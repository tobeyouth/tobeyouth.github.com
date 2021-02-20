---
layout: post
title: "learn draft-js"
modified: 2017-08-22 11:24:56 +0800
tags: [javascript, react, draft-js]
---

### what is draft-js

`draft-js` 是一个基于 `react` 和 `immutable-js` 的富文本编辑器, 可以方便的集成到 `react` 代码中。


### 富文本编辑器要解决的问题

编辑器最终要解决的是两个方面问题：

#### 输入

1. 同步 `state` 和 `input` 的值
2. 监听 `input` 的`onChange` 事件


#### 富文本

1. 单纯的文本不足以满足复杂的形式
2. 可编辑的元素没有 `onChange` 事件


### 富文本内容

富文本的内容可以分为以下几个部分

- EditorState
- ContentState
- ContentBlock
- Entity
- SelectionState

它们之间的大致关系可以粗浅的立即为下图（箭头为包含关系）

![箭头表示包含关系，函数名为可返回箭头右侧实例的方法](https://docs.google.com/drawings/d/e/2PACX-1vSEPu-3OFXads-C4Vy3TU31edu_Bulzimw3mebzoNI0AFpYkiB9NTldaIbeT0j6LTIGS4KZ2MyxRpKH/pub?w=480&h=360)


### [EditorState](https://draftjs.org/docs/api-reference-editor-state)

为了解决以上两方面的问题，`draft-js` 引入了`State` 和 `onChange`。
对于熟悉 `react`的工程师来说，这两个概念已经不是新鲜的概念了。

其中`State`作为所有交互和输入数据的状态，被包装成了一个不可变的数据类型`EditorState`。

`EditorState` 的数据类型依托于

[Record](http://facebook.github.io/immutable-js/docs/#/Record/Record)，其中包括了以下几类数据，用于描述`draft-js`
编辑器中的所有状态

1. 当前文本内容, current text content state
2. 当前选中内容, current selection state
3. 内容装饰, decorated representation of the contents
4. 操作记录, Undo/redo stacks
5. 最近的内容变化的类型, The most recent type of change made to the contents

整个`editorState` 就是要编辑的所有状态，所以**最好是将所有状态的修改都通过 editorState 提供的 Api 来进行，而不要通过
immutable-js 的 Api 来进行**.

---- 

#### [ContentState](https://draftjs.org/docs/api-reference-content-state)

`ContentState`可以理解为内容的状态，所有对内容的更新，本质上都是更新这个状态的实例

---- 


#### [ContentBlock](https://draftjs.org/docs/api-reference-content-block)

`ContentBlock`是在 `ContentState` 中实际描述内容的单元，每个 `ContentBlock` 实例包括了以下几部分：

- Plain text contents of the block
- Type, e.g. paragraph, header, list item
- Entity, inline style, and depth information

每个 `ContentBlock`都有其唯一的 key，可以通过 key 来指定需要操作的 `ContentBlock`。


##### Decorators

在 `ContentBlock`中，有时会对一些文本进行特殊展示，这种情况下可以使用 `Decorator`来对其进行装饰，使其在编辑器中展示出特别的样式和 html 结构。

每一个 `Decorator` 对象都有以下这两个属性：

``` javascript
strategy: handleStrategy (contentBlock, callback, contentState) => {},
component: HandleSpan (props) => {}
```

看一个具体的例子，下面的这段代码匹配到了 `@` 和 `#` 并返回了富文本

``` javascript

// strategy
const HANDLE_REGEX = /\@[\w]+/g;
const HASHTAG_REGEX = /\#[\w\u0590-\u05ff]+/g;

function handleStrategy(contentBlock, callback, contentState) {
  findWithRegex(HANDLE_REGEX, contentBlock, callback);
}

function hashtagStrategy(contentBlock, callback, contentState) {
  findWithRegex(HASHTAG_REGEX, contentBlock, callback);
}

function findWithRegex(regex, contentBlock, callback) {
  const text = contentBlock.getText();
  let matchArr, start;
  while ((matchArr = regex.exec(text)) !== null) {
    start = matchArr.index;
    callback(start, start + matchArr[0].length);
  }
}

// component
const HandleSpan = (props) => {
  return <span {...props} style={styles.handle}>{props.children}</span>;
};

const HashtagSpan = (props) => {
  return <span {...props} style={styles.hashtag}>{props.children}</span>;
};

```

这里需要注意的是 `callback` 方法实际接收的是匹配到的文本内容的起始位置和结束位置，也可以称为 `range`。


另外，`Decorator` 需要通过 
```
EditorState.set(editorState, {decorator: decorators})
```
的方式来添加到 editorState 中，这样才可以正常使用。

---- 

#### [Entity](https://draftjs.org/docs/api-reference-entity)

`entity` 是一个最小单元的富文本对象，每一个 `entity` 对象都有三个属性：

- type: 表示这是什么类型的对象, 例如：'LINK', 'PHOTO'
- mutability: 表示是否可以在 editor 中修改这个对象
- data: 容纳所有富文本数据的地方，例如 `type` 为 'LINK' 的话，data 可以写成这样 `{'url': 'https://foo.com'}`


`entity` 对象用于描述 `contentState` 中的`selectionState`, 举个例子，这是一个**创建 entity -> 获取 entityKey -> 使用
Modifier 将 entity 应用到 selectionState **

```
const contentState = editorState.getCurrentContent();
const contentStateWithEntity = contentState.createEntity(
  'LINK',
  'MUTABLE',
  {url: 'http://www.zombo.com'}
);
const entityKey = contentStateWithEntity.getLastCreatedEntityKey();
const contentStateWithLink = Modifier.applyEntity(
  contentStateWithEntity,
  selectionState,
  entityKey
);
```

###### tips:

重要的事情再强调一遍，Entity 中的 `mutability` 跟 `Immutable Record` 对象不是一回事儿，这个属性仅仅是标记在 editor 中当前的entity 是否会改变。

这个属性有三个可选择的值，类型都是字符串：

- IMMUTABLE: 当前 entity 不可变
- MUTABLE: 当前 entity 可变
- SEGMENTED: 当前 entity 部分可变


#### [SelectionState](https://draftjs.org/docs/api-reference-selection-state)

顾名思义，`SelectionState`就是 [Selection](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection) 在编辑器中的状态；通过`SelectionState`可以获取到与 Selection 相关的`ContentBlock`。

---- 

除了表示内容的数据结构外，draft-js 还提供了一些其他的 Utils 供开发者使用


- RichUtils: 提供一些获取、操作 editorState 的静态方法
- AutomicBlockUtils：提供一些方法来操作原子 Block
- KeyBindingUtils：顾名思义，提供一些 key command 相关的绑定方法
- Modifier：提供一些操作 ContentState 的方法
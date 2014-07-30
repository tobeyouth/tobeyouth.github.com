---
layout: post
title: "XML Schema"
description: ""
category: 
tags: ['XML','数据']
---
{% include JB/setup %}


`XML Schema`是一种通过XML实现的数据格式约定，可以通过schema来约束数据的格式，应该算是类似各种编程语言中`interface`的概念。

通过`XML Schema`的方式定义数据格式，最大的好处应该就是还算得上便于理解，另外可以跟数据公用XML解析器。

闲话少说，还是直接来介绍一下`XML Schema`吧。

在`XML Schema`的规则中，需要先在头部声明文档类型：

        <?xml version="1.0"?>

以证明这是一份XML文件，接下来，要声明`schema`的解析方式：

       <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

再接下来，是写入document标签：
    
         <xs:element name="DOCUMENT" type="DOCUMENT_type"/>

然后，就可以在document中写入你所定义的规则了。
所谓规则，其实可以分为这么几大类：
- 元素
- 组合
- 属性
- 类型

其中`元素`和`组合`可以看做一组，即元素的集合，就是组合。`属性`和`类型`也可以当做一组，即属性的集合就是一个类型。在这几组中，不论是`组合`还是`类型`，都可以通过`<xs:simpleType>`或者`<xs:complexType>`来组织。

例如我要创造一个人的基本信息的`schema`，其中基本信息包括`name`,`age`,`family`，其中`family`是其家庭成员的集合，该条规则可以这样定义：

        <xs:complexType name="person" type="person_type">
            <xs:sequence>
                <xs:element name="name" type="name_type" />
                <xs:element name="age" type="age_type" />
                <xs:element name="family" type="family_type" />
            </xs:sequence>
        </xs:complexType>
        <xs:simpleType name="name_type">
           <xs:restriction base="xs:string">
                <xs:maxLength value="6"/>
                <xs:minLength value="1"/>
            </xs:restriction>
        </xs:simpleType>
        <xs:simpleType name="age_type">
            <xs:restriction base="xs:integer">
                <xs:minInclusive value="0"/>
                <xs:maxInclusive value="120"/>
            </xs:restriction>
        </xs:simpleType>
        <xs:complexType name="family_type">
            <xs:sequence>
                <xs:elment name="person" />
            </xs:sequence>
        </xs:complexType>

通过这个schema可以看出来，`simpleType`和`compexType`是两个比较重要的标签形式，用于定义`类型`或者`组合`。基本的区分方法就是，如果`只需要基本类型的定义`，那么可以使用`simpleType`；如果`需要定义的对象中，还包括其他对象`，那么应该使用`complexType`。

对于xml格式的数据，除了定义内容之外，还可以定义标签的属性，例如我们想要定义这样一个标签：

        <time country="china">12:30</time>

其中有个`country`属性，这个属性可以这样定义：

        <xs:complexType name="time_type">
            <attribute name="country" base="xs:string">
                <xs:minLength value="1" />
            </attribute>
        </xs:complexType>

另外，可以看到还有`xs:string`和`xs:integer`这样的类型限制，这样的类型限制对应的是不同的基本类型，在schema的语法中，有这么几种常见的基本类型：

        xs:string
        xs:decimal
        xs:integer
        xs:boolean
        xs:date
        xs:time

在基本类型中，还可以通过一些其他属性，来增加数据规则的详细程度，下面以`xs:string`为例，说一下可以在schema中可以通过哪些方式进行更加细致的限定。

首先，可以通过`maxLength`和`minLength`来做字符串的最大长度和最小长度的限定，如上面的`name_type`这条规则一样：
        <xs:simpleType name="name_type">
           <xs:restriction base="xs:string">
                <xs:maxLength value="6"/>
                <xs:minLength value="1"/>
            </xs:restriction>
        </xs:simpleType>

其次，还可以指定规则，将数据限定在几种固定值之间：

        <xs:simpleType name="car_type">
            <xs:restriction base="xs:string">
                <xs:enumeration value="Audi"/>
                <xs:enumeration value="Golf"/>
                <xs:enumeration value="BMW"/>
            </xs:restriction>
        </xs:simpleType>

再次，还可以通过正则表达式的方法，来指定数据的规则：

        <xs:simpleType>
            <xs:restriction base="xs:string">
                <xs:pattern value="[a-z]"/>
            </xs:restriction>
        </xs:simpleType>

除了上面所说的`minLength`,`enumeration`和`pattern`属性可以定义数据类型的内容之外，还可以通过以下多种方式，来对数据的内容进行限定：

        
        enumeration:限定一组数值
        fractionDigits:定义所允许的最大的小数位数。必须大于等于0
        length:定义所允许的字符或者列表项目的精确数目。必须大于或等于0
        maxExclusive:定义数值的上限。所允许的值必须小于此值
        maxLength:定义所允许的字符或者列表项目的最大数目。必须大于或等于0
        minExclusive:定义数值的下限。所允许的值必须大于此值
        minInclusiv:定义数值的下限。所允许的值必须大于或等于此值
        minLength:定义所允许的字符或者列表项目的最小数目。必须大于或等于0
        pattern:定义可接受的字符的精确序列(正则表达式)
        totalDigits:定义所允许的阿拉伯数字的精确位数。必须大于0
        whiteSpace:定义空白字符（换行、回车、空格以及制表符）的处理方式

以上只是简单的介绍了以下`XML Schema`，更多详细的信息，可以参考(http://www.w3school.com.cn/schema/schema_intro.asp)[http://www.w3school.com.cn/schema/schema_intro.asp]



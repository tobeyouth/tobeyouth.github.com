---
layout: post
title: "XML Schema"
description: ""
category: 
tags: ['XML','����']
---
{% include JB/setup %}


`XML Schema`��һ��ͨ��XMLʵ�ֵ����ݸ�ʽԼ��������ͨ��schema��Լ�����ݵĸ�ʽ��Ӧ���������Ƹ��ֱ��������`interface`�ĸ��

ͨ��`XML Schema`�ķ�ʽ�������ݸ�ʽ�����ĺô�Ӧ�þ��ǻ�����ϱ�����⣬������Ը����ݹ���XML��������

�л���˵������ֱ��������һ��`XML Schema`�ɡ�

��`XML Schema`�Ĺ����У���Ҫ����ͷ�������ĵ����ͣ�

        <?xml version="1.0"?>

��֤������һ��XML�ļ�����������Ҫ����`schema`�Ľ�����ʽ��

       <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

�ٽ���������д��document��ǩ��
    
         <xs:element name="DOCUMENT" type="DOCUMENT_type"/>

Ȼ�󣬾Ϳ�����document��д����������Ĺ����ˡ�
��ν������ʵ���Է�Ϊ��ô�����ࣺ
- Ԫ��
- ���
- ����
- ����

����`Ԫ��`��`���`���Կ���һ�飬��Ԫ�صļ��ϣ�������ϡ�`����`��`����`Ҳ���Ե���һ�飬�����Եļ��Ͼ���һ�����͡����⼸���У�������`���`����`����`��������ͨ��`<xs:simpleType>`����`<xs:complexType>`����֯��

������Ҫ����һ���˵Ļ�����Ϣ��`schema`�����л�����Ϣ����`name`,`age`,`family`������`family`�����ͥ��Ա�ļ��ϣ�������������������壺

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

ͨ�����schema���Կ�������`simpleType`��`compexType`�������Ƚ���Ҫ�ı�ǩ��ʽ�����ڶ���`����`����`���`�����������ַ������ǣ����`ֻ��Ҫ�������͵Ķ���`����ô����ʹ��`simpleType`�����`��Ҫ����Ķ����У���������������`����ôӦ��ʹ��`complexType`��

����xml��ʽ�����ݣ����˶�������֮�⣬�����Զ����ǩ�����ԣ�����������Ҫ��������һ����ǩ��

        <time country="china">12:30</time>

�����и�`country`���ԣ�������Կ����������壺

        <xs:complexType name="time_type">
            <attribute name="country" base="xs:string">
                <xs:minLength value="1" />
            </attribute>
        </xs:complexType>

���⣬���Կ�������`xs:string`��`xs:integer`�������������ƣ��������������ƶ�Ӧ���ǲ�ͬ�Ļ������ͣ���schema���﷨�У�����ô���ֳ����Ļ������ͣ�

        xs:string
        xs:decimal
        xs:integer
        xs:boolean
        xs:date
        xs:time

�ڻ��������У�������ͨ��һЩ�������ԣ����������ݹ������ϸ�̶ȣ�������`xs:string`Ϊ����˵һ�¿�����schema�п���ͨ����Щ��ʽ���и���ϸ�µ��޶���

���ȣ�����ͨ��`maxLength`��`minLength`�����ַ�������󳤶Ⱥ���С���ȵ��޶����������`name_type`��������һ����
        <xs:simpleType name="name_type">
           <xs:restriction base="xs:string">
                <xs:maxLength value="6"/>
                <xs:minLength value="1"/>
            </xs:restriction>
        </xs:simpleType>

��Σ�������ָ�����򣬽������޶��ڼ��̶ֹ�ֵ֮�䣺

        <xs:simpleType name="car_type">
            <xs:restriction base="xs:string">
                <xs:enumeration value="Audi"/>
                <xs:enumeration value="Golf"/>
                <xs:enumeration value="BMW"/>
            </xs:restriction>
        </xs:simpleType>

�ٴΣ�������ͨ��������ʽ�ķ�������ָ�����ݵĹ���

        <xs:simpleType>
            <xs:restriction base="xs:string">
                <xs:pattern value="[a-z]"/>
            </xs:restriction>
        </xs:simpleType>

����������˵��`minLength`,`enumeration`��`pattern`���Կ��Զ����������͵�����֮�⣬������ͨ�����¶��ַ�ʽ���������ݵ����ݽ����޶���

        
        enumeration:�޶�һ����ֵ
        fractionDigits:���������������С��λ����������ڵ���0
        length:������������ַ������б���Ŀ�ľ�ȷ��Ŀ��������ڻ����0
        maxExclusive:������ֵ�����ޡ��������ֵ����С�ڴ�ֵ
        maxLength:������������ַ������б���Ŀ�������Ŀ��������ڻ����0
        minExclusive:������ֵ�����ޡ��������ֵ������ڴ�ֵ
        minInclusiv:������ֵ�����ޡ��������ֵ������ڻ���ڴ�ֵ
        minLength:������������ַ������б���Ŀ����С��Ŀ��������ڻ����0
        pattern:����ɽ��ܵ��ַ��ľ�ȷ����(������ʽ)
        totalDigits:����������İ��������ֵľ�ȷλ�����������0
        whiteSpace:����հ��ַ������С��س����ո��Լ��Ʊ�����Ĵ���ʽ

����ֻ�Ǽ򵥵Ľ���������`XML Schema`��������ϸ����Ϣ�����Բο�(http://www.w3school.com.cn/schema/schema_intro.asp)[http://www.w3school.com.cn/schema/schema_intro.asp]



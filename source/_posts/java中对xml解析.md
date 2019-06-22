---
title: java中对xml解析
date: 2019-06-20 09:40:39
categories: 
- java基础知识 
tags:
- java
- xml
description: 本文介绍如何使用jdk自带工具对xml进行解析。
---
# 一、从输入流获取Document

1. 获取资源对应的输入流
```
InputStream is = App.class.getResourceAsStream("/xml/doc.xml");
```
2. 获取文档构建器工厂DocumentBuilderFactory,该工厂可以设置对文档解析时的一些参数。
```
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
//解析器是否将xml文档中CDATA节点转换为文本，并附加到相邻的文本节点，默认为false
dbf.setCoalescing(false);
//解析器是否扩展xml文档中实体引用节点，默认为false
dbf.setExpandEntityReferences(false);
//解析器是否忽略xml文档中的注释，默认为false
dbf.setIgnoringComments(false);
//解析器是否提供对命名空间的支持，默认为false
dbf.setNamespaceAware(false);
//解析器是否忽略元素内容中的空白，默认为false
dbf.setIgnoringElementContentWhitespace(false);
//解析器是否对xml文档进行dtd验证,默认为false
dbf.setValidating(false);
//是否处理xi:include标签，默认为false
dbf.setXIncludeAware(false);
```
3. 获取DocumentBuilder
```
DocumentBuilder db = dbf.newDocumentBuilder();
//设置如何查找dtd文件
InputStream dtdInputstream = App.class.getResourceAsStream("doc.dtd");
db.setEntityResolver((PublicId, systemId) -> new InputSource(dtdInputstream));
```
4. 从输入流中解析出Document
```
Document document = db.parse(is);
```
# 二、解析Document
```
//使用XPath解析Document获取节点
XPathFactory factory = XPathFactory.newInstance();
XPath xpath = factory.newXPath();
Node node = (Node) xpath.evaluate("/root", document, XPathConstants.NODE);
//获取node后可以使用node的方法获取子节点或者属性
node.getFirstChild().getAttributes().item(0).getNodeValue();
```

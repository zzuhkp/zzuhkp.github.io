---
title: Mybatis如何对xml进行解析
date: 2019-06-22 23:07:39
categories: 
- Mybatis源码阅读
tags:
- Mybatis
- 源码
description: Mybatis对xml的解析能够将xml映射为Configuration类...
---

# 一、概述
Mybatis对Xml的解析主要使用org.apache.ibatis.parsing包中的类，主要分为**节点解析和节点中变量的解析**，该包中有以下几个类：
- GenericTokenParser：查找文本内容中的变量，并使用该类包含的TokenHandler引用解析出变量值。
- ParsingException：未使用。
- PropertyParser：将静态内部类VariableTokenHandler引用传递给GenericTokenParser解析出xml节点属性值中的变量。
- TokenHandler：描述如何处理变量的接口，由PropertyParser内部静态类VariableTokenHandler实现。
- XNode：使用XPathParser解析节点的内容、使用PropertyParser解析xml节点属性中变量的值。
- XPathParser：提供从InputStream中创建Document能力，并能根据表达式对节点进行解析。

流程图如下：

{% asset_img 'Mybatis XML解析序列图.jpg' Mybatis XML解析序列图 %}

# 二、节点解析
Mybatis使用XPathParser类对节点进行解析，该类还提供了从输入流中创建Document的能力。

## 2.1 Document创建
XPathParser部分代码如下：
```
public class XPathParser {

  //xml文本对应的文档
  private final Document document;
  //是否进行dtd校验
  private boolean validation;
  //该类定义了如何解析dtd
  private EntityResolver entityResolver;
  //xml中可能使用到的变量
  private Properties variables;
  //使用XPath解析节点
  private XPath xpath;
  
  public XPathParser(InputStream inputStream, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    this.document = createDocument(new InputSource(inputStream));
  }
  
  public XPathParser(Reader reader, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    this.document = createDocument(new InputSource(reader));
  }
  
  public XPathParser(String xml, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    this.document = createDocument(new InputSource(new StringReader(xml)));
  }  

  private void commonConstructor(boolean validation, Properties variables, EntityResolver entityResolver) {
    this.validation = validation;
    this.entityResolver = entityResolver;
    this.variables = variables;
    XPathFactory factory = XPathFactory.newInstance();
    this.xpath = factory.newXPath();
  }
  
  private Document createDocument(InputSource inputSource) {
    // important: this must only be called AFTER common constructor
    try {
      DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
      factory.setValidating(validation);

      factory.setNamespaceAware(false);
      factory.setIgnoringComments(true);
      factory.setIgnoringElementContentWhitespace(false);
      factory.setCoalescing(false);
      factory.setExpandEntityReferences(true);

      DocumentBuilder builder = factory.newDocumentBuilder();
      builder.setEntityResolver(entityResolver);
      builder.setErrorHandler(new ErrorHandler() {
        @Override
        public void error(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void fatalError(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void warning(SAXParseException exception) throws SAXException {
        }
      });
      return builder.parse(inputSource);
    } catch (Exception e) {
      throw new BuilderException("Error creating document instance.  Cause: " + e, e);
    }
  }
```
可以看到XPathParser构造方法的代码基本一致，最终都调用了commonConstructor和createDocument方法。
- commonConstructor方法主要对成员变量进行初始化。
- createDocument方法使用JDK自带的类创建Document对象。

## 2.2 节点解析
Mybatis对Node节点的解析主要使用了XPathParser的evalNode方法，该方法使用了XPath表达式对节点解析。XPathParser类对Node解析的部分代码如下：
```
public class XPathParser {

  /**
   * 解析表达式的通用方法
   * @param expression
   * @param root
   * @param returnType
   * @return
   */
  private Object evaluate(String expression, Object root, QName returnType) {
    try {
      return xpath.evaluate(expression, root, returnType);
    } catch (Exception e) {
      throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
    }
  }
  
  /**
   * 将表达式解析为节点
   * @param root
   * @param expression
   * @return
   */
  public XNode evalNode(Object root, String expression) {
    Node node = (Node) evaluate(expression, root, XPathConstants.NODE);
    if (node == null) {
      return null;
    }
    return new XNode(this, node, variables);
  }

  /**
   * 将表达式解析为字符串
   *
   * @param root
   * @param expression
   * @return
   */
  public String evalString(Object root, String expression) {
    String result = (String) evaluate(expression, root, XPathConstants.STRING);
    result = PropertyParser.parse(result, variables);
    return result;
  }
  
  /**
   * 将表达式解析为整型
   *
   * @param root
   * @param expression
   * @return
   */
  public Integer evalInteger(Object root, String expression) {
    return Integer.valueOf(evalString(root, expression));
  }
  
} 
```

可以看到XPathParser提供通用的evaluate方法对表达式进行解析，并提供了特定的其他方法用于解析出节点、字符串、整型变量等。
- 解析出的节点将被包装为XNode类，XNode类中对节点内容的解析使用XPathParser提供的方法。
- 解析出的字符串将PropertyParser.parse方法解析出字符串中的变量值。

# 三、变量解析

XPathParser将XPath表达式解析为字符串时将调用PropertyParser.parse方法解析出字符串中的变量对应的值，例如："jdbc:${url}/test",将解析出变量url的值并替换${url}。

PropertyParser部分代码如下：
```
public class PropertyParser {

  private static final String KEY_PREFIX = "org.apache.ibatis.parsing.PropertyParser.";

  /**
   * 是否开启默认值的键
   * 3.4.2版本开始
   */
  public static final String KEY_ENABLE_DEFAULT_VALUE = KEY_PREFIX + "enable-default-value";

  /**
   * 变量名和默认值的分隔符的键
   * 3.4.2版本开始
   */
  public static final String KEY_DEFAULT_VALUE_SEPARATOR = KEY_PREFIX + "default-value-separator";

  /**
   * 默认不开启变量的默认值
   */
  private static final String ENABLE_DEFAULT_VALUE = "false";

  /**
   * 默认分隔符为:
   */
  private static final String DEFAULT_VALUE_SEPARATOR = ":";

  private PropertyParser() {
    // Prevent Instantiation
  }

  public static String parse(String string, Properties variables) {
    VariableTokenHandler handler = new VariableTokenHandler(variables);
    GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
    return parser.parse(string);
  }

  private static class VariableTokenHandler implements TokenHandler {
    /**
     * xml中properties标签设置的变量
     */
    private final Properties variables;
    /**
     * 是否开启默认值
     */
    private final boolean enableDefaultValue;
    /**
     * 变量名和默认值的分隔符
     */
    private final String defaultValueSeparator;

    private VariableTokenHandler(Properties variables) {
      this.variables = variables;
      this.enableDefaultValue = Boolean.parseBoolean(getPropertyValue(KEY_ENABLE_DEFAULT_VALUE, ENABLE_DEFAULT_VALUE));
      this.defaultValueSeparator = getPropertyValue(KEY_DEFAULT_VALUE_SEPARATOR, DEFAULT_VALUE_SEPARATOR);
    }

    private String getPropertyValue(String key, String defaultValue) {
      return (variables == null) ? defaultValue : variables.getProperty(key, defaultValue);
    }

    @Override
    public String handleToken(String content) {
      if (variables != null) {
        //变量不为空时处理变量名
        String key = content;
        if (enableDefaultValue) {
          final int separatorIndex = content.indexOf(defaultValueSeparator);
          String defaultValue = null;
          if (separatorIndex >= 0) {
            //利用分隔符将变量名和默认值分开
            key = content.substring(0, separatorIndex);
            defaultValue = content.substring(separatorIndex + defaultValueSeparator.length());
          }
          if (defaultValue != null) {
            //从设置的变量中解析出xml文本内容中的变量名对应的变量值
            return variables.getProperty(key, defaultValue);
          }
        }
        if (variables.containsKey(key)) {
          return variables.getProperty(key);
        }
      }
      return "${" + content + "}";
    }
  }

}
```
- 阅读PropertyParser源码可以发现，mybatis从3.4.2开始支持为变量设置默认值，并且默认值特定默认是关闭的。如果需要开启可以在xml配置文件properties节点中设置。
- PropertyParser使用内部类VariableTokenHandler处理变量名和默认值，从properties中获取变量名对应的值。
- PropertyParser.parse方法调用GenericTokenParser类中parse方法从文本中解析出变量名，并使用VariableTokenHandler提供的方法从变量名中解析出变量的值。

GenericTokenParser源码如下，并已经做出了注释。
```
public class GenericTokenParser {

  /**
   * 变量开始标识
   */
  private final String openToken;
  /**
   * 变量结束标识
   */
  private final String closeToken;
  /**
   * 变量处理器
   */
  private final TokenHandler handler;

  public GenericTokenParser(String openToken, String closeToken, TokenHandler handler) {
    this.openToken = openToken;
    this.closeToken = closeToken;
    this.handler = handler;
  }

  /**
   * 将文本中的变量替换为对应的变量值
   * 以openToken为“${”和closeToken为“}”
   *
   * @param text
   * @return
   */
  public String parse(String text) {
    //文本为空之间返回
    if (text == null || text.isEmpty()) {
      return "";
    }
    // 搜索变量左侧符号openToken在文本内的位置
    int start = text.indexOf(openToken);
    //找不到openToken直接返回
    if (start == -1) {
      return text;
    }
    //将文本转换为字符数组，便于向StringBuilder中拼接非变量值和变量值
    char[] src = text.toCharArray();
    //未处理的字符在字符数组str的偏移量
    int offset = 0;
    //最终解析的值
    final StringBuilder builder = new StringBuilder();
    //变量名称，根据名称使用TokenHandler从设置的属性中查找值
    StringBuilder expression = null;
    //只要在未处理的字符数组中还存在openToken的字符就一直处理
    while (start > -1) {
      //openToken前存在转义字符“\”,处理转义字符
      if (start > 0 && src[start - 1] == '\\') {
        //存在转义字符则拼接非变量值和openToken
        builder.append(src, offset, start - offset - 1).append(openToken);
        //计算偏移量,偏移量为${在text的索引位置加上“${”的长度
        offset = start + openToken.length();
      } else {
        //找到openToken，并且openToken前不存在转义字符
        //初始化变量名
        if (expression == null) {
          expression = new StringBuilder();
        } else {
          expression.setLength(0);
        }
        //最终解析的值拼接openToken左侧的非变量内容
        builder.append(src, offset, start - offset);
        //重新计算偏移量
        offset = start + openToken.length();
        //从偏移量开始在文本中搜索closeToken的位置
        int end = text.indexOf(closeToken, offset);
        //当closeToken存在时
        while (end > -1) {
          //处理closeToken前的转义字符“\”
          if (end > offset && src[end - 1] == '\\') {
            //变量名拼接非变量内容和closeToken,${abc\}def},即abc\}
            expression.append(src, offset, end - offset - 1).append(closeToken);
            //重新计算偏移量
            offset = end + closeToken.length();
            //从偏移量开始，在文本中重新计算CloseToken的索引位置
            end = text.indexOf(closeToken, offset);
          } else {
            //变量名拼接偏移量开始到closeToken位置之间的内容,即${abc\}def}中的def或${abc}中的abc
            expression.append(src, offset, end - offset);
            //重新计算偏移量
            offset = end + closeToken.length();
            break;
          }
        }
        //找不到closeToken
        if (end == -1) {
          //拼接从${后面所有的内容
          builder.append(src, start, src.length - start);
          //偏移量为文本长度值
          offset = src.length;
        } else {
          //找到closeToken,获取处理好的变量名对应的值拼接到最终解析的值中，handleToken从设置的Properties中获取值
          builder.append(handler.handleToken(expression.toString()));
          //重新计算偏移量
          offset = end + closeToken.length();
        }
      }
      //重新计算openTokend的索引位置
      start = text.indexOf(openToken, offset);
    }
    //处理最后closeToken后面的内容
    if (offset < src.length) {
      //最终解析的值中拼接closeToken后面的内容
      builder.append(src, offset, src.length - offset);
    }
    return builder.toString();
  }
}
```

GenericTokenParser的parse方法能够在文本中查找多个变量，并对转义字符进行了处理。该方法最终调用TokenHandler引用，即VariableTokenHandler类的handleToken方法从变量名解析出变量值。

# 总结

通过以上的介绍，Mybatis已经能够完成对xml的解析工作。对Xml的解析能够用于对Mybatis配置文件的解析，将配置文件映射为Configuration类。
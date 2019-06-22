---
title: java中获取类路径资源文件的几种方式
date: 2019-06-20 09:36:32
categories: 
- java基础知识 
tags:
- java
description: java中获取类路径资源文件主要通过Class及ClassLoader提供的方法，最终都调用了ClassLoader中的方法。
---
# 一 Class.getResource
```
String name1 = "/prop/system.properties";
URL url1 = App.class.getResource(name1);
```
# 二 Class.getResourceAsStream
```
String name1 = "/prop/system.properties";
InputStream inputStream1 = App.class.getResourceAsStream(name1);
```
# 三 ClassLoader.getResource
```
String name2 = "prop/system.properties";
 URL url2 = App.class.getClassLoader().getResource(name2);
```
# 四 ClassLoader.getResourceAsStream
```
String name2 = "prop/system.properties";
InputStream inputStream2 = App.class.getClassLoader().getResourceAsStream(name2);
```
# 五 ResourceBundle.getBundle
```
String name3 = "prop/system";
ResourceBundle resourceBundle = ResourceBundle.getBundle(name3);
```
注意资源名称不包含.properties
# Class、ClassLoader获取资源文件区别
- Class.getResource和Class.getResourceAsStream中，资源名称以"/"开头表示绝对定位，为类路径根目录下的资源文件， 否则资源名称被解析为以Class类的包名为父目录
- ClassLoader.getResource和ClassLoader.getResourceAsStream中，资源名称不能以"/"开头，否则将找不到文件。

# 源码分析
Class类部分源码如下：
```
public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type,AnnotatedElement {

    public java.net.URL getResource(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResource(name);
        }
        return cl.getResource(name);
    }
    
    public InputStream getResourceAsStream(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResourceAsStream(name);
        }
        return cl.getResourceAsStream(name);
    }
    
    private String resolveName(String name) {
        if (name == null) {
            return name;
        }
        if (!name.startsWith("/")) {
            Class<?> c = this;
            while (c.isArray()) {
                c = c.getComponentType();
            }
            String baseName = c.getName();
            int index = baseName.lastIndexOf('.');
            if (index != -1) {
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else {
            name = name.substring(1);
        }
        return name;
    }
    
    ClassLoader getClassLoader0() { return classLoader; }

}
```

- Class的getResource和getResourceAsStream首先调用resolveName方法解析资源名称，然后调用ClassLoader的getResource或getResourceAsStream方法；如果Class类中的域classLoader
为null还将调用ClassLoader.getSystemResource或ClassLoader.getSystemResourceAsStream方法获取系统资源。
- Class的resolveName方法解析资源名称时对方法参数进行处理，如果以"/"开头则把开头的"/"截取，否则取包名为父目录和方法参数进行拼接。

ClassLoader部分源码如下：

```
public abstract class ClassLoader {

    public URL getResource(String name) {
        URL url;
        if (parent != null) {
            url = parent.getResource(name);
        } else {
            url = getBootstrapResource(name);
        }
        if (url == null) {
            url = findResource(name);
        }
        return url;
    }
    
    public InputStream getResourceAsStream(String name) {
        URL url = getResource(name);
        try {
            return url != null ? url.openStream() : null;
        } catch (IOException e) {
            return null;
        }
    }
}

```

- ClassLoader的getResourceAsStream方法调用了getResource方法
- ClassLoader的getResource方法首先尝试使用父类加载器获取资源，未获取到则调用findResource方法查找资源，findResource方法由子类覆盖。

# 总结

- Class.getResource、 Class.getResourceAsStream、ClassLoader.getResourceAsStream最终都调用ClassLoader.getResource
- 如果将properties文件转换为java类Properties，可以使用ResourceBundle.getBundle方法。
---
title: Mybatis资源获取工具类Resources
date: 2019-06-20 09:41:46
categories: 
- Mybatis源码阅读
tags:
- Mybatis
- 源码
description: Mybatis工具类Resources在获取Mybatis配置文件，Mybatis xml dtd文件中都有使用到，本文对Resources工具类进行源码分析。
---
# 一 Resources包含的方法

Resources提供各种从类路径获取资源的方法，主要依赖内部的classLoaderWrapper完成资源的查找，提供的方法主要有以下几种
- getResourceAsFile 
- getResourceAsProperties
- getResourceAsReader
- getResourceAsStream
- getResourceURL
- getUrlAsProperties
- getUrlAsReader
- getUrlAsStream

这些方法主要是返回的类型不同，getResourceXXX方法最终都调用了classLoaderWrapper.getResourceAsURL或classLoaderWrapper.getResourceAsStream方法。
Resources类的部分代码如下：
```
public class Resources {

  private static ClassLoaderWrapper classLoaderWrapper = new ClassLoaderWrapper();

  public static InputStream getResourceAsStream(ClassLoader loader, String resource) throws IOException {
    InputStream in = classLoaderWrapper.getResourceAsStream(resource, loader);
    if (in == null) {
      throw new IOException("Could not find resource " + resource);
    }
    return in;
  }
  
  public static URL getResourceURL(ClassLoader loader, String resource) throws IOException {
    URL url = classLoaderWrapper.getResourceAsURL(resource, loader);
    if (url == null) {
      throw new IOException("Could not find resource " + resource);
    }
    return url;
  }
}
```
可以看到，如果classLoaderWrapper在类路径下找不到资源，将抛出异常，而不是返回null。

# 二 ClassLoaderWrapper分析

ClassLoaderWrapper是对多个类加载器的包装，使用多个类加载器获取类路径下的资源，部分源码如下

```
public class ClassLoaderWrapper {

  ClassLoader defaultClassLoader;
  ClassLoader systemClassLoader;
  
  /**
   * 获取包装的类加载器列表
   */
  ClassLoader[] getClassLoaders(ClassLoader classLoader) {
    return new ClassLoader[]{
        classLoader,
        defaultClassLoader,
        Thread.currentThread().getContextClassLoader(),
        getClass().getClassLoader(),
        systemClassLoader};
  }

  InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
    for (ClassLoader cl : classLoader) {
      if (null != cl) {

        // try to find the resource as passed
        InputStream returnValue = cl.getResourceAsStream(resource);

        // now, some class loaders want this leading "/", so we'll add it and try again if we didn't find the resource
        if (null == returnValue) {
          returnValue = cl.getResourceAsStream("/" + resource);
        }

        if (null != returnValue) {
          return returnValue;
        }
      }
    }
    return null;
  }
  
  URL getResourceAsURL(String resource, ClassLoader[] classLoader) {

    URL url;

    for (ClassLoader cl : classLoader) {

      if (null != cl) {

        // look for the resource as passed in...
        url = cl.getResource(resource);

        // ...but some class loaders want this leading "/", so we'll add it
        // and try again if we didn't find the resource
        if (null == url) {
          url = cl.getResource("/" + resource);
        }

        // "It's always in the last place I look for it!"
        // ... because only an idiot would keep looking for it after finding it, so stop looking already.
        if (null != url) {
          return url;
        }

      }

    }

    // didn't find it anywhere.
    return null;

  }
}

```

getResourceAsStream和getResourceAsURL方法循环使用包装的类加载器获取资源，如果获取到则返回，否则返回null。
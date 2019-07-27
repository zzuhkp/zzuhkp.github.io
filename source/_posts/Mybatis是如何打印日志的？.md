---
title: Mybatis是如何打印日志的？
date: 2019-06-30 12:23:40
categories: 
- Mybatis源码阅读
tags:
- Mybatis
- 源码
description: Mybatis使用具体的日志框架打印Mybatis中的日志，并且代理了JDBC中的相关类，如Connection，因此在sql执行时能打印相关信息，那么Mybatis是如何查找日志框架和代理JDBC类的呢？
---
# 概述

Mybatis的日志使用了工厂模式，分别包装了不同的日志框架，例如slf4j,common logging,log4j2,jdk logging等，Mybatis会按照默认设定的顺序从类路径中查找存在的日志框架作为实现，同时用户也可以定义自己的日志实现，具体的打印工作由日志框架来完成。

# 一、Mybatis如何获取日志实现类的
## 1.1  Mybatis中如何获取日志对象
Mybatis使用“org.apache.ibatis.logging.LogFactory#getLog(java.lang.Class<?>)”方法获取日志对象，关键源码如下：
```
public final class LogFactory {
  /**
   * 缓存日志实现类的构造器
   */
  private static Constructor<? extends Log> logConstructor;
  
  public static Log getLog(Class<?> aClass) {
    return getLog(aClass.getName());
  }

  public static Log getLog(String logger) {
    try {
      return logConstructor.newInstance(logger);
    } catch (Throwable t) {
      throw new LogException("Error creating logger for logger " + logger + ".  Cause: " + t, t);
    }
  }  

}
```
可以看到，先获取了类的静态变量logConstructor，即日志类的构造器，然后通过构造器对日志类进行实例化，然后进行缓存logConstructor。

## 1.2 日志实现类在LogFactory的缓存
LogFactory缓存了日志实现类的构造器，构造器的设置主要有两种方式。

1. 第一种是LogFactory静态代码块中尝试设置Mybatis自带日志实现类的构造器。
2. 第二种是设置自定义日志实现类的构造器。

### 1.2.1 LogFactory静态代码块缓存日志实现类构造器
```
public final class LogFactory {

  static {
    tryImplementation(LogFactory::useSlf4jLogging);
    tryImplementation(LogFactory::useCommonsLogging);
    tryImplementation(LogFactory::useLog4J2Logging);
    tryImplementation(LogFactory::useLog4JLogging);
    tryImplementation(LogFactory::useJdkLogging);
    tryImplementation(LogFactory::useNoLogging);
  }

  public static synchronized void useSlf4jLogging() {
    setImplementation(org.apache.ibatis.logging.slf4j.Slf4jImpl.class);
  }
  
  public static synchronized void useCommonsLogging() {
    setImplementation(org.apache.ibatis.logging.commons.JakartaCommonsLoggingImpl.class);
  }
  
  public static synchronized void useLog4J2Logging() {
    setImplementation(org.apache.ibatis.logging.log4j2.Log4j2Impl.class);
  }
  
  public static synchronized void useLog4JLogging() {
    setImplementation(org.apache.ibatis.logging.log4j.Log4jImpl.class);
  }  
  
  public static synchronized void useJdkLogging() {
    setImplementation(org.apache.ibatis.logging.jdk14.Jdk14LoggingImpl.class);
  } 
  
  public static synchronized void useNoLogging() {
    setImplementation(org.apache.ibatis.logging.nologging.NoLoggingImpl.class);
  }  
  
  private static void tryImplementation(Runnable runnable) {
    if (logConstructor == null) {
      try {
        runnable.run();
      } catch (Throwable t) {
        // ignore
      }
    }
  }

  private static void setImplementation(Class<? extends Log> implClass) {
    try {
      Constructor<? extends Log> candidate = implClass.getConstructor(String.class);
      Log log = candidate.newInstance(LogFactory.class.getName());
      if (log.isDebugEnabled()) {
        log.debug("Logging initialized using '" + implClass + "' adapter.");
      }
      logConstructor = candidate;
    } catch (Throwable t) {
      throw new LogException("Error setting Log implementation.  Cause: " + t, t);
    }
  }
  
}
```

- Logfactory在初始化时即执行静态代码块按照一定顺序执行tryImplementation方法尝试设置静态变量logConstructor，在tryImplementation代码中加入了判断，如果logConstructor不为null则不会再尝试执行Runable方法设置logConstructor，并且忽略了在类路径找不到对应日志框架时的异常。
- tryImplementation方法的参数为Runable对象，Runable的run方法调用Logfactory的useSlf4jLogging、useCommonsLogging等方法，这些方法调用Logfactory的setImplementation方法对Mybatis日志实现类进行实例化，并且保存日志实现类的构造器引用到logConstructor。因为tryImplementation方法已经捕获了异常，因此程序不会异常终止。
- setImplementation方法的参数为Mybatis的日志实现类的Class对象，这些日志实现类均使用了相似的方法包装具体的日志框架，打印日志的工作由具体的日志框架来完成。

### 1.2.2 缓存自定义日志实现类构造器
自定义日志实现类的设置步骤：
1. 自定义日志实现类CustomLog实现org.apache.ibatis.logging.Log
2. 在mybatis配置文件properties中配置logImpl即可，如下
```
<configuration>
    <properties>
        <property name="logImpl" value="com.zzuhkp.practice.CustomLog"/>
    </properties>
</configuration>
```
Mybatis的Configuration构造器中注册了一些日志实现类的别名，因此我们也能自己指定Mybatis中自带的具体的日志实现类，如SLF4J。Configuration构造方法的部分源码如下：
```
public class Configuration {

  public Configuration() {
    ...省略部分代码

    typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
    typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
    typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
    typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
    typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
    typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
    typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
    
    ...省略部分代码
  }

}
```

那么Mybatis如何设置自定义日志实现类的呢？源码藏在了XMLConfigBuilder类的parseConfiguration方法中，部分源码如下：

```
public class XMLConfigBuilder extends BaseBuilder {

  private void parseConfiguration(XNode root) {
    try {
      ... 省略部分代码
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      ... 省略部分代码
      loadCustomLogImpl(settings);
      ... 省略部分代码
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
  
  private void loadCustomLogImpl(Properties props) {
    Class<? extends Log> logImpl = resolveClass(props.getProperty("logImpl"));
    configuration.setLogImpl(logImpl);
  }
  
}

public class Configuration {

  public void setLogImpl(Class<? extends Log> logImpl) {
    if (logImpl != null) {
      this.logImpl = logImpl;
      LogFactory.useCustomLogging(this.logImpl);
    }
  }

}

public final class LogFactory {

  public static synchronized void useCustomLogging(Class<? extends Log> clazz) {
    setImplementation(clazz);
  }
  
}
```

XMLConfigBuilder类对配置文件解析时获取setting配置的logImpl值，然后会调用LogFactory.useCustomLogging方法，最后会调用LogFactory.setImplementation方法进行设置。

## 1.3 Mybatis日志实现类是如何封装日志框架的
Mybatis日志实现类封装了日志框架，保存日志框架的日志对象到成员变量中，具体的打印工作由日志框架的日志对象来完成。

Mybatis日志实现类有一个共同的接口Log,Log接口主要定义了不同日志级别的打印方法，源码如下：
```
public interface Log {

  boolean isDebugEnabled();

  boolean isTraceEnabled();

  void error(String s, Throwable e);

  void error(String s);

  void debug(String s);

  void trace(String s);

  void warn(String s);

}

```

以Slf4j的日志实现类Slf4jImpl为例，部分源码如下：
```
public class Slf4jImpl implements Log {

  private Log log;

  public Slf4jImpl(String clazz) {
    Logger logger = LoggerFactory.getLogger(clazz);

    if (logger instanceof LocationAwareLogger) {
      try {
        // check for slf4j >= 1.6 method signature
        logger.getClass().getMethod("log", Marker.class, String.class, int.class, String.class, Object[].class, Throwable.class);
        log = new Slf4jLocationAwareLoggerImpl((LocationAwareLogger) logger);
        return;
      } catch (SecurityException | NoSuchMethodException e) {
        // fail-back to Slf4jLoggerImpl
      }
    }

    // Logger is not LocationAwareLogger or slf4j version < 1.6
    log = new Slf4jLoggerImpl(logger);
  }

  @Override
  public boolean isDebugEnabled() {
    return log.isDebugEnabled();
  }

  @Override
  public void debug(String s) {
    log.debug(s);
  }
}

class Slf4jLocationAwareLoggerImpl implements Log {

  private final LocationAwareLogger logger;

  Slf4jLocationAwareLoggerImpl(LocationAwareLogger logger) {
    this.logger = logger;
  }

  @Override
  public boolean isDebugEnabled() {
    return logger.isDebugEnabled();
  }

  @Override
  public void debug(String s) {
    logger.log(MARKER, FQCN, LocationAwareLogger.DEBUG_INT, s, null, null);
  }

}

class Slf4jLoggerImpl implements Log {

  private final Logger log;

  public Slf4jLoggerImpl(Logger logger) {
    log = logger;
  }

  @Override
  public boolean isDebugEnabled() {
    return log.isDebugEnabled();
  }

  @Override
  public void debug(String s) {
    log.debug(s);
  }

}

```
Slf4jImpl类保存了Mybatis的其他日志对象引用到成员变量，其他日志对象则封装了slf4j的日志对象。在实例化Slf4jImpl时获取slf4j的日志对象Logger，然后检查slf4j的版本，如果slf4j的版本号在1.6及以上则保存Slf4jLocationAwareLoggerImpl日志对象到成员变量，否则保存Slf4jLoggerImpl日志对象到成员变量。

另外Mybatis在pom文件中的dependency中设置了optional的值为true，因此能够编译通过，而在我们实际使用Mybatis中则会使用某一个具体的日志框架。
```
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>1.7.26</version>
      <optional>true</optional>
    </dependency>
```
# 二、Mybatis中JDBC执行的日志打印

Mybatis使用JDK动态代理对JDBC中的Connection、PreparedStatement、ResultSet、Statement进行代理，当执行JDBC中的相关方法时，会使用Log打印相关的日志。具体如下：
{% asset_img 'Mybatis日志框架类图.jpg' Mybatis日志框架类图 %}
BaseExecutor中的getConnection方法会获取Connection的代理对象。以ConnectionLogger为例，源码如下：
```
public abstract class BaseExecutor implements Executor {

  protected Connection getConnection(Log statementLog) throws SQLException {
    Connection connection = transaction.getConnection();
    if (statementLog.isDebugEnabled()) {
      return ConnectionLogger.newInstance(connection, statementLog, queryStack);
    } else {
      return connection;
    }
  }
  
}

public final class ConnectionLogger extends BaseJdbcLogger implements InvocationHandler {

  private final Connection connection;

  private ConnectionLogger(Connection conn, Log statementLog, int queryStack) {
    super(statementLog, queryStack);
    this.connection = conn;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] params)
      throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, params);
      }
      if ("prepareStatement".equals(method.getName())) {
        if (isDebugEnabled()) {
          debug(" Preparing: " + removeBreakingWhitespace((String) params[0]), true);
        }
        PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
        stmt = PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
        return stmt;
      } else if ("prepareCall".equals(method.getName())) {
        if (isDebugEnabled()) {
          debug(" Preparing: " + removeBreakingWhitespace((String) params[0]), true);
        }
        PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
        stmt = PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
        return stmt;
      } else if ("createStatement".equals(method.getName())) {
        Statement stmt = (Statement) method.invoke(connection, params);
        stmt = StatementLogger.newInstance(stmt, statementLog, queryStack);
        return stmt;
      } else {
        return method.invoke(connection, params);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }

  public static Connection newInstance(Connection conn, Log statementLog, int queryStack) {
    InvocationHandler handler = new ConnectionLogger(conn, statementLog, queryStack);
    ClassLoader cl = Connection.class.getClassLoader();
    return (Connection) Proxy.newProxyInstance(cl, new Class[]{Connection.class}, handler);
  }

  public Connection getConnection() {
    return connection;
  }

}

public abstract class BaseJdbcLogger {

  protected void debug(String text, boolean input) {
    if (statementLog.isDebugEnabled()) {
      statementLog.debug(prefix(input) + text);
    }
  }

  protected void trace(String text, boolean input) {
    if (statementLog.isTraceEnabled()) {
      statementLog.trace(prefix(input) + text);
    }
  }
  
  private String prefix(boolean isInput) {
    char[] buffer = new char[queryStack * 2 + 2];
    Arrays.fill(buffer, '=');
    buffer[queryStack * 2 + 1] = ' ';
    if (isInput) {
      buffer[queryStack * 2] = '>';
    } else {
      buffer[0] = '<';
    }
    return new String(buffer);
  }

}
```
- BaseExecutor.getConnection方法调用ConnectionLogger.newInstance获取Connection的代理对象ConnectionLogger，ConnectionLogger持有Connection和Log的引用，当执行Connection的方法时，会使用Log打印相关的日志信息。调用Connection的方法获取PreparedStatement时会调用PreparedStatementLogger.newInstance获取PreparedStatement的代理对象，获取Statement时会调用StatementLogger.newInstance方法获取Statement的代理对象。
- 另外打印debug和trace级别的日志时还会设置日志的前缀，使用BaseJdbcLogger类的prefix方法获取前缀，所以我们能够在日志上看到前缀“\==> ”或者“<== ”。

# 总结
Mybatis日志框架中使用的设计模式包括工厂方法、代理模式值得我们学习。
  
---
title: java基础知识之序列化serializable
date: 2018-08-11 01:06:54
categories: 
- java基础知识 
tags:
- java
- 序列化
- serializable
description: 序列化是java中的基础知识，在远程过程调用RPC中需要对java类进行序列化和反序列化。
---

**一、序列化概念**：将对象在内存中的状态保存下来，在需要的时候获取。  
- 序列化：将对象转换为字节数组，以便在网络传输或存储。  
- 反序列化：将字节数组转换为对象。  

**二、序列化特点**:  
- 类必须实现Serializable接口，父类未实现Serializable接口则父类不参与序列化，父类实现Serializable接口后子类不需要显式实现Serializable接口；
- 类的静态变量不参与序列化；
- transient关键字修饰的变量不参与序列化；
- 序列化ID必须一致；
- 对象的实例变量引用其他对象，引用的对象也将参与序列化  

**三、序列化方法：**  
1. 创建OutputStream；  
`OutputStream outputStream = new FileOutputStream("object.txt");`  
2. 创建ObjectOutputStream；  
`ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);`
3. 调用ObjectOutputStream的writeObject方法进行序列化；  
`objectOutputStream.writeObject(obj);`  
4. 关闭输出流；  
`objectOutputStream.close();`  

**四、反序列化方法**
1. 创建InputStream；  
`InputStream inputStream = new FileInputStream("object.txt");`
2. 创建ObjectInputStream；  
`ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);`
3. 调用ObjectInputStream的readObject方法进行反序列化； 
`Object obj = objectInputStream.readObject();`  
4. 关闭输出流；  
`objectInputStream.close();`

**五、序列化实例**  

**5.1 实例一:类必须实现Serializable接口**  
- Serializable接口是一个标记接口，本身并没有定义方法，但是类必须实现Serializable接口才能进行序列化，否则进行序列化时将抛出NotSerializableException异常。 
```java
public interface Serializable {
}
```

- 类User包含username和password两个成员变量，并未实现Serializable接口。

```java
package com.zzuhkp.javanote.serializable;

public class User{
    private String username;
    private String password;

    public User() {
        System.out.println("User no arg constructor");
    }

    public User(String username, String password) {
        System.out.println("User with arg constructor");
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```
- 测试类Main对User类中的成员变量赋值，进行序列化后然后反序列化打印User信息；

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        writeObj(user);
        System.out.println(readObj());
    }

    public static void writeObj(Object obj) throws IOException {
        OutputStream outputStream = new FileOutputStream("object.txt");
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
        objectOutputStream.writeObject(obj);
        objectOutputStream.close();
    }

    public static Object readObj() throws IOException, ClassNotFoundException {
        InputStream inputStream = new FileInputStream("object.txt");
        ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
        Object obj = objectInputStream.readObject();
        objectInputStream.close();
        return obj;
    }

}
```

- 程序运行后抛出java.io.NotSerializableException异常，如下所示；

```java
User with arg constructor
Exception in thread "main" java.io.NotSerializableException: com.zzuhkp.javanote.serializable.User
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at com.zzuhkp.javanote.serializable.Main.writeObj(Main.java:16)
	at com.zzuhkp.javanote.serializable.Main.main(Main.java:9)
```

**5.2 实例二 未实现Serializable的父类不参与序列化**
- 创建类CommonVO,其只有一个成员变量id，该类未实现Serializable接口；

```java
package com.zzuhkp.javanote.serializable;

public class CommonVO {
    private String id;

    public CommonVO(){
        System.out.println("CommonVO no arg constructor");
    }

    public CommonVO(String id) {
        System.out.println("CommonVO with arg constructor");
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    @Override
    public String toString() {
        return "CommonVO{" +
                "id='" + id + '\'' +
                '}';
    }
}
```

- 使User类继承CommonVO类；

```java
package com.zzuhkp.javanote.serializable;

public class User extends CommonVO implements Serializable {
    //省略代码和实例一相同，未做改变
}
```

- 修改测试类如下所示：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        user.setId("123");
        writeObj(user);
        User user1= (User) readObj();
        System.out.println(user1.toString());
        System.out.println("id:"+user1.getId());
    }

    //省略代码和实例一相同，未做改变

}
```

- 程序运行结果如下所示，可以看到反序列化并没有把前面设置的父类成员变量id的值打印出来，并且可以看到反序列化时程序调用了User的无参构造方法创建了User的实例；

```java
CommonVO no arg constructor
User with arg constructor
CommonVO no arg constructor
User{username='zzuhkp', password='123456'}
id:null
```
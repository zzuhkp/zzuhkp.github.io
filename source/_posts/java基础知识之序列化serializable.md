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

# 一、序列化概念
将对象在内存中的状态保存下来，在需要的时候获取。  
- 序列化：将对象转换为字节序列，以便在网络传输或存储。  
- 反序列化：将字节序列转换为对象。  

# 二、序列化特点  
- 类必须实现Serializable接口，父类未实现Serializable接口则父类不参与序列化，父类实现Serializable接口后子类不需要显式实现Serializable接口；
- 类的静态变量不参与序列化；
- transient关键字修饰的变量不参与序列化；
- 序列化对象的成员变量serialVersionUID和反序列化的对象的成员变量serialVersionUID值必须一致；
- 对象的实例变量引用其他对象，引用的对象也将参与序列化  

# 三、序列化步骤
1. 创建OutputStream；  
`OutputStream outputStream = new FileOutputStream("object.txt");`  
2. 创建ObjectOutputStream；  
`ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);`
3. 调用ObjectOutputStream的writeObject方法进行序列化；  
`objectOutputStream.writeObject(obj);`  
4. 关闭输出流；  
`objectOutputStream.close();`  

# 四、反序列化步骤
1. 创建InputStream；  
`InputStream inputStream = new FileInputStream("object.txt");`
2. 创建ObjectInputStream；  
`ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);`
3. 调用ObjectInputStream的readObject方法进行反序列化； 
`Object obj = objectInputStream.readObject();`  
4. 关闭输出流；  
`objectInputStream.close();`

# 五、序列化实例  

## 5.1 实例一:类必须实现Serializable接口 
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

## 5.2 实例二 未实现Serializable的父类不参与序列化
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

## 5.3 实例三 类的静态变量和transient修饰的变量不参与序列化；
- 修改实例一中的User如下所示；

```java
package com.zzuhkp.javanote.serializable;

import java.io.Serializable;

public class User  implements Serializable {
    public static String USER_TYPE="1";
    private transient String password;
    
    //省略代码和实例一中的User一致
}
```

- 修改测试类如下所示；

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        writeObj(user);
        User.USER_TYPE="2";
        User user1= (User) readObj();
        System.out.println(user1.toString());
        System.out.println(User.USER_TYPE);
    }

    //省略代码和实例一中的Main类保持一致

}
```

- 序列化后修改User类的静态变量USER_TYPE，程序运行，打印结果如下。反序列化后获取的静态变量的值和序列化之前并不保持一致，说明类的静态变量不参数序列化；transient修饰的password变量在反序列化后并未打印出序列化之前设置的值，说明transient修饰的成员变量也不参与序列化。

```java
User with arg constructor
User{username='zzuhkp', password='null'}
2
```
## 5.4 实例四 序列化ID serialVersionUID必须一致
- 修改User类如下所示，注意serialVersionUID的值为1L。

```java
package com.zzuhkp.javanote.serializable;

import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;
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

- 新建SocketClient类作为Socket连接的客户端，代码如下：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;
import java.net.Socket;

public class SocketClient {

    public static void main(String[] args) throws IOException {
        Socket socket=new Socket("127.0.0.1",8080);
        User user = new User("zzuhkp", "123456");
        OutputStream outputStream = socket.getOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
        objectOutputStream.writeObject(user);
        objectOutputStream.close();
    }
}
```

- 新建项目，并在新项目中新建User类,User类的serialVersionUID为2L，其他内容和原项目User保值一致，代码如下；

```java
package com.zzuhkp.javanote.serializable;

import java.io.Serializable;

public class User  implements Serializable {
    private static final long serialVersionUID = 2L;
    
    //省略代码和实例四上面User内容一致
}
```

- 在新项目中新建SocketServer类作为Socket的服务端，代码如下：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ServerSocket serverSocket=new ServerSocket(8080);
        Socket socket=serverSocket.accept();

        InputStream inputStream = socket.getInputStream();
        ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
        Object obj = objectInputStream.readObject();
        objectInputStream.close();
        System.out.println(obj.toString());
    }
}
```

- 分别运行SocketServer和SocketClient，SocketServer程序运行报InvalidClassException异常，并提示 stream classdesc serialVersionUID = 1, local class serialVersionUID = 2，说明serialVersionUID必须保持一致才能序列化成功。

```java
Exception in thread "main" java.io.InvalidClassException: com.zzuhkp.javanote.serializable.User; local class incompatible: stream classdesc serialVersionUID = 1, local class serialVersionUID = 2
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:687)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1883)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1749)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2040)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1571)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:431)
	at com.zzuhkp.javanote.serializable.SocketServer.main(SocketServer.java:15)
```

# 六、序列化对象的writeObject方法和readObject方法

- 序列化时ObjectOutputStream将通过反射获取并尝试调用序列化对象的`private void writeObject(ObjectOutputStream objectOutputStream)`方法进行序列化，如果不存在该方法则ObjectOutputStream使用默认的序列化方式进行序列化。在writeObject方法中可以先调用`objectOutputStream.defaultWriteObject()`方法进行默认的序列化，然后调用ObjectOutputStream类的其他方法序列化；
- 反序列化时ObjectInputStream将通过反射获取并尝试调用序列化对象的`private void readObject(ObjectInputStream objectInputStream)`方法进行反序列化，如果不存在该方法则ObjectInputStream使用默认的反序列化方式进行反序列化。在readObject中可以先调用`objectInputStream.defaultReadObject()`方法进行默认的反序列化，然后调用ObjectInputStream类的其他方法反序列化；  
- 修改User类如下所示：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String username;
    private transient String password;

    private void writeObject(ObjectOutputStream objectOutputStream) throws IOException, ClassNotFoundException {
        objectOutputStream.defaultWriteObject();
        objectOutputStream.writeObject(this.password);
    }

    private void readObject(ObjectInputStream objectInputStream) throws IOException, ClassNotFoundException {
        objectInputStream.defaultReadObject();
        this.password = (String) objectInputStream.readObject();
    }
    
    //省略代码和实例一中的User类代码相同
}
```

- 修改实例一中的Main类如下所示：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        writeObj(user);
        User user1= (User) readObj();
        System.out.println(user1.toString());
    }

    //省略代码和实例一中的Main类代码一致
}
```

- 运行结果如下，说明使用自定义的序列化方式对transient修饰的成员变量序列化已经生效。

```java
User with arg constructor
User{username='zzuhkp', password='123456'}
```

# 七、序列化接口Externalizable

Externalizable接口继承Serializable接口，并声明了两个方法用于分别对对象进行序列化和反序列化，查看Externalizable的主要源码如下：

```
package java.io;

import java.io.ObjectOutput;
import java.io.ObjectInput;

public interface Externalizable extends java.io.Serializable {
    
    void writeExternal(ObjectOutput out) throws IOException;
    
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

一个类实现Externalizable接口，该类即可参与序列化，此时需要我们自己调用writeExternal方法进行序列化，调用readExternal方法进行反序列化，并且实现Serializable接口进行的默认序列化方式不会生效。

修改实例一中的User类如下：

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class User implements Externalizable {
    private static final long serialVersionUID = 1L;
    private String username;
    private String password;

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(this.username);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        this.username = (String) in.readObject();
    }

    //省略代码和实例一中User的代码一致

}
```

修改Main类的代码如下：

```
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        writeObj(user);
        User user1= (User) readObj();
        System.out.println(user1.toString());
    }

    //省略代码和实例一中Main方法的代码一致

}
```

在User类中只对成员变量username进行序列化和反序列化，运行结果如下所示,说明我们自定义的序列化和反序列化已经生效：

```java
User with arg constructor
User no arg constructor
User{username='zzuhkp', password='null'}
```

# 八、序列化对象的writeReplace和readResolve方法

- 如果序列化对象包含Object writeReplace() throws ObjectStreamException方法，在序列化时将使用writeReplace方法的返回值作为序列化的对象替代原序列化对象；
- 如果序列化对象包含Object readResolve() throws ObjectStreamException方法，在反序列化时将使用readResolve方法的返回值作为反序列化的对象替代原反序列化对象。

修改实例一中的User类如下所示，注意添加了writeReplace()和 readResolve()方法。

```
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String username;
    private String password;

    private Object writeReplace() throws ObjectStreamException{
        System.out.println("writeReplace() is called");
        User user=new User("writeReplace Name","123");
        return user;
    }

    private Object readResolve() throws ObjectStreamException{
        System.out.println("readResolve() is called");
        User user=new User("readResolve Name","456");
        return user;
    }

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

测试类Main类代码如下:

```java
package com.zzuhkp.javanote.serializable;

import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        User user = new User("zzuhkp", "123456");
        writeObj(user);
        User user1= (User) readObj();
        System.out.println(user1.toString());
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

运行结果如下，说明反序列化的对象已经被readResolve()方法的返回值替代。

```java
User with arg constructor
writeReplace() is called
User with arg constructor
readResolve() is called
User with arg constructor
User{username='readResolve Name', password='456'}
```
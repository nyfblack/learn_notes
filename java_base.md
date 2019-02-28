# 1.transient关键字

## 1.1 transient的作用及使用方法

​     对象实现了Serilizable接口，就可以被自动序列化；

​	在实际开发过程中，常常会遇到类的有些属性需要序列化，而其他属性不需要被序列化，如：用户的一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

​      java 的transient关键字，实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

```java
public class TransientTest {
    public static void main(String[] args) {
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");
        
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        try {
            //序列化
            ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("C:/user.txt"));
            os.writeObject(user); // 将User对象写进文件             
            os.flush();
            os.close();
            
            //反序列化
            ObjectInputStream is = new ObjectInputStream(new FileInputStream("C:/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据             
            is.close();
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.err.println("password: " + user.getPasswd());
        } catch (Exception e) {
            e.printStackTrace();
        } 
    }
}
class User implements Serializable {
    private static final long serialVersionUID = 8294180014912103005L;  
    
    private String username;
    private transient String passwd;
   
    public String getUsername() {return username;}
    public void setUsername(String username) {this.username = username;}
    public String getPasswd() {return passwd;}
    public void setPasswd(String passwd) {this.passwd = passwd;}
}

//read before Serializable: //username: Alexia//password: 123456 
//read after Serializable: //username: Alexia//password: null
```



## 1.2 transient使用小结

- 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

- transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

- 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

  反序列化后类中static型变量的值为当前JVM中对应static变量的值，而不是反序列化得出的。

```java
public class TransientTest {
    
    public static void main(String[] args) {
        
        User user = new User();
        user.setUsername("Alexia");
        user.setPasswd("123456");
        
        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.err.println("password: " + user.getPasswd());
        
        try {
            //序列化
            ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("C:/user.txt"));
            os.writeObject(user); // 将User对象写进文件             
            os.flush();
            os.close();
            // 在反序列化之前改变username的值             
            User.username = "jmwang";
            ObjectInputStream is = new ObjectInputStream(new FileInputStream("C:/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据             
            is.close();
            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.err.println("password: " + user.getPasswd());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
class User implements Serializable {
    private static final long serialVersionUID = 8294180014912103005L;  
    
    private String username;
    private transient String passwd;
   
    public String getUsername() {return username;}
    public void setUsername(String username) {this.username = username;}
    public String getPasswd() {return passwd;}
    public void setPasswd(String passwd) {this.passwd = passwd;}
    }
}

//read before Serializable: //username: Alexia //password: 123456 
//read after Serializable: //username: jmwang //password: null
```

​	在Java中，对象的序列化可以通过实现两种接口来实现，若实现的是Serializable接口，则所有的序列化将会自动进行，若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。

```java
public class ExternalizableTest implements Externalizable {
    private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";
    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(content);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException,ClassNotFoundException {
        content = (String) in.readObject();
    }

    public static void main(String[] args) throws Exception {
        ExternalizableTest et = new ExternalizableTest();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream(new File("test")));
        out.writeObject(et);

        ObjectInput in = new ObjectInputStream(new FileInputStream(new File("test")));
        et = (ExternalizableTest) in.readObject();
        System.out.println(et.content);
        out.close();
        in.close();
    }
}
//输出：是的，我将会被序列化，不管我是否被transient关键字修饰
```


# 2.单线程池

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
executorService.execute(() -> { });
```




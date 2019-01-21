## 1.Lambda语法

### 1.1 无参数

```java
() -> {
    for (int i = 100; i >= 0; i--){
        System.out.println(i);
    }
}
```

### 1.2 含有一个参数

```java
ActionListener listener = event -> { 
    // 或 (event) -> . . . or (ActionEvent event) -> . . .
    System.out.println("The time is " + new Date());
};
```

### 1.3 含有多个参数

```java
Comparator<String> comp = (first, second) -> {
    // 或 (String first, String second) ->
    first.length() - second.length();
};
```

## 2.函数式接口

这是Java在解决函数式编程，引入Lambda表达式的同时引入的一个概念。

**函数式接口**：定义的一个接口，接口里面必须有且只有一个方法，这样的接口就成为函数式接口。 

​	在可以使用Lambda表达式的地方，方法声明时必须包含一个函数式的接口。任何函数式接口都可以使用Lambda表达式替换。 

```java
@FunctionalInterface	//用此注解标注的接口，只能是函数式接口；
public interface FunctionInterface {
    void test();
}
```

## 3.调用方法

### 3.1 方法引用

- object::instanceMethod

```java
this::equals  // x -> this.equals(x);
    
Timer t = new Timer(1000, super::greet);
```

- Class::staticMethod

```java
Timer t = new Timer(1000, System.out::println);  //相当于  x -> System.out.println(x); 

Math::pow  //  相当于  (x, y) -> Math.pow(x, y); 
```

- Class::instanceMethod

```java
//相当于  (x, y) -> x.compareToIgnoreCase(y);
Arrays.sort(strings, String::compareToIgnoreCase);  
```

### 3.2 构造器引用

- 单个对象

```java
ArrayList<String> names = . . .;

Stream<Person> stream = names.stream().map(Person::new);

List<Person> people = stream.collect(Collectors.toList());
```

- 对象数组

```java
Person[] people = stream.toArray(Person[]::new);

int[]::new  // x -> new int[x];
```

## 4.变量作用域

1. Lambda表达式能够引用外部方法的final类型（或相当于final类型：只赋值一次）的对象，不能引用非final类型的对象。

```java
public static void repeatMessage(String text, int delay){
    ActionListener listener = event ->{
        System.out.println(text); // OK
        Toolkit.getDefaultToolkit().beep();
    };
    new Timer(delay, listener).start();
}

public static void repeat(String text, int count){
     for (int i = 1; i <= count; i++){
         ActionListener listener = event ->{
             // Error: Cannot refer to changing i
             System.out.println(i + ": " + text); 
         };
         new Timer(1000, listener).start();
     }
 }
```

 

2. Lambda表达式不能够修改外部方法的final类型（或相当于final类型）的对象的引用。

```java
public static void countDown(int start, int delay){
     ActionListener listener = event -> {
         start--; // Error: Can't mutate captured variable
         System.out.println(start);
     };
     new Timer(delay, listener).start();
 }
```



3. Lambda表达式的语句体与Lambda表达式所在的方法具有相同的作用域，所以Lambda表达式中定义的变量名不能与Lambda表达式所在的方法中的变量名同名。

```java
Path first = Paths.get("/usr/bin");
// Error: Variable first already defined
Comparator<String> comp = (first, second) -> first.length() - second.length();
```

 

4. Lambda表达式的语句体中的this与Lambda表达式所在的方法中的this相同。

```java
public class Application () {
     public void init () {
         ActionListener listener = event -> {
             System.out.println(this.toString()); // this 指向Application对象的实例
             ...
         }...
     }
 }
```

## 5.应用Lambda表达式的场景 

- Running the code in a separate thread 在一个单独的线程中执行代码

- Running the code multiple times 多次执行同一份代码

- Running the code at the right point in an algorithm (for example, the comparison operation in sorting)  在一个算法中，在一个恰当的时间点执行代码

- Running the code when something happens (a button was clicked, data has arrived, and so on)

当事件发生时执行代码

- Running the code only when necessary 当需要的时候才执行代码



 



 






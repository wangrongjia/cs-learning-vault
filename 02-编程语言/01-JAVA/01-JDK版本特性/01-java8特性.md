---
dg-publish: true
---
#  what's new in Java 8
官网： http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html
##  Lambda 表达式
lambda表达式是一段可以传递的代码，它的核心思想是将面向对象中的传递数据变成传递行为
Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）  
以下是lambda表达式的重要特征:

>1. 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。   
>2. 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。  
>3. 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。   
>4. 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。  

demo:
```java
class Test {

  public static void main(String[] args) {

    new Thread(new Runnable() {
      public void run() {
        System.out.println("java7");
      }
    }).start();

    //这是一个语法糖
    new Thread(() -> System.out.println("java8")).start();

  }
}
```

Lambda 表达式主要用来定义行内执行的方法类型接口,免去了使用匿名方法的麻烦    
```java
  interface MathOperation {
    int operation(int a, int b);
  }

  // 类型声明
  MathOperation addition = (int a, int b) -> a + b;

  private int operate(int a, int b, MathOperation mathOperation) {
    return mathOperation.operation(a, b);
  }

  public static void main(String args[]) {
    Java8Tester tester = new Java8Tester();
    System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
  }
```
### 注意点1 lambda表达式引用外层局部变量，不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误，也就是说在lambda表达式内部，其引用的外层局部变量默认都被final修饰。

```java
public class Java8Tester {
  public static void main(String args[]) {
    int num = 1;
    Converter<Integer, String> s = (param) -> {
      System.out.println(String.valueOf(param + num++));//报错：Local variable num defined in an enclosing scope must be final or effectively final
    };
    s.convert(2); // 输出结果为 3
  }

  public interface Converter<T1, T2> {
    void convert(int i);
  }
}
```
### 注意点2 lambda表达式的参数和外层局部变量不能重名

## 方法引用
在学习lambda表达式之后，我们通常使用lambda表达式来创建匿名方法。然而，有时候我们仅仅是调用了一个已存在的方法。如下：  
```java
Arrays.sort(arr,(s1,s2)->s1.compareToIgnoreCase(s2));
```
 在Java8中，我们可以直接通过方法引用来简写lambda表达式中已经存在的方法。  
```java
Arrays.sort(stringsArray, String::compareToIgnoreCase);
```
这种特性就叫做方法引用(Method Reference)。
方法引用的四种方式：
类型	| 示例
---|---
引用静态方法 | ContainingClass::staticMethodName
引用某个对象的实例方法	 | containingObject::instanceMethodName
引用某个类型的任意对象的实例方法 | 	ContainingType::methodName
引用构造方法	 | ClassName::new

```java
class Test {

  public static int compare(String a, String b) {
    return a.compareTo(b);
  }

  public int compareObject(String a, String b) {
    return a.compareTo(b);
  }

  public static void main(String[] args) {

    String[] arr = {"a", "b", "c"};

    // 不使用lambda表达式的写法
    Arrays.sort(arr, new Comparator<String>() {
      public int compare(String a, String b) {
        return a.compareTo(b);
      }
    });

    // 静态引用  
    Arrays.sort(arr, Test::compare);    //因为arr是字符串数组，所以可以传入compare方法(字符串有compareTo方法)

    // 实例方法引用，可以是自己的方法，父类的，或者类型上的方法引用
    Arrays.sort(arr, new Test()::compareObject);// 当引用的方法有泛型时，泛型写在::之后，方法之前

    // 构造方法引用
    Supplier<Test> sup = () -> new Test();

    Supplier<Test> supp = Test::new;
  }

}
```
## 接口的默认方法

Java 8 新增了接口的默认方法。
以前 接口的方法默认都是用   public abstract修饰的 
java8 之后 默认方法就是接口的方法用default修饰的时候可以有方法体，还可以有静态方法
```java
class Test {
  interface inter {
    default void print() {
      System.out.println("inter");
    }

    static void staticPrint() {
      System.out.println("static inter");
    }
  }

  interface inter1 {
    default void print() {
      System.out.println("inter1");
    }

  }

  class inner implements inter, inter1 {
    // 当一个类实现的多个接口中有同名的默认方法时，需要指定其中一个接口的方法作为类的方法或者自己重写一个同名方法
    public void print() {
      inter.super.print();// 指定inter接口的print方法作为类的方法
    }

  }

  public static void main(String[] args) {
    // 可以直接调用接口的默认方法和静态方法，不需要重写
    inter.staticPrint();
    new Test().new inner().print();
  }
}
```
## 函数式接口
函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
@FunctionalInterface  修饰 表名这是一个函数式接口  只可以有一个抽象方法  
函数式接口可以被隐式转换为lambda表达式。
函数式接口可以现有的函数友好地支持 lambda。
JDK 1.8之前已有的函数式接口:
java.lang.Runnable
java.util.concurrent.Callable
java.security.PrivilegedAction
java.util.Comparator
java.io.FileFilter
java.nio.file.PathMatcher
java.lang.reflect.InvocationHandler
java.beans.PropertyChangeListener
java.awt.event.ActionListener
javax.swing.event.ChangeListener
JDK 1.8 新增加的函数接口：

java.util.function
java.util.function 它包含了很多类，用来支持 Java的 函数式编程，该包中的函数式接口有：


其中的四个核心  demo:
```java
class Test {

  public static void main(String[] args) {

    /**
     * @name 消费型接口
     * @use Consumer<T>
     * @param T 传入参数
     * @fun 接受一个参数 无返回值
     */
    Consumer<String> con = (str) -> System.out.println(str);
    con.accept("我是消费型接口!");

    /**
     * @name 供给型接口
     * @use Supplier<R>
     * @param R 返回值类型
     * @fun 无参数 有返回值
     */
    Supplier<Date> supp = Date::new;
    Date date = supp.get();
    System.out.println("当前时间:" + date);

    /**
     * @name 函数型接口
     * @use Function<T,R>
     * @param T 传入参数
     * @return R 返回值类型
     * @fun 接受一个参数 有返回值
     */
    Function<String, String> fun = (str) -> "hello," + str;
    String str = fun.apply("123");
    System.out.println(str);
    /**
     * @name 断定型接口
     * @use Predicate<T>
     * @param T 传入参数
     * @return Boolean 返回一个Boolean型值
     * @fun 接受一个参数 返回Boolean型值
     */
    Predicate<Integer> pre = (num) -> num > 0;
    Boolean flag = pre.test(10);
    System.out.println(flag);
  }
}
```

## stream
Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。
Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。
Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。
这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。
元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

### 什么是 Stream？
Stream（流）是一个来自数据源的元素队列并支持聚合操作

>元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。  
>**数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。  
>**聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。   

和以前的Collection操作不同， Stream操作还有两个基础的特征：

>**Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。   
>**内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。    

### 生成流
在 Java 8 中, 集合接口有两个方法来生成流：

>**stream()** − 为集合创建串行流。  
>**parallelStream() **− 为集合创建并行流。  

demo:
```java
 public static void main(String args[]){
      System.out.println("使用 Java 7: ");
        
      // 计算空字符串
      List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
      System.out.println("列表: " +strings);
      long count = getCountEmptyStringUsingJava7(strings);
        
      System.out.println("空字符数量为: " + count);
      count = getCountLength3UsingJava7(strings);
        
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      // 删除空字符串
      List<String> filtered = deleteEmptyStringsUsingJava7(strings);
      System.out.println("筛选后的列表: " + filtered);
        
      // 删除空字符串，并使用逗号把它们合并起来
      String mergedString = getMergedStringUsingJava7(strings,", ");
      System.out.println("合并字符串: " + mergedString);
      List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
        
      // 获取列表元素平方数
      List<Integer> squaresList = getSquares(numbers);
      System.out.println("平方数列表: " + squaresList);
      List<Integer> integers = Arrays.asList(1,2,13,4,15,6,17,8,19);
        
      System.out.println("列表: " +integers);
      System.out.println("列表中最大的数 : " + getMax(integers));
      System.out.println("列表中最小的数 : " + getMin(integers));
      System.out.println("所有数之和 : " + getSum(integers));
      System.out.println("平均数 : " + getAverage(integers));
      System.out.println("随机数: ");
        
      // 输出10个随机数
      Random random = new Random();
        
      for(int i=0; i < 10; i++){
         System.out.println(random.nextInt());
      }
        
      System.out.println("使用 Java 8: ");
      System.out.println("列表: " +strings);
        
      count = strings.stream().filter(string->string.isEmpty()).count();
      System.out.println("空字符串数量为: " + count);
        
      count = strings.stream().filter(string -> string.length() == 3).count();
      System.out.println("字符串长度为 3 的数量为: " + count);
        
      filtered = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.toList());
      System.out.println("筛选后的列表: " + filtered);
        
      mergedString = strings.stream().filter(string ->!string.isEmpty()).collect(Collectors.joining(", "));
      System.out.println("合并字符串: " + mergedString);
        
      squaresList = numbers.stream().map( i ->i*i).distinct().collect(Collectors.toList());
      System.out.println("Squares List: " + squaresList);
      System.out.println("列表: " +integers);
        
      IntSummaryStatistics stats = integers.stream().mapToInt((x) ->x).summaryStatistics();
        
      System.out.println("列表中最大的数 : " + stats.getMax());
      System.out.println("列表中最小的数 : " + stats.getMin());
      System.out.println("所有数之和 : " + stats.getSum());
      System.out.println("平均数 : " + stats.getAverage());
      System.out.println("随机数: ");
        
      random.ints().limit(10).sorted().forEach(System.out::println);
        
      // 并行处理
      count = strings.parallelStream().filter(string -> string.isEmpty()).count();
      System.out.println("空字符串的数量为: " + count);
   }
    
   private static int getCountEmptyStringUsingJava7(List<String> strings){
      int count = 0;
        
      for(String string: strings){
        
         if(string.isEmpty()){
            count++;
         }
      }
      return count;
   }
    
   private static int getCountLength3UsingJava7(List<String> strings){
      int count = 0;
        
      for(String string: strings){
        
         if(string.length() == 3){
            count++;
         }
      }
      return count;
   }
    
   private static List<String> deleteEmptyStringsUsingJava7(List<String> strings){
      List<String> filteredList = new ArrayList<String>();
        
      for(String string: strings){
        
         if(!string.isEmpty()){
             filteredList.add(string);
         }
      }
      return filteredList;
   }
    
   private static String getMergedStringUsingJava7(List<String> strings, String separator){
      StringBuilder stringBuilder = new StringBuilder();
        
      for(String string: strings){
        
         if(!string.isEmpty()){
            stringBuilder.append(string);
            stringBuilder.append(separator);
         }
      }
      String mergedString = stringBuilder.toString();
      return mergedString.substring(0, mergedString.length()-2);
   }
    
   private static List<Integer> getSquares(List<Integer> numbers){
      List<Integer> squaresList = new ArrayList<Integer>();
        
      for(Integer number: numbers){
         Integer square = new Integer(number.intValue() * number.intValue());
            
         if(!squaresList.contains(square)){
            squaresList.add(square);
         }
      }
      return squaresList;
   }
    
   private static int getMax(List<Integer> numbers){
      int max = numbers.get(0);
        
      for(int i=1;i < numbers.size();i++){
        
         Integer number = numbers.get(i);
            
         if(number.intValue() > max){
            max = number.intValue();
         }
      }
      return max;
   }
    
   private static int getMin(List<Integer> numbers){
      int min = numbers.get(0);
        
      for(int i=1;i < numbers.size();i++){
         Integer number = numbers.get(i);
        
         if(number.intValue() < min){
            min = number.intValue();
         }
      }
      return min;
   }
    
   private static int getSum(List numbers){
      int sum = (int)(numbers.get(0));
        
      for(int i=1;i < numbers.size();i++){
         sum += (int)numbers.get(i);
      }
      return sum;
   }
    
   private static int getAverage(List<Integer> numbers){
      return getSum(numbers) / numbers.size();
   }
```

## Optional类
Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。  
Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。   
Optional 类的引入很好的解决空指针异常。  

demo:
```java
class Test {
  public static void main(String args[]) {

    Test java8Tester = new Test();
    Integer value1 = null;
    Integer value2 = new Integer(10);

    // Optional.ofNullable - 允许传递为 null 参数
    Optional<Integer> a = Optional.ofNullable(value1);

    // Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
    Optional<Integer> b = Optional.of(value2);
    System.out.println(java8Tester.sum(a, b));
  }

  public Integer sum(Optional<Integer> a, Optional<Integer> b) {

    // Optional.isPresent - 判断值是否存在

    System.out.println("第一个参数值存在: " + a.isPresent());
    System.out.println("第二个参数值存在: " + b.isPresent());

    // Optional.orElse - 如果值存在，返回它，否则返回默认值
    Integer value1 = a.orElse(new Integer(0));

    // Optional.get - 获取值，值需要存在
    Integer value2 = b.get();
    return value1 + value2;
  }
}
```

只能解决空指针null问题  对于字符串  是否是空字符串等的判断还是需要

## 日期时间 API 
Java 8通过发布新的Date-Time API (JSR 310)来进一步加强对日期与时间的处理。  
在旧版的 Java 中，日期时间 API 存在诸多问题，其中有：  
>**非线程安全** − java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。  
>**设计很差**− Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。  
>**时区处理麻烦** − 日期类并不提供国际化，没有时区支持，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。  

Java 8 在 java.time 包下提供了很多新的 API。以下为两个比较重要的 API：  
>**Local(本地)**− 简化了日期时间的处理，没有时区的问题。  
>**Zoned(时区)** − 通过制定的时区处理日期时间。  

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。  

local demo : 
```java
class Test {
  public static void main(String args[]) {
    Test java8tester = new Test();
    java8tester.testLocalDateTime();
  }

  public void testLocalDateTime() {

    // 获取当前的日期时间
    LocalDateTime currentTime = LocalDateTime.now();
    System.out.println("当前时间: " + currentTime);

    LocalDate date1 = currentTime.toLocalDate();
    System.out.println("date1: " + date1);

    Month month = currentTime.getMonth();
    int day = currentTime.getDayOfMonth();
    int seconds = currentTime.getSecond();

    System.out.println("月: " + month + ", 日: " + day + ", 秒: " + seconds);

    LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
    System.out.println("date2: " + date2);

    // 12 december 2014
    LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
    System.out.println("date3: " + date3);

    // 22 小时 15 分钟
    LocalTime date4 = LocalTime.of(22, 15);
    System.out.println("date4: " + date4);

    // 解析字符串
    LocalTime date5 = LocalTime.parse("20:15:30");
    System.out.println("date5: " + date5);
  }
}
```
zone demo:

```java
class Test {
  public static void main(String args[]){
    Test java8tester = new Test();
    java8tester.testZonedDateTime();
 }
  
 public void testZonedDateTime(){
  
    // 获取当前时间日期
    ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
    System.out.println("date1: " + date1);
      
    ZoneId id = ZoneId.of("Europe/Paris");
    System.out.println("ZoneId: " + id);
      
    ZoneId currentZone = ZoneId.systemDefault();
    System.out.println("当期时区: " + currentZone);
 }
}
```

月份 | MONTH
--- | ---
一月 | JANUARY
二月 | FEBRUARY
三月 | MARCH
四月 | APRIL
五月 | MAY
六月 | JUNE 
七月 | JULY
八月 | AUGUEST
九月 | SEPTEMPER
十月 | OCTOBER
十一月 |  NOVEMBER
十二月 | DECEMBER

---
dg-publish: true
---

　　IOC-Inversion of Control，即控制反转。它不是什么技术，而是一种设计思想。

　　    传统的创建对象的方法是直接通过 new 关键字，而 spring 则是通过 IOC 容器来创建对象，也就是说我们将创建对象的控制权交给了 IOC 容器。我们可以用一句话来概括 IOC：

**IOC 让程序员不在关注怎么去创建对象，而是关注与对象创建之后的操作，把对象的创建、初始化、销毁等工作交给spring容器来做**

```java

public class HelloWorldPo {

  private String name;

  public String getName() {

    return name;

  }

  public void setName(String name) {

    this.name = name;

  }

    //静态工厂方法

    public static HelloWorldPo getInstances(){

        return new HelloWorldPo();

    }

  @Override

  public String toString() {

    return "HelloWorldPo [name=" + name + "]";

  }

}

```

传统的创建对象的方式

```java

  @Test

  public void testHello(){

    //通过new创建实例对象

    HelloWorldPo helloWorldPo=new HelloWorldPo();

    helloWorldPo.setName("张三");

    System.out.println(helloWorldPo.toString());

  }

  //通过反射创建实例对象

  @Test

  public void testInvoke() throws Exception{

    Class class1 = Class.forName("com.oaec.po.HelloWorldPo");

    //获取属性对象

    Field field = class1.getDeclaredField("name");

    //获取方法对象

    Method method = class1.getMethod("setName", String.class);

    //通过反射创建实例对象

    HelloWorldPo helloWorldPo=(HelloWorldPo) class1.newInstance();

    //通过反射调用方法

    method.invoke(helloWorldPo, "王五");

    System.out.println(helloWorldPo.toString());

  }

```

**使用spring容器创建对象**

  

1.使用默认的构造函数

首先，在项目的src根路径创建applicationContext.xml

```xml

<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

    xsi:schemaLocation="http://www.springframework.org/schema/beans

        http://www.springframework.org/schema/beans/spring-beans.xsd">

      <!-- id:bean的名称，在IOC容器中必须唯一，推荐指定名称，不要省略，

    如果省略的话，会使用类名作为id

    class:全类名，在IOC容器中通过反射创建实例，所以要求必须有class属性，

        在继承的继承的情况下可以省略，

            property:属性注入

     -->

    <!-- 默认情况下，启动 spring 容器便创建对象（遇到bean便创建对象）

    　   在spring的配置文件bean中有一个属性 lazy-init="default/true/false"

 　　　　如果lazy-init为"default/false"在启动spring容器时创建对象（默认情况）

　　　　 如果lazy-init为"true",在context.getBean时才要创建对象

 　　 在第一种情况下可以在启动spring容器的时候，检查spring容器配置文件的正确性，

    如果再结合tomcat,如果spring容器不能正常启动，整个tomcat就不能正常启动。

    但是这样的缺点是把一些bean过早的放在了内存中，如果有数据，则对内存来是一个消耗。

　　    反过来，在第二种情况下，可以减少内存的消耗，但是不容易发现错误-->

    <!-- Spring框架支持以下五种bean的作用域：

    singleton : bean在每个Spring ioc 容器中只有一个实例。

    prototype：一个bean的定义可以有多个实例。

    request：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。

    session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

    global-session：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

    缺省的Spring bean 的作用域是Singleton。 -->

  <!-- 默认构造函数 -->

  <bean id="helloWorldByDefaultConstructor" class="com.po.HelloWorldPo">

    <property name="name" value="构造"></property>

  </bean>

  <!-- 静态工厂 -->

  <bean id="helloWorldByStaticFactory" factory-method="getInstances" class="com.po.HelloWorldPo">

  <property name="name" value="静态"></property>

  </bean>

  <!-- 实例工厂 -->

  <bean id="instanceFactory" class="com.po.HelloWorldInstanceFactory"></bean>

  <bean id="helloWorldByInstanceFactory" factory-bean="instanceFactory" class="com.po.HelloWorldPo">

  <property name="name" value="实例"></property>

  </bean>

</beans>

```

测试方法

```java

  //构造方法

  @Test

  public void testConstructor(){

    //spring是个容器，负责管理对象的生命周期

//    需要实例化IOC容器，容器产生哪些实例，是根据配置文件来产生的

//    容器实例化之后，才可以从IOC容器获取Bean的实例并使用

//    如何实例化IOC容器

//    1.BeanFacotry .IOC容器的基本实现，面向的是Spring本身

//    2.ApplicationContxt.是BeanFactory的子接口，ApplicationContext是面向

//    spring框架的使用者，所有的应用基本上都是满足

//    ApplicationContext实现类：

//      1.ClassPathXmlApplicationContext 从类路径下加载配置文件（src）

//        2.FileSystemXmlApplicationContext 从系统文件中加载配置文件（F://cxx/xxx/xxx.xml）

//    ConfigurableApplicationContext 扩展ApllicationContext.

//    功能：启动，刷新，关闭上下文的功能

//      ApplicationContext 特点：在初始化上下文时就已经实例化好所有的的实例（单例）

//             针对web项目，上下文使用 WebApplicationContext  

    AbstractApplicationContext applicationContext

    = new ClassPathXmlApplicationContext("applicationContext.xml");

    //在容器中获取 class类型为HelloWorldPo 的bean

    //容器中只有一个class类型相同的bean 才可以使用 getBean(Class);

    HelloWorldPo helloWorldPo=(HelloWorldPo) applicationContext.getBean("helloWorldByDefaultConstructor");

    System.out.println(helloWorldPo.toString());

  }

```

2.静态工厂方法    

```java

  //静态工厂

  @Test

  public void testStaticFactory(){

    //spring是个容器，负责管理对象的生命周期

    AbstractApplicationContext applicationContext

    = new ClassPathXmlApplicationContext("applicationContext.xml");

    HelloWorldPo helloWorldPo=(HelloWorldPo) applicationContext.getBean("helloWorldByStaticFactory");

    System.out.println(helloWorldPo.toString());

  }

```

3.实例工厂

创建实例工厂类

```java

public class HelloWorldInstanceFactory {

  public HelloWorldPo getInstance() {

    HelloWorldPo helloWorldPo = new HelloWorldPo();

    return helloWorldPo;

  }

}

```

```java

  //实例工厂

  @Test

  public void testInstanceFactory(){

    //spring是个容器，负责管理对象的生命周期

    AbstractApplicationContext applicationContext

    = new ClassPathXmlApplicationContext("applicationContext.xml");

    HelloWorldPo helloWorldPo=(HelloWorldPo) applicationContext.getBean("helloWorldByInstanceFactory");

    System.out.println(helloWorldPo.toString());

  }

```

bean的继承

```xml

<!-- 子 Bean 从父 Bean 中继承配置, 包括 Bean 的属性配置

子 Bean 也可以覆盖从父 Bean 继承过来的配置

父 Bean 可以作为配置模板, 也可以作为 Bean 实例.

若只想把父 Bean 作为模板, 可以设置 <bean> 的abstract 属性为 true, 这样 Spring 将不会实例化这个 Bean

并不是 <bean> 元素里的所有属性都会被继承. 比如: autowire, abstract 等.

也可以忽略父 Bean 的 class 属性, 让子 Bean 指定自己的类,

而共享相同的属性配置. 但此时 abstract 必须设为 true

 -->

<!-- 父bean -->

 <bean id="father"

 p:name="父亲" p:car-ref="car"

 abstract="true"

  ></bean>

  <bean id="son1" class="com.oaec.po.Person"

  parent="father"

  p:name="大儿子"

  scope="prototype"

  ></bean>

```

加载配置文件

创建jdbc.properties

```properties

jdbc.driverClass=com.mysql.jdbc.Driver

jdbc.url=jdbc:mysql://localhost:3306/mybatis2?characterEncoding=utf-8

jdbc.user=root

jdbc.password=123456

```

```xml

  <!-- 加载配置文件 -->

  <context:property-placeholder location="classpath:jdbc.properties"/>

  <bean id="jdbcTool" class="com.oaec.po.JDBCtool"

  p:driver="${jdbc.driverClass}"

   p:url="${jdbc.url}"

   p:username="${jdbc.user}"

   p:password="${jdbc.password}"

  ></bean>

```

前置后置的配置

```java

/**

 * 1.构造方法、工厂方法初始化bean

 * 2.属性赋值

 * 3.后置处理器的before方法

 * 4.bean的初始化方法

 * 5.后置处理器的after方法

 * 6.bean可以使用

 * 7.容器关闭，销毁bean

 *

 */

public class MyProcessor implements BeanPostProcessor{

  //在初始化之前调用    

  /**

   * obj:当前执行生命周期的对象

   * id：当前执行声明周期的对象的id

   */

  @Override

  public Object postProcessBeforeInitialization(Object obj, String id)

      throws BeansException {

    System.out.println("后置处理器before.obj="+obj+",id="+id);

    return obj;

  }

  //在初始化方法之后调用

    @Override

    public Object postProcessAfterInitialization(Object obj, String id)

        throws BeansException {

      System.out.println("后置处理器after.obj="+obj+",id="+id);

      return obj;

    }

}

```

```xml

  <!-- 初始化后置处理器 -->

  <bean class="com.oaec.po.MyProcessor"></bean>

  <!-- 指定此bean的初始化方法和销毁方法

  1、构造方法或者工厂创建car实例

  2、对属性赋值或者关联其他对象

  3、调用初始化方法

  4、此bean可以使用

  5、当容器关闭时，调用bean的销毁方法

   -->

  <bean id="car" class="com.oaec.po.Car"

  p:name="dazhong" p:price="300" p:color="red"

  ></bean>

```

```java

  @Test

  public void testLife() {

      ConfigurableApplicationContext ac=

          new ClassPathXmlApplicationContext("bean-life.xml");

      Car car=(Car) ac.getBean("car");

      System.out.println(car.toString());

      ac.close();

  }

/*  运行结果

  car 构造方法

  car ...setName...

  后置处理器before.obj=Car [name=dazhong, color=red, price=300],id=car

  后置处理器after.obj=Car [name=dazhong, color=red, price=300],id=car

  Car [name=dazhong, color=red, price=300]*/

```

配置注解的方式实现IOC

```xml

  <context:component-scan base-package="com.oaec.scan" use-default-filters="true">

    <context:include-filter type="assignable" expression="com.dao.UserDao"/>

      <context:exclude-filter type="annotation"

       expression="org.springframework.stereotype.Controller"/>

  </context:component-scan>

<!--  base-package:设置要扫描的包名

    扫描包下所有的java文件，

    use-default-filters 为true的话会扫描@Component的所有子类注解

    自定义注解扫描细节的话 设置成false

  context:include-filter:要包含哪些注解/类

  context:exclude-filter:不包含哪些注解/类

  type：扫描的类型

      1、annotation 是否标注了特定的注解

      2、assignable 目标类是否继承或者实现了某个特定的类 -->

```

在配置文件中导入其他配置文件

```xml

<!-- 导入其他配置文件 -->

  <import resource="classpath:bean-ref.xml"/>

```
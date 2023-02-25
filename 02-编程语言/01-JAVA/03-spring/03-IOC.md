IOC 让程序员不在关注怎么去创建对象，而是关注与对象创建之后的操作，把对象的创建、初始化、销毁等工作交给spring容器来做

## Spring 容器创建对象的三种方式

### 1.利用默认的构造方法

```xml

    <!--

    创建对象的第一种方式：利用无参构造器

    id:唯一标识符

    class:类的全类名

      -->

<bean id="helloIoc" class="com.ys.ioc.HelloIoc" ></bean>  

```

### 2.利用静态工厂方法

```java

public class HelloStaticFactory {

    public HelloStaticFactory(){

        System.out.println("HelloStaticFactory constructor");

    }

    //静态工厂方法

    public static HelloIoc getInstances(){

        return new HelloIoc();

    }

}

```

```xml

<bean id="helloStaticFactory" factory-method="getInstances" class="com.ys.ioc.HelloStaticFactory"></bean>

```

**注意：spring容器只负责调用静态工厂方法，而这个静态工厂方法内部实现由程序员完成**

### 3.利用实例工厂方法

```java

public class HelloInstanceFactory {

    public HelloInstanceFactory(){

        System.out.println("实例工厂方法构造函数");

    }

    //利用实例工厂方法创建对象

    public HelloIoc getInstance(){

        HelloIoc instanceIoc = new HelloIoc();

        return instanceIoc;

    }

}

```

```xml

    <bean id="instanceFactory" class="com.ys.ioc.HelloInstanceFactory"></bean>

    <bean id="instance" factory-bean="instanceFactory" factory-method="getInstance"></bean>

```

## Spring 容器创建对象的时机

默认初始化容器时创建

bean 的lazy-init属性为true,使用时创建，context.getBean()时

## spring的bean中的scope："singleton/prototype/request/session/global session"

Spring框架支持以下五种bean的作用域：

**singleton** : bean在每个Spring ioc 容器中只有一个实例。

**prototype**：一个bean的定义可以有多个实例。

**request**：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。

**session**：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

**global-session**：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

缺省的Spring bean 的作用域是Singleton。

## Spring 容器生命周期

```xml

<bean id="springLifeCycle" init-method="init" destroy-method="destroy" class="com.ys.ioc.SpringLifeCycle"></bean>

```

指定初始化，销毁方法

　分析：spring 容器的声明周期

　　 　   1、spring容器创建对象

        　　2、执行init方法

        　　3、调用自己的方法

        　　4、当spring容器关闭的时候执行destroy方法

　　注意：当scope为"prototype"时，调用 close（） 方法时是不会调用 destroy 方法的
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

## IOC
https://www.bilibili.com/video/BV1N8411J7zK?p=42&vd_source=4ea256e9431ae8b0e6d59aeaa6c35daf

控制反转：理论思想，原来的对象是由使用者来进行控制，有了spring之后，可以把整个对象交给spring来帮我们进行管理 
Dl：依赖注入，把对应的属性的值注入到具体的对象中，@Autowired, populateBean完成属性值的注入 
容器：存储对象，使用map结构来存储，在spring中一般存在三级缓存，singletonObjects存放完整的bean对象， 整个bean的生命周期，从创建到使用到销毁的过程全部都是由容器来管理（bean的生命周期） 
分： 
1、一般聊ioc容器的时候要涉及到容器的创建过程（beanFactory, DefaultListableBean Factory)，向bean工厂中设置一些参数 (BeanPostProcessor,Aware接口的子类）等等属性 
2、加载解析bean对象，准备要创建的bean对象的定义对象beanDefinition, ( x m I或者注解的解析过程） 
3、beanFactoryPostProcessor的处理，此处是扩展点，PlaceHolderConfigurSu pport,ConfigurationClassPostProcessor 
4、BeanPostProcessor’的注册功能，方便后续对bean对象完成具体的扩展功能 
5、通过反射的方式讲BeanDefinition：对象实例化成具体的bean对象， 
6、 bean对象的初始化过程（项充属性，调用aware子类的方法，调用Bean Post Processor前置处理方法，调用init-method方法，调用 BeanPostProcessor的后置处理方法） 
7.生成完整的bean对象
8.销毁过程

面试官，这是我对IOC的整体理解，包含了一些详细的处理过程，您看一下有什么问题，可以指点我一下（允许你把整个流程说完） 
老师，我没看过源码怎么办？ 
具体的细节我记不太清了，但是spring中的bean都是通过反射的方式生成的，同时其中包含了很多的扩展点，比如最常用的对 Bean Factory的扩展，对bean的扩展（对占位符的处理），我们在公司对这方面的使用是比较多的，除此之外IOC中最核心的也就是填 
充具体bean的属性，和生命周期（背一下）。 


引申问题点：
## bean的生命周期
[[01-bean的生命周期#生命周期一览图]]
1. 实例化bean,反射的方式生成对象，分配内存空间
2. 填充bean的属性：populateBean,循环依赖的问题（三级缓存)
3. 调用aware接口相关的方法：invokeAwareMethod（完成BeanName,BeanFactory,BeanClassLoader对象的属性设置）
4. 调用BeanPostProcessor中的前置处理方法：使用比较多的有（ApplicationContextPostProcessor，设置 ApplicationContext,Environment,ResourceLoader, EmbeddValueResolver等对象）
5. 调用initmethod方法：invokeInitmethod()，判断是否实现了initializingBean接口，如果有，调用afterPropertiesSet方法，没有就不调用
6. 调用Bean PostProcessor的后置处理方法：spring的aop就是在此处实现的，AbstractAutoProxyCreator注册Destuction相关的回调接口：钩子函数
7. 获取到完整的对象，可以通过getBean的方式来进行对象的获取
8. 销毁流程， 判断是否实现了DispoableBean接口，2，调用destroyMethod方法

## @Autowired, populateBean  具体过程

容器：存储对象，使用map结构来存储
## 循环依赖
![[循环依赖-AB.png]]

关键点：三级缓存，提前暴露对象，aop
总：
什么是循环依赖问题，A依赖B,B依赖A 先说明bean的创建过程：实例化，初始化（填充属性）
分 ：
1. 先创建A对象，实例化A对象，此时A对象中的b属性为空，填充属性b
2. 从容器中查找B对象，如果找到了，直接赋值不存在循环依赖问题（不通），找不到直接创建B对象
3. 实例化B对象，此时B寸象中的a属性为空，填充属性a
4. 从容器中查找A对象，找不到，直接创建

此时，如果仔细琢磨的话，会发现A对象是存在的，只不过此时的A对象不是一个完整的状态，只完成了实例化但是未完成初始化， 如果在程序调用过程中，拥有了某个对象的引用，能否在后期给他完成赋值操作，可以优先把非完整状态的对象优先赋值，等待后续操作 来完成赋值，相当于提前暴露了某个不完整对象的引用，所以解决问题**的核心在于实例化和初始化分开操作，这也是解决循环依赖问题的 关键**。

当所有的对象都完成实例化和初始化操作之后，还要把完整对象放到容器中，此时在容器中存在两种状态的对象，完成实例化但未完成初始化和完整状态
因为都在容器中，所以要使用不同的map结构来进行存储，此时就有了一级缓存和二级缓存，如果一级缓存有了，那么二级缓存中就不会存在同名的对象，因为他们的查找顺序是1,2, 3这样的方式来查找的。**一级缓存中放的是完整对象，二级缓 存中放的是非完整对象**

为什么需要三级缓存？**三级缓存**的value类型是ObjectFactory，是一个函数式接口，存在的意义是**保证在整个容器的运行过程中同名的bean对象只能有一个**。

如果一个对象需要被代理，或者说需要生成代理对象，那么要不要优先生成一个普通对象？要
普通对象和代理对象是不能同时出现在容器中的，因此当一个对象需要被代理的时候，就要使用代理对象覆盖掉之前的普通对象，在 实际的调用过程中，是没有办法确定什么时候对象被使用，所以就要求当某个对象被调用的时候，优先判断此对象是否需要被代理，类似 于一种回调机制的实现，因此传入Iambda表达式的时候，可以通过Iambda表达式来执行对象的慈盖过程，getEarlyBeanReference() 因此，**所有的bean对象在创建的时候都要优先放到三级缓存中，在后续的使用过程中，如果需要被代理则返回代理对象，如果不需 要被代理，则直接返回普通对象**


---
dg-publish: false
---
所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

现在有一张表 User,然后我们要在程序中实现对 User 表的增加和删除操作。

　　要求：增加和删除操作都必须要记录日志。

## 使用jdk动态代理实现
### 创建 UserService 接口
```java
public interface UserService {
    //添加 user
    public void addUser(User user);
    //删除 user
    public void deleteUser(int uid);
}
```
### 创建 UserService的实现类
```java
public class UserServiceImpl implements UserService{
    @Override
    public void addUser(User user) {
        System.out.println("增加 User");
    }
    @Override
    public void deleteUser(int uid) {
        System.out.println("删除 User");
    }
}
```
### 创日志类 MyLog
```java
public class MyLog {
    //开启事务
    public void before(){
        System.out.println("开启");
    }
    //提交事务
    public void after(){
        System.out.println("结束");
    }
}
```
### 动态代理类ObjectInterceptor.java 
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
 
import com.ys.aop.one.MyTransaction;
 
public class ObjectInterceptor implements InvocationHandler{
    //目标类
    private Object target;
    //切面类（这里指日志类）
    private MyLog myLog;
     
    //通过构造器赋值
    public ObjectInterceptor(Object target,MyLog myLog){
        this.target = target;
        this.transaction = transaction;
    }
     
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        //开启
        this.myLog.before();
        //调用目标类方法
        method.invoke(this.target, args);
        //结束
        this.myLog.after();
        return null;
    }
     
}
```

## aop术语
　　1.target：目标类，需要被代理的类。例如：UserService

　　2.Joinpoint(连接点):所谓连接点是指那些可能被拦截到的方法。例如：所有的方法

　　3.PointCut 切入点：已经被增强的连接点。例如：addUser()

　　4.advice 通知/增强，增强代码。例如：after、before

　　5. Weaving(织入):是指把增强advice应用到目标对象target来创建新的代理对象proxy的过程.

　　6.proxy 代理类：通知+切入点

　　7. Aspect(切面): 是切入点pointcut和通知advice的结合

## AOP 的通知类型　
　Spring按照通知Advice在目标类方法的连接点位置，可以分为5类

前置通知 org.springframework.aop.MethodBeforeAdvice
在目标方法执行前实施增强，比如上面例子的 before()方法
后置通知 org.springframework.aop.AfterReturningAdvice
在目标方法执行后实施增强，比如上面例子的 after()方法
环绕通知 org.aopalliance.intercept.MethodInterceptor
在目标方法执行前后实施增强
异常抛出通知 org.springframework.aop.ThrowsAdvice
在方法抛出异常后实施增强
引介通知 org.springframework.aop.IntroductionInterceptor
　　　　　　在目标类中添加一些新的方法和属性

## 使用 Spring AOP 解决上面的需求
```xml
    <!--1、 创建目标类 -->
    <bean id="userService" class="com.ys.aop.UserServiceImpl"></bean>  
    <!--2、创建切面类（通知）  -->
    <bean id="transaction" class="com.ys.aop.one.MyTransaction"></bean>
     
    <!--3、aop编程 
        3.1 导入命名空间
        3.2 使用 <aop:config>进行配置
                proxy-target-class="true" 声明时使用cglib代理
                如果不声明，Spring 会自动选择cglib代理还是JDK动态代理
            <aop:pointcut> 切入点 ，从目标对象获得具体方法
            <aop:advisor> 特殊的切面，只有一个通知 和 一个切入点
                advice-ref 通知引用
                pointcut-ref 切入点引用
        3.3 切入点表达式
            execution(* com.ys.aop.*.*(..))
            选择方法         返回值任意   包             类名任意   方法名任意   参数任意
     
    -->
    <aop:config>
        <!-- 切入点表达式 -->
        <aop:pointcut expression="execution(* com.ys.aop.*.*(..))" id="myPointCut"/>
        <aop:aspect ref="transaction">
            <!-- 配置前置通知，注意 method 的值要和 对应切面的类方法名称相同 -->
            <aop:before method="before" pointcut-ref="myPointCut"></aop:before>
            <aop:after-returning method="after" pointcut-ref="myPointCut"/>
        </aop:aspect>
    </aop:config>
```

### 切入点表达式
```xml
execution（modifiers-pattern? ref-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?）
　　　　　　　　　　   类修饰符           返回值           方法所在的包                  方法名                     方法抛出的异常
```
### springAOP 的具体加载步骤
　1、当 spring 容器启动的时候，加载了 spring 的配置文件

　　2、为配置文件中的所有 bean 创建对象

　　3、spring 容器会解析 aop:config 的配置

   　　　　1、解析切入点表达式，用切入点表达式和纳入 spring 容器中的 bean 做匹配

       　　　　如果匹配成功，则会为该 bean 创建代理对象，代理对象的方法=目标方法+通知

       　　　　如果匹配不成功，不会创建代理对象

　　4、在客户端利用 context.getBean() 获取对象时，如果该对象有代理对象，则返回代理对象；如果没有，则返回目标对象

　　　　说明：如果目标类没有实现接口，则 spring 容器会采用 cglib 的方式产生代理对象，如果实现了接口，则会采用 jdk 的方式

### 注解的方式

```xml
	<context:component-scan base-package="com"></context:component-scan>
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

```java
@Component
@Aspect
public class LogAspect {
	
//	
//	合并切入点表达式 使用pointCut来声明切点 @Pointcut（切入点表达式）
//	必须声明在一个没有方法体的方法上面
//	使用  @Before(此方法名())
 	@Pointcut("execution (public int com.oaec.po.Calc.*(int,int))")
	private void logPointCut(){}
//	通知如何访问当前连接点的信息？
// 	JoinPoint 连接点。可以通过JoinPoint来访问当前连接细节，例如方法名。方法参数
	@Before("logPointCut()")
	public void beforeLog(JoinPoint joinPoint){
		String methodName=joinPoint.getSignature().getName();
		Object[] args = joinPoint.getArgs();
//		d当前被代理的类
		String className=joinPoint.getTarget().getClass().getName();
		System.out.println("前置日志:"+className
				+","+methodName+","+Arrays.toString(args));
		
	}
	//后置通知，连接点完成之后执行，连接点返回结果或者是抛出异常，都会执行后置通知
	
	@After("logPointCut()" )
	public void afterLog(JoinPoint joinPoint){
		String methodName=joinPoint.getSignature().getName();
		String className=joinPoint.getTarget().getClass().getName();
		System.out.println("后置通知:"+methodName+","+className);
	}
//	返回通知
//	如果想要在连接点返回结果的时候记录日志，需要使用返回通知
//	如果想要获取返回的结果，在返回通知中，将returning属性添加到@AfterReturning注解中
//	必须在方法签名中定义同名参数，aop在运行就会将结果传递给result这个参数
	@AfterReturning(value="logPointCut()",returning="result")
	public void logAfterReturnint(JoinPoint joinPoint,Object result){
		String methodName=joinPoint.getSignature().getName();
		String className=joinPoint.getTarget().getClass().getName();
		System.out.println("返回通知:"+methodName+
				","+className+",result="+result);
	}
//	异常通知
//	如果想要获取当前是哪个异常，将属性throwing 加入注解中，同时，方法参数中加入同名参数
//	同名参数类型必须为 分类或者是其超类
	@AfterThrowing(value="logPointCut()",throwing="e")
	public void logAfterThrowing(JoinPoint joinPoint,Exception e){
		String methodName=joinPoint.getSignature().getName();
		String className=joinPoint.getTarget().getClass().getName();
		System.out.println("异常通知:"+methodName+
				","+className+",e="+e);
		
	}
	//如果只捕获某个特定的异常，将参数定位这个特定的异常
}
```

```java
//验证切面
@Component
@Aspect
public class VerificationAspect {
			
//	环绕通知，功能最强大，
//	可以完成 前面四个通知
	//环绕通知必须有返回值，返回值就是目标方法的返回值
//	如果想在环绕通知中获取连接点的信息 ProceedingJoinPoint 作为参数
//	ProceedingJoinPoint .proceed() 可以执行目标方法
	@Around("execution(* com.oaec.*..*(..))")
	public Object vAroundMethod(ProceedingJoinPoint joinPoint) throws Throwable{
		Object result=null;
		try {
			//前置通知
			beforeV();
			//执行目标方法
			result=joinPoint.proceed();
			//返回通知
			afterReturningV();
		} catch (Throwable e) {
			//异常通知
			afterThrowingV();
			e.printStackTrace();
			throw e;
			
			
		}
		//后置通知
		afterV();
		return result;
	}
	
	public void beforeV(){
		System.out.println("环绕通知中，，，，，前置通知");
	}
	public void afterReturningV(){
		System.out.println("环绕通知中，，，返回通知");
	}
	public void afterThrowingV(){
		System.out.println("环绕通知中....异常通知");
	}
	public  void  afterV(){
		
		System.out.println("环绕通知中....后置通知");
	}
}
```

## 切面的基本使用
```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

  @Before("execution(* com.example.service.*.*(..))")
  public void logBefore(JoinPoint joinPoint) {
  // 这段代码定义了一个切面，其中定义了一个 `@Before` 注解的方法，在 `com.example.service` 包中所有方法执行前都会被调用。
    System.out.println("Entering method: " + joinPoint.getSignature().getName());
  }
}
```
1.  `@Before`：在方法执行前运行
2.  `@After`：在方法执行后运行
3.  `@Around`：在方法执行前和执行后都运行
4.  `@AfterReturning`：在方法正常执行后运行
5.  `@AfterThrowing`：在方法抛出异常后运行
6.  `@Pointcut`：定义切入点表达式，可以多次引用
通过使用这些注解，你可以非常方便地在切面中实现各种功能，例如**日志记录、安全性校验、性能监控**等。


这段代码定义了一个切面，它的作用是在执行 `com.example.service` 包中的所有方法时统计并输出方法的执行时间。通过使用 `@Around` 注解，切面方法会在目标方法执行前和执行后都被调用，并可以通过调用 `joinPoint.proceed()` 方法来执行目标方法。
```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class PerformanceAspect {

  @Around("execution(* com.example.service.*.*(..))")
  public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long elapsedTime = System.currentTimeMillis() - start;
    System.out.println("Method execution time: " + elapsedTime + " ms");
    return result;
  }
}

```

## aop的实现原理

Spring AOP 的实现原理是**动态代理**。

在 AOP 中，通常将切面称为“**额外的功能**”，而将要在其上添加切面的类或方法称为“目标对象”。在 Spring AOP 中，目标对象通过代理对象来实现，即在**目标对象外面包装了一层代理对象，在代理对象中实现了切面功能**。

当你在代码中调用目标对象的方法时，**实际上是调用的代理对象的方法**，代理对象会在实际调用目标对象的方法前后添加切面功能，从而实现了 AOP 的效果。

Spring AOP 支持两种代理模型：**JDK 动态代理和 CGLIB 代理**。在 JDK 动态代理中，代理对象需要**实现与目标对象相同的接口**，在 CGLIB 代理中，代理对象是**通过扩展目标对象类**来实现的，通常使用 CGLIB 代理的场景是当目标对象没有接口时。

**AOP的实现依赖的是spring拓展点中的BeanPostProcessor的后置处理方法**
1. 代理对象的创建过程（advice，切面，切点）
2. 通过jdk或者cglib的方式来生成代理对象
3. 在执行方法调用的时候，会调用到生成的字节码文件中，直接回找到。dynamicAdvisorInterceptor类中的intercept方法，从此 方法开始执行
4. 根据之前定义好的通知来生成拦截器链
5. 从拦截器链中依次获取每一个通知开始进行执行，在执行过程中，为了方便找到下一个通知是哪个，会有一个 CglibMethod Invocation的对象，找的时候是从一1的位置一次开始查找并且执行的。
## cglib为什么不能代理private,final
https://heapdump.cn/article/3333875

**cglib是通过动态生成代理类的子类实现代理功能****，所生成的子类重写了代理类的所有方法（不包括final方法，至于private方法后文会讲到），cglib生成的子类中再通过方法拦截的方式实现对代理类方法的代码织入。**

**说到这，重点来了，cglib实质上是通过继承父类并重写父类的方法达到生成代理类的，那么自然的，final类和final方法一定无法通过cglib代理，在生成的class文件中也不会找到对应的final方法。**
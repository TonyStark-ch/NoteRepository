## 11. AOP
### 11.1 什么是AOP
AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编方式和运行期动态代理实现程序 功能的统一维护的一种技术。AOP是OOP的延是软件开发中的一个热点，也是Spring框架中的一个重要内容，
是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦 合度降低，提高程序的可重用性，同时提高了开发的效率。
> 个人理解的是减少重复代码。如果执行一个方法的前后需要打印日志，那么用AOP会大大减少重复代码（个人感觉AOP
> 和动态代理是一样的，只不过spring封装之后简化了配置），AOP的底层就是用动态代理来实现的。
###11.2 11.2 AOP在Spring中的作用
>提供声明式事务；允许用户自定义切面
- AOP中的名词理解：
    - 连接点（Join point）：指方法，在Spring AOP中，一个连接点总是代表一个方法的执行。（所有方法都可以叫连接点）
    - 切入点：切入点是指我们要对哪些Join point进行拦截的定义。（需要增加方法的连接点叫切入点），即执行增加功能的位置。
    - 通知（Advice）：在切面的某个特定的连接点（Join point）上执行的动作。即增加的功能。
    - 目标对象（Target Object）： 被通知对象。即增加功能的对象。
    - 代理（Proxy）：向目标对象应用通知之后创建的对象。
    - 切面（Aspect）：简单来说就是新增加功能的模块化。被抽取的公共模块，可能会横切多个对象。
    - 引入（Introduction）：（也被称为内部类型声明（inter-type
      declaration））。声明额外的方法或者某个类型的字段。Spring允许引入新的接口（以及一个对应的实现）到任何被代理的对象。例如，你可以使用一个引入来使bean实现IsModified接口，以便简化缓存机制。
    - 织入（Weaving）：指把增强应用到目标对象来创建新的代理对象的过程。Spring是在运行时完成织入。
- spring中支持5中advice
    - 前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

    - 返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。

    - 抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。

    - 后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

    - 环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。
      环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。
      环绕通知是最常用的一种通知类型。大部分基于拦截的AOP框架，例如Nanning和JBoss4，都只提供环绕通知。
###11.3 使用Spring实现AOP
- spring的具体实现：
    - 导入AOP织入包
  ```xml
   <dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
   </dependency>
  ```
    - 使用Spring的API接口【主要是SpringAPI接口实现】(切点)
        - 在service包下，定义UserService业务接口和UserServiceImpl实现类
  ```java
      public interface UserService {
        public void add();
        public void delete();
        public void update();
        public void select();
      }
  ```

  ```java
  public class UserServiceImpl implements UserService{
    @Override
    public void add() {
        System.out.println("增加了一个用户");
    }
    @Override
    public void delete() {
        System.out.println("删除了一个用户");
    }
    @Override
    public void update() {
        System.out.println("修改了一个用户");
    }
    @Override
    public void select() {
        System.out.println("查询了一个用户");
    }
  }
  ```
    - 在log包下，定义我们的增强类，一个Log前置增强和一个AfterLog后置增强类
  ```java
  public class beforeLog implements MethodBeforeAdvice {
    /*target方法前执行*/
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("目标函数："+o.getClass()+"方法"+method.getName());
    }
  }
  ```
  ```java
  public class afterLog implements AfterReturningAdvice {
    /*target方法后执行*/
    @Override
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("目标函数："+o1.getClass()+"方法"+method.getName()+o);
    }
  }
  ```
    - 最后去spring的文件中注册 , 并实现aop切入实现 , 注意导入约束，配置applicationContext.xml文件
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  https://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/aop
  https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--注册bean-->
    <bean id="userService" class="org.spring.code.service.UserServiceImpl"/>
    <bean id="afterLog" class="org.spring.code.log.afterLog"/>
    <bean id="beforeLog" class="org.spring.code.log.beforeLog"/>

    <!--方式一：使用原生Spring API接口-->
    <!--配置aop:需要导入aop的约束-->
    <aop:config>
        <!--切入点：expression：表达式，execution(要执行的位置！* * * * *)-->
        <aop:pointcut id="pointcut" expression="execution(* org.spring.code.service.UserServiceImpl.*(..))"/>

        <!--执行环绕增加！-->
        <aop:advisor advice-ref="beforeLog" pointcut-ref="pointcut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
    </aop:config>

  </beans>
  ```
  > execution(* *.*.*. (*))（分别指代返回值类型 包名.类名.方法名 参数）<br/>
  参数中(..)所有参数，*指所有
    - 自定义类来实现AOP【主要是切面定义】(用切面方式来实现)
        - 在diy包下定义自己的DiyPointCut切入类
  ```java
  public class DiyPointCut {
    public void before(){
      System.out.println("======方法执行前======");
    }

    public void after(){
      System.out.println("======方法执行后======");
    }
  }
  ```
  ```xml
      <!--自定义切面-->
    <bean id="diy" class="org.spring.code.diy.DiyPointCut"/>
    <aop:config>
        <aop:aspect ref="diy">
            <!--定义好切点-->
            <aop:pointcut id="pointCut" expression="execution(* org.spring.code.service.UserServiceImpl.* (..))"/>
            <!--执行环绕增加！-->
            <aop:before method="before" pointcut-ref="pointcut"/>
            <aop:after method="after" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>
  ```
>推荐使用第一种，定制化程度高

  - 注释法
    - 在diy包下定义注解实现的DiynPointCut增强类
  ```java
public class DiyPointCut {
    @Before("execution(* org.spring.code.service.UserServiceImpl.* (..))")
    public void before() {
        System.out.println("======方法执行前======");
    }
    @After("execution(* org.spring.code.service.UserServiceImpl.* (..))")
    public void after() {
        System.out.println("======方法执行后======");
    }
}
  ```

  ```xml
      <!--方式三：使用注解-->
  <bean id="annotationPointCut" class="org.spring.code.diy.DiyPointCut"/>
        <!--开启注解支持！ JDK(默认是 proxy-target-class="false")  cglib（proxy-target-class="true"）-->
  <aop:aspectj-autoproxy/>
  ```

###11.4 注释的对应
类上的`@Aspect`---->`<aop:aspect ref="diy">`---定义切面
方法上的`@Before("execution(* org.spring.code.service.UserServiceImpl.* (..))")`---->`<aop:before method="before" pointcut-ref="pointcut"/>`---执行环绕增加

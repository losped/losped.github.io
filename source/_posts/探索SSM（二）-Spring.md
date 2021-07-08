---
title: 探索SSM（二）-Spring
date: 2019.02.13 16:32:05
tags: JW
categories: JW
---


# Spring
##### 注意~~
①必须在所有使用了dao的地方,包括调用它的servcie都要进行@Autowired注入,否则之后的注入就会失败..
②注解静态实例对象
在Springframework里，我们是不能@Autowired一个静态变量。类加载后静态成员是在内存的共享区，因为当类加载器加载静态变量时，Spring上下文尚未加载。所以类加载器不会在bean中正确注入静态类，并且会失败。
**解决方法**：
方式一：初始化到非静态对象再赋值
```
@Component
public class TestClass {

   private static AutowiredTypeComponent component;

   @Autowired
   private AutowiredTypeComponent autowiredComponent;

   @PostConstruct
   private void beforeInit() {
      component = this.autowiredComponent;
   }

   // 调用静态组件的方法
   public static void testMethod() {
      component.callTestMethod();
   }

}
```
方式二：直接用Spring框架工具类获取bean，定义成局部变量使用。
```
public class TestClass {

    // 调用静态组件的方法
   public static void testMethod() {
      AutowiredTypeComponent component = SpringApplicationContextUtil.getBean("component");
      component.callTestMethod();
   }

}
```


**Spring优势**
方便解耦，简化开发
    Spring就是一个大工厂，可以将所有对象创建和依赖关系维护交给 Spring 管理
AOP 编程的支持
    Spring 提供面向切编程，可以方便的实现对序进行权限拦截、运监控等功能 提供面向切编程
声明式事务的支持
    只需要通过配置就可以完成对事务的管理，而无手动编程


## web.xml

```
<!--  每次请求都会创建一个工厂类 次请求都会创建一个工厂类 ,服务器端的资源就浪费了 ,一般情况下一个工程只有Spring 的工厂 类就OK 了. * 将工厂在服务器启动的时候创建好 ,将这个工厂放入到 ServletContext 域中 .每次获取工厂从 ServletContext 域中进行获取  -->
配置监听器 :
<listener >
    <listener -class >org.springframework.web.context.ContextLoaderListener </ listener-class >
</ listener >
<context -param >
    <param-name >contextConfigLocation </param-name >
    <param-value >classpath:applicationContext.xml </param-value >
</context-param >
```

getBean操作
```
WebApplicationContext applicationContext = WebApplicationContextUtils. getWebApplicationContext (ServletActionContext. getServletContext());
CustomerService customerService = (CustomerService) applicationContext .getBean( "customerService" );
```


## applicationContext-service.xml
**配置注解扫描**
```
<! -- Spring 的注解开发 :组件扫描 (类上注解 : 可以直接使用属性注入的解 可以直接使用属性注入的解 ) -- >
<context:component -scan base -package="com.itheima.spring.demo1"/>
```

**引入外部属性文件**
```
<context:property -placeholder location="classpath:jdbc.properties"/>
```

在相关的类上添加注解 :
```
@Component (value= "userDao ") public class UserDaoImpl implements UserDao {

    @Override public void sayHello() {
        System. out .println( "Hello Spring Annotation ..." );
        }
    }
```

**注解解释**
1.2.2.1  Spring 中提供 @Component 的三个衍生注解 :( **功能目前来讲是一致的 **)  
@Controller :WEB 层 *
@Service :业务层 业务层 *
@Repository :持久层 持久层
这三个注解是为了让标类本身的用途清晰

1.2.2.2  注解的属性
@Value :用于注入普通类型 .
@Autowired :自动装配 :
    * 默认按类型进行装配
    * 按名称注入 按名称注入 :
       * @Qualifier: 强制使用名称注入 .
@Resource 相当于 相当于 : * @Autowired 和@Qualifier 一起使用 .

1.2.2.3 Bean 的作用范围注解 :
@Scope:
    * singleton: 单例
    * prototype: 多例

1.2.2.4 Bean的生命周期配置：
@PostConstruct :相当于 init-method
@PreDestroy :相当于 destroy-method


## AOP（applicationContext-trans.xml）

底层代理机制 :
 Spring 的 AOP 的底层用到两种代理机制：
    * JDK 的动态代理 :针对实现了接口的类产生代理 .
    * Cglib 的动态代理 :针对没有实现接口的类产生代理 应用的是底层字节码增强技术, 生成当前类的子类对象 .

**注解解释**
Joinpoint(连接点): 所谓连接点是指那些被拦截到的点。在spring中,这些点指的是方法,因为spring只支持方法类型的连接点. Pointcut(切入点): 所谓切入点是指我们要对哪些Joinpoint进行拦截的定义.
Advice(通知/增强): 所谓通知是指拦截到Joinpoint之后所要做的事情就是通知.通知分为前置通知,后置通知,异常通知,最终通知,环绕通知(切面要完成的功能)
Introduction(引介): 引介是一种特殊的通知在不修改类代码的前提下, Introduction可以在运行期为类动态地添加一些方法或Field. Target(目标对象): 代理的目标对象
Weaving(织入): 是指把增强应用到目标对象来创建新的代理对象的过程.
    spring采用动态代理织入，而AspectJ采用编译期织入和类装在期织入
Proxy（代理）: 一个类被AOP织入增强后，就产生一个结果代理类 Aspect(切面): 是切入点和通知（引介）的结合

1.2.7.6 通知类型
@Before 前置通知 ：在目标方法执行之前执行
@AfterReturing 后置通知 ：在目标方法执行之后执行
@Around 环绕通知 ：在目标方法执行前和后执行
@After 异常抛出通知：在目标方法执行出现异常的时候执行
@AfterThrowing 最终通知 ：无论目标方法是否出现异常最终通知都会执行

##### AspectJ进行AOP开发 :**XML的方式**

1.2.7.7 切入点表达式
```
execution( 表达式 )
表达式 :
[方法访问修饰符] 方法返回值 包名 .类名 .方法名 (方法的参数 )
public * cn.itcast.spring.dao.*.*(..)
* cn.itcast.spring.dao.*.*(..)
* cn.itcast.spring.dao.UserDao+.*(..) *
```

1.2.7.8 编写一个切面类
```
    public class MyAspectXml {
         // 前置增强
        public void before(){
            System. out .println( "前置增强 ===========" );
        }
    }
```

1.2.7.9 配置完成增强
```
<! -- 配置切面类 -- >
<bean id ="myAspectXml" class ="cn.itcast.spring.demo3.MyAspectXml" ></ bean >

<! -- 进行 aop 的配置 -- >
<aop:config >
    <! -- 配置切入点表达式 :哪些类的方法需要进行增强 哪些类的方法需要进行增强 -- >
    <aop:pointcut expression ="execution(* cn.itcast.spring.demo3.O rderDao.save(..))" id ="pointcut1" />
    <! -- 配置切面 -- >
    <aop:aspect ref ="myAspectXml" >
        <aop:before method ="before" pointcut -ref ="pointcut1" />
    </ aop:aspect >
</ aop:config >
```

##### AspectJ进行AOP开发 :**注解的方式**（常规）

**1.2.1.5 开启 aop 注解的自动代理 :**
<aop:aspectj -autoproxy/>

1.2.1.8 配置切面 :
<! -- 配置切面类 -- >
<bean id ="myAspectAnno" "myAspectAnno" class ="cn.itcast.spring.demo4.MyAspectAnno" ></ bean >

1.2.1.9 编写切面类
```
@Aspect
public class MyAspectAnno {
@Before ("MyAspectAnno.pointcut1()" )
    public void before(){
        System. out .println( "前置通知 ===========" );
    }

@AfterReturning ("MyAspectAnno.pointcut2()" )
    public void afterReturning(){
        System. out .println( "后置通知 ===========" );
    }

@Around ("MyAspectAnno.pointcut3()" )
    public Object around(ProceedingJoinPoint joinPoint ) throws Throwable{
     System. out .println( "环绕前通知 ==========" );
     Object obj = joinPoint .proceed();
     System. out .println( "环绕后通知 ==========" );
     return obj ;
     }

@AfterThrowing ("MyAspectAnno.pointcut4()" ) public void afterThrowing(){
    System. out .println( "异常抛出通知 ========" );
    }

@After ("MyAspectAnno.pointcut4()" )
    public void after(){
    System. out .println( "最终通知 ==========" );
    }

@Pointcut ("execution(* cn.itcast.spring.demo4.ProductDao.save(..))" )
     private void pointcut1(){}
@Pointcut ("execution(* cn.itcast.spring.demo4.ProductDa o.update(..))" )
    private void pointcut2(){}
@Pointcut ("execution(* cn.itcast.spring.demo4.ProductDao.delete(..))" )
     private void pointcut3(){}
@Pointcut ("execution(* cn.itcast.spring.demo4.ProductDao.find(..))" )
     private void pointcut4(){} }
```

## Spring事务管理

**连接池的配置**
①内置连接池
```
<-- 配置 Spring 的内置连接池 -- >
<bean id ="dataSource" class ="org.springframework.jdbc.datasource.DriverManagerDataSource" >
    <property name ="driverClassName" value ="com.mysq l.jdbc.Driver" />
    <property name ="url" value ="jdbc:mysql:///spring_day02" />
    <property name ="username" value ="root" />
    <property name ="password" value ="123" />
</ bean >
```

②DBCP连接池
```
<-- 配置 DBCP 连接池 -- >
<bean id ="dataSource" "dataSource" class ="org.apache.commons.dbcp.BasicDataSource" >
    <property name ="driverClassName" value ="com.mysql.jdbc.Driver" />
    <property name ="url" value ="jdbc:mysql:///spring_day02" />
    <property name ="username" value ="root" />
    <property name ="password" value ="123" />
</ bean >
```

③c3p0连接池
```
<! -- 配置 C3P0 连接池 -- >
<bean id ="dataSource" "dataSource" class ="com.mchange.v2.c3p0.ComboPooledDataSource" >
    <property name ="driverClass" value ="com.mysql.jdbc.Driver" />
    <property name ="jdbcUrl" value ="jdbc:mysql:///spring_day02" />
    <property name ="user" value ="root" />
    <property name ="password" value ="123" />
</ bean >
```

**事务的传播行为**
```
保证同一个事务中
PROPAGATION_REQUIRED 支持当前事务，如果不存在 就新建一个(默认)
PROPAGATION_SUPPORTS 支持当前事务，如果不存在，就不使用事务
PROPAGATION_MANDATORY 支持当前事务，如果不存在，抛出异常

保证没有在同一个事务中
PROPAGATION_REQUIRES_NEW 如果有事务存在，挂起当前事务，创建一个新的事务
PROPAGATION_NOT_SUPPORTED 以非事务方式运行，如果有事务存在，挂起当前事务
PROPAGATION_NEVER 以非事务方式运行，如果有事务存在，抛出异常
PROPAGATION_NESTED 如果当前事务存在，则嵌套事务执行
```

**1.5.2 Spring的编程式事务-手动编写**
![spring](/images/探索SSM（二）-Spring/spring.jpg)

**1.5.3 Spring的声明式事务管理：XML 方式**

1.5.3.3 配置事务管理器
```
<-- 事务管理器 -- >
<bean id ="transactionManager" class ="org.sp ringframework.jdbc.datasource.DataSourceTransactionManager" >
    <property name ="dataSource" ref ="dataSource" />
</ bean >
```

1.5.3.4 配置事务的通知
```
<! -- 配置事务的增强 -- >
<tx:advice id ="txAdvice" "txAdvice" transaction -manager ="transactionManager" >
    <tx:attributes > <! -- isolation="DEFAULT" 隔离级别 propagation="REQUIRED" 传播行为 read -only="false" 只读 timeout=" -1" 过期时间 rollback -for="" -Exception no -rollback -for="" +Exception -- >
        <tx:method name ="transfer" propagation ="REQUIRED" />
    </ tx:attributes >
</ tx:advice >
```

1.5.3.5 配置 aop 事务
<aop:config >
    <aop:pointcut expression ="execution(* cn.itcast.transaction.demo2.AccountServiceImpl.transfer(..))" id ="pointcut1" />
    <aop:advisor advice -ref ="txAdvice" pointcut -ref ="pointcut1" />
</ aop:config >

**1.5.3 Spring的声明式事务管理：注解方式**

1.5.4.1 引入 jar包:
![spring1](/images/探索SSM（二）-Spring/spring1.jpg)

1.5.4.3 配置事务管理器 :
```
<! -- 配置事务管理器 -- >
<bean id ="transactionManager" class ="org.springframework.jdbc.datasource.DataSourceTransactionManager" >
    <property name ="dataSource" ref ="dataSource" />
</bean >
```

**1.5.4.4 开启事务管理的注解 :**
```
<! -- 开启注解事务管理 -- >
<tx:annotation -driven transaction -manager ="transactionManager" />
```

1.5.4.5 在使用事务的类上添加一个注解： @Transactional
![spring2](/images/探索SSM（二）-Spring/spring2.jpg)



e.g. 本文仅供个人笔记使用，借鉴部分网上资料。

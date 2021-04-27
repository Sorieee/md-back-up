---
title: Spring-study
date: 2021-03-28 09:32:16
tags:
---

https://www.bilibili.com/video/BV1WZ4y1H7du?p=39&spm_id_from=pageDriver

# Spring注解

## 原始注解

| 注解           |                            |
| -------------- | -------------------------- |
| @Component     |                            |
| @Controller    |                            |
| @Service       |                            |
| @Repository    |                            |
| @Autowired     |                            |
| @Qualifier     | 指定按名字注入             |
| @Resource      | = @Autowired  + @Qualifier |
| @Value         |                            |
| @Scope         |                            |
| @PostConstruct |                            |
| @PreDetroy     |                            |

## 新注解

* 非自定义的bean
* 加载properties文件的配置
* 组件扫描
* 引入其他文件。

| 注解            |                                              |
| --------------- | -------------------------------------------- |
| @Configuration  | 指定为配置类，创建容器会从该类加载注解       |
| @ComponentScan  | 扫描包                                       |
| @Bean           | 使用在方法上，将方法返回值存储到Spring容器中 |
| @PropertySource | 加载properties文件                           |
| @Import         | 导入其他配置类                               |

# Junit

如何获取容器？

* 让Spring Junit创建Spring容器
* 将测试Bean直接注入在测试类中。



步骤：

1. 导致jar包
2. @Runwith
3. @ContextConfiguration
4. @Autowired

```xml
	<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.9.RELEASE</version>
    </dependency>
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration("classpath:appContext.xml")
@ContextConfiguration(classes = {MyConfig.class})
public class SpringJunitTest {
    @Autowired
    private UserService userService;

    @Test
    public void test1() {
        userService.saveUser();
    }
}
```

# XML-AOP

Aspect Oriented Programming

* 作用：在程序运行期间，在不修改源码的情况下，对方法进行增强。
* 优势：减少重复代码，提高开发效率，并且便于维护。

## 底层实现

​	动态代理。在运行期间通过动态代理生成代理对象，代理对象方法执行时进行增强功能的介入。

​	常用动态代理技术。

* JDK代理：基于接口。
* cglib：基于父类。

### 基于JDK的动态代理

![](https://pic.imgdb.cn/item/605fde1e8322e6675cac11ad.jpg)

```java
public class ProxyTest {
    public static void main(String[] args) {
        Target t = new Target();
        Advice advice = new Advice();
        // 返回动态代理对象
        TargetInterface proxy1 = (TargetInterface) Proxy.newProxyInstance(
                t.getClass().getClassLoader(),
                t.getClass().getInterfaces(), // 目标对象相同的接口字节码对象数组
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        advice.before();
                        method.invoke(t, args);
                        advice.afterReturning();
                        return null;
                    }
                }
        );
        proxy1.save();
    }
}
```



### 基于cglib的动态代理

![](https://pic.imgdb.cn/item/605fde2b8322e6675cac1702.jpg)

​	目前版本已经集成到spring-core中。

```java
public class ProxyTest {
    public static void main(String[] args) {
        Target t = new Target();
        Advice advice = new Advice();
        // 返回动态代理对象
        // 1. 创建增强器
        Enhancer enhancer = new Enhancer();
        // 2. 设置父类
        enhancer.setSuperclass(t.getClass());
        // 3. 设置回调
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                advice.before();
                method.invoke(t);
                advice.afterReturning();
                return null;
            }
        });
        Target proxy = (Target)enhancer.create();
        proxy.save();
    }
}
```

## 相关概念

* Target(目标对象): 代理目标对象。
* Proxy(代理)：一个类被AOP织入增强后，会产生一个结果代理类。
* JoinPoint(连接点)： 拦截到的点，Spring指的方法，spring只支持方法类型的连接点。
* Pointcut(切入点): 切入点是我们要对哪些JointPoint的进行拦截的定义。
* Advice(通知/增强)：所谓通知时指拦截到JoinPoint之后要做的事情就是通知。
* Aspect(切面) : 切入点和通知的结合。
* Weaving(织入)：是指把增强应用到目标对象来创建新的代理对象的过程。spring采用动态代理织入，而AspectJ采用编译器织入和类装载期织入。



## 快速入门



* 导入包

```xml
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
```

* 创建目标接口和目标类(内部有切点)。
* 创建切面类(内部有增强方法)。
* 将目标类和切面类的对象创建权交给spring
* 在xml织入关系。

```xml
<bean id="target" class="top.sorie.spring_aop.aop.Target"></bean>
        <bean id="myAspect" class="top.sorie.spring_aop.aop.MyAspect"></bean>
        <aop:config>
                <aop:aspect ref="myAspect">
                        <aop:before method="before" pointcut="execution(public void top.sorie.spring_aop.aop.Target.save())"></aop:before>
                </aop:aspect>
        </aop:config>
```



* 测试代码。

## 切点表达式

```xml
<aop:before method="before" pointcut="execution(public void top.sorie.spring_aop.aop.Target.save())"></aop:before>
```

表达式

```
execution([修饰符] 返回值类型 包名.类名.方法名(参数))
```

* 访问修饰符可以省略
* 返回值 、包名、类、方法名都可以使用*代表任意。
* 参数类别可以使用两个点..表示任意个数，任意类型的参数列表。

```
execution(* top.sorie.aop.*.*(..)); // aop的中子包
execution(* top.sorie.aop..*.*(..)); // aop及其子包
```



## 通知类型

| 名称         | 标签                    | 说明                   |
| ------------ | ----------------------- | ---------------------- |
| 前置通知     | `<aop:before>`          | 在                     |
| 后置通知     | `<aop:after-returning>` | 之后通知，有异常不通知 |
| 环绕通知     | `<aop:around>`          | 之前和之后都执行。     |
| 异常抛出通知 | `<aop:afterThrowing>`   | 异常通知               |
| 最终通知     | `<aop:after>`           | 无论有异常没有都会通知 |

```java
public Object around(ProceedingJointPoint pjp) throws Throwable {
    // xxx
    // 切入方法
    Object obj = pjp.proceed();
    // xxx
    return obj;
}

public void afterThrowing() {
    
}
```

## 抽取切点表达式

```xml
<aop:config>
        <aop:aspect ref="myAspect">
                <aop:pointcut id="xx" expression="execution(public void top.sorie.spring_aop2.aop.Target.save())"/>
                <aop:before method="before" pointcut-ref="xx"></aop:before>
        </aop:aspect>
</aop:config>
```

# 注解-AOP

## 快速入门

开发步骤：

1. 创建目标接口和目标类。
2. 创建切面类。
3. 将切面类和目标类的对象创建权交给spring。
4. 在切面类中使用注解配置织入关系。
5. 在配置文件中开启组件扫描和AOP自动代理。
6. 测试。

xml中

```xml
<aop:aspect-autoproxy/>
```

```java
@Component
@Aspect
public class MyAspect {

    @Before("execution(public void top.sorie.spring_aop2.anno.Target.save())")
    public void before() {
        System.out.println("before!");
    }
}
```

## 类型

| 名称         | 标签             | 说明                   |
| ------------ | ---------------- | ---------------------- |
| 前置通知     | `@Before`        | 在                     |
| 后置通知     | `@AfterReturing` | 之后通知，有异常不通知 |
| 环绕通知     | `@Around`        | 之前和之后都执行。     |
| 异常抛出通知 | `@AfterThrowing` | 异常通知               |
| 最终通知     | `@After`         | 无论有异常没有都会通知 |

## 表达式抽取

```java
@Component
@Aspect
public class MyAspect {
	@Pointcut("execution(public void top.sorie.spring_aop2.anno.Target.save())")
    public void myPoint(){}
    
    @Before("MyAspect.mypoint()")
    public void before() {
        System.out.println("before!");
    }
}
```

# 注解

```java
//即使用jdk默认代理模式，AspectJ代理模式是CGLIB代理模式
//如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP
//如果目标对象实现了接口，可以强制使用CGLIB实现AOP (此例子我们就是强制使用cglib实现aop)
//如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

//用于定义配置类，可替换xml配置文件
@Configuration
//开启AspectJ 自动代理模式,如果不填proxyTargetClass=true，默认为false，
@EnableAspectJAutoProxy(proxyTargetClass=true)
//扫描注入类
@ComponentScan(basePackages = "com.dyh.ioc.*")
@Component
@Aspect
public class AopAspectConfiguration {
    //声明切入点
    //第一个*表示 方法  返回值（例如public int）
    //第二个* 表示方法的全限定名（即包名+类名）
    //perform表示目标方法参数括号两个.表示任意类型参数
    //方法表达式以“*”号开始，表明了我们不关心方法返回值的类型。然后，我们指定了全限定类名和方法名。对于方法参数列表，
    //我们使用两个点号（..）表明切点要选择任意的perform()方法，无论该方法的入参是什么
    //execution表示执行的时候触发
    @Pointcut("execution(* *(..))")
    public void point(){
        //该方法就是一个标识方法，为pointcut提供一个依附的地方
    }

    @Before("point()")
    public void before(){
        System.out.println("Before");
    }
    @After("point()")
    public void after(){
        System.out.println("After");
    }
}
```

# Spring JdbcTemplate

1. 导入spring-jdbc和spring-tx。
2. 创建数据表和实体。
3. 创建JdbcTemplate对象。
4. 执行数据库操作。

```xml
		<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.9.RELEASE</version>
        </dependency>
```

```java
@Test
public void test1() throws Throwable{
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/springdemo");
    dataSource.setUser("root");
    dataSource.setPassword("cquisse");
    JdbcTemplate jdbcTemplate = new JdbcTemplate();
    // 设置数据源
    jdbcTemplate.setDataSource(dataSource);
    int row = jdbcTemplate
            .update("insert into account(name, money) values(?, ?)", "Tom", 5000);
    System.out.println(row);
}
```

```sql
CREATE TABLE account (
	id int NOT NULL AUTO_INCREMENT,
	name char (50) NOT NULL,
	money int NOT NULL,
	PRIMARY KEY(id)
);
```

## Spring产生JdbcTemplate对象

```xml
<bean id = "dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="user" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>
<bean id = "jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name = "dataSource" ref="dataSource"></property>
</bean>
```

## 常用操作

```java
int row = jdbcTemplate
        .update("insert into account(name, money) values(?, ?)", "Tom", 5000);
int row = jdbcTemplate
                .update("delete from account where name = ?", "Tom");        
```

```java
        List<Account> accountList = jdbcTemplate.query("select * from account", new BeanPropertyRowMapper<Account>(Account.class));
        Account obj = jdbcTemplate.queryForObject("select * from account where name = ?", new BeanPropertyRowMapper<Account>(Account.class), "tom");
        Long cnt = jdbcTemplate.queryForObject("select count(*) from account", Long.class);

```

# 事务

## 编程式事务

### PlatformTransactionManager

​	PlatformTransactionManager接口是spring的事务管理器

| 方法                                                      | 说明               |
| --------------------------------------------------------- | ------------------ |
| TransactionStatus getTransaction(Transaction definition); | 获取事务的状态信息 |
| void commit(TransactionStatus status);                    | 提交事务           |
| void rollback(TransactionStatus status);                  | 回滚事务           |

![](https://pic.imgdb.cn/item/60648f538322e6675cd3759d.jpg)

### TransactionDefinition

| 方法                          | 说明               |
| ----------------------------- | ------------------ |
| int getIsolationLevel();      | 获取事务的隔离级别 |
| int getPropogationBehavior(); | 获得事务的传播行为 |
| int getTimeout();             | 获得超时时间       |
| boolean isReadyOnly();        | 是否只读           |

 **事务隔离级别**

```java
int ISOLATION_DEFAULT = -1;
int ISOLATION_READ_UNCOMMITTED = 1; // 解决脏读
int ISOLATION_READ_COMMITTED = 2; // 解决不可重复读
int ISOLATION_REPEATABLE_READ = 4; // 解决幻读
int ISOLATION_SERIALIZABLE = 8; // 都可以
```

**事务传播行为**

```java
int PROPAGATION_REQUIRED = 0; // 没有事务就创建，否则加入。默认值
int PROPAGATION_SUPPORTS = 1; // 有就有，没有就没有
int PROPAGATION_MANDATORY = 2; // 使用当前事务，没有就抛出异常
int PROPAGATION_REQUIRES_NEW = 3; // 新建事务，如果有事务，挂起
int PROPAGATION_NOT_SUPPORTED = 4; // 非事务执行，有就挂起
int PROPAGATION_NEVER = 5; // 存在事务，就嵌套执行，没有事务，就和REQUIRED类似
int PROPAGATION_NESTED = 6;
```

超时时间：默认-1，代表没有超时限制。有以秒为单位进行设置。

是否只读：建议查询时设置为只读。

**TransactionStatus**

| 方法                        | 说明           |
| --------------------------- | -------------- |
| boolean hasSavepoint();     | 是否存储回滚点 |
| boolean isCompleted();      | 事务是否完成   |
| boolean isNewTransaction(); | 是否新事务     |
| boolean isRollbackOnly();   | 事务是否回滚   |

## 基于XML的声明式事务

​	顾名思义就是采用声明的方式来处理事务。

**作用**

* 不侵入开发组件。
* 可以随时移除，无需重新编译。



​	Spring声明式事务控制底层就是AOP。



```xml
xmlns:tx="http://www.springframework.org/schema/tx"


	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                <property name="dataSource" ref="dataSource"></property>
        </bean>

        <tx:advice id="txAdvice" transaction-manager="transactionManager">
                <tx:attributes>
                        <tx:method name="*" isolation="DEFAULT" propagation="REQUIRED" timeout="-1" 
                                read-only="false"/>
                </tx:attributes>
        </tx:advice>
<!--        AOP织入-->
        <aop:config>
                <aop:advisor advice-ref="txAdvice" pointcut="execution(* top.sorie.spring_jdbc.service.*.*(..))"></aop:advisor>
        </aop:config>
```

` <tx:attributes>`中可以单独配置

**要点**

* 配置平台事务管理器。
* 事务通知配置。
* 配置织入。

## 基于注解的声明式事务

```java
@Override
@Transactional(isolation = Isolation.DEFAULT, propagation = Propagation.REQUIRED)
// 也可以加载类上
public void trans() {
    // xxx
}
```

```xml
        <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
```

也可以

```java
@EnableTransactionManagement
@ComponentScan("com.atguigu.tx")
@Configuration
public class TxConfig {
     
    //数据源
    @Bean
    public DataSource dataSource() throws Exception{
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("123456");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        return dataSource;
    }
     
    @Bean
    public JdbcTemplate jdbcTemplate() throws Exception{
        //Spring对@Configuration类会特殊处理；给容器中加组件的方法，多次调用都只是从容器中找组件
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource());
        return jdbcTemplate;
    }
     
    //注册事务管理器在容器中
    @Bean
    public PlatformTransactionManager transactionManager() throws Exception{
        return new DataSourceTransactionManager(dataSource());
    }
}　
```

# Spring MVC

## 基本环境搭建

```java
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ApplicationContext app = new ClassPathXmlApplicationContext("appContext.xml");
        DemoService demoService = app.getBean(DemoService.class);
        demoService.save();
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0"  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans">
    <servlet>
        <servlet-name>UserServlet</servlet-name>
        <servlet-class>top.sorie.ademo.web.UserServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>UserServlet</servlet-name>
        <url-pattern>/userServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

## 集成ContextLoaderListener

​	在Web项目中，可以使用ServletContextListener监听Web引用启动。启动就加载配置文件。创建ApplicationContext，存储到最大的域servletContext域中，就可以在域中获得ApplicationContext;

```java
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        ApplicationContext app = new ClassPathXmlApplicationContext("appContext.xml");
        ServletContext servletContext = sce.getServletContext();
        servletContext.setAttribute("app", app);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0"  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans">
<!--    监听器-->
    <listener>
        <listener-class>top.sorie.ademo.listener.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>UserServlet</servlet-name>
        <servlet-class>top.sorie.ademo.web.UserServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>UserServlet</servlet-name>
        <url-pattern>/userServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

```java
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = req.getServletContext();
        ApplicationContext app = (ApplicationContext)servletContext.getAttribute("app");
        DemoService demoService = app.getBean(DemoService.class);
        demoService.save();
    }
}
```

### 优化

```java
public class ContextLoaderListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {

        ApplicationContext app = new ClassPathXmlApplicationContext("appContext.xml");
        ServletContext servletContext = sce.getServletContext();
        String config = servletContext.getInitParameter("contextConfig");
        servletContext.setAttribute("app", app);
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {

    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0"  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans">
<!--   全局初始化参数-->
    <context-param>
        <param-name>contextConfig</param-name>
        <param-value>appContext.xml</param-value>
    </context-param>
    <!--    监听器-->
    <listener>
        <listener-class>top.sorie.ademo.listener.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>UserServlet</servlet-name>
        <servlet-class>top.sorie.ademo.web.UserServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>UserServlet</servlet-name>
        <url-pattern>/userServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

## WebApplicationContextUtils

以上分析无需自己手动实现。

我们只需要

* 在web.xml配置监听器(配置spring-web 的maven)
* 

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" version="3.0"  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans">
<!--   全局初始化参数-->
    <context-param>
        <param-name>contextConfig</param-name>
        <param-value>classpath:appContext.xml</param-value>
    </context-param>
    <!--    监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>UserServlet</servlet-name>
        <servlet-class>top.sorie.ademo.web.UserServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>UserServlet</servlet-name>
        <url-pattern>/userServlet</url-pattern>
    </servlet-mapping>
</web-app>
```

```java
public class UserServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = req.getServletContext();
        WebApplicationContext app = WebApplicationContextUtils.getWebApplicationContext(servletContext);
        DemoService demoService = app.getBean(DemoService.class);
        demoService.save();
    }
}
```

## Spring MVC简介

![](https://pic.imgdb.cn/item/6067fc718322e6675cd99bca.jpg)

**开发步骤**

1. 导包。
2. 配置Servlet(DispatherServlet)。
3. 编写Controller。
4. 注解配置Controller中业务方法的映射地址。
5. 配置spring-mvc.xml(配置组件扫描)。



```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.9.RELEASE</version>
</dependency>
```

web.xml中

```xml
<servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

```java
@Controller
public class UserController {

    @RequestMapping("/quick")
    public String save() {
        System.out.println("Controller save");
        return "success.jsp";
    }
}
```

### 快速搭建

spring-mvc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd ">

    <!-- 开启spring的扫描注入，使用如下注解 -->
    <!-- @Component,@Repository,@Service,@Controller-->
    <!-- 错误点 com.hf.controller 不能扫描到service ，dao等-->
    <context:component-scan base-package="top.sorie.ademo.controller"/>

    <!-- 开启springMVC的注解驱动，使得url可以映射到对应的controller -->
    <mvc:annotation-driven />

    <!-- 视图解析 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

```java
@Controller
public class SeDemoController {

    @RequestMapping("/quick")
    public String save() {
        System.out.println("Controller save");
        return "success";
    }
}
```



web.xml

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd "
         version="3.0">
    <display-name>Archetype Created Web Application</display-name>

    <!-- 配置引用spring-mvc.xml -->
    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 启动时加载spring-mvc.xml -->
            <param-value>classpath:spring-mvc.xml</param-value>
            <!--<param-value>/WEB-INF/spring-mvc.xml</param-value>-->
        </init-param>
        <!-- 启动时加载spring-mvc.xml 的优先级高-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

中间遇到过404，重新加载artifact就可以了。

![](https://pic.imgdb.cn/item/606848788322e6675c32644f.jpg)

![](https://pic.imgdb.cn/item/606848a18322e6675c3287d1.jpg)

**开发步骤**

1. 导入SpringMVC相关包。
2. 配置核心控制器DispatcherServlet。
3. 创建Controller和视图页面。
4. 配置Controller业务方法的映射地址。
5. 配置SpringMVC核心文件spring-mvc.xml
6. 客户端发起请求测试。

## 组件解析

**SpringMVC执行流程**

![](https://pic.imgdb.cn/item/60684a318322e6675c33e613.jpg)

1. 请求到前端控制器DispatcherServlet。
2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3. 处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器，并返回给DispatchServlet。
4. DispatchServlet调用HandlerAdapter处理器适配器。
5. HandlerAdapter经过适配调用具体的处理器。
6. Controller执行完成返回ModelAndView。
7. HandlerAdapter将Controller执行结果ModelAndView返回给DispatchServlet。
8. DispatcherServlet将ModelAndView传给ViewResolver视图解析器。
9. ViewResolver解析后返回具体的View。

### 注解解析

**@RequestMapping**

用于建立URL和处理请求方法之间的对应关系。

* 类上：1级目录。
* 方法上，2级访问目录，和1级目录拼接。
* value：url，和path作用一样。
* method：方法类型。
* params: 请求的参数限制条件。
* params={"accountName"},表示请求参数中必须有accountName。
* params={"money!100"},表示请求参数中money不能是100。

**扫描**

```xml
<context:component-scan base-package="top.sorie.ademo">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

## 数据响应方式

* 页面跳转
  * 直接返回字符串。
  * 通过ModelAndView对象返回。
* 回写数据
  * 直接返回字符串。
  * 返回对象或集合。

### 页面跳转

**返回字符串**

* 会将返回的字符串与视图解析器的前后缀拼接后跳转。



![](https://pic.imgdb.cn/item/60684e638322e6675c3a00ef.jpg)

**返回ModelAndView对象**

```java
@RequestMapping("/quick")
public ModelAndView save() {
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("success");
    modelAndView.addObject("username", "sorie");
    return modelAndView;
}

@RequestMapping("/quick3")
public ModelAndView save3(ModelAndView modelAndView) {
    modelAndView.setViewName("success");
    modelAndView.addObject("username", "sorie");
    return modelAndView;
}
@RequestMapping("/quick4")
public String save4(Model model) {
    model.addAttribute("username", "sorie");
    return "success";
}
@RequestMapping("/quick5")
public String save5(HttpServletRequest req) {
    req.addAttribute("username", "sorie");
    return "success";
}
```

## 回写数据

**直接返回字符串**

`resp.getWriter().print("Hello World")`

```java
@RequestMapping("/quick6")
public void save6(HttpServletRequest req, HttpServletResponse resp) {
    resp.getWriter().print("Hello World")`;
}
@RequestMapping("/quick7")
@ResponseBody
public String save7() {
    return "Hello World";
}
```

@ResponseBody告知SpringMVC，方法返回的字符串不是跳转而是直接在http响应体里放回。

**返回json**

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.3</version>
</dependency>
```

```java
@RequestMapping("/quick8")
@ResponseBody
public String quick8() throws IOException {
    User user = new User();
    user.setUsername("sorie");
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(user);
    return json;
}
```

**配置转换**

```java
@RequestMapping("/quick8")
@ResponseBody
public User quick8() throws IOException {
    User user = new User();
    user.setUsername("sorie");
    return user;
}
```

配置处理器映射器

spring-mvc.xml

```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            </bean>
        </list>
    </property>
</bean>
```

**driven**

在方法上添加@ResponseBody比较麻烦，可以用

```xml
<mvc:annotation-driven />
```

代替。

​	可以自动加载处理映射器和处理适配器，可用它代替。它会默认集成jackson对象或集合的json格式字符串的转换。

**小结**

页面跳转

* 直接返回字符串。
* 返回ModelAndView。

回写数据

* 直接返回字符串。
* 返回对象或集合。

## 获取参数

### **基本类型**

```java
@RequestMapping("/quick8")
@ResponseBody
public void quick8(String username, int age) throws IOException {
    return;
}
```

### **POJO参数**

```java
@RequestMapping("/quick9")
@ResponseBody
public void quick9(User user) throws IOException {
    return;
}
```

### **数组类型**

url:${baseUrl}/strs=111&strs=222&strs=333

```java
@RequestMapping("/quick10")
@ResponseBody
public void quick10(String[] strs) throws IOException {
    return;
}
```

### **集合类型**

不能这么写！！！！！！！！！！！！

```java
@RequestMapping("/quick11")
@ResponseBody
public void quick11(List<User> users) throws IOException {
    return;
}
```

得用一个对象把List封起来。

或者可以用@RequestBody就可以了

```java
@RequestMapping("/quick12")
@ResponseBody
public void quick12(@RequestBody List<User> users) throws IOException {
    return;
}
```

注意开放静态资源访问权限

```xml
<mvc:resource mapping="/js/**" location="/js/"/>
```

或者

```xml
<mvc:default-servlet-handler/>
```

### **请求乱码问题**

可以设置过滤器来进行编码的过滤

web.xml。

```xml
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  <filter>
```

### **参数绑定**

```java
@RequestMapping("/quick8")
@ResponseBody
public void quick8(@RequestParam(value="name")String username, int age) throws IOException {
    return;
}
```

**@RequestParam**

* value:与请求参数名称。
* required：必填，默认true。
* defaultValue：当没有指定请求参数时，则使用指定的默认值赋值。

### **获取Restful风格参数**

```java
@RequestMapping("/quick8/{name}")
@ResponseBody
public void quick8(@PathVariable(value="name")String username) throws IOException {
    return;
}
```

### **自定义转换器**

步骤：

* 定位转换器实现Converter接口。
* 在配置文件中声明转换器。
* 在`<annotation-driven>`中引用转换器。

```java
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String s) {
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
        Date date = null;
        try {
            date = format.parse(s);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```

spring-mvc.xml

```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="top.sorie.ademo.converter.DateConverter"></bean>
            </list>
        </property>
</bean>
<mvc:annotation-driven conversion-service="conversionService"/>
```

* Spring MVC已经默认提供了一些类型转换器，比如客户端的字符串转换成int型。
* 但是不是所有数据类型都提供了，比如日期。

### **获取请求头**

**@RequestHeader**

* value:请求头的名称。
* required：是否必须携带此请求头。

```java
@RequestMapping("/quick8/{name}")
@ResponseBody
public void quick8(@RequestHeader(value="User-Agent", required=false) String userAgent) throws IOException {
    return;
}
```

**@CookieValue**

* value：指定cookie的名称。
* required：是否必须携带此cookie。

```java
@RequestMapping("/quick8/{name}")
@ResponseBody
public void quick8(@CookieValue(value="JSESSIONID", required=false) String jsessionId) throws IOException {
    return;
}
```

### 文件上传

**jsp三要素**

* 表单项: type="file"
* 提交方式是post
* enctype="multipart/form-data"

**上传原理**

* 多部分表单时，request.getParameter()将失效。
* enctype="application/x-www-form-unlencoded"时，正文内容是:key=value&key=value
* 当取值是"multipart/form-data"时，请求正文内容就变成多部分形式。

**单文件上传步骤**

* 导入fileupload和io坐标
* 配置文件上传解析器。
* 编写文件上传代码。

```xml
	<dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.3</version>
        </dependency>
```

解析器

```xml
<!--    文件上传解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<!--    总大小-->
        <property name="maxUploadSize" value="5000000000"/>
<!--    每个文件大小-->
        <property name="maxUploadSizePerFile" value="5000000000"/>
<!--    编码-->
        <property name="defaultEncoding" value="UTF-8"/>
    </bean>
```

上传代码

```java
@RequestMapping("/quick18")
    @ResponseBody
    public void quick18(String name, MultipartFile uploadfile) throws IOException {
        uploadfile.transferTo(new File("D:\\upload\\" + uploadfile.getOriginalFilename()));
}
```

**上传多文件**

```java
@RequestMapping("/quick18")
    @ResponseBody
    public void quick18(String name, MultipartFile[] uploadfile) throws IOException {
   
}
```

## Spring MVC拦截器

​	类似于Servlet开发中的过滤器Filter，对处理器进行预处理和后处理。

​	拦截器按一定顺序连接成一条链，称为拦截器链。



**拦截器和过滤器区别**

| 区别     | 过滤器                                                      | 拦截器                                                       |
| -------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| 使用范围 | servlet规范中一部分，所有Java Web工程都可以使用             | SpringMVC可用                                                |
| 拦截范围 | 在url-pattern中配置了/*之后，可以对所有要访问的资源进行拦截 | 只会拦截访问的控制方法，如果访问的是jsp，html，css，image，js不会拦截 |

**快速入门**

* 创建拦截器实现HandlerInterceptor接口。
* 配置拦截器。
* 测试。

```java
public class MyInterceptor implements HandlerInterceptor {
    // 目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        return false; // true 放行 false不放行
    }

    // 目标方法执行之后 视图返回之前
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    //
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

spring-mvc.xml

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="top.sorie.ademo.interceptor.MyInterceptor"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

**说明**

| 方法            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| preHandle       | 请求处理器前。返回true才会调用下一个拦截器的preHandle方法    |
| postHandle      | 请求处理之后调用，视图渲染之前，可以对请求处理之后的ModelAndView对象进行操作。 |
| afterCompletion | 整个请求结束后，渲染了对应视图之后执行。                     |

**用户登录拦截**

```
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/user/login"/>
        <bean class="top.sorie.ademo.interceptor.MyInterceptor2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

```java
public class MyInterceptor2 implements HandlerInterceptor {
    // 目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        // 登录完成后
        User user = (User)session.getAttribute("user");
        if (user == null) {
            response.sendRedirect(request.getContextPath() + "login.jsp");
            return false;
        }
        return true;
    }

    // 目标方法执行之后 视图返回之前
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    //
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

```java
	@RequestMapping("/login")
    public String quick18(String username, String password, HttpSession session) throws IOException {
   	User user = userService.login(usernam, password);
    if (user != null) {
        session.setAttribute("user", user);
        return "redirect:/index.jsp";
    }
    return "redirect:/login.jsp";
}
```

## 统一异常处理

**思路**

预期异常和运行时异常。前者捕获，后者主要通过代码规范和测试减少发生。

![](https://pic.imgdb.cn/item/606876fc8322e6675c64f251.jpg)

**两种方式**

* Spring MVC提供的SimpleMappingExceptionHandler。
* 实现异常处理接口HandlerExceptionResolver，自定义自己的异常处理器。



**简单异常处理器**

![](https://pic.imgdb.cn/item/606877e98322e6675c65f542.jpg)

**自定义异常处理器**

处理步骤

* 创建异常处理器类实现HandlerExpcetionResolver。
* 配置异常处理器。
* 编写异常页面。
* 测试异常跳转。

```java
public class MyExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        ModelAndView modelAndView = new ModelAndView();
        if (ex instanceof ClassCastException) {
            modelAndView.addObject("info", "类转换异常");
        }
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

spring-mvc.xml

```xml
<bean class="top.sorie.ademo.resolver.MyExceptionResolver"/>
```

# MyBatis

**原始jdbc操作**

![](https://pic.imgdb.cn/item/60687c238322e6675c6a719a.jpg)

1. 注册驱动
2. 获得连接
3. 获得statement。
4. 执行查询。
5. 遍历结果集。
6. 封装实体。
7. 释放资源。

**问题**

* 频繁创建释放造成系统资源浪费从而影响性能。
* sql语句硬编码，不容易维护。
* 查询时，需要手动封装进实体，插入时，需要手动设置数据。

**解决方案**

* 池化。
* sql语句抽取到xml中。
* 反射和内省，自动将实体和表进行属性与字段的自动映射。



## 简介

* mybatis是一个基于Java的持久层框架。
* 通过XML和注解完成statement配置。
* 通过ORM思想解决了实体和数据库映射问题。

## 快速入门

**开发步骤**

* 导包。
* 创建数据表。
* 编写实体类。
* 编写映射文件UserMapper.xml
* 编写核心文件SqlMapConfig.xml
* 编写测试类。

```sql
CREATE TABLE user (
   id int NOT NULL AUTO_INCREMENT,
   name char (50) NOT NULL,
   password char (50) NOT NULL,
   PRIMARY KEY(id)
);
```

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.5</version>
</dependency>
```

UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="userMapper">
    <select id="findAll" resultType="top.sorie.ademo.entity.User">
        select * from user
    </select>
</mapper>
```

sqlMapConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    数据源环境-->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/springdemo"/>
                <property name="username" value="root"/>
                <property name="password" value="cquisse"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="top/sorie/mapper/UserMapper.xml"></mapper>
    </mappers>
</configuration>
```

**编写测试代码**

```java
public class MybatisTest {
    @Test
    public void test1() throws Exception {
        InputStream resourceStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 获取session工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceStream);
        // 获得session会话对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 执行操作 参数：namespace+id
        List<User> userList = sqlSession.selectList("userMapper.findAll");
        System.out.println(userList);
        // 释放资源
        sqlSession.close();
    }
}
```

## **映射文件概述**

![](https://pic.imgdb.cn/item/60693a848322e6675c151383.jpg)

## 增删改实例

**增**

UserMapper.xml

```xml
   <insert id="save" parameterType="top.sorie.ademo.entity.User">
        insert into user values(#{id}, #{username}, #{password}) 
    </insert>
```

```java
 @Test
    public void test1() throws Exception {
        InputStream resourceStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 获取session工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceStream);
        // 获得session会话对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 执行操作 参数：namespace+id
        User user = new User();
        user.setUserName("tom");
        user.setPassword("abc");
		sqlSession.insert("userMapper.save", user);
        sqlSession.commit();
        // 释放资源
        sqlSession.close();
    }
```

**更新**

```java
User user = new User();
user.setId(2);
user.setUserName("tom");
user.setPassword("abc");
sqlSession.update("userMapper.update", user);
```

**删除**

```java
sqlSession.delete("userMapper.delete", 8);
```

## 核心文件概述

* configuration配置
  * properties 属性
  * setting 设置
  * typeAliases 类型别名
  * typeHandlers 类型处理器
  * objectFactory 对象工程
  * plugins 插件
  * environments 环境
    * environment 环境变量
    * transactionManager 事务管理器
    * dataSource 数据源
  * databaseProvider 数据库厂商标识
  * mappers 映射器

### **environments标签**

![](https://pic.imgdb.cn/item/60693e278322e6675c1929da.jpg)

transactionManager有两种

* JDBC： 直接使用JDBC的提交和回滚。依赖从数据源得到的连接来管理事务作用域。
* MANAGED:几乎什么不做，不提交和回滚一个连接，让容器来管理事务的声明周期。默认情况会关闭连接，一些容器不希望这么做，可以将closeConnection属性设置为false来阻止默认关闭行为。



datasource有三种类型：

* UNPOOLED: 只是每次被请求时打开和关闭连接池。
* POOLED: 池化
* JNDI: 为了能在EJB或应用服务器这类容器中使用，容器可以集中在外部配置数据源，然后放置一个JNDI上下文的引用。

### mapper标签

加载方式有几种

* 类路径，用resource。
* 全限定资源定位符(URL),用url
* 映射器接阔实现类的全限定名， class
* 将包内映射器接口实现全部注册为映射器 `<package name="org.mybaties.builder/>"`

### properties标签

![](https://pic.imgdb.cn/item/606940128322e6675c1acf07.jpg)

**typeAliases**

![](https://pic.imgdb.cn/item/606940558322e6675c1b05b5.jpg)

已经设置好的别名

| 别名    | 数据类型 |
| ------- | -------- |
| string  | String   |
| long    | Long     |
| int     | Integer  |
| double  | Double   |
| boolean | Boolean  |
| ...     | ...      |

## API

**工程构建器SqlSessionFactoryBuilder**

![](https://pic.imgdb.cn/item/606941db8322e6675c1c4daf.jpg)

**工厂对象SqlSessionFactory**

| 方法                         | 解释                         |
| ---------------------------- | ---------------------------- |
| openSession()                | 默认开启事务，但是不会提交。 |
| openSession(bool autoCommit) | 参数为是否自动提交           |

**SqlSession会话对象**

![](https://pic.imgdb.cn/item/6069425d8322e6675c1cb5d5.jpg)

## Dao层实现

### 传统开发方式

1. 编写UserDao接口

```java
public interface UserMapper {
    List<User> findAll() throws IOException;
}
```

```java
public interface UserMapperImpl implements UserMapper {
    List<User> findAll() throws IOException {
        InputStream resourceStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        // 获取session工厂对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceStream);
        // 获得session会话对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        // 执行操作 参数：namespace+id
        List<User> userList = sqlSession.selectList("userMapper.findAll");
        System.out.println(userList);
        // 释放资源
        sqlSession.close();
    }
}
```

### 代理开发

​	只需要程序员编写Mapper接口，由Mybatis框架根据接口定义创建接口的动态代理对象。

​	需要遵循如下规范：

* namespace和mapper接口全限定名相同。
* 接口方法名和Mapper.xml中定义的每个statement的id相同。
* 接口输入参数和mapper.xml中定义的每个sql的parameterType类型相同。
* 输出参数类型也相同。

![](https://pic.imgdb.cn/item/606943e98322e6675c1dfcc0.jpg)

![](https://pic.imgdb.cn/item/6069440b8322e6675c1e1a3f.jpg)

## 深入映射文件

### **动态sql**

**sql-if**

```xml
<select id="findByCondition" parameterType="top.sorie.ademo.entity.User"
        resultType="top.sorie.ademo.entity.User">
    select * from user where 1=1
    <if test="id!=0">
        and id=#{id}
    </if>
    <if test="username!=null">
        ans username=#{username}
    </if>
    <if test="password">
        and password=#{password}
    </if>
</select>
```

改进where

```xml
<select id="findByCondition" parameterType="top.sorie.ademo.entity.User"
        resultType="top.sorie.ademo.entity.User">
    select * from user
    <where>
        
    </where>
</select>
```

**foreach**

```java
<select id="findByIds" parameterType="list"
        resultType="top.sorie.ademo.entity.User">
    select * from user
    <where>
        <foreach collection="array" open="id in(" close=")" item="id" separator=",">
            #{id}
        </foreach>
    </where>
</select>
```

```java
public List<User> findByIds(List<Integer> ids);
```

**sq片段抽取**

![](https://pic.imgdb.cn/item/6069475e8322e6675c20d335.jpg)

## 深入核心配置文件

### **typeHandlers标签**

![](https://pic.imgdb.cn/item/606947ce8322e6675c212cf7.jpg)

**自定义开发步骤**

1. 定义转换类继承类BaseTypeHandler<T>。
2. 覆盖4个未实现的方法，其中setNonNullParameter为java程序设置数据到数据库的回调方法，getNullableResult为查询时mysql的字符串类型转换成java的Type类型的方法。
3. 在MyBatis核心配置文件中进行注册。
4. 测试转换是否正确。

```java
public class DateTypeHandler extends BaseTypeHandler<Date> {
    // java转数据库
    @Override
    public void setNonNullParameter(PreparedStatement preparedStatement, int i, Date date, JdbcType jdbcType) throws SQLException {
        long time = date.getTime();
        preparedStatement.setLong(i, time);
    }
    // 数据库转java
    @Override
    public Date getNullableResult(ResultSet resultSet, String s) throws SQLException {
        long data = resultSet.getLong(s);
        Date date = new Date(s);
        return date;
    }

    // 数据库转java
    @Override
    public Date getNullableResult(ResultSet resultSet, int i) throws SQLException {
        long data = resultSet.getLong(i);
        Date date = new Date(data);
        return date;
    }

    // 数据库转java
    @Override
    public Date getNullableResult(CallableStatement callableStatement, int i) throws SQLException {
        long data = callableStatement.getLong(i);
        Date date = new Date(data);
        return date;
    }
}
```

**注册**

```xml
<typeHandlers>
    <typeHandler handler="top.sorie.ademo.handler.DateTypeHandler">		</typeHandler>
</typeHandlers>
```

### plugins

开发步骤：

* 导入包
* 配置插件。
* 测试。

```xml
<dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>3.7.5</version>
        </dependency>
```

配置

```xml
<plugins>
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql"/>
        </plugin>
    </plugins>
```

分页

```java
PageHelper.startPage(1, 3);
// 执行操作 参数：namespace+id
List<User> userList = sqlSession.selectList("userMapper.findAll");
System.out.println(userList);
// 获取分页相关参数
PageInfo<User> pageInfo = new PageInfo<User>(userList);
        System.out.println("当前页：" + pageInfo.getPageNum());
        System.out.println("每页显示条数：" + pageInfo.getPageSize());
        System.out.println("总条数：" + pageInfo.getTotal());
        System.out.println("总页数：" + pageInfo.getPages());
        System.out.println("上一页：" + pageInfo.getPrePage());
        System.out.println("下一页：" + pageInfo.getNextPage());
        System.out.println("是否第一页：" + pageInfo.isIsFirstPage());
        System.out.println("是否最后一页：" + pageInfo.isIsLastPage());

```

## 多表操作

### **1对1**

![](https://pic.imgdb.cn/item/60694d988322e6675c27b2ad.jpg)

![](https://pic.imgdb.cn/item/60694e128322e6675c28ba20.jpg)

或者



![](https://pic.imgdb.cn/item/60694e6d8322e6675c297b93.jpg)

### **1对多**

![](https://pic.imgdb.cn/item/60694ec58322e6675c2a2f8a.jpg)

### **多对多**

类似于1对多的配置

## 注解开发

* @Insert
* @Update
* @Delete
* @Select
* @Result
* @Results
* @One
* @Many

**增删改查**

![](https://pic.imgdb.cn/item/606953248322e6675c2f3c8b.jpg)

```xml
 <mappers>
        <package name="top.sorie.mapper"/>
 </mappers>
```

**复杂查询**

![](https://pic.imgdb.cn/item/606953a78322e6675c2fa99c.jpg)



### **1对1**

![](https://pic.imgdb.cn/item/606954468322e6675c309770.jpg)

![](https://pic.imgdb.cn/item/606954d18322e6675c31b91c.jpg)

### **1对多**

![](https://pic.imgdb.cn/item/606955288322e6675c326a5f.jpg)

### **多对多**

![](https://pic.imgdb.cn/item/606955958322e6675c3314ed.jpg)

## 配置SqlSessionFactory

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.5</version>
</dependency>
```

applicationContext.xml

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="xxx"></property> 
    <property name=configLocation value="classpath:sqlMapConfig-spring.xml"
</bean>
 <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="top.sorie.mapper"></property>
    </bean>
```

![](https://pic.imgdb.cn/item/60695ac08322e6675c37c15b.jpg)

sqlMapConfig-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeHandlers>
        <typeHandler handler="top.sorie.ademo.handler.DateTypeHandler"></typeHandler>
    </typeHandlers>
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <property name="dialect" value="mysql"/>
        </plugin>
    </plugins>

    <mappers>
        <package name="top.sorie.mapper"/>
    </mappers>

</configuration>
```

# Maven基础

​	maven是一个项目管理工具，主要作用是在项目开发阶段对Java项目进行依赖管理和项目构建。

​	依赖管理: 对jar包进行管理，通过导入坐标，相当于将jar包导入到了当前项目中。

​	项目构建：通过maven的一个命令就可以完成项目从清理、编译、测试、报告、打包、部署整个过程。

![](https://pic.imgdb.cn/item/6069a6e88322e6675c870d8b.jpg)

## **仓库类型**

本地仓库

远程仓库

* 中央仓库: http://repo2.maven.org/maven2/
* 私服：自己搭建。
* 其他公共远程仓库：比如 http://repo.maven.apache.org/maven2/

## **常用命令**

* clean: 清理
* compile：编译
* test:测试
* package：打包
* install：安装

## 书写规范

```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.5</version>
</dependency>
```

## 依赖范围

![](https://pic.imgdb.cn/item/6069a91f8322e6675c891b1b.jpg)

没写的时候，依赖范围是compile

## 依赖传递

![](https://pic.imgdb.cn/item/6069aa8b8322e6675c8a9b92.jpg)

## 依赖冲突

​	比如 spring-webmvc-4.2.4依赖spring-beans-4.2.4, spring-aop-5.0.2依赖spring-beans-5.0.2,就造成了依赖冲突。

### 解决冲突

1. 使用maven一同的依赖调解原则：
   * 第一声明优先原则。
   * 路径近者优先原则。
2. 排除依赖。
3. 锁定版本。



#### 第一声明优先原则

​	根据坐标导入顺序，来决定使用哪一个依赖。

​	不推荐。



#### 路径近者优先原则

​	如果把刚才的spring-beans依赖，直接写到pom文件中，那么项目就不会再使用其他依赖传递过来的spring-beans。



####  排除依赖

![](https://pic.imgdb.cn/item/6069ae0c8322e6675c8df064.jpg)

#### 版本锁定

​	企业常用，锁定版本确定依赖jar包版本，锁定后，不考虑依赖的声明顺序或依赖路径，以锁定的版本为准添加到工程中。

​	使用方式：

* 在dependencyManagement标签中锁定依赖的版本。
* 在dependencies标签中声明需要导入的坐标。

![](https://pic.imgdb.cn/item/6069af5a8322e6675c8f22b0.jpg)

![](https://pic.imgdb.cn/item/6069af998322e6675c8f8b8c.jpg)

**classpath:和classpath*:**

​	classpath*可以加载其他jar包的配置文件。

## 分模块构建maven工程

常见拆分方式有两种：

* 按业务拆分：规模大。
* 按层拆分：中小。

![](https://pic.imgdb.cn/item/606b1d5a8322e6675cd32d3e.jpg)



### maven工程继承

​	在maven工程之间也可以继承，子工程继承父工程后，就可以使用父工程中引入的依赖。继承的目的是为了消除重复代码。

​	在被继承的Maven项目的POM部分定义是。

```xml
<groupId>top.sorie</groupId>
<artifactId>ademo</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>ademo</name>
<packaging>pom</packaging>
```

​	父工程打包方式必须为pom，所以我们区分是否父工程，就看打包方式是否为pom。

​	继承Maven项目POM中最关键的部分就是。

```xml
<parent>
    <groupId>top.sorie</groupId>
	<artifactId>ademo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>ademo</name>
</parent>
```

​	如果没有将packing 指定为pom ，那么子模块之间将无法正常的进行依赖传递。

​	我们执行的maven命令的时候将首先对父项目执行，而后当 父项目 的packing 类型为 pom 时，将对所有的子模块执行同样的命令，否则将无法执行同样的命令，那么依赖的传递将无法由maven 编译或者打包命令 得以执行。

### 聚合

​	在maven的pom.xml中，可以使用`<modules>`标签将其他maven工程聚合到一起，聚合的目的是为了进行统一操作。

​	需要打包的时候，只需要在工程中执行一次打包命令，其下所有的聚合工程都会被打包。

```xml
<modules>
    <module>../spring_aop2</module>
</modules>
```



**建议在父工程中只锁定jar包版本。**

### 部署

​	部署web即可。

## Maven私服

### 安装nexus

略

### 将第3方jar安装到maven私服

1. 下载Oracle的jar包。
2. 在maven的setting.xml配置文件中配置第三方的server信息。

```xml
<server>
	<id>thirdparty</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

3. 执行mvn deploy进行安装

![](https://pic.imgdb.cn/item/606dac548322e6675c1fecc9.jpg)

### nexus仓库类型

* hosted：宿主仓库，部署自己的jar到这个类型的仓库，包括Releases和SnapShots两部分，Releases为公司内部发布版本仓库、Snapshots为公司内部测试版本仓库。
* proxy，代理仓库，用于代理远程的公共仓库，如中央仓库。用户连接私服，私服自动去中央仓库下载jar包或插件。
* group，仓库组，用来组合多个hosted/proxy仓库，通常我们配置自己的maven连接仓库组。

### 将项目发布到maven私服

* 配置maven的setting.xml。

```xml
<server>
	<id>releases</id>
    <username>admin</username>
    <password>admin123</password>
</server>
<server>
	<id>snapshots</id>
    <username>admin</username>
    <password>admin123</password>
</server>
```

* 配置项目的pom.xml。

添加到要上传的项目pom中确定上传路径

![](https://pic.imgdb.cn/item/606dafb68322e6675c2391d1.jpg)

* 执行mvn deploy命令。

### 从私服下载jar到本地仓库

* 在maven的setting.xml文件配置下载模板。
* 在maven的setting.xml文件中配置激活下载模板。



![](https://pic.imgdb.cn/item/606db2978322e6675c26b1f8.jpg)

![](https://pic.imgdb.cn/item/606db4398322e6675c29607c.jpg)

### 将jar包安装到本地仓库

![](https://pic.imgdb.cn/item/606db6858322e6675c2c8568.jpg)

### 将jar包

# Git基础

## 和SVN区别

​	SVN是集中版本控制系统，版本集中放在中央服务器。

​	缺点：

* 服务器单点故障。
* 容错性差。



​	分布式仓库：

* 远程仓库。
* 本地仓库。

![](https://pic.imgdb.cn/item/606db85b8322e6675c2e9ce4.jpg)

## 使用流程

![](https://pic.imgdb.cn/item/606db8ee8322e6675c2f4173.jpg)

```cmd
# 设置用户信息
git config --global user.name "sorie"
git config --global user.email ""
# 查看配置信息
git config --list
git config user.name
# 以上配置会保存在~/.gitconfig文件
```

## 获取

* 本地初始化
* 克隆

```cmd
# 本地
git init
# 克隆
git clone
```

## 工作目录、暂存区以及版本库概念

![](https://pic.imgdb.cn/item/606dbc258322e6675c33be18.jpg)

## 文件状态

* untracked

* tracked

  * Unmodified
  * Modified
  * Staged 已暂存

  

```cmd
# 查看文件状态
git status
# 更简洁 
git status -s
# 跟踪
git add xxx.txt
# 将已跟踪文件变为为跟踪
git reset HEAD xxx.txt
# 删除
git rm
```

## 忽略列表

创建.gitignore文件，列出文件模式。

![](https://pic.imgdb.cn/item/606dbe7d8322e6675c36a9eb.jpg)

## 日志记录

```cmd
git log
```

## 远程仓库操作

```cmd
# 查看远程仓库
git remote [-v]
# 添加远程仓库
git remote add origin https://xxx.com/XXX/reop1.git
# 从远程仓库克隆
git clone https://xxx.com/XXX/reop1.git
# 移除无效远程仓库 只影响本地
git remote rm origin
# 抓取 不自动merge
git fetch
gut merge origin/master
# 抓取 自动合并
git pull
```

## 分支操作

```cmd
# 列出所有本地分制
git branch
# 列出所有远程分制
git branch -r
# 列出所有本地和远程分制
git branch -a
# 创建本地分支
git branch b1
# 切换分支
git checkout b1
# 推送到远程分支
git push origin b1
# 在master分支下 合并
git merge b1
# 删除分支 如果要删除的分支中进行了一些开发操作，就不会删除, 坚持删除用-D
git branch -d b1
```

## 标签操作

```cmd
# 列出已有标签
git tag
# 查看tag信息
git show [tag]
# 创建
git tag Tagname
# 提交指定标签
git push [remote] [tag]
# 检出分支， 指向某个tag
git checkout -b [branch] [tag]
# 删除标签
git tag -d [tag]
# 删除远程标签
git push origin :refs/tags/[tag]
```

## TortoiseGit

略

## IDEA.GIT

略

## SSH

https://blog.csdn.net/fanbaodan/article/details/90545224

# Apach Dubbo

## 软件架构演进过程

​	单体->垂直->SOA->微服务。

### 单体架构

![](https://pic.imgdb.cn/item/60704e4e8322e6675c6c4265.jpg)

架构说明：

​	全部功能集中在一个项目内(All in one)。

架构优点：

​	架构简单，前期开发成本低，项目周期短，适合小型项目。

架构缺点：

​	全部功能集成在一个工程中，对于大型项目不易开发、扩展和维护。

​	技术栈受限，只能使用一种语言开发。

​	系统性能扩展只能通过扩展集群节点，成本高。



### 垂直架构

![](https://pic.imgdb.cn/item/6070501f8322e6675c6dfc70.jpg)

架构说明：

​	按业务进行切割，形成小的单体项目。

架构有点：

​	技术栈可扩展。

架构缺点：

​	系统扩展只能通过集群的方式。

​	项目之间功能冗余，数据冗余，耦合性强。

### SOA架构

​	全称Service-Oriented Architecture，即面向服务的架构。

​	站在功能角度，把业务逻辑抽象成可服用的服务，通过服务的编排实现业务的快速再生。目的：把原先固有的业务功能转变为通用的业务服务，实现业务逻辑的快速复用。

![](https://pic.imgdb.cn/item/6070526d8322e6675c7019ec.jpg)

架构说明：

​	将复制功能或模块抽取成组件的形式。对外提供服务，在项目与服务之间使用ESB(企业服务总线)的形式作为通信桥梁。

架构优点：

​	重复功能或模块抽取为服务，提高开发效率。

​	可重用性高。

​	可维护性好。

架构缺点：

​	各系统之间业务不同，很难确定功能或模块是重复的。

​	抽取服务的粒度大。

​	系统和服务之间耦合度高。

### 微服务架构

![](https://pic.imgdb.cn/item/607058578322e6675c75e48f.jpg)

架构说明：

​	将系统服务层完全独立出来，抽取为一个一个的微服务。

​	抽取的粒度更细，遵循单一原则。

​	采取轻量级框架协议传输。

架构优点：

​	服务拆分粒度更细，有利于提高开发效率。

​	可以针对不同服务指定对应的优化方案。

​	适用于互联网，产品迭代周期更短。

架构缺点：

​	粒度太细导致服务太多，维护成本高。

​	分布式系统开发的技术成本高，对团队挑战大。

## Appache Dubbo概述

### 简介

​	高性能Java RPC框架，可以和Spring框架无缝集成。

**什么是RPC**

​	remote precedure call，远程过程调用。不同服务器需要通过网络来表达调用的语义和传达调用的数据。

​	Java中RPC框架，广泛使用的有RMI、Hessian、Dubbo等。

​	http://dubbo.apache.org

​	提供了三大核心能力：面向接口的远程方法调用，智能容错的负载均衡，以及服务自动注册和发现。

### Dubbo架构

![](https://pic.imgdb.cn/item/60706b228322e6675c88b0b8.jpg)

| 节点        | 角色说明                               |
| ----------- | -------------------------------------- |
| `Provider`  | 暴露服务的服务提供方                   |
| `Consumer`  | 调用远程服务的服务消费方               |
| `Registry`  | 服务注册与发现的注册中心               |
| `Monitor`   | 统计服务的调用次数和调用时间的监控中心 |
| `Container` | 服务运行容器                           |

**调用关系说明**

1. 服务容器负责启动，加载，运行服务提供者。
2. 服务提供者在启动时，向注册中心注册自己提供的服务。
3. 服务消费者在启动时，向注册中心订阅自己所需的服务。
4. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
5. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
6. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

## 服务注册中心Zookeeper

### 介绍

​	Appache Hadoop子项目，是一个树形的目录服务，支持变更推送，适合作为Dubbo服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。

![](https://pic.imgdb.cn/item/60706d468322e6675c8af736.jpg)

### 安装

* 安装jdk

* 上传压缩包到linux系统。

* 解压缩压缩包。

  tar -zxvf xxx.tar.gz

* 进入目录，创建data目录。

* 进入conf目录，将zoo_samle.cfg改名为zoo.cfg(改为)

* 修改zoo.cfg文件，修改dataDir=/xxx/data

### 启动停止Zookeeper

```cmd
./zkServer.sh start
./zkServer.sh stop
# 查看状态
./zkServer.sh status
```

## Dubbo快速入门

### 服务提供方开发

* 创建maven工程(打包方式是war)dubbodemo_provider,在pom.xml中导入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.4</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.sorie</groupId>
    <artifactId>dubbo_provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>dubbo_provider</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring.version>5.0.5.RELEASE</spring.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.6.1</version>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <configuration>
                    <port>8081</port>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

web.xml

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd "
         version="3.0">
    <display-name>Archetype Created Web Application</display-name>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:app*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

3. 创建服务接口和实现

```java
package top.sorie.dubbo_provider.service.impl;

import com.alibaba.dubbo.config.annotation.Service;
import top.sorie.dubbo_provider.service.HelloService;

@Service // dubboe注解
public class HelloServiceImpl implements HelloService {
    @Override
    public String sayHello(String name) {
        return "Hello " + name;
    }
}
```

4. 配置

applicationContext-service.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
<!--    每个dubbo应用都需要指定一个唯一名称-->
    <dubbo:application name="dubbo-provider"></dubbo:application>
<!--    指定服务注册中心-->
    <dubbo:registry address="zookeeper://192.168.0.103:2181"/>
<!--    协议和port 默认是20880-->
    <dubbo:protocol name="dubbo" port="20881"/>
<!--    扫描指定包，加入@Service注解的类会被发布为服务-->
    <dubbo:annotation package="top.sorie.dubbo_provider.service.impl"/>
</beans>
```

5. 启动Tomcat

```sh
tomcat7:run
```

### 服务消费方开发

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
-->
<!-- $Id: pom.xml 642118 2008-03-28 08:04:16Z reinhard $ -->
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

  <modelVersion>4.0.0</modelVersion>
  <packaging>war</packaging>

  <name>dubbo_consumer</name>
  <groupId>top.sorie</groupId>
  <artifactId>dubbo_consumer</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <java.version>1.8</java.version>
    <spring.version>5.0.5.RELEASE</spring.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jms</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>dubbo</artifactId>
      <version>2.6.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.6.1</version>
    </dependency>
    <dependency>
      <groupId>com.github.sgroschupf</groupId>
      <artifactId>zkclient</artifactId>
      <version>0.1</version>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <configuration>
          <port>8082</port>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>

  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:applicationContext.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>

</web-app>
```

Controller

```java
package top.sorie.controller;

import com.alibaba.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import top.sorie.service.HelloService;

@Controller
@RequestMapping("hello")
public class HelloController {
    @Reference
    private HelloService helloService;

    @RequestMapping("/hello")
    @ResponseBody
    public String sayHello(String name) {
        return helloService.sayHello(name);
    }
}

```

applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
  <!--    每个dubbo应用都需要指定一个唯一名称-->
  <dubbo:application name="dubbo_consumer"/>
  <!--    指定服务注册中心-->
  <dubbo:registry address="zookeeper://192.168.0.103:2181"/>
  <!--    扫描指定包-->
  <dubbo:annotation package="top.sorie"/>
</beans>
```

运行测试

注意包名一致。

### 测试

### Dubbo-admin

https://blog.csdn.net/fly910905/article/details/86212400

修改配置

```sh
dubbo.registry.address=zookeeper://192.168.0.103:2181
dubbo.admin.root.password=root
dubbo.admin.guest.password=guest
```

http://localhost:8080/dubbo-admin/

![image-20210410124421523](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210410124421523.png)

## 更好

* 把service提出来单独一个工程。
* Dubbo底层基于代理技术，为Service创建代理对象，远程调用时通过此代理对象完成的，是基于Netty框架完成。
* Zookeeper可以支持集群模式。

# Vue2.x(todo)

## 介绍

​	Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与[现代化的工具链](https://cn.vuejs.org/v2/guide/single-file-components.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#libraries--plugins)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 声明式渲染

​	Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统：

```html
<html>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <body>
        <div id="app">
            {{message}}
        </div>
    </body>
    <script>
        var app = new Vue({
            el: "#app",
            data: {
                message: 'Hello Vue!'
            }
        })
    </script>
</html>
```

**例子 悬停效果**

```html
<html>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <body>
        <div id="app2">
            <span v-bind:title="message">
                悬停
            </span>
        </div>
    </body>
    <script>
        var app = new Vue({
            el: "#app2",
            data: {
                message: 'Duang~'
            }
        })
    </script>
</html>
```



`v-bind` attribute 被称为**指令**。指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊 attribute。

### 条件循环

**条件显示**

```html
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>
```

```js
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```

**循环**

```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```

```js
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '整个牛项目' }
    ]
  }
})
```

### 处理输入

`v-on`添加一个事件监听器

```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转消息</button>
</div>
```

```js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```



Vue 还提供了 `v-model` 指令，它能轻松实现表单输入和应用状态之间的双向绑定。

```html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```

```js
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```

### 组件化应用构建

​	组件系统是 Vue 的另一个重要概念，因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。

![](https://pic.imgdb.cn/item/6071386d8322e6675c2f13eb.jpg)

在 Vue 里，一个组件本质上是一个拥有预定义选项的一个 Vue 实例：

```js
// 定义名为 todo-item 的新组件
Vue.component('todo-item', {
  template: '<li>这是个待办项</li>'
})

var app = new Vue(...)
```

比如：

```html
<ol>
  <!-- 创建一个 todo-item 组件的实例 -->
  <todo-item></todo-item>
</ol>
```

```js
Vue.component('todo-item', {
  // todo-item 组件现在接受一个
  // "prop"，类似于一个自定义 attribute。
  // 这个 prop 名为 todo。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```

然后通过`v-bind`将待办事项传到循环的每个组件中。

```html
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供 todo 对象
      todo 对象是变量，即其内容可以是动态的。
      我们也需要为每个组件提供一个“key”，稍后再
      作详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>
```

```js
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})
```

​	大型应用中，有必要将整个应用程序划分为组件，以使开发更易管理。

```html
<div id="app">
  <app-nav></app-nav>
  <app-view>
    <app-sidebar></app-sidebar>
    <app-content></app-content>
  </app-view>
</div>
```

## Vue实例

### 创建实例

```js
var vm = new Vue({
  // 选项
})
```

​	一个 Vue 应用由一个通过 `new Vue` 创建的**根 Vue 实例**，以及可选的嵌套的、可复用的组件树组成。举个例子，一个 todo 应用的组件树可以是这样的：

```
根实例
└─ TodoList
   ├─ TodoItem
   │  ├─ TodoButtonDelete
   │  └─ TodoButtonEdit
   └─ TodoListFooter
      ├─ TodosButtonClear
      └─ TodoListStatistics
```

### 数据与方法

​	当一个 Vue 实例被创建时，它将 `data` 对象中的所有的 property 加入到 Vue 的**响应式系统**中。当这些 property 的值发生改变时，视图将会产生“响应”，即匹配更新为新的值。

```js
// 我们的数据对象
var data = { a: 1 }

// 该对象被加入到一个 Vue 实例中
var vm = new Vue({
  data: data
})

// 获得这个实例上的 property
// 返回源数据中对应的字段
vm.a == data.a // => true

// 设置 property 也会影响到原始数据
vm.a = 2
data.a // => 2

// ……反之亦然
data.a = 3
vm.a // => 3
```

​	当这些数据改变时，视图会进行重渲染。值得注意的是只有当实例被创建时就已经存在于 `data` 中的 property 才是**响应式**的。也就是说如果你添加一个新的 property，比如：

```js
vm.b = 'hi'
```

​	如果你知道你会在晚些时候需要一个 property，但是一开始它为空或不存在，那么你仅需要设置一些初始值。比如：

```js
data: {
  newTodoText: '',
  visitCount: 0,
  hideCompletedTodos: false,
  todos: [],
  error: null
}
```

​	唯一的例外是使用 `Object.freeze()`，这会阻止修改现有的 property，也意味着响应系统无法再*追踪*变化。

```js
var obj = {
  foo: 'bar'
}

Object.freeze(obj)

new Vue({
  el: '#app',
  data: obj
})
```

​	除了数据 property，Vue 实例还暴露了一些有用的实例 property 与方法。它们都有前缀 `$`，以便与用户定义的 property 区分开来。例如：

```js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

​	每个 Vue 实例在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。同时在这个过程中也会运行一些叫做**生命周期钩子**的函数，这给了用户在不同阶段添加自己的代码的机会。

​	比如 [`created`](https://cn.vuejs.org/v2/api/#created) 钩子可以用来在一个实例被创建之后执行代码：

```js
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```

​	也有一些其它的钩子，在实例生命周期的不同阶段被调用，如 [`mounted`](https://cn.vuejs.org/v2/api/#mounted)、[`updated`](https://cn.vuejs.org/v2/api/#updated) 和 [`destroyed`](https://cn.vuejs.org/v2/api/#destroyed)。生命周期钩子的 `this` 上下文指向调用它的 Vue 实例。

> 不要在选项 property 或回调上使用[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`，`this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。



![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

# Spring Boot

## 概述

### 什么是Spring Boot

​	Spring项目中的子工程。

​	一般把Spring Boot称为搭建程序的脚手架，或者是便捷搭建基于Spring工程的脚手架。帮助开发人员快速构建庞大的spring项目，尽可能减少一切xml配置，做到开箱即用，迅速上手。

### 为什么要学习Spring Boot

* 解决复杂的配置问题。
* 解决混乱的依赖管理。

> Spring Boot 简化了Spring的应用开发，只需要run，就能创建一个独立的，生产级别的Spring应用。



​	可以使用Spring Boot创建java应用，并使用java -jar启动它，就能得到一个生产级别的web工程。

### Spring Boot的特点

​	主要特点：

* 创建独立应用，为所有Spring开发者提供一个非常快速、广泛接受的入门体验。
* 直接嵌入应用服务器，如tomcat、jetty、undertow等；不需要去部署war包。
* 提供固定的启动器依赖去简化组件配置，实现开箱即用。通过自己设置参数，即可快速使用。
* 自动地配置Spring和其他有需要的第三方依赖。
* 提供一些大型项目中场景的非功能性特性，如内嵌服务器、安全、指标、健康监测、外部化配置。
* 绝对没有代码生成、也无需XML配置。

## 入门

* 创建工程。
* 添加依赖(起步依赖, spring-boot-starter-web);
* 创建启动类。
* 创建控制器。
* 测试。



​	默认端口8080。

## Java代码方式配置

* 可以通过@Value方式获取配置文件配置项并结合@Bean注册组件到Spring



常用注解

* `@Configuration`:声明一个类作为配置类，代替xml文件。
* `@Bean`: 声明在方法上，将方法的返回值加入Bean容器。
* `@Value`: 属性注入。
* `@PropertySource`:指定外部属性文件。



以连接池为例：

* 添加依赖。
* 创建数据库。
* 创建数据库连接参数的配置文件jdbc.properties。
* 创建配置类。
* 改造处理器类注入数据源并使用。



```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.17</version>
</dependency>
```

jdbc.properties

```properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/springdemo
jdbc.username=root
jdbc.password=root
```

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class JDBCConfig {
    @Value("jdbc.url")
    private String url;

    @Value("jdbc.driverClassName")
    private String driverClassName;

    @Value("jdbc.username")
    private String username;

    @Value("jdbc.password")
    private String password;

    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

## Spring Boot属性注入方式

**目标**：能够使用@ConfigurationProperties实现Spring Boot配置文件配置项读取和应用。

**分析**：

可以使用Spring Boot提供的注解@ConfigurationProperties，该注解可以将Spring Boot配置文件(默认必须为application.properties或application.yml)中配置项读取到一个对象中。

**实现步骤**：

1. 创建配置项类JdbcProperties类。
2. 将jdbc.properties改为application.properties。
3. 将jgbcProperties注入到jgbcConfig。
4. 测试验证

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

```java
package top.sorie.springbootdemo.conf;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "jdbc")
public class JDBCProperties {

    private String url;

    private String driverClassName;

    private String username;

    private String password;

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getDriverClassName() {
        return driverClassName;
    }

    public void setDriverClassName(String driverClassName) {
        this.driverClassName = driverClassName;
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
}
```

```java
package top.sorie.springbootdemo.conf;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@EnableConfigurationProperties(JdbcProperties.class)
public class JDBCConfig {
    @Autowired
    JdbcProperties jdbcProperties;
    @Bean
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(jdbcProperties.getDriverClassName());
        dataSource.setUrl(jdbcProperties.getUrl());
        dataSource.setUsername(jdbcProperties.getUrl());
        dataSource.setPassword(jdbcProperties.getPassword());
        return dataSource;
    }
}
```

优势：

* 松散绑定：

  * 不严格要求属性文件名和变量名一致。支持驼峰，下划线等等转换，甚至支持对象引导。
  * meta-data support: 元数据支持，帮助IDE生成属性提示。

  

### 更优雅的实现

```java
	@Bean
	@ConfigurationProperties(prefix="jdbc") //自动设置到返回值中
    public DataSource dataSource() {
        return new DataSource();
    }
```

## 多个yml

**目标**: 多个yml文件在application.yml中去配置激活。

​	多个yml配置文件，必须是application-***.yml的格式，并且需要在application.yml中激活。

​	如果properties和yml相同名字共存，都会起效，以properties为主。

```yml
# 激活配置文件
spring:
  profiles:
    active:
      jdbc
```

## 自动配置原理(todo)

​	`@SpringBootApplication`注解是一个组合注解/配置注解。

https://blog.csdn.net/qq_28289405/article/details/81302498 

```java

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration // 表示是一个配置类
@EnableAutoConfiguration // 
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}  // 扫描当前包及其子包
)
public @interface SpringBootApplication {
}

```

spring.factories

@ConditionalOnMissingBean

@ConditionalOnBean

如何查看配置项

![](https://pic.imgdb.cn/item/6071bfb78322e6675cbfdf18.jpg)

* 在`META-INF\spring.factories`文件中定义了很多自动配置类；根据在pom.xml添加的起步依赖自动配置组件。
* 通过图上的流程修改application配置文件，改变自动配置的默认参数。

## Lombok

​	@Data会生成get/set/hashCode/equals/toString等方法。

​	@Slf4j:自动提供log变量，实际上用的是slf4j的日志功能。



## SpringMVC端口和访问静态资源

```yml
server:
	port: 8081
```

![](https://pic.imgdb.cn/item/6071c19c8322e6675cc18dee.jpg)

## 拦截器

* 编写拦截器(实现接口HandlerInterceptor)。
* 编写配置类实现WebMvcConfigurer，在该类添加注解@Configuration，类中添加各种组件。
* ![](https://pic.imgdb.cn/item/6071c2428322e6675cc22694.jpg)

## 事务和连接池

* 事务配置。
  * 添加事务相关的起步依赖，mysql相关依赖。
  * 添加业务类UserService使用事务注解@Transactional。
* 数据库连接池Hikari。
  * 只需要配置参数。



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
</dependency>
```

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springdemo
    username: root
    password: cquisse
```

## Mybatis

* 添加起步依赖。
* 配置MyBatis：实体类别名包，日志，映射文件等。
* 配置扫描。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

```java
@SpringBootApplication
@MapperScan("top.sorie.springbootdemo")
public class SpringbootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }

}
```

```xml
mybatis:
  type-aliases-package: top.sorie.springbootdemo.pojo
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 通用Mapper

https://github.com/abel533/Mapper/wiki/3.config

​	实现自动拼接sql语句，所有mapper都不需要编写任何方法，不用编写sql语句。

* 添加起步依赖。
* 改造UserMapper继承`Mapper<T>`;
* 修改启动引导类Application的Mapper扫描注解。
* 修改User视图类添加jpa注解。
* 改造UserService实现功能。

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```

```java
public interface UserMapper extends Mapper<User> {
}
```

```xml
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
//@MapperScan("top.sorie.springbootdemo")
@MapperScan("top.sorie.springbootdemo")
public class SpringbootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }

}
```

```java
@Data
@Table(name="user")
public class User {
    @Id
    // 主键回填
    @KeySql(useGeneratedKeys = true)
    private Integer id;
    private String name;
}
```

```java
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;
    public User getUser(Integer id) {
        if (id == null) {
            return null;
        }
        return userMapper.selectByPrimaryKey(id);
    }
}
```

## Junit

junit5

* 添加起步依赖
* 编写测试类

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class UserServiceTest {

    @Autowired
    private UserService userService;
    @Test
    void getUser() {
        System.out.println(userService.getUser(1));
    }
}
```

## Redis

windows启动redis

```
redis-server.exe redis.windows.conf
```



* 添加起步依赖。
* 配置连接参数。
* 编写测试类应用RedisTemplate操作Redis的5中数据类型(string/hash/list/set/storted set)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```
spring:
  redis:
    host: localhost
    port: 6379
```

```java
@Test
public void test1() {
    redisTemplate.opsForValue().set("name", "sorie");
    redisTemplate.boundValueOps("name").set("sorie");
    System.out.println("name=" + redisTemplate.opsForValue().get("name"));
}
```

## 部署

* 需要添加打包组件将项目的资源、配置、依赖包打到一个jar包中；可以使用`mvn package`;
* 部署: java -jar 包名

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

# RabbitMQ

## 概述

​	消息传输过程中，保存消息的容器。多用于分布式系统之间进行通信。

## 优势和劣势

**优势**

* 应用解耦。
* 异步提速。
* 削峰填谷。

**劣势**

* 系统可用性降低。

  * MQ宕机，会对业务造成影响。如何保证MQ高可用。

* 系统复杂度提高。

  * 如何保证消息没有被重复消费？怎么处理消息丢失的情况？怎么保证消息传递的顺序性。

* 一致性问题。

  * A系统处理完业务，通过MQ给B,C,D三个系统发消息数据。如果B系统、C系统处理成功，D系统处理失败。如何保证消息数据处理的一致性。

  

何时使用MQ

* 生产者不需要从消费者获得反馈。引入消息队列之前的直接调用，其接口返回值应该为空，这才让明明下层的动作还没做，上层当成动作完成，继续往后走。
* 容许短暂的不一致性。
* 收益超过加入MQ、管理MQ这些的成本。



## 对比

![](https://pic.imgdb.cn/item/6072cc998322e6675cb45899.jpg)

## 简介

​	AMOP，Advanced Message Queuing Protocol(高级消息队列协议)。基于此协议的客户端与消息中间件之间可传递消息，并不受客户端/中间件不同产品、不同开发语言等条件的限制。

![](https://pic.imgdb.cn/item/6072cda68322e6675cb55dc8.jpg)



![](https://pic.imgdb.cn/item/6072cdf68322e6675cb5abd0.jpg)

* Broker：接收和分发消息的应用。RabbitMQ Server就是Message Broker。
* Virtual host：处于多租户和安全因素设计的，把AMQP的基础组件划分到一个虚拟的分组中，类似于网络中的namespace概念。多个不同的用户使用同一个RabbitMQ server提供服务时，可以划分出多个vhost，每个用户在自己的vhost创建exchange/queue等。
* Connection: publisher/consumer和broker之间的TCP连接。
* Channel: 如果每一次访问都简历一个Connection，消息量大的时候，开销巨大，效率低。Channel是在connection内部建立的逻辑连接。如果应用程序支持多线程，通常每个thread创建单独的channel进行通讯。AMQP method包含了channel id帮助客户端和message broker识别channel，所以channel之间是隔离的。Channel作为轻量级的Connection极大减少了操作系统建立TCP Connection的开销。
* Exchange：broker的第一站，根据分发规则，匹配查询表中的routing key分发消息到queue中去。常用的类型有: direct(p2p), topic(publish-subscribe)and fanout(multicast)。
* Queue: 存储并等待被消费。
* Binding：exchange和queue之间的虚拟连接，binding中可以包含routing key。Binding信息被保存到exchange的查询表中，用于message的分发依据。

**工作模式**

* 简单模式。
* work queues。
* Publish/Subscirbe发布与订阅模式。
* Routing路由模式。
* Topics主题模式。
* RPC远程调用模式。

![](https://pic.imgdb.cn/item/6072f4d68322e6675cde6a4a.jpg)

## JMS

​	JMS即Java消息服务应用程序接口，是一个Java平台中关于面向消息中间件的API。

## 安装和配置

​	官网：http://www.rabbitmq.com

### 安装依赖环境

```bash
yum install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
```

### 安装Erlang

```
rpm -ivh erlang-23.3.1-1.el8.x86_64.rpm
```

如果出现Failed dependencies说明gblic版本太低，则升级。

### 安装socat和rabbitmq

```
yum install socat
rpm -ivh rabbitmq-server-3.8.14-1.el8.noarch.rpm
```

### 启动/停止

```sh
service rabbitmq-server start
systemctl start rabbitmq-server.service

service rabbitmq-server stop
systemctl stop rabbitmq-server.service

service rabbitmq-server restart
systemctl restart rabbitmq-server.service
```

### 开启管理界面及配置

https://www.imooc.com/article/305858



```sh
# 开启管理界面
rabbitmq-plugins enable rabbitmq_management
# 修改默认配置信息 需要去下载 放在/etc/rabbitmq/rabbitmq.conf
# 比如修改密码，配置等等，例如: loockback_users中的guest只保留guest
loopback_users.guest = true
```

https://github.com/rabbitmq/rabbitmq-server/blob/v3.8.x/deps/rabbit/docs/rabbitmq.conf.example

访问

http://192.168.0.105:15672/

登录失败

https://www.cnblogs.com/FengGeBlog/p/13905541.html

```
rabbitmqctl add_user admin admin
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
rabbitmqctl set_user_tags admin administrator
rabbitmqctl list_users
```

## 快速入门

​	使用简单模式完成消息传递。

![](https://pic.imgdb.cn/item/6076fba48322e6675c1821fd.jpg)

​	步骤：

* 创建工程(生产者，消费者)。
* 分别添加依赖。
* 编写生产者发送消息。
* 编写消费者消费消息。
* 测试

### 生产者

```xml
<dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.7.3</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```java
public class Producer_HelloWorld {
    public static void main(String[] args) throws Throwable{
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置参数
        factory.setHost("192.168.0.105"); // 默认localhost
        factory.setPort(5672);
        factory.setVirtualHost("/hello_world"); // 默认 /
        factory.setUsername("admin"); // 默认值guest
        factory.setPassword("admin"); // 默认值guest
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建Channel
        Channel channel = connection.createChannel();
        // 创建队列Queue
        /***
         * queue: 队列名称
         * durable: 是否持久化
         * exclusive:
         *      * 是否独占，只能有一个消费者。
         *      * 当Connection关闭时，是否删除队列。
         * autoDelete: 是否自动删除。当没有Consumer时，自动删除掉。
         * arguments: 参数。
         */
        // 如果没有队列，则创建，有则不会。
        channel.queueDeclare("hello", true, false, false, null);
        // 发送消息
        /***
         * 1. exchange: 交换机名称, 简单模式使用默认交换机 ""。
         * 2. routingKey: 路由名称
         * 3. props: 配置信息。
         * 4. body: 发送消息数据。
         */
        String body = "hello rabbitmq";
        channel.basicPublish("", "hello", null, body.getBytes());
        // 释放资源
        channel.close();
        connection.close();
    }
}
```

### 消费者

```java
import com.rabbitmq.client.*;

import java.io.IOException;

public class Consumer_HelloWorld {
    public static void main(String[] args) throws Throwable {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置参数
        factory.setHost("192.168.0.105"); // 默认localhost
        factory.setPort(5672);
        factory.setVirtualHost("/hello_world"); // 默认 /
        factory.setUsername("admin"); // 默认值guest
        factory.setPassword("admin"); // 默认值guest
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建Channel
        Channel channel = connection.createChannel();
        // 创建队列Queue
        /***
         * queue: 队列名称
         * durable: 是否持久化
         * exclusive:
         *      * 是否独占，只能有一个消费者。
         *      * 当Connection关闭时，是否删除队列。
         * autoDelete: 是否自动删除。当没有Consumer时，自动删除掉。
         * arguments: 参数。
         */
        // 如果没有队列，则创建，有则不会。
        channel.queueDeclare("hello", true, false, false, null);
        // 接收消息
        /***
         * queue: 队列名称
         * autoAck: 是否自动确认
         * callback: 回调函数。
         */
        Consumer consumer = new DefaultConsumer(channel) {
            /***
             * 回调方法，收到消息会自动执行该方法
             * @param consumerTag 消息标识
             * @param envelope 获取对应信息(比如交换机、路由key..)
             * @param properties 配置信息
             * @param body 消息
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag:" + consumerTag);
                System.out.println("exchange:" + envelope.getExchange());
                System.out.println("routingKey:" + envelope.getRoutingKey());
                System.out.println("properties:" + properties);
                System.out.println("body:" + new String(body));
            }
        };
        channel.basicConsume("hello", true, consumer);
    }
}
```

## 工作模式

![](https://pic.imgdb.cn/item/6078285f8322e6675c17be53.jpg)

​	和简单模式相比，多一个或者一些消费端。多个消费端共同消费同一个队列中的消息。

**应用场景**： 任务过重或者任务较多的情况使用工作队列可以提高任务处理的速度。

​	方式和之前一样，只是多一份代码监听队列。

## Pub/Sub订阅模式

![](https://pic.imgdb.cn/item/60782beb8322e6675c1f20a9.jpg)

* P: 生产者。
* C: 消费者。
* X：Exchange，交换机。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别的队列、递交给所有队列、或是将消息对齐。常见类型：
  * Fanout：广播。交给所有队列。
  * Direct：定向，把消息交给符合指定routing key的队列。
  * Topic：通配符，把消息交给符合routing pattern(路由模式)的队列。
* 交换机只负责转发消息，不具备存储消费的能力，如果没有任何队列与之绑定，消息就会丢失。

**消费者**

```java
public class Producer_Pub_Sub {
    public static void main(String[] args) throws Throwable {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置参数
        factory.setHost("192.168.0.105"); // 默认localhost
        factory.setPort(5672);
        factory.setVirtualHost("/hello_world"); // 默认 /
        factory.setUsername("admin"); // 默认值guest
        factory.setPassword("admin"); // 默认值guest
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建Channel
        Channel channel = connection.createChannel();
        // 创建交换机
        /***
         * exchange： 交换机名称
         * type： 交换机类型
         *    DIRECT 定向
         *    FANOUT 广播
         *    TOPIC  通配符的方式
         *    HEADERS 参数匹配
         * durable： 是否持久化。
         * autoDelete： 自动删除。
         * internal： 内部使用，一般false
         * arguments： 参数
         */
        String exchangeName = "test_fanout";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT, true, false, false, null);
        // 创建队列
        String queue1 = "test_fanout_queue1";
        String queue2 = "test_fanout_queue2";
        channel.queueDeclare(queue1, true, false, false , null);
        channel.queueDeclare(queue2, true, false, false , null);
        // 绑定队列和交换机
        /***
         * 1. queue: 队列名称
         * 2. exchange： 交换机
         * 3. routingkey: 广播类型设置为空字符串
         */
        channel.queueBind(queue1, exchangeName, "");
        channel.queueBind(queue2, exchangeName, "");

        // 发送消息
        String body = "日志信息";
        channel.basicPublish(exchangeName, "", null, body.getBytes());
        // 释放资源
        channel.close();
        connection.close();
    }
}
```

## Routing 路由模式



* 队列与交换机的绑定，不是任意绑定，而是要指定一个RoutingKey。
* 消息队列发送方在想Exchange发送消息时，也必须指定消息的RoutingKey。
* Exchange不再把消息交给每一个绑定的队列，而是根据消息的RoutingKey进行判断，只有队列的RoutingKey和消息的RoutingKey完全一致时，才会接收到消息。

![](https://pic.imgdb.cn/item/60796c768322e6675c4f83a3.jpg)

```java
public class Producer_Routing {
    public static void main(String[] args) throws Throwable {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置参数
        factory.setHost("192.168.0.102"); // 默认localhost
        factory.setPort(5672);
        factory.setVirtualHost("/hello_world"); // 默认 /
        factory.setUsername("admin"); // 默认值guest
        factory.setPassword("admin"); // 默认值guest
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建Channel
        Channel channel = connection.createChannel();
        // 创建交换机
        /***
         * exchange： 交换机名称
         * type： 交换机类型
         *    DIRECT 定向
         *    FANOUT 广播
         *    TOPIC  通配符的方式
         *    HEADERS 参数匹配
         * durable： 是否持久化。
         * autoDelete： 自动删除。
         * internal： 内部使用，一般false
         * arguments： 参数
         */
        String exchangeName = "test_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true, false, false, null);
        // 创建队列
        String queue1 = "test_direct_queue1";
        String queue2 = "test_direct_queue2";
        channel.queueDeclare(queue1, true, false, false , null);
        channel.queueDeclare(queue2, true, false, false , null);
        // 绑定队列和交换机
        /***
         * 1. queue: 队列名称
         * 2. exchange： 交换机
         * 3. routingkey: 广播类型设置为空字符串
         */
        channel.queueBind(queue1, exchangeName, "error");
        channel.queueBind(queue2, exchangeName, "info");
        channel.queueBind(queue2, exchangeName, "error");
        channel.queueBind(queue2, exchangeName, "warning");

        // 发送消息
        String body = "日志信息";
        channel.basicPublish(exchangeName, "info", null, body.getBytes());
        // 释放资源
        channel.close();
        connection.close();
    }
}
```

消费者和Pub/Sub模式一致。

## Topics通配符模式

![](https://pic.imgdb.cn/item/60797c5f8322e6675c7087b3.jpg)

```java
public class Producer_Topic {
    public static void main(String[] args) throws Throwable {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 设置参数
        factory.setHost("192.168.0.102"); // 默认localhost
        factory.setPort(5672);
        factory.setVirtualHost("/hello_world"); // 默认 /
        factory.setUsername("admin"); // 默认值guest
        factory.setPassword("admin"); // 默认值guest
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建Channel
        Channel channel = connection.createChannel();
        // 创建交换机
        /***
         * exchange： 交换机名称
         * type： 交换机类型
         *    DIRECT 定向
         *    FANOUT 广播
         *    TOPIC  通配符的方式
         *    HEADERS 参数匹配
         * durable： 是否持久化。
         * autoDelete： 自动删除。
         * internal： 内部使用，一般false
         * arguments： 参数
         */
        String exchangeName = "test_topic";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC, true, false, false, null);
        // 创建队列
        String queue1 = "test_topic_queue1";
        String queue2 = "test_topic_queue2";
        channel.queueDeclare(queue1, true, false, false , null);
        channel.queueDeclare(queue2, true, false, false , null);
        // 绑定队列和交换机
        /***
         * 1. queue: 队列名称
         * 2. exchange： 交换机
         * 3. routingkey： 系统名称.日志级别
         */
        channel.queueBind(queue1, exchangeName, "#.error");
        channel.queueBind(queue1, exchangeName, "order.*");
        channel.queueBind(queue2, exchangeName, "*.*");

        // 发送消息
        String body = "日志信息";
        channel.basicPublish(exchangeName, "order.info", null, body.getBytes());
        // 释放资源
        channel.close();
        connection.close();
    }
}
```

## Spring整合RabbitMQ

### 生产者

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.sorie</groupId>
    <artifactId>spring-rabbit-producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-rabbit-producer</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.6</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

略

## Spring Boot整合RabbitMQ

### 生产者

* 创建生产者SpringBoot工程。
* 引入依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

* 定义交换机、队列以及绑定关系的配置类。
* 注入RabbitTemplate，调用方法，完成消息发送。

```yml
# 配置基本信息
spring:
  rabbitmq:
    host: 192.168.0.105
    port: 5672
    password: admin
    username: admin
    virtual-host: /hello_world
```

```java
package top.sorie.springbootrabbitproducer.conf;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    public static final String EXCHANGE_NAME = "boot_topic_exchange";
    public static final String QUEUE_NAME = "boot_queue";
    @Bean("bootExchange")
    public Exchange bootExchange() {
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
    }

    @Bean("bootQueue")
    public Queue bootQueue() {
        return QueueBuilder.durable(QUEUE_NAME).build();
    }

    @Bean
    public Binding bindQueueExchange(@Qualifier("bootQueue") Queue queue,
                                     @Qualifier("bootExchange")Exchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();
    }
}
```

### 消费者

```java
@Component
public class RabbitMQListener {

    public static final String QUEUE_NAME = "boot_queue";
    @RabbitListener(queues = QUEUE_NAME)
    public void ListenerQueue(Message message) {
        System.out.println(new String(message.getBody()));
    }
}
```

## 高级特性

* 可靠性投递。
* Consumer ACK。
* 消费端限流。
* TTL。
* 死信队列。
* 延迟队列。
* 日志与监控。
* 消息可靠性分析与追踪。
* 管理

**应用问题**

* 消息可靠性保障。
* 消息幂等性处理。

### 可靠投递

​	两种方式控制消息的可靠性模式。

* confirm确认模式。
* return退回模式。

投递路径：

producer->rabbitmq broker->exchange->queue->consumer

* 消息从producer到exchange则会返回一个confirmCallback。
* 消息从exchange->queue投递失败则会返回一个returnCallback。

#### confirm模式

步骤：

* 确认模式开启。
* 在末班类定义ConfirmCallBack。

```yml
# 配置基本信息
spring:
  rabbitmq:
    host: 192.168.0.105
    port: 5672
    password: admin
    username: admin
    virtual-host: /hello_world
    publisher-confirm-type: SIMPLE

#    /**
#    * Use {@code RabbitTemplate#waitForConfirms()} (or {@code waitForConfirmsOrDie()}
#    * within scoped operations.
#    */
#    SIMPLE,
#
#    /**
#    * Use with {@code CorrelationData} to correlate confirmations with sent
#    * messsages.
#    */ 
#    CORRELATED,
#
#    /**
#    * Publisher confirms are disabled (default).
#    */
#    NONE
```

```java
@Test
void test1() {

    rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
        /***
         *
         * @param correlationData 配置信息
         * @param ack exchange交换机是否接受到消息
         * @param cause 失败原因
         */
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            System.out.println("ConfirmCallback");
        }
    });
    rabbitTemplate.convertAndSend(EXCHANGE_NAME, "boot.haha", "boot mq hello");
}
```

#### return模式

* 开启回退模式。
* 设置ReturnCallBack。
* 设置Exchange处理的模式。
  * 如果没有路由到Queue，则丢弃。
  * 如果没有路由到Queue，返回消息发送方

```java
spring:
  rabbitmq:
    host: 192.168.0.105
    port: 5672
    password: admin
    username: admin
    virtual-host: /hello_world
    publisher-confirm-type: SIMPLE
    publisher-returns: true
```

```java
@Test
void test2() {
    rabbitTemplate.setMandatory(true);
    rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
        @Override
        public void returnedMessage(ReturnedMessage returned) {

        }
    });
    rabbitTemplate.convertAndSend(EXCHANGE_NAME, "boot.haha", "boot mq hello");
}
```

也提供了事务但是性能较差

channel下列方法可以完成事务控制。

* txSelect(), 用于当前channel设置为transaction模式。
* txCommit(), 提交事务。
* txRollback(), 回滚事务。



* NONE值是禁用发布确认模式，是默认值
* CORRELATED值是发布消息成功到交换器后会触发回调方法。
* SIMPLE值经测试有两种效果，其一效果和CORRELATED值一样会触发回调方法，其二在发布消息成功后使用rabbitTemplate调用waitForConfirms或waitForConfirmsOrDie方法等待broker节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie方法如果返回false则会关闭channel，则接下来无法发送消息到broker;
  

### Consumer ACK

​	ack指Acknowledge，确认。表示消费端收到消息后的确认方式。

* 自动确认: acknowledge="none"。 收到就返回，出现异常丢失
* 手动确认：acknowledge="manual"。 业务处理没有问题时，手动返回。
* 根据异常情况确认: acknowledge="auto"。

https://blog.csdn.net/LoveLacie/article/details/105398970

​	手动确认：

​	channel.basicAck();

​	channel.basicNack();

```yml
spring:
  rabbitmq:
    host: 192.168.253.128 #ip
    username: guest
    password: guest
    virtual-host: /
    port: 5672
    listener:
      simple:
        acknowledge-mode: manual #开启
```



```java
@Component
public class MyAckListener {

    /**
     *
     * @param message 队列中的消息;
     * @param channel 当前的消息队列;
     * @param tag 取出来当前消息在队列中的的索引,
     * 用这个@Header(AmqpHeaders.DELIVERY_TAG)注解可以拿到;
     * @throws IOException
     */

    @RabbitListener(queues = "direct_boot_queue")
    public void myAckListener(String message, Channel channel, @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {

        System.out.println(message);
        try {

            // 处理业务逻辑
            /**
             * 无异常就确认消息
             * basicAck(long deliveryTag, boolean multiple)
             * deliveryTag:取出来当前消息在队列中的的索引;
             * multiple:为true的话就是批量确认,如果当前deliveryTag为5,那么就会确认
             * deliveryTag为5及其以下的消息;一般设置为false
             */
            channel.basicAck(tag, false);
        }catch (Exception e){
            /**
             * 有异常就绝收消息
             * basicNack(long deliveryTag, boolean multiple, boolean requeue)
             * requeue:true为将消息重返当前消息队列,还可以重新发送给消费者;
             *         false:将消息丢弃
             */
            channel.basicNack(tag,false,true);
        }
        
    }

}
```



### 并发&限流

https://blog.csdn.net/linsongbin1/article/details/100658415

**限流**

![](https://pic.imgdb.cn/item/607ac3608322e6675cdfd18e.jpg)

* 确保ack机制为手动。
* 配置属性。

```java
@Component
public class SpringBootMsqConsumer {
    @RabbitListener(queues = "spring-boot-direct-queue",concurrency = "5-10",containerFactory = "mqConsumerlistenerContainer")
    public void receive(Message message) {
        System.out.println("receive message:" + new String(message.getBody()));
    }
}
```

```java

@Configuration
public class RabbitMqConfig {

    @Autowired
    private CachingConnectionFactory connectionFactory;

    @Bean(name = "mqConsumerlistenerContainer")
    public SimpleRabbitListenerContainerFactory mqConsumerlistenerContainer(){
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setPrefetchCount(50);
        return factory;
    }
}
```







**并发**

yml配置

```yml
spring:
  rabbitmq:
    listener:
      simple:
        concurrency: 5
        max-concurrency: 10
```

@RabbitListener配置

```java
@RabbitListener(queues = "spring-boot-direct-queue",concurrency = "5-10")
    public void receive(Message message) {
        System.out.println("receive message:" + new String(message.getBody()));
    }

```

### TTL

​	全称 Time To Live(存活时间/过期时间)。

​	达到存活时间后，还没被消费，会自动清除。

​	RabbitMQ可以对消息设置过期时间，也可以对整个(Queue)设置过期时间。

![](https://pic.imgdb.cn/item/607ad74b8322e6675c0f290a.jpg)



**admin界面统一过期时间**

![](https://pic.imgdb.cn/item/607ad7b08322e6675c108913.jpg)

**代码设置**

https://www.jianshu.com/p/341c63cf0459

指定消息设置

```java
@RestController
public class TTLController {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostMapping("/testTTL")
    public String testTTL() {
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setExpiration("20000"); // 设置过期时间，单位：毫秒
        byte[] msgBytes = "测试消息自动过期".getBytes();
        Message message = new Message(msgBytes, messageProperties);
        rabbitTemplate.convertAndSend("TTL_EXCHANGE", "TTL", message);
        return "ok";
    }
}
```

`RabbitMQ`只会对队列头部的消息进行过期淘汰。如果单独给消息设置TTL，先入队列的消息过期时间如果设置比较长，后入队列的设置时间比较短。会造成消息不会及时地过期淘汰，导致消息的堆积。

代码统一设置

```java
@Configuration
public class TTLQueueRabbitConfig {
    @Bean
    public Queue TTLQueue() {
        Map<String, Object> map = new HashMap<>();
        map.put("x-message-ttl", 30000); // 队列中的消息未被消费则30秒后过期
        return new Queue("TTL_QUEUE", true, false, false, map);
    }

    @Bean
    public DirectExchange TTLExchange() {
        return new DirectExchange("TTL_EXCHANGE", true, false);
    }

    @Bean
    public Binding bindingDirect() {
        return BindingBuilder.bind(TTLQueue()).to(TTLExchange()).with("TTL");
    }
}
```

如果同时指定了`Message TTL`和`Queue TTL`，则优先较小的那一个。

### 死信队列

​	Dead Letter Exchange。当消息称为Dead Message后，可以被重新发送到另一个交换机，这个交换机就是DLX。

![](https://pic.imgdb.cn/item/607adcfa8322e6675c1c7066.jpg)

​	成为死信的三种情况：

* 消息队列长度达到限制。
* 消息拒接消息，basicNack/basicReject, 并且不把消息重新放入原目标队列，requeue=false。
* 原队列存在消息过期设置，消息达到超时时间未消费。



绑定方式：

​	给队列设置参数: x-dead-letter-exchange和x-dead-letter-routing-key。

* 声明正常的队列和交换机。
* 声明死信的队列和交换机。



https://blog.csdn.net/u014748504/article/details/108147081

```java
@Bean("deadLetterExchange")
    public Exchange deadLetterExchange() {
        return ExchangeBuilder.directExchange("DL_EXCHANGE").durable(true).build();
    }
@Bean("deadLetterQueue")
public Queue deadLetterQueue() {
    Map<String, Object> args = new HashMap<>(2);
    //       x-dead-letter-exchange    声明  死信交换机
        args.put("x-dead-letter-exchange", "DL_EXCHANGE");
//       x-dead-letter-routing-key    声明 死信路由键
        args.put("x-dead-letter-routing-key", "KEY_R");
        return QueueBuilder.durable("DL_QUEUE").withArguments(args).build();
    }

@Bean("redirectQueue")
public Queue redirectQueue() {
    return QueueBuilder.durable("REDIRECT_QUEUE").build();
}
 
/**
 * 死信路由通过 DL_KEY 绑定键绑定到死信队列上.
 *
 * @return the binding
 */
@Bean
public Binding deadLetterBinding() {
    return new Binding("DL_QUEUE", Binding.DestinationType.QUEUE, "DL_EXCHANGE", "DL_KEY", null);
 
}
 
/**
 * 死信路由通过 KEY_R 绑定键绑定到死信队列上.
 *
 * @return the binding
 */
@Bean
public Binding redirectBinding() {
    return new Binding("REDIRECT_QUEUE", 	Binding.DestinationType.QUEUE, "DL_EXCHANGE", "KEY_R", null);
}
```
```yaml
app:
  rabbitmq:
    # 队列定义
    queue:
      # 正常业务队列
      user: user-queue
      # 死信队列
      user-dead-letter: user-dead-letter-queue
    # 交换机定义
    exchange:
      # 正常业务交换机
      user: user-exchange
      # 死信交换机
      common-dead-letter: common-dead-letter-exchange
```

```java
/**
 * 队列与交换机定义与绑定
 *
 * @author futao
 * @date 2020/4/7.
 */
@Configuration
public class Declare {
 
   /**
     * 用户队列
     *
     * @param userQueueName 用户队列名
     * @return
     */
    @Bean
    public Queue userQueue(@Value("${app.rabbitmq.queue.user}") String userQueueName,
                           @Value("${app.rabbitmq.exchange.common-dead-letter}") String commonDeadLetterExchange) {
        return QueueBuilder
                .durable(userQueueName)
                //声明该队列的死信消息发送到的 交换机 （队列添加了这个参数之后会自动与该交换机绑定，并设置路由键，不需要开发者手动设置)
                .withArgument("x-dead-letter-exchange", commonDeadLetterExchange)
                //声明该队列死信消息在交换机的 路由键
                .withArgument("x-dead-letter-routing-key", "user-dead-letter-routing-key")
                .build();
    }
 
    /**
     * 用户交换机
     *
     * @param userExchangeName 用户交换机名
     * @return
     */
    @Bean
    public Exchange userExchange(@Value("${app.rabbitmq.exchange.user}") String userExchangeName) {
        return ExchangeBuilder
                .topicExchange(userExchangeName)
                .durable(true)
                .build();
    }
 
    /**
     * 用户队列与交换机绑定
     *
     * @param userQueue    用户队列名
     * @param userExchange 用户交换机名
     * @return
     */
    @Bean
    public Binding userBinding(Queue userQueue, Exchange userExchange) {
        return BindingBuilder
                .bind(userQueue)
                .to(userExchange)
                .with("user.*")
                .noargs();
    }
 
    /**
     * 死信交换机
     *
     * @param commonDeadLetterExchange 通用死信交换机名
     * @return
     */
    @Bean
    public Exchange commonDeadLetterExchange(@Value("${app.rabbitmq.exchange.common-dead-letter}") String commonDeadLetterExchange) {
        return ExchangeBuilder
                .topicExchange(commonDeadLetterExchange)
                .durable(true)
                .build();
    }
 
   /**
     * 用户队列的死信消息 路由的队列
     * 用户队列user-queue的死信投递到死信交换机`common-dead-letter-exchange`后再投递到该队列
     * 用这个队列来接收user-queue的死信消息
     *
     * @return
     */
    @Bean
    public Queue userDeadLetterQueue(@Value("${app.rabbitmq.queue.user-dead-letter}") String userDeadLetterQueue) {
        return QueueBuilder
                .durable(userDeadLetterQueue)
                .build();
    }
 
    /**
     * 死信队列绑定死信交换机
     *
     * @param userDeadLetterQueue      user-queue对应的死信队列
     * @param commonDeadLetterExchange 通用死信交换机
     * @return
     */
    @Bean
    public Binding userDeadLetterBinding(Queue userDeadLetterQueue, Exchange commonDeadLetterExchange) {
        return BindingBuilder
                .bind(userDeadLetterQueue)
                .to(commonDeadLetterExchange)
                .with("user-dead-letter-routing-key")
                .noargs();
    }
 
}
```

### 延迟队列

![](https://pic.imgdb.cn/item/607afc518322e6675c68071a.jpg)

​	rabbitmq没有提供延迟队列功能。

​	可以会用TTL+死信队列组合实现延迟队列的效果。

![](https://pic.imgdb.cn/item/607afcd98322e6675c6944e3.jpg)

https://blog.csdn.net/u010096717/article/details/82148681

### 日志与监控

​	默认日志存放位置：/var/log/rabbitmq/rabbit@xxx.log

​	包含MQ的版本号、Erlang版本号、MQ服务节点名称、cookie的hash值、RabbitMQ配置文件地址、内存限制、磁盘限制、默认账户guest的创建以及权限配置等等。

![](https://pic.imgdb.cn/item/607bb0fb8322e6675c6f77f1.jpg)



**rabbitctl管理和监控**

```sh
# 查看队列
rabbitmqctl list_queues
# 查看exchanges
rabbitmqctl list_exchanges
# 查看用户
rabbitmqctl list_users
# 虚拟host
rabbitmqctl list_vhosts
# 查看连接
rabbitmqctl list_connections
# 查看消费者
rabbitmqctl list_consumers
# 查看环境变量
rabbitmqctl environment
# 查看未被确认的队列
rabbitmqctl list_queues name message_unacknowledged
# 查看单个队列的内存使用
rabbitmqctl list_queues name memory
# 查看准备就绪的队列
rabbitmqctl list_queues name message_ready
```

### 消息追踪

![](https://pic.imgdb.cn/item/607bd35e8322e6675cb5b9be.jpg)

**firehose**	

​	firehose的机制是将生产者投递给rabbitmq的教习，rabbitmq投递给消费者的消息按指定格式发送到默认的exchange上。amq.rabbitmq.trace，它是一个topic类型的exchange。发送到这个exchange上的消息队列routingkey为publish.exchangename和deliver.queuename。

​	注意：打开trace会影响写入功能，适当打开后需关闭。

```sh
rabbitmqctl trace_on
rabbitmqctl trace_off
```

**rabbitmq_tracing**

​	实现上和firehose一样，但是多个GUI包装。

```sh
rabbitmq-plugins enable rabbitmq_tracing
```

![](https://pic.imgdb.cn/item/607bd74c8322e6675cbd9219.jpg)

如果想只接受发送的:publish.#

如果想只接受接收的:deliver.#

## 消息补偿

​	![](https://pic.imgdb.cn/item/607bd7e78322e6675cbecacb.jpg)

## 幂等性保障

![](https://pic.imgdb.cn/item/607bd97c8322e6675cc2085e.jpg)

## 集群搭建

### 单机多实例部署

```sh
# 启动第一个节点
RAABITMQ_NODE_PORT=5673 RAABITMQ_NODE_NAME=rabbit1 rabbitmq-server start
# 启动第二个节点 还要指定web管理插件
RAABITMQ_NODE_PORT=5674 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port, 15763}]" RAABITMQ_NODE_NAME=rabbit4 rabbitmq-server start
# 结束命令
rabbitmqctl -n nodeName stop
```

rabbit1操作为主节点

```sh
rabbitmqctl -n rabbit1 stop_app
rabbitmqctl -n rabbit1 reset
rabbitmqctl -n rabbit1 start_app
```

rabbit2操作为从节点

```sh
rabbitmqctl -n rabbit2 stop_app
rabbitmqctl -n rabbit2 reset
rabbitmqctl -n rabbit2 join_cluster rabbit1@'xxx' # 'xxx'是自己的主机名。
rabbitmqctl -n rabbit2 start_app
```

​	主节点才有数据、从节点没有数据。怎么解决？

### 镜像队列

![](https://pic.imgdb.cn/item/607bdc9c8322e6675cc88970.jpg)

可以通过命令创建

```sh
rabbitmqctl set_policy my_ha "^" '{"ha-mode":"all"}'
```

### 负载均衡-HAProxy

​	提供高可用、负载均衡以及TCP、HTTP应用的代理，支持虚拟主机。它实现了一种事件驱动、单一进程模型。此模型支持了非常大的并发连接数。

**安装**

https://my.oschina.net/u/4316924/blog/4051213

下载地址：http://pkgs.fedoraproject.org/repo/pkgs/haproxy/

```sh
# 下载
wget http://pkgs.fedoraproject.org/repo/pkgs/haproxy/haproxy-1.7.9.tar.gz/sha512/d1ed791bc9607dbeabcfc6a1853cf258e28b3a079923b63d3bf97504dd59e64a5f5f44f9da968c23c12b4279e8d45ff3bd39418942ca6f00d9d548c9a0ccfd73/haproxy-1.7.9.tar.gz
# 上传haproxy源码包
tar -zxvf haproxy-1.7.9.tar.gz -C /usr/local
# 编译、安装
cd /usr/local/haproxy-1.7.9
make TARGET=linux31 PREFIX=/usr/local/haproxy
make install PREFIX=/usr/local/haproxy
mkdir /etc/haproxy
# 赋权
groupadd -r -g 149 haproxy
useradd -g haproxy -r -s /sbin/nologin -u 149 haproxy
# 创建haproxy配置文件
vim /etc/haproxy/haproxy.cfg
```

**配置haproxy**

/etc/haproxy/haproxy.cfg

```sh
global
	log 127.0.0.1 local0 info
	maxconn 5120
	chroot /user/local/haproxy
	uid 99
	gid 99
	daemon
	quiet
	nbproc 20
	pidfile /var/run/haproxy.pid
	
defaults
	log global
	
	mode tcp
	
	option tcplog
	option dontlognull
	retries 3
	option redispath
	maxconn 200
	clitimout 60s
	srvtimeout 15s
# fornt-end IP for comsumers ans producers
listen rabbitmq_cluster
	bind 0.0.0.0:5672
	
	mode tcp
	# balance url_param userid
	# balance url_param session_id check_post 64
	# balance hdr(User-Agent)
	# balance hdr(host)
	# balance hdr(Host) user_domain_only
	# balance rdp-cookie
	# balance leastconn
	# balance source //ip
	
	balance roudbin
		server node1 127.0.0.1:5763 check inter 5000 rise 2 fall 2
		server node2 127.0.0.1:5674 check inter 5000 rise 2 fall 2
		
listen stats
	bind 127.16.98.133:8100
	mode http
	option httplog
	stats enable
	stats uri /rabbitmq-stats
	stats refresh 5s		
```

**启动HaProxy负载**

```sh
/usr/local/haproxy/sbin/haproxy -f /etc/haproxy/haproxy.cfg
# 查看haproxy进程状态
ps -ef | grep haproxy
# 访问如下地址对mq节点进行监控
http://172.16.98.133:8100/rabbitmq-stats
```

代码访问5672

![](https://pic.imgdb.cn/item/607be8a88322e6675ce39dac.jpg)

![](https://pic.imgdb.cn/item/607be8e08322e6675ce41098.jpg)

# Spring Cloud

​	API Gateway网关是一个服务器，是系统的唯一入口。为每个客户端提供一个定制的API。

​	网关可以承担其他职责，身份认证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。

**微服务特点**

* 单一职责。
* 微。
* 面向服务。
* 自治：服务间相互独立。
  * 团队独立。
  * 技术独立。
  * 前后端分离。
  * 数据库分离: 没合服务都使用自己的数据源
  * 部署独立。

| 功能     | SOA                  | 微服务                         |
| -------- | -------------------- | ------------------------------ |
| 组件大小 | 大块业务逻辑         | 单独任务或小块业务逻辑         |
| 耦合     | 通常松耦合           | 总是松耦合                     |
| 管理     | 着重中央管理         | 着重分散管理。                 |
| 目标     | 确保应用能够交互操作 | 易维护、易扩展、更清量级的交互 |

微服务是使用小服务或单一业务来开发单个应用的方式或途径。

## 服务调用方式

### RPC和HTTP

* RPC: 远程过程调用, RPC基于Socket，工作在会话层，自定义数据格式，速度快，效率高。
* Http: 工作在应用层，规定了数据传输的格式。缺点是消息封装臃肿，优势是对提供和调用放都没有任何技术限定，自由灵活，更符合微服务理念。



​	**区别**： RPC的机制是根据语言API来定义的，而不是根据网络的应用来定义的。

### RestTemplate

​	一般有以下三种http客户端。

* httpClient。
* okHttp。
* JDK原始URLConnection



spring提供RestTemplate对上述工具进行封装。

```java
@SpringBootApplication
@MapperScan("top.sorie.springbootdemo.mapper")
public class SpringbootDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
    }
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
@RestController
public class HelloController {
    @Autowired
    private DataSource dataSource;
    @Autowired
    private UserService userService;

    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("hello")
    public String hello() {
        String url = "http://localhost:8080/user/1";
        User user = restTemplate.getForObject(url, User.class);
        System.out.println(user);
        return user.toString();
    }

    @GetMapping("/user/{id}")
    public User queryUser(@PathVariable("id") Integer id) {
        User user = userService.getUser(id);
        return user;
    }
}
```

##  初识Spring Cloud

* 后台硬。
* 技术强。
* 群众基础好。
* 使用方便。

## 简介

​	https://spring.io/projects/spring-cloud

​	主要设计的组件有：

* Eureka: 注册中心。
* Zuul，Gateway: 服务网关。
* Ribbon: 负载均衡。
* Feign: 服务调用。
* Hystrix或Resilence4j: 熔断器。

![](https://pic.imgdb.cn/item/607c29f28322e6675c64a050.jpg)

## 版本

​	命名比较特殊，是A到Z为首字母的一些单词(实际上是伦敦地铁站的名字)。

![](https://pic.imgdb.cn/item/607c2b978322e6675c68393e.jpg)

## 创建微服务工程

* 创建微服务父工程、用户护肤工程、服务消费工程

  sorie-cloud、user-service、consumer-demo



* sorie-cloud: 添加springboot父坐标和管理其他组件的依赖。
* user-service: 整合mybatis查询数据库中用户数据, 提供查询用户服务。
* consumer-demo: 利用查询用户服务获取用户数据并输出到浏览器。

父工程依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.sorie</groupId>
    <artifactId>sorie-cloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sorie-cloud</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
        <mapper.starter.version>2.1.5</mapper.starter.version>
        <mysql.version>5.1.46</mysql.version>
    </properties>
    <modules>
        <module>user-service</module>
        <module>consumer-demo</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper-spring-boot-starter</artifactId>
                <version>${mapper.starter.version}</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-config</artifactId>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

```xml
            <dependency>
<!--                还需要继承来自这个的坐标-->
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
```

## 搭建user-service工程

需求:可以访问http://localhost:9091/user/1输出用户数据。

* 添加起步依赖，web，通用mapper。
* 创建启动引导类和配置文件。
* 修改配置文件中给的参数。
* 编写代码UserMappper，UserService，UserController。
* 测试。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.5</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.sorie</groupId>
    <artifactId>sorie-cloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sorie-cloud</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
        <mapper.starter.version>2.1.5</mapper.starter.version>
        <mysql.version>8.0.23</mysql.version>
    </properties>
    <modules>
        <module>user-service</module>
        <module>consumer-demo</module>
    </modules>
    <dependencyManagement>
        <dependencies>
            <dependency>
<!--                还需要继承来自这个的坐标-->
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper-spring-boot-starter</artifactId>
                <version>${mapper.starter.version}</version>
            </dependency>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-config</artifactId>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

 编写代码 略。

## 搭建配置consumer-demo工程

​	编写测试类使用restTemplate访问user-service的路径根据id查询用户。

* 添加起步依赖。
* 创建启动引导类和配置文件。
* 编写测试代码。

```java
@RestController
public class UserController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/hello")
    public void hello() {
        String url = "http://localhost:9091/user/1";
        User user = restTemplate.getForObject(url, User.class);
        System.out.println(user);
    }
}
```

**问题**

* url硬编码，不好维护。
* consumer需要以及user-service的地址，如果出现变更，得不到通知，地址将失效。
* consumer不清楚user-service的状态，服务宕机也不知道。
* user-service只有一台服务，不具备高可用性。
* 即使user-service形成集群，consumer还需要自己实现负载均衡。

总结：

* 服务管理。
  * 如何自动注册和发现。
  * 如何实现状态监管。
  * 如何实现动态路由。
* 服务如何实现负载均衡。
* 服务如何解决容灾问题。

## Eureka注册中心

​	Devops的思想是系统可以通过一组过程、方法或系统；提高应用发布和运维的效率，降低管理成本。

​	Eureka负责管理、记录服务提供者的信息。服务调用者无需自己寻找服务，而是把自己的需求高速Eureka，然后Eureka会把符合你需求的服务告诉你。

​	服务提供方与Eureka之间通过"心跳"机制进行监控，当某个服务提供方出现问题，Eureka自然会把它从服务列表中提出。

### 原理图

![](https://pic.imgdb.cn/item/607db0ed8322e6675c1da59c.jpg)

* Eureka: 注册中心，对外暴露自己的地址。
* 提供者: 启动后向Eureka注册自己的信息。
* 消费者: 向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新。
* 心跳(续约)：提供者定期通过http方式向Eureka刷新自己的状态。

### 搭建eureka-server工程

​	Eureka是服务注册中心，只做服务注册，自动并不提供服务也不消费服务。可以搭建Web工程使用Eureka。

​	搭建步骤:

* 创建工程。
* 添加起步依赖。
* 编写启动启动引导类(添加Eureka的服务注解)和配置文件。
* 修改配置文件(端口, 应用名称)。
* 启用测试。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```

```xml
server:
  port: 10086
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
    # 不注册自己
    register-with-eureka: false
    # 不拉取服务
    fetch-registry: false
```

### 服务注册与发现

​	可以根据服务名称调用。



**注册**

* 服务注册：在服务提供工程user-service上添加Eureka客户端依赖; 自动将服务注册到Eureka服务地址列表。
  * 添加依赖。
  * 改造启动引导类; 添加开启Eureka客户端发现的注解。
  * 修改配置文件: 设置Eureka服务地址。



```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springdemo
    username: root
    password: cquisse
  application:
    name: user-service
server:
  port: 9091

mybatis:
  type-aliases-package: top.sorie.soriecloud.pojo
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

**发现**

服务发现：在服务消费工程consumer-demo上添加Eureka客户端依赖; 可以使用工具类根据服务名称获取对应的服务地址列表。

* 添加依赖。
* 改造启动引导类; 添加开启Eureka客户端发现的注解。
* 修改配置文件: 设置Eureka服务地址。
* 改造处理器类，我可以使用工具DiscoveryCilent根据服务名称获取对应的服务地址列表。

![](https://pic.imgdb.cn/item/607dbb038322e6675c312407.jpg)



```java
@SpringBootApplication
@MapperScan("top.sorie.soriecloud.mapper")
@EnableEurekaClient
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

报错TypeNotPresentExceptionProxy

添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-context</artifactId>
</dependency>
```

调整springboot和 cloud版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>2020.0.1</spring-cloud.version>
        <mapper.starter.version>2.1.5</mapper.starter.version>
        <mysql.version>8.0.23</mysql.version>
    </properties>
```

```java
@Autowired
private DiscoveryClient discoveryClient;
@GetMapping("/hello")
public void hello() {
    List<ServiceInstance> instanceList = discoveryClient.getInstances("user-service");
    ServiceInstance instance = instanceList.get(0);
    String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/user/1";
    User user = restTemplate.getForObject(url, User.class);
    System.out.println(user);
}
```

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/springdemo
    username: root
    password: cquisse
  application:
    name: user-service
server:
  port: 9091

mybatis:
  type-aliases-package: top.sorie.soriecloud.pojo
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

### 高可用配置

​	Eureka Server是一个web应用，可以启动多个实例(配置不同端口), 保证Eureka Server高可用。

​	核心角色：

* 服务注册中心
  * Eureka的服务端应用，提供服务注册和发现功能。
* 服务提供者
  * 提供服务的应用，可以使SpringBoot应用，也可以是其他任意技术实现，只要对外提供的是Rest风格服务即可。
* 服务消费者

**高可用**

​	服务同步。

​	多个Eureka Server之间也会互相注册为服务，当服务提供者注册到Eureka Server集群中的某个节点时，该节点会把服务的信息同步给集群中的每个节点，从而实现数据同步。

![](https://pic.imgdb.cn/item/607ec6308322e6675cc3a36e.jpg)

​	如果有三个，每个都要注册到其他Eureka服务中。

​	配置两个

​	10086

​	10087

```yaml
server:
  port: ${port:10086}
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url:
      defaultZone: ${defaultZone:http://127.0.0.1:10087/eureka}
    register-with-eureka: true
    fetch-registry: true
```

`-Dport=10086 -DdefaultZone=http://127.0.0.1:10087/eureka`

`-Dport=10087 -DdefaultZone=http://127.0.0.1:10086/eureka`

![](https://pic.imgdb.cn/item/607ec7748322e6675cc65582.jpg)

user-service如下配置

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka,http://127.0.0.1:10087/eureka
```

### 客户端与服务端配置

​	配置eureka客户端user-service的注册、续约等配置项，配置eureka客户端consumer-demo的获取服务间隔时间; 了解失效剔除和自我保护。

* Eureka客户端
  * user-service
    * 服务地址使用ip方式。
    * 续约
  * consumer-demo
    * 获取服务地址的频率



​	服务提供者在启动时，回去检查`register-with-eureka=true`是否正确，默认就是true。如果是true，会向EurekaServer发送一个Rest请求，并携带自己的元数据信息，Eureka Server会把这些信息保存到一个双层Map结构中。

* 第一层Map的Key就是服务id，一般是`spring.application.name`属性。
* 第二层是实例id。一般host+serviceId+port，例如，`localhost:user-service:8081`
* 值是服务实例对象。



服务端

* 失效剔除
* 自我保护

**ip注册**

​	默认注册时使用的是主机名或localhost，如果想用ip进行注册，可以在user-service中添加如下配置

```yaml
eureka:
	instance:
		ip-address: 127.0.0.1
		prefer-ip-address: true
```

**服务续约**

​	注册完成后，服务提供者会维持一个心跳，告诉EurekaServer:"我还活着", 称为续约(renew)。

```yaml
eureka:
	instance:
		# 服务时效时间 90秒，默认90秒
		lease-expiration-duration-in-seconds: 90
		# 续约间隔，默认30秒
		lease-renewal-interval-in-seconds: 30
```

**拉取服务列表**

```yaml
eureka:
	client:
		registry-fetch-interval-seconds: 30
```

**失效剔除**

​	当正常进行关闭服务时，会触发一个下线请求。

​	有时候服务因为内存溢出或网络故障的原因不能提供服务。服务注册在启动时会创建一个定时任务，默认每隔一段时间(60秒)将当前清单中超时(默认90秒)没有续约的服务剔除，称为失效剔除。

​	`eureka.server.evication-interval-timer-in-ms`对其进行i修改，单位是毫秒。

**自我**欧虎

​	关停服务会看到一条警告。

![](https://pic.imgdb.cn/item/607ecfe58322e6675cd7cceb.jpg)

​	触发了Eureka的自我保护机制。当服务未按时进行心跳续约时，会统计服务实例最近15分钟的心跳续约比例是否低于了85%。生产环境下，因为网络延迟等原因，心跳失败的比例很有可能超标, 此时就把服务剔除列表不妥当，因为服务可能没有宕机。在这段时间不会剔除任何服务实例，直到网络恢复。生产环境很有效，保证了大多数服务依然可用。不过也有可能获取到失败的服务实例，此时，服务调用者必须做好服务失败的容错。

​	可以关闭自我保护：

```yaml
eureka:
	server:
		enable-self-preservation: false # 默认打开
```

## Ribbon-负载均衡

​	可以通过负载均衡算法，从地址列表中获取地址进行服务调用。

​	Ribbon默认提供了很多负载均衡算法，例如轮询、随机等。也可以自定义负载均衡算法。

### 应用

* 启动多个user-service实例。
* 修改RestTemplate实例化方法，添加负载均衡注解；
* 修改ConsumerController。
* 测试

consumer

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

```java
    @GetMapping("/hello")
    public void hello() {
//        List<ServiceInstance> instanceList = discoveryClient.getInstances("user-service");
//        ServiceInstance instance = instanceList.get(0);
//        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/user/1";
        String url = "http://user-service/user/1";
        User user = restTemplate.getForObject(url, User.class);
        System.out.println(user);
    }
```

如果想改变策略, 在consumer-demo配置文件中添加：

```yaml
user-service:
	ribbon:
		NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## Hystrix-熔断器

​	是一块提供保护机制的组件。

​	是Netflix开源的一个延迟和容错库，用于隔离访问远程服务、第三方库，防止出现级联失败。

https://www.bilibili.com/video/BV1nX4y1K748?from=search&seid=1547387143300458150

https://blog.csdn.net/loushuiyifan/article/details/82702522

https://blog.csdn.net/goldenfish1919/article/details/108192745

https://blog.csdn.net/weixin_35537635/article/details/112521696

https://zhuanlan.zhihu.com/p/142942311

### **雪崩效应**

​	分布式系统环境下，服务间类似依赖非常常见，一个业务调用通常依赖多个基础服务。如下图，对于同步调用，当库存服务不可用时，商品服务请求线程被阻塞，当有大批量请求调用库存服务时，最终可能导致整个商品服务资源耗尽，无法继续对外提供服务。并且这种不可用可能沿请求调用链向上传递，这种现象被称为雪崩效应。

![img](https://static.oschina.net/uploads/space/2018/0122/170502_7fqS_2663573.png)



​	造成雪崩原因归结为以下三点:

* 服务提供者不可用。
* 重试加大流量。
* 服务消费者不可用。

### **解决方案**

* 请求缓存: 支持一个请求和返回结果做缓存处理。
* 请求合并: 将相同的请求进行合并，然后调用批处理接口。
* 服务隔离: 限制调用分布式的资源，某一个调用服务不会影响其他服务的调用。
* 服务熔断: 牺牲局部服务，保全整体系统稳定性的措施。
* 服务降级: 服务熔断以后，客户端调用自己本地方法返回缺省值。



​	最终结果就是：一个服务不可用，导致一系列服务的不可用。





![](https://pic.imgdb.cn/item/607edff58322e6675cfb87db.jpg)

​	3个状态:

* Closed: 关闭状态，所有请求都正常访问。
* Open: 打开，所有请求都会被降级。Hystrix会对请求情况计数，当第一定时间内请求失败百分比达到阈值，则触发熔断。默认失败比例是50%，请求次数最低不低于20次。
* Half Open: 半开状态，不是永久的，断路器打开后会进入休眠时间，默认5s。随后断路器会自动进入半开状态，此时会释放部分请求通过。如果这些请求时健康的，则关闭断路器，否则继续保持打开，再次进行休眠计时。

### 环境准备

* eureka-server: 注册中心
* eureka-server02: 注册中心
* provider-service: 商品服务，提供了商品列表查询接口，根据主键查询商品接口，根据多主键查询商品接口。
* order-service-rest: 订单服务, 基于Ribbon通过RestTemplate调用商品服务。
* order-server-feign: 订单服务，基于Feign通过声明式调用商品服务。

### Jmeter

**安装**

略

通过bin目录下的jmeter.bat运行。

**修改配置**

​	bin目录编辑jmeter.properties文件，修改37行和1085行两处代码(可能有一点差距)。

```properties
language=zh_CN
sampleresult.default.encoding=UTF-8
```

**基本使用**

* 添加线程组
  * 修改线程数以及循环数。
* 添加HTTP请求
  * 配置HTTP请求。
* 添加察看结果树。
  * 查看结果。

### 请求缓存

​	将请求结果存储在缓存中。

**安装redis**

​	Hystrix自带缓存有两个缺点：

* 本地缓存，集群情况无法同步。
* 不支持缓存容器。

**添加依赖**

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>    
```

**配置redis**

![](https://pic.imgdb.cn/item/608132fb563420b6472ad3f6.jpg)

**配置类**

​	添加Redis配置类重写序列化规则。

​	Redis默认使用JDKSerialize，序列化的数据时Json的5倍大小。

```java
package top.sorie.soriecloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.cache.RedisCacheWriter;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
public class RedisConfig {

    // 重写redis序列化
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置String类型key序列化器
        template.setKeySerializer(new StringRedisSerializer());
        // String类型的value序列化器
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        // 设置String类型key序列化器
        template.setHashKeySerializer(new StringRedisSerializer());
        // String类型的value序列化器
        template.setHashKeySerializer(new GenericJackson2JsonRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    // 重写Cache序列化
    @Bean
    public RedisCacheManager redisCacheManager(RedisTemplate redisTemplate) {
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisTemplate.getConnectionFactory());
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                // 设置默认过期时间30min
                .entryTtl(Duration.ofMinutes(30))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisTemplate.getKeySerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisTemplate.getValueSerializer()));
        return new RedisCacheManager(redisCacheWriter, redisCacheConfiguration);
    }
}
```

**开启缓存处理**

```java
@SpringBootApplication
@MapperScan("top.sorie.soriecloud.mapper")
@EnableEurekaClient
@EnableCaching
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

**业务层开启缓存**

![](https://pic.imgdb.cn/item/608135fb563420b6474862a2.jpg)

![](https://pic.imgdb.cn/item/60813611563420b647493ae2.jpg)

### 请求合并

![](https://pic.imgdb.cn/item/608137ad563420b64757881d.jpg)

**请求合并的缺点**

* 需要等待。



步骤:

* 添加依赖

```java
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
```

```java
	@HystrixCommand
    public List<User> getUsers(List<Integer> ids) {
        return new ArrayList<>();
    }

    // 处理请求的方法一定要支持异步，返回值必须是Future<T>
    // 合并请求
    @HystrixCollapser(batchMethod = "getUsers",
            // 请求方式
            scope = com.netflix.hystrix.HystrixCollapser.Scope.GLOBAL,
            collapserProperties = {
                // 间隔多久的请求会合并
                @HystrixProperty(name = "timeDelayInMilliseconds", value = "20"),
                // 批处理累计的最大请求数
                @HystrixProperty(name = "maxRequestInBatch", value = "200")
            }
    )
    public Future<User> getUser(Integer id) {
        System.out.println("getUser");
        return null;
    }
```

开启hystrix

```
@SpringBootApplication
@MapperScan("top.sorie.soriecloud.mapper")
@EnableEurekaClient
@EnableCaching
@EnableHystrix
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

测试

![](https://pic.imgdb.cn/item/60813aa7563420b6476fae09.jpg)

### 服务隔离

**线程池隔离**

​	没有线程池隔离带而项目所有接口都运行在同一个`ThreadPool`中，某一个接口压力过大或出现故障时，会导致资源耗尽从而影响到其他接口的调用而引发服务雪崩效应。

​	每次都开启一个单独线程运行。它的隔离是通过线程池，每个隔离粒度都是个线程池，互相不干扰。线程池隔离方式，等于多了一层的保护措施，可以通过hystrix直接设置超时，超时后直接返回。

**隔离前**

![](https://pic.imgdb.cn/item/60815a0a563420b64752be37.jpg)

![](https://pic.imgdb.cn/item/60815a24563420b6475378be.jpg)

**隔离后**

![](https://pic.imgdb.cn/item/60815a4c563420b6475499c1.jpg)

**信号量隔离**

​	每次调用线程，当前请求通过计数信号进行限制，当信号量大雨了最大请求数`maxConcuurentRequests`时，进行限制，调用`fallback`接口快速返回。信号量的调用是同步的，也就是说，每次调用都得阻塞调用方的线程，知道结果返回。这就导致无法对访问做超时(只能依靠协议超时，无法主动释放)。

![](https://pic.imgdb.cn/item/60815b73563420b6475d61d9.jpg)

#### 线程池隔离案例



```java
@HystrixCommand(groupKey = "userService-pool", // 服务名称，相同名称使用同一个线程池
        commandKey = "getUsers", // 接口名称，默认为方法名
        threadPoolKey = "userService-pool", // 线程池名称，相同名称使用同一个线程池
        commandProperties = {
                // 超时时间，默认 1000ms
                @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",
                        value = "5000"
                )
            },
        threadPoolProperties = {
                // 线程池大小
                @HystrixProperty(name = "coreSize", value = "6"),
                // 队列等待阈值(默认-1)
                @HystrixProperty(name = "maxQueueSize", value = "100"),
                // 线程存活时间, 默认1min
                @HystrixProperty(name = "keepAliveTimeMinutes", value = "2"),
                // 超出队列等待阈值执行拒绝策略
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "100")
        },
        // 托底数据方法
        fallbackMethod = "getUserFallBack"
    )
public List<User> getUsers(List<Integer> ids) {
    List<User> users = new ArrayList<>();
    return new ArrayList<>();
}

public List<User> getUserFallBack(List<Integer> ids) {
    return new ArrayList<>();
}
```

![](https://pic.imgdb.cn/item/6081764e563420b64748c794.jpg)

#### 信号量隔离案例

```java
@HystrixCommand(
        commandProperties = {
                // 超时时间，默认 1000ms
                @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",
                        value = "5000"
                ),
                // 信号量隔离
                @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_STRATEGY,
                    value = "SEMAPHORE"),
                // 信号量最大并发， 调小一些方便模拟高并发
                @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_SEMAPHORE_MAX_CONCURRENT_REQUESTS,
                    value = "6")
        },
        // 托底数据方法
        fallbackMethod = "getUserFallBack"
)
public List<User> semGetUsers(List<Integer> ids) {
    List<User> users = new ArrayList<>();
    return new ArrayList<>();
}
```

![](https://pic.imgdb.cn/item/608178d1563420b64760c85f.jpg)

#### 异同点

| 隔离方式   | 是否支持超时 | 是否支持熔断 | 隔离原理             | 是否是异步调用       | 资源消耗 |
| ---------- | ------------ | ------------ | -------------------- | -------------------- | -------- |
| 线程池隔离 | 支持         | 支持         | 每个服务单独用线程池 | 支持同步或异步       | 大       |
| 信号量隔离 | 不支持       | 支持         | 通过信号量的计数器   | 同步调用，不支持异步 | 小       |

**线程池隔离**

* 请求线程和调用Provider线程不是同一条。
* 支持超时，可以直接返回。
* 支持熔断，当线程池达到最大线程数后，再请求会触发 fallback接口进行熔断。
* 隔离原理: 每个服务单独使用线程池。
* 支持同步或异步。
* 资源消耗大，大量线程的上下文切换、排队、调度等，容易造成机器负载高。
* 无法传递HttpHeader。

**信号量隔离**

* 请求线程和调用Provider线程同一条。
* 不支持超时。
* 支持熔断，信号量达到`maxConcurrentRequests`后。再请求会触发`fallback`进行熔断。
* 隔离原理:  通过信号量的计数器。
* 同步调用， 不支持异步。
* 资源消耗小，只是个计数器。
* 可以传递Http Header。

**总结**

* 请求并发大，耗时长，采用线程隔离策略。可以保证大量的线程可用，不会因为服务原因一直处于阻塞或等待状态，快速失败返回。还有就是对依赖服务的网络请求的调用和访问，会涉及timeout这种问题都使用线程池隔离。
* 请求并发大，耗时小。采用信号量隔离策略。不会占用线程太长时间，减少线程切换的开销，提高缓存服务的效率。还有就是适合访问不是对外依赖的访问，而是对内部一些比较复杂的业务逻辑的访问，像访问这种系统内部的代码，不涉及任何网络请求，做信号量的普通限流就可以了。因为不需要捕获timeout类似的问题，并发量突然太高，稍微耗时一些导致很多线程卡在这里，所以进行一个基本的资源隔离和访问，避免内部复杂的低效率代码，导致大量的线程被卡住。

### 服务熔断

​	指软件系统中，某些原因使得服务出现了过载现象，为防止造成整个系统故障，从而采用的一种保护措施，所以很多地方把熔断称为过载保护。

![](https://pic.imgdb.cn/item/60817dce563420b64792bf07.jpg)

```java
@HystrixCommand(
        commandProperties = {
                // 10s内失败百分比大于阈值，触发熔断，默认20
                @HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_REQUEST_VOLUME_THRESHOLD,
                        value = "10"
                ),
                // 请求错误率大于50%就启动熔断器，然后for循环发起重试
                @HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_ERROR_THRESHOLD_PERCENTAGE,
                        value = "50"),
                // 熔断多少秒去重试 默认5s
                @HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_SLEEP_WINDOW_IN_MILLISECONDS,
                        value = "5")
        },
        // 托底数据方法
        fallbackMethod = "getUserFallBack"
)
public List<User> breakGetUsers(List<Integer> ids) {
    List<User> users = new ArrayList<>();
    return new ArrayList<>();
}
```

### 服务降级

![](https://pic.imgdb.cn/item/60818119d1a9ae528f36de09.jpg)



```java
@HystrixCommand(
        // 托底数据方法
        fallbackMethod = "getUserFallBack"
)
public List<User> degradationGetUsers(List<Integer> ids) {
    List<User> users = new ArrayList<>();
    return new ArrayList<>();
}
```

### Hystrix服务监控

![](https://pic.imgdb.cn/item/60829dd0d1a9ae528f86fe95.jpg)

​	除了实现服务容错之外，Hystrix还提供了近乎实时的监控功能，将服务执行结果和运行指标，通过`Acturator`进行收集，然后访问`/actuator/hystrix.stream`即可看到实时的监控数据。

**添加依赖**

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
```

开启stream端点

![](https://pic.imgdb.cn/item/60829ee2d1a9ae528f8eb852.jpg)

**启动类**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: hystrix.stream
```

#### 监控中心

​	监控中心是Hystrix提供了一套可视化`Hystrix-Dashbord`, 可以非常友好的看到当前环境中服务运行的状态。`Hystrix-Dashbord`是一款针对`Hystrix`实时进行监控的工具。可以看到请求响应时间，请求成功等。



![](https://pic.imgdb.cn/item/6082a00cd1a9ae528f98350c.jpg)

**添加依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
   <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
```

**启动类**

```
@EnableHystrixDashboard
```

然后通过

xxx/hystrix访问

![](https://pic.imgdb.cn/item/6082a10dd1a9ae528fa0e814.jpg)

![](https://pic.imgdb.cn/item/6082a167d1a9ae528fa3e793.jpg)

#### 聚合监控

![](https://pic.imgdb.cn/item/6082a1eed1a9ae528fa86dbc.jpg)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

**配置文件**

![](https://pic.imgdb.cn/item/6082a2dcd1a9ae528fb0398b.jpg)

**启动类**

```
@EnableTurbine
```

## Feign

​	Feign也叫伪装。

​	Fueugb可以把Rest请求进行隐藏，伪装成类似SpringMVC的Controller一样。你不用再自己拼接url，拼接参数等等操作。一切都交给Feign去做。

​	项目主页: https://github.com/OpenFeign/feign

### 快速入门

* 导入起步依赖。
* 开启Feign功能。
* 编写Feign客户端。
* 编写处理器ConsumerFeignController。
* 测试。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.0.0.M2</version>
</dependency>
```

`@EnableFeignClients`

Client的service接口添加

![](https://pic.imgdb.cn/item/6086e050d1a9ae528fa36e3d.jpg)

### Feign负载均衡

​	本身已经继承了Ribbon依赖和自动配置。

​	添加配置就可以。

![](https://pic.imgdb.cn/item/6086e11cd1a9ae528faf24f0.jpg)

### Feign雪崩处理

https://blog.csdn.net/magi1201/article/details/85840281

添加依赖

在client端，@FeigenClient可以设置fallback方法

feign默认集成Hystrix。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.0.0.M2</version>
</dependency>
```

```yaml
feign:
	hystrix:
		enabled: true
```

![](https://pic.imgdb.cn/item/608298eed1a9ae528f637aca.jpg)

![](https://pic.imgdb.cn/item/60829927d1a9ae528f64fe2a.jpg)

启动类添加注解

```java
@EnableFeignClients
```

![](https://pic.imgdb.cn/item/6086e1a7d1a9ae528fb74e69.jpg)

支持factory。

通过匿名内部类。

https://blog.csdn.net/fxq8866/article/details/77113106

![](https://pic.imgdb.cn/item/60829ae8d1a9ae528f71905f.jpg)

### 请求压缩

​	支持对请求和响应进行gzip亚索。

![](https://pic.imgdb.cn/item/6086e220d1a9ae528fbd6b97.jpg)

### 日志级别

​	四种

* NONE: 不记录任何日志，默认值。
* BASIC: 仅记录请求的方法，URL和响应码和执行时间。
* HEADERS: BASIC的基础上，额外记录了请求和响应的头信息。
* FULL: 记录所有请求和响应的明细，包括头信息、请求体、元数据。

![](https://pic.imgdb.cn/item/6086e2fed1a9ae528fc9134c.jpg)

然后在@FeignClient中添加配置类

![](https://pic.imgdb.cn/item/6086e32fd1a9ae528fcb8ac2.jpg)

## Gateway网关

### 简介

* 基于Filter链提供网关基本功能: 安全、监控/埋点、限流等。
* 为微服务提供简单、有效且统一的API路由管理方式。
* 替代Netflix Zuul的一套解决方案。



​	核心是一系列的过滤器，通过过滤器可以将客户端发送的请求路由到对应的微服务。它是加载整个微服务最前沿的防火墙和代理期，隐藏微服务结点IP端口信息，从而加强去安全保护。它本身也是一个微服务，需要注册到Eureka服务注册中心。

​	核心功能是: 过滤和路由。

![](https://pic.imgdb.cn/item/6086e4b7d1a9ae528fdffcc2.jpg)

### 核心概念

* **路由(route)** 路由信息的组成: 由一个ID、一个目的ID、一组断言工厂、一组Filter组成，如果路由断言为真，说明请求URL和路由匹配。
* **断言(Predicate)** Spring Cloud Gateway中的断言函数输入类型是Spring 5.0框架中的ServerWebExchange。Spring Cloud Gateway的断言函数允许开发者去定义匹配来自于Http Request中任何信息比如请求头和参数。
* **过滤器(Filter)** 一个标准的Spring WebFilter。Spring Cloud Gateway的Filter分为两种类型，分别是Gateway Filter和Global Filter。过滤器Filter将会对请求和响应进行修改处理。

### 入门

​	搭建网关服务工程测试网关服务作用。

​	通过网关系统sorie-gateway包含有/user的请求，理由到http://127.0.0.1/user/用户id

* 创建工程。
* 添加起步依赖。
* 编写启动引导类和配置文件。
* 修改配置文件，设置路由信息。
* 启动测试。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.sorie.sorie-cloud</groupId>
    <artifactId>sorie-gateway</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>top.sorie</groupId>
        <artifactId>sorie-cloud</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

```yml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，任意
        - id: user-service-route
          uri: http://127.0.0.1:9091
        # 路由断言
          predicates:
            - Path=/user/**
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

### 面向服务的路由

​	如果将路由服务地址写死，明显是不合理的，在Spring Cloud Gateway中可以通过配置动态路由解决。

​	应该根据服务名称，去Eureka注册中心查找 服务对应的所有实例列表，然后进行动态路由。

​	只需要修改地址。

```yaml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，任意
        - id: user-service-route
        # 代理的服务地址; lb表示从eureka中获取具体服务
          uri: lb://user-service
        # 路由断言
          predicates:
            - Path=/user/**
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

​	默认使用ribbon进行负载均衡。

### 路由前缀处理

​	可以对请求到网关服务的地址添加或去除前缀。

分析：

​	提供服务地址: http://127.0.0.1:9091/user/8

* 添加前缀: 对请求地址添加前缀路径之后再作为代理服务地址。
* 去除前缀: 将请求地址汇总路径去除一些前缀路由之后再作为代理的服务地址。



​	在gateway中可以通过配置路由器的过滤器PrefixPath，实现映射路径中地址的添加。

```yaml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，任意
        - id: user-service-route
        # 代理的服务地址; lb表示从eureka中获取具体服务
          uri: lb://user-service
        # 路由断言
          predicates:
            - Path=/**
          filters:
          # 添加请求路径的前缀
          	- PrefixPath=/user
          # 表示过滤1个路径，2代表了两个，以此类推。
          	- StripPathfix=1
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

```yaml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，任意
        - id: user-service-route
        # 代理的服务地址; lb表示从eureka中获取具体服务
          uri: lb://user-service
        # 路由断言
          predicates:
            - Path=/api/user/**
          filters:
          # 表示过滤1个路径，2代表了两个，以此类推。
          	- StripPathfix=1
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

### 过滤器

#### 简介

​	作为网关的其中一个重要功能，就是实现请求的鉴权。这个动作往往是通过网关提供的过滤器来实现的。前面的`路由前缀`章节中的功能也是使用过滤器来实现的。

* Gateway自带过滤器有几十个，常见的如下：

| 过滤器名称           | 说明                         |
| -------------------- | ---------------------------- |
| AddRequestHeader     | 对匹配上的请求加上Header     |
| AddRequestParameters | 对匹配上的请求路由添加参数   |
| AddResponseHeader    | 对从网关返回的响应添加Header |
| StripFix             | 对匹配上的请求路径去除前缀   |

![](https://pic.imgdb.cn/item/60878015d1a9ae528f10da04.jpg)

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

* 过滤器类型:Gateway实现方式上, 有两种过滤器:

  * 局部过滤器: 通过`spring.cloud.gateway.routes.filters`配置在具体路由下，只作用在当前路由上; 自带过滤器都可以配置或自定义按照自带过滤器的方式。如果配置`spring.cloud.gateway.default-filters`上会对所有路由生效也算是全局的过滤器; 但是这些过滤器的实现都是要实现GatewayFilterFactory。
  * 全局过滤器: 不需要在配置文件中使用，作用在所有路由上; 实现`GlobalFilter`接口即可。

  

#### 执行声明周期

​	类似于Spring MVC拦截器: pre和post，代表请求执行前和执行后。

![image-20210427113103369](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210427113103369.png)

#### 使用场景

* 请求鉴权: 一般`GatewayFilterChain`执行filter前，如果发现没有访问权限，直接就返回空。
* 异常处理: 一般`GatewayFilterChain`执行方法后，记录异常并返回。
* 服务调用时长统计: `GatewayFilterChain`执行Filter方法前后根据时间统计。

#### 自定义局部过滤器

​	按默认过滤器编写配置一个自定义局部过滤器，该过滤器可以通过配置文件中的参数名称获取请求的参数值。

* 配置过滤器。
* 编写过滤器。
* 测试。

```yaml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，任意
        - id: user-service-route
        # 代理的服务地址; lb表示从eureka中获取具体服务
          uri: lb://user-service
        # 路由断言
          predicates:
            - Path=/api/user/**
          filters:
          # 表示过滤1个路径，2代表了两个，以此类推。
          	- StripPathfix=1
          	- MyParam=name
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

![](https://pic.imgdb.cn/item/608788c3d1a9ae528f4f14c9.jpg)

```java
public class MyParamGatewayFilterFactory extends AbstractGatewayFilterFactory<MyParamGatewayFilterFactory.Config> {
    static final String PARAM_NAME = "param";
    public MyParamGatewayFilterFactory() {
        super(Config.class);
    }
    public List<String> shortcutFieldOrder() {
        return Arrays.asList(PARAM_NAME);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return ((exchange, chain) -> {
            // 获取请求参数中对应参数名的参数值
            ServerHttpRequest request = exchange.getRequest();
            if (request.getQueryParams().containsKey(config.param)) {
                request.getQueryParams().get(config.param).forEach(System.out::println);
            }
            return chain.filter(exchange);
        });
    }

    public static class Config {
        private String param;

        public String getParam() {
            return param;
        }

        public void setParam(String param) {
            this.param = param;
        }
    }

}
```

#### 已定义全局过滤器

​	定义一个全局过滤器检查请求中是否携带有token参数。

* 编写全局过滤器。
* 测试。



```java
package top.sorie.soriecloud.filter;

import org.apache.commons.lang.StringUtils;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class MyGlobalTokenFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("全局过滤器");
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if (StringUtils.isBlank(token)) {
            // 设置未授权
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        // 值越小越先执行
        return 1;
    }
}
```

#### 其他配置说明

​	网关的负载均衡和熔断参数配置。

![](https://pic.imgdb.cn/item/60878ce1d1a9ae528f738af8.jpg)

#### 跨域配置

![](https://pic.imgdb.cn/item/60878d5bd1a9ae528f77ae56.jpg)



### 高可用

​	启动多个Gateway服务，注册到Eureka，形成集群。服务内部访问，就通过Eureka自动均衡。

​	更多的是网布访问，无法通过Eureka进行负载均衡，那该怎么办?

​	通过其他服务网关对Gateway进行代理，比如与Nginx。

### 和Feign的区别

* 作为整个应用的流量入口，接收所有的请求，如PC、移动端等，并且将不同的请求转发至不同的微服务模块，其作用可视为nginx；大部分情况下用作权限鉴定、服务端流量控制等。
* Feign则是将当前微服务的部分服务接口暴露出来，并且主要用于各个微服务之间的服务调用。

## Spring Cloud Config分布式配置中心

​	分布式系统中，服务数量非常多，配置文件分散在不同的微服务项目中，管理不方便。为了方便配置文件集中管理，需要分布式配置中心组建。在Spring Cloud中， 提供了Spring Cloud Config。 它支持配置文件放在配置服务的本地，也支持放在远程Git仓库。

​	架构如下：

![](https://pic.imgdb.cn/item/60879048d1a9ae528f8ed46f.jpg)

### 搭建配置中心微服务

​	公开git仓库，配置配置中心微服务config-server

* 创建git仓库。
* 搭建配置中心config-server。

**创建配置文件**

​	{application}-{profile}.yml或properties。

![](https://pic.imgdb.cn/item/608792aed1a9ae528fa3a48c.jpg)

本地config-server

```yml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>top.sorie.sorie-cloud</groupId>
    <artifactId>config-server</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>top.sorie</groupId>
        <artifactId>sorie-cloud</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(SorieGatewayApplication.class, args);
    }
}
```

![](https://pic.imgdb.cn/item/608792e0d1a9ae528fa56c9e.jpg)

![](https://pic.imgdb.cn/item/60879319d1a9ae528fa78568.jpg)

### 获取配置中心配置

​	改造user-service, 配置文件信息不再由微服务项目提供，而是从配置中心获取。

* 添加起步依赖。
* 修改配置文件。
* 测试验证。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>3.0.3</version>
</dependency>
```

添加配置文件bootstap.yml(比较固定的)

![](https://pic.imgdb.cn/item/608795add1a9ae528fc093b7.jpg)

## Spring Cloud Bus

### 简介

​	修改配置项时，没有及时更新。

​	用轻量的消息代理将分布式的节点连接起来，用于广播配置文件的更改或服务监控管理。也就是消息总线可以为微服务做监控，也可以实现应用程序之间项目通信。Spring Cloud Bug可选的消息代理有RabbitMQ和Kafka。

​	![](https://pic.imgdb.cn/item/60879768d1a9ae528fd200f9.jpg)

​	启动RabbitMQ通过修改码云中的配置文件后发送Post请求实现及时更新用户微服务中的配置项。

* 启动RabbitMQ。
* 修改配置中心config-server。
* 修改服务提供工程user-service。
* 测试。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
    <version>3.1.1</version>
</dependency>
```

![](https://pic.imgdb.cn/item/60879928d1a9ae528fe50ff4.jpg)

**改造用户服务**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-bus</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
    <version>3.1.1</version>
</dependency>
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

![](https://pic.imgdb.cn/item/60879993d1a9ae528fe9e0ef.jpg)

在使用信息的bean中添加刷新配置的注解。

![](https://pic.imgdb.cn/item/608799a2d1a9ae528fea89b9.jpg)

发送post请求至配置中心的以下地址

/actuator/bus-refresh

![](https://pic.imgdb.cn/item/60879ad5d1a9ae528ff880fd.jpg)

![](https://pic.imgdb.cn/item/60879bb2d1a9ae528f029e18.jpg)

# ES(todo)

# Node

​	node.js是一个介于Chrome JavaScript运行时建立的一个平台。

​	是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行JavaScript的速度非常快，性能非常好。

## 安装

​	略

## 模块化编程

​	编写模块文件，使用require引入模块后使用node.js执行

demo1.js

```js
var a = 1;
var b = 2;
console.log(a + b);
```

运行

```
node demo1.js
```

​	每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的。对其他文件不可见。

```js
exports.add=function(a, b) {
	return a + b;
}
```

​	每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性(module,exports)是对外的接口。加载某个模块，实际上是加载该模块的module.exports属性。

```js
var demo = require('./demo3_1');
console.log(demo.add(400, 600));
```

##  创建Nodejs web服务器

​	引入http模块监听8888端口实现输出字符

demo4.js

```js
// 引入http
var http = require("http");
// 创建并监听web服务器
http.createServer(function (req, resp) {
    // 发送http响应头部
    resp.writeHead(200, {"Conten-Type": "text"});
    // 发送响应数据
    resp.end("Hello World\n");
}).listen(8888);
```

写多条

```js
// 引入http
var http = require("http");
// 创建并监听web服务器
http.createServer(function (req, resp) {
    // 发送http响应头部
    resp.writeHead(200, {"Conten-Type": "text"});
    // 发送响应数据
    for (int i = 0; i < 10; i++) {
        resp.write("Hello World\n");
    }
}).listen(8888);
```

## 处理Nodejs web请求参数

​	引入http和url模块创建web容器并使用url解析请求路径中的参数并输出。

* 创建Web服务器。
* 引入url模块。
* 利用url解析请求地址中的请求和值。

```js
// 引入http
var http = require("http");
var url = require("url");
// 创建并监听web服务器
http.createServer(function (req, resp) {
    // 发送http响应头部
    resp.writeHead(200, {"Conten-Type": "text"});
    // 第二个参数如果为true代表解析到一个对象中
    var params = url.parse(req.url, true),query;
    // 发送响应数据
    for (var key in params) {
        resp.write(key + " = " + params[key]);
        resp.write("\n");
    }
}).listen(8888);
```

## NPM

​	是node包管理和分发工具，其实我们可以把npm理解为前端的maven。

### 命令

```sh
# 初始化工程
npm init
# 本地安装(本项目安装)
npm install express [--s] [-g]
# 根据package.json安装
npm install
```

​	package-lock.json是当node-mdoules或package.json发生变化时自动生成的文件。主要是确定当前安装包的依赖，忽略更新。

![](https://pic.imgdb.cn/item/6087a426d1a9ae528f5f69a2.jpg)

### 切换镜像源略

## webpack

**安装**

​	是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。

![](https://pic.imgdb.cn/item/6087a5d0d1a9ae528f71208f.jpg)



```sh
npm install webpack -g
npm install webpack-cli -g
```

### 打包js

​	创建两个js文件，使用webpack命令打包到dist/bundle.js文件中并测试。

* 创建两个js文件。
* 创建入口文件main.js。
* 创建webpack配置文件。
* 运行webpack命令。
* 创建index.html页面进行测试。

webpack.config.js

```js
var path = require("path");
module.exports = {
    // 入口文件
    entry: "./src/main.js",
    output: {
        // 路径
        path: path.resolve(__direname, "./dist"),
        filename: "bundle.js"
    }
}
```

打包命令

```sh
webpack
```

### 打包css

​	安装style-loader css-loader组件，创建并使用css文件，使用webpack命令打包js文件到`dist/bundle.js`文件并测试。

* 安装转换css的组件。
* 修改配置文件。
* 创建css文件。
* 修改入口文件，加载css文件。
* 打包测试。

```sh
cnpm install style-loader css-loader --save-dev
```

​	Webpack本身只能处理JS模块，如果要处理其他类型的文件，需要使用loader进行转换。

​	Loader可以理解为是模块和资源的转换器，它本身是一个函数，接受源文件作为参数，返回转换的结果。这样我们就可以通过require来加载任何的模块或文件。比如CoffeeScript、JSX、LESS或图片。

![](https://pic.imgdb.cn/item/6087a8cdd1a9ae528f90478e.jpg)

![](https://pic.imgdb.cn/item/6087a8eed1a9ae528f918bb6.jpg)

![](https://pic.imgdb.cn/item/6087a90dd1a9ae528f92bfda.jpg)

# ES6(todo)

todo


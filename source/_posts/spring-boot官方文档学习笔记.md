---

title: spring boot官方文档学习笔记
date: 2020-10-21 21:51:43
tags: [Spring, Spring Boot]
---

本笔记基于官方文档2.3.4

https://spring.io/projects/spring-boot#learn

# Spring Boot起步(Getting Started)

https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started

本章包含内容：

* 连同安装说明(installtion instructions)对Spring Boot进行介绍
* 引导创建第一个Spring Boot应用，并介绍一些核心规则。

## 1.介绍Spring Boot

​	Spring Boot可以帮助我们创建独立的(stande-alone), 生产级别的Spring应用。大多数Spring Boot应用只需要极少的配置。

​	你可以通过`java -jar`或传统的war包部署。官方也提供了运行"Spring script"的命令行工具。

​	Spring Boot的主要目标有：

* 从根本上为所有Spring开发提供更快, 更广的入门体验。
* 开箱即用, 如果需求与默认值有所出入, 也可以很快完成自己的配置。
* 提供了一系列大型项目的通用非功能性功能(例如嵌入式服务器，安全性, 指标, 健康检查, 以及外部配置)。
* 绝对没有代码生成, 也没有对XML配置的需求。

## 2. 系统需求

​	Spring Boot 2.3.4需要Java8-Java14。也需要Spring Framwork 5.2.9.RELEASE及以上版本。

​	构建工具如下：

| Build Tool | Version                                                      |
| :--------- | :----------------------------------------------------------- |
| Maven      | 3.3+                                                         |
| Gradle     | 6 (6.3 or later). 5.6.x is also supported but in a deprecated form |

### 2.1 Servlet容器

| Name         | Servlet Version |
| :----------- | :-------------- |
| Tomcat 9.0   | 4.0             |
| Jetty 9.4    | 3.1             |
| Undertow 2.0 | 4.0             |

​	也可以在任何适配Servlet 3.1+的容器中部署Spring Boot应用。

## 3. 安装Spring Boot

​	检查Java SDK是否是1.8以上

```
$ java -version
```

​	我们可以从[Spring Boot CLI](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started-installing-the-cli Spring Boot CLI, "Spring Boot CLI")开始, 或者阅读"经典"安装说明。

### 通过Idea创建Spring Boot应用

​	这里我们选择Idea创建Spring Boot应用。更多创建的方式请参照官网。

在idea中创建Spring Boot项目。

![](https://pic.imgdb.cn/item/5f899ee81cd1bbb86bb36d57.jpg)

创建一个叫MySpringStudy的项目。

![](https://pic.imgdb.cn/item/5f899f421cd1bbb86bb3a6f6.jpg)

![](https://pic.imgdb.cn/item/5f899f581cd1bbb86bb3b6b2.jpg)

项目结构如下：

![](https://pic.imgdb.cn/item/5f899f971cd1bbb86bb3dd4d.jpg)

删除src后，再按类似的流程创建ch1helloworld 模块。

### 4.1 pom.xml

打开POM文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>top.sorie</groupId>
    <artifactId>ch1helloworld</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>ch1helloworld</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
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

可以运行该应用测试一下。

### 4.2  添加ClassPath Dependencies

​	Spring Boot提供了一系列"Starter", 可以将这些Jar包添加到我们的项目中。我们刚才在`pom`的'parent'部分设置了`spring-boot-starter-parent`。它是一个特殊的starter, 提供了许多有用的默认Maven配置。还提供了一个`dependency-management`模块([详情](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-dependency-management)),  所以可以忽略dependency的 version。

​	因为我们要部署一个Web应用, 所以我们要添加`spring-boot-starter-web`依赖。在此之前,我们可以使用命令查看我们当前有哪些依赖:

```cmd
$mvn dependency:tree
[INFO] top.sorie:ch1helloworld:jar:0.0.1-SNAPSHOT
[INFO] +- org.springframework.boot:spring-boot-starter:jar:2.3.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot:jar:2.3.4.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-context:jar:5.2.9.RELEASE:compile
[INFO] |  |     +- org.springframework:spring-aop:jar:5.2.9.RELEASE:compile
[INFO] |  |     +- org.springframework:spring-beans:jar:5.2.9.RELEASE:compile
[INFO] |  |     \- org.springframework:spring-expression:jar:5.2.9.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.3.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.3.4.RELEASE:compile
[INFO] |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  \- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.13.3:compile
[INFO] |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.13.3:compile
[INFO] |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.30:compile
[INFO] |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO] |  +- org.springframework:spring-core:jar:5.2.9.RELEASE:compile
[INFO] |  |  \- org.springframework:spring-jcl:jar:5.2.9.RELEASE:compile
[INFO] |  \- org.yaml:snakeyaml:jar:1.26:compile
[INFO] \- org.springframework.boot:spring-boot-starter-test:jar:2.3.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test:jar:2.3.4.RELEASE:test
[INFO]    +- org.springframework.boot:spring-boot-test-autoconfigure:jar:2.3.4.RELEASE:test
[INFO]    +- com.jayway.jsonpath:json-path:jar:2.4.0:test
[INFO]    |  +- net.minidev:json-smart:jar:2.3:test
[INFO]    |  |  \- net.minidev:accessors-smart:jar:1.2:test
[INFO]    |  |     \- org.ow2.asm:asm:jar:5.0.4:test
[INFO]    |  \- org.slf4j:slf4j-api:jar:1.7.30:compile
[INFO]    +- jakarta.xml.bind:jakarta.xml.bind-api:jar:2.3.3:test
[INFO]    |  \- jakarta.activation:jakarta.activation-api:jar:1.2.2:test
[INFO]    +- org.assertj:assertj-core:jar:3.16.1:test
[INFO]    +- org.hamcrest:hamcrest:jar:2.2:test
[INFO]    +- org.junit.jupiter:junit-jupiter:jar:5.6.2:test
[INFO]    |  +- org.junit.jupiter:junit-jupiter-api:jar:5.6.2:test
[INFO]    |  |  +- org.apiguardian:apiguardian-api:jar:1.1.0:test
[INFO]    |  |  +- org.opentest4j:opentest4j:jar:1.2.0:test
[INFO]    |  |  \- org.junit.platform:junit-platform-commons:jar:1.6.2:test
[INFO]    |  +- org.junit.jupiter:junit-jupiter-params:jar:5.6.2:test
[INFO]    |  \- org.junit.jupiter:junit-jupiter-engine:jar:5.6.2:test
[INFO]    |     \- org.junit.platform:junit-platform-engine:jar:1.6.2:test
[INFO]    +- org.mockito:mockito-core:jar:3.3.3:test
[INFO]    |  +- net.bytebuddy:byte-buddy:jar:1.10.14:test
[INFO]    |  +- net.bytebuddy:byte-buddy-agent:jar:1.10.14:test
[INFO]    |  \- org.objenesis:objenesis:jar:2.6:test
[INFO]    +- org.mockito:mockito-junit-jupiter:jar:3.3.3:test
[INFO]    +- org.skyscreamer:jsonassert:jar:1.5.0:test
[INFO]    |  \- com.vaadin.external.google:android-json:jar:0.0.20131108.vaadin1:test
[INFO]    +- org.springframework:spring-test:jar:5.2.9.RELEASE:test
[INFO]    \- org.xmlunit:xmlunit-core:jar:2.7.0:test
[INFO] --------------------------------------------------------------
```

Idea中可以在右方查看

![](https://pic.imgdb.cn/item/5f904df51cd1bbb86b8a697b.jpg)

`spring-boot-starter-parent`本身没有提供任何依赖。现在我们添加一下`spring-boot-starter-web`

```
   <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
```

​	再运行`mvn dependency:tree`, 我们可以看到一系列其他的依赖, 包含Tomcat和Spring Boot本身。

### 4.3 编写代码

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ImportResource;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import top.sorie.ch1helloworld.config.TestConfigBean;

@EnableAutoConfiguration
@RestController
public class Ch1helloworldApplication {
    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Ch1helloworldApplication.class, args);
    }
}
```

#### 4.3.1 @RestController和@RequestMapping注解

​	类上的第一个注解是@RestController。它是一个模板化的注解, 提醒我们这个类扮演一个指定的角色。在这里, 我们的类是一个Web @Controller， 所以Spring认为它可以处理Web请求。

​	`@RequestMapping`提供路由信息, 告诉Spring 任意HTTP request(路径是`/`)可以映射到`home`方法上。然后`@RequestController`告诉Spring 将返回值的String直接返回给调用者。

> 这两个注解都是Spring MVC的注解。可以查看[MVC section](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) 查看更多细节。

#### 4.3.2@EnableAutoConfiguration注解

​	这个注解告诉Sping Boot "猜猜"你想怎么配置Spring(基于你添加的依赖)。因为`spring-boot-starter-web`添加了Tomcat和SpringMVC, 自动配置假设你正在部署一个web应用,并创建相应的Spring应用。

> **Starters和自动配置**
>
> 自动配置被设计成可以和"Starters"相互工作得很好, 但是两者不是直接绑定。你可以自由选择jar包依赖。Spring Boot仍然可以为你的应用做好自动配置。

### 4.3.3 main方法

​	我们的main方法调用了`SpringApplication.run()`方法。`SpringApplication`引导我们的应用启动Spring, 并且启动自动配置的Tomcat服务。我们将`Ch1helloworldApplication.class`作为一个参数传进run方法, 告诉`SpringApplication`它是初始的Spring component。`args`数组传递了命令行参数。

## 4.4 运行示例

```
mvn spring-boot:run
```

```cmd
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.3.4.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

​	我们打开浏览器访问`localhost:8080`,可以在页码看到输出

```
Hello World!
```

​	如果想优雅地结束应用, 按`ctrl+c`

## 4.5 创建可运行的jar包

​	为了创建可运行的jar包，需要加入`spring-boot-maven-plugin`在我们的pom.xml中。

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

> `spring-boot-starter-parent`包含`<executions>`配置。如果不使用parent POM, 需要自己声明配置。 [plugin documentation](https://docs.spring.io/spring-boot/docs/2.3.4.RELEASE/maven-plugin/reference/html/#getting-started)

保存pom后运行`mvn package`。

```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 0, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO]
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ ch1helloworld ---
[INFO] Building jar: F:\studySrc\myspringstudy\ch1helloworld\target\ch1helloworld-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.3.4.RELEASE:repackage (repackage) @ ch1helloworld ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13.358 s
[INFO] Finished at: 2020-10-22T23:21:26+08:00
[INFO] Final Memory: 27M/455M
[INFO] ------------------------------------------------------------------------
```

可以在`target`文件夹中看到ch1helloworld-0.0.1-SNAPSHOT.jar。大概10MB，我的是16MB。

也可以看到ch1helloworld-0.0.1-SNAPSHOT.jar.original也在target的文件夹内，它是Maven在SpringBoot重新打包前创建的。

如果想运行

```cmd
$ java -jar target/ch1helloworld-0.0.1-SNAPSHOT.jar
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.4.RELEASE)

2020-10-22 23:26:47.591  INFO 22848 --- [           main] t.s.c.Ch1helloworldApplication           : Starting Ch1helloworldApplication v0.0.1-SNAPSHOT on DESKTOP-I81IC1V with PID 22848 (F:\studySrc\myspringstudy\ch1helloworld\target\ch1helloworld-0.0.1-SNAPSHOT.jar started by 81929 in F:\studySrc\myspringstudy\ch1helloworld)
2020-10-22 23:26:47.595  INFO 22848 --- [           main] t.s.c.Ch1helloworldApplication           : No active profile set, falling back to default profiles: default
2020-10-22 23:26:50.785  INFO 22848 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-10-22 23:26:50.810  INFO 22848 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-10-22 23:26:50.810  INFO 22848 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.38]
2020-10-22 23:26:50.955  INFO 22848 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-10-22 23:26:50.956  INFO 22848 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 3166 ms
2020-10-22 23:26:51.279  INFO 22848 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-10-22 23:26:51.558  INFO 22848 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-10-22 23:26:51.572  INFO 22848 --- [           main] t.s.c.Ch1helloworldApplication           : Started Ch1helloworldApplication in 4.844 seconds (JVM running for 5.739)
```

和之前一样，想优雅地结束，按`ctrl+c`

# 使用Spring

本章内容：

* 创建系统, 自动配置以及如何运行你的应用。
* 也包含了一些Spring Boot的最佳实践。

## 1. Build Systems

​	建议使用Maven和Gradle构建，其他虽然也可以，但是支持不一定好。

### 1.1 依赖管理

​	每个Spring Boot的发布版本都提供了一系列它支持的依赖。在实践中，这些依赖不可以不指定版本。当我们升级Spring Boot本身，这些依赖也会随之更新。

> 每个版本的Spring Boot都会携带一个基础版本的Spring Framework。我们强烈建议你不要指定它的版本。

### 1.2 Maven

略

### 1.3 Gradle

略

### 1.4 Ant

略

### 1.5 起步依赖

**Table 1. Spring Boot application starters**

| Name                                          | Description                                                  |
| :-------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`                         | Core starter, including auto-configuration support, logging and YAML |
| `spring-boot-starter-activemq`                | Starter for JMS messaging using Apache ActiveMQ              |
| `spring-boot-starter-amqp`                    | Starter for using Spring AMQP and Rabbit MQ                  |
| `spring-boot-starter-aop`                     | Starter for aspect-oriented programming with Spring AOP and AspectJ |
| `spring-boot-starter-artemis`                 | Starter for JMS messaging using Apache Artemis               |
| `spring-boot-starter-batch`                   | Starter for using Spring Batch                               |
| `spring-boot-starter-cache`                   | Starter for using Spring Framework’s caching support         |
| `spring-boot-starter-data-cassandra`          | Starter for using Cassandra distributed database and Spring Data Cassandra |
| `spring-boot-starter-data-cassandra-reactive` | Starter for using Cassandra distributed database and Spring Data Cassandra Reactive |
| `spring-boot-starter-data-couchbase`          | Starter for using Couchbase document-oriented database and Spring Data Couchbase |
| `spring-boot-starter-data-couchbase-reactive` | Starter for using Couchbase document-oriented database and Spring Data Couchbase Reactive |
| `spring-boot-starter-data-elasticsearch`      | Starter for using Elasticsearch search and analytics engine and Spring Data Elasticsearch |
| `spring-boot-starter-data-jdbc`               | Starter for using Spring Data JDBC                           |
| `spring-boot-starter-data-jpa`                | Starter for using Spring Data JPA with Hibernate             |
| `spring-boot-starter-data-ldap`               | Starter for using Spring Data LDAP                           |
| `spring-boot-starter-data-mongodb`            | Starter for using MongoDB document-oriented database and Spring Data MongoDB |
| `spring-boot-starter-data-mongodb-reactive`   | Starter for using MongoDB document-oriented database and Spring Data MongoDB Reactive |
| `spring-boot-starter-data-neo4j`              | Starter for using Neo4j graph database and Spring Data Neo4j |
| `spring-boot-starter-data-r2dbc`              | Starter for using Spring Data R2DBC                          |
| `spring-boot-starter-data-redis`              | Starter for using Redis key-value data store with Spring Data Redis and the Lettuce client |
| `spring-boot-starter-data-redis-reactive`     | Starter for using Redis key-value data store with Spring Data Redis reactive and the Lettuce client |
| `spring-boot-starter-data-rest`               | Starter for exposing Spring Data repositories over REST using Spring Data REST |
| `spring-boot-starter-data-solr`               | Starter for using the Apache Solr search platform with Spring Data Solr |
| `spring-boot-starter-freemarker`              | Starter for building MVC web applications using FreeMarker views |
| `spring-boot-starter-groovy-templates`        | Starter for building MVC web applications using Groovy Templates views |
| `spring-boot-starter-hateoas`                 | Starter for building hypermedia-based RESTful web application with Spring MVC and Spring HATEOAS |
| `spring-boot-starter-integration`             | Starter for using Spring Integration                         |
| `spring-boot-starter-jdbc`                    | Starter for using JDBC with the HikariCP connection pool     |
| `spring-boot-starter-jersey`                  | Starter for building RESTful web applications using JAX-RS and Jersey. An alternative to [`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-web) |
| `spring-boot-starter-jooq`                    | Starter for using jOOQ to access SQL databases. An alternative to [`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-data-jpa) or [`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-jdbc) |
| `spring-boot-starter-json`                    | Starter for reading and writing json                         |
| `spring-boot-starter-jta-atomikos`            | Starter for JTA transactions using Atomikos                  |
| `spring-boot-starter-jta-bitronix`            | Starter for JTA transactions using Bitronix. Deprecated since 2.3.0 |
| `spring-boot-starter-mail`                    | Starter for using Java Mail and Spring Framework’s email sending support |
| `spring-boot-starter-mustache`                | Starter for building web applications using Mustache views   |
| `spring-boot-starter-oauth2-client`           | Starter for using Spring Security’s OAuth2/OpenID Connect client features |
| `spring-boot-starter-oauth2-resource-server`  | Starter for using Spring Security’s OAuth2 resource server features |
| `spring-boot-starter-quartz`                  | Starter for using the Quartz scheduler                       |
| `spring-boot-starter-rsocket`                 | Starter for building RSocket clients and servers             |
| `spring-boot-starter-security`                | Starter for using Spring Security                            |
| `spring-boot-starter-test`                    | Starter for testing Spring Boot applications with libraries including JUnit, Hamcrest and Mockito |
| `spring-boot-starter-thymeleaf`               | Starter for building MVC web applications using Thymeleaf views |
| `spring-boot-starter-validation`              | Starter for using Java Bean Validation with Hibernate Validator |
| `spring-boot-starter-web`                     | Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default embedded container |
| `spring-boot-starter-web-services`            | Starter for using Spring Web Services                        |
| `spring-boot-starter-webflux`                 | Starter for building WebFlux applications using Spring Framework’s Reactive Web support |
| `spring-boot-starter-websocket`               | Starter for building WebSocket applications using Spring Framework’s WebSocket support |



**Table 2. Spring Boot production starters**

| Name                           | Description                                                  |
| :----------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-actuator` | Starter for using Spring Boot’s Actuator which provides production ready features to help you monitor and manage your application |

**Table 3. Spring Boot technical starters**

| `spring-boot-starter-jetty`         | Starter for using Jetty as the embedded servlet container. An alternative to [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) |
| ----------------------------------- | ------------------------------------------------------------ |
| `spring-boot-starter-log4j2`        | Starter for using Log4j2 for logging. An alternative to [`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-logging) |
| `spring-boot-starter-logging`       | Starter for logging using Logback. Default logging starter   |
| `spring-boot-starter-reactor-netty` | Starter for using Reactor Netty as the embedded reactive HTTP server. |
| `spring-boot-starter-tomcat`        | Starter for using Tomcat as the embedded servlet container. Default servlet container starter used by [`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-web) |
| `spring-boot-starter-undertow`      | Starter for using Undertow as the embedded servlet container. An alternative to [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) |

## 2. 构建代码

### 2.1 使用默认包

​	当一个类没有包含`package`声明，它被认为在默认包中。但是不鼓励使用默认包，可能会导致以下注解无法使用: @ComponentScan`, `@ConfigurationPropertiesScan`, `@EntityScan`, or `@SpringBootApplication。

### 2.2 定位Main Application Class

​	我们建议把main application class放在根package中。[`@SpringBootApplication`](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-using-springbootapplication-annotation)注解通常指定你的主类，它也隐式地为项目定义了一个基本的搜索包。比如你想创建一个JPA应用，就会去搜索你添加`@SpringBootApplication`注解类的包下去搜索`@Entity`注解的项。

> @EnableAutoConfiguration` and `@ComponentScan也可以起到和@SpringBootApplication类似的效果。

比如创建类如下：

```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`Application.java`会声明`main`方法，然后添加`@SpringBootApplication`注解

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## 3. 配置类

​	建议使用`Configuration`类。不建议使用XML配置。

### 3.1 导入额外配置类

​	如果不能将所有的配置放到配置类中。可以使用`@Import`注解去导入额外的配置。也可以选择将配置类的包路径放到`@ComponentScan`注解中。

### 3.2 Import XML Configuration

​	如果非要使用XML配置，可以使用`@ImportResource`加载XML配置。

## 4. 自动配置

​	通过在任意配置类上添加`@EnableAutoConfiguration`或`@SpringBootApplication`注解, 来开启自动配置。

> 通常只建议在primary @Configuration class上加这两个注解

### 4.1  逐步替换自动配置

​	自动配置时非侵入性的，可以在任意点定义你的配置类去替换自动配置的某一部分。比如, 如果你添加自己的`DataSource`, 那么默认的嵌入database支持回退。

​	如果想找到是怎样自动设置的,你可以以`--debug`开关。然后会启用debug logs，打印出一些报告。

### 4.2 禁用特定的自动配置类

​	可以在`@SpringBootApplication`类中禁用他们。

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
```

​	如果这个类没在classpath, 可以使用`excludeName`去指定(根据类的全限定名)。如果你更喜欢用`@EnableAutoConfiguration`, `@exclude`和`@excludeName`也可以使用。最后，你也可以在`spring.autoconfigure.exclude`属性中设置要排除的类。

> 可以在注解和属性中通识定义exclusions。

> 尽管自动配置类是public, 一般认为只有使用类型名字禁用自动配置时唯一的方法。一些内部配置类或者bean方法不建议直接使用。

## 5. Spring Bean以及依赖注入

​	一般使用`@ComponentScan`和`@Autowired`来进行依赖注入。

​	如果你将你的代码组织在Application Class作为根目录的包下，你可以无需在`@ComponentScan`加任何参数。程序所有的components都会自动注入成Spring Beans。

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果有一个构造器，可以忽略`@Autowired`。

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

> 注意代码将 `riskAssessor`设置为了 final field, 避免该bean被修改。

## 6. Using the @SpringBootApplication Annotation

​	单个的`@SpringBootApplication`用来启用一下三个特性：

* `@EnableAutoConfiguration`: 启用自动配置机制。
* `@ComponentScan`: 启用 `@Component`扫描。
* `@Configuration`: 允许在上下文中导入额外的Bean或者往外的配置类。

也可以不用`@SpringBootApplication`，比如去掉包扫描和configuration properties

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ MyConfig.class, MyAnotherConfig.class })
public class Application {

    public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
    }

}
```

## 7. 运行程序

### 7.1 在IDE里运行

​	略

### 7.2 jar包运行

```
$ java -jar ch1helloworld-0.0.1-SNAPSHOT.jar
```

也可以按debug模式运行

```
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n  -jar ch1helloworld-0.0.1-SNAPSHOT.jar
```

### 7.3 使用Maven Plugin运行

```
$ mvn spring-boot:run
$ export MAVEN_OPTS=-Xmx1024m
```

### 7.4 使用Gradle Plugin

```java
$ gradle bootRun
```

如果想使用java参数

```java
$ export JAVA_OPTS=-Xmx1024m
```

### 7.5 热替换

​	JRebel支持热替换。

​	`spring-boot-devtools`包含对程序快速重启的支持。

## 8. Developer Tools

Maven:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

Gradle:

```
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

> 如果你用`jar -jar`运行程序，否则它会从一个特殊的ClassLoader开始启动。如果你不想使用dev-tools,可以设置系统变量禁用它。
>
> -Dspring.devtools.restart.enabled=false

> 标记dependency为 optional在Maven中或者使用`developmentOnly`在Gradle中，确保devtools不会被传递到项目的其他模块

> 重打包默认不包含devtools。如果你想使用某个远程开发工具功能，在Maven中需要设置`excludeDevtools`为false。

### 8.1 默认属性

​	Spring Boot 支持对一些库进行缓存来提高性能。比如模板引擎缓存避免我们重复的解析模板文件。另外一方面，Spring MVC可以添加HTTP 缓存头部到response上用以提供静态资源。

​	不过这个缓存属性是反生产库的，所以`devtools`默认禁用了这个属性。

​	缓存开关一般在`application.properties`中设置。比如, Thymeleaf提供了`spring.thymeleaf.cache`属性。`spring-boot-devtools`会自动应用开发时合适的属性。

​	因为你在开发时需要更多的信息(关于web request)。dev-tools会打开`DEBUG` logging。会给我们提供一些incoming request, 以及 response outcome的信息。如果想记录所有request的细节，可以打开`spring.mvc.log-request-details` or `spring.codec.log-request-details`

> 如果不想默认设置的属性,可以设置`spring.devtools.add-properties`设置为`false`

### 8.2 Automatic Restart

​	如果classpath路径下有文件发生改变，devtools就会自动重启应用。

> Triggering a restart
>
> 导致类路径更新的方式取决于所使用的IDE，如果是Eclipse，保存一个变更的变更文件就会导致restart。但是在Intellij中，building the Project有相同的效果。

> 和LiveReload一起使用体验更好。[See the LiveReload section](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-livereload) for details。
>
> 如果你使用JRebel，自动重启会禁用为了支持动态类加载。

> DevTools依赖于应用程序上下文的关闭钩子，在重新启动时关闭它。如果禁用了关机钩子，它就不能正常工作.
>
> SpringApplication.setRegisterShutdownHook(false)

> DevTools会忽略掉一些名字叫做
>
> `spring-boot`, `spring-boot-devtools`, `spring-boot-autoconfigure`, `spring-boot-actuator`, and `spring-boot-starter`的项目。

> DevTools需要在`ApplicationContext`中定制`ResourceLoader`, 如果程序已经提供了几个，它会被包装起来。直接覆盖`ApplicationContext`的`getResource`方法是不支持的。

#### 8.2.1 Loggin changes in condition evaluation

​	默认情况下, 应用每次重启，都会记录一个条件评估的增量报告。这个报告展示了对你项目自动配置的变化，比如增加和移除了bean以及设置配置属性。

​	禁用该报告：

```properties
spring.devtools.restart.log-condition-evaluation-delta=false
```

#### 8.2.2 Excluding Resources

​	如果确定某个资源的修改不需要触发重启。默认情况下，改变`/META-INF/maven`, `/META-INF/resources`, `/resources`, `/static`, `/public`, or `/templates`不会触发重启，但是会触发 [live reload](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。如果想将这些排除在外，可以设置以下属性。

```properties
spring.devtools.restart.exclude=static/**,public/**
```

> 如果想保持默认的属性，并且排除其他项，使用`spring.devtools.restart.additional-exclude`(这句话没看懂 todo)

#### 8.2.3 监控额外的路径

​	如果你想监控的路径不在classpath下。可以添加`spring.devtools.restart.additional-path`属性。可以使用`spring.devtools.restart.exclude`来控制变化时触发重新加载还是live reload。

#### 8.2.4 禁用重启

​	`spring.devtools.restart.enabled`禁用重启

​	大部分情况下在`application.properties`设置就可以了。但是在一些特定库下这种设置不起作用。可以在main函数中禁用。

```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 8.2.5 Using a Trigger File

​	如果使用了一个持续编译修改文件的IDE，可能更像在指定的时间重启。可以添加一个"trigger file"。它是一个你如果向触发更新，就必须修改的文件。

​	`spring.devtools.restart.trigger-file`设置你的trigger文件路径。

​	如果你文件路径如下：

```
src
+- main
   +- resources
      +- .reloadtrigger
```

那么设置

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

> 如果向将这个设置为全局设置 [global setting](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings)
>
> 这样，所有的项目都会表现出相同的行为。

> Some IDEs have features that save you from needing to update your trigger file manually. [Spring Tools for Eclipse](https://spring.io/tools) and [IntelliJ IDEA (Ultimate Edition)](https://www.jetbrains.com/idea/) both have such support. With Spring Tools, you can use the “reload” button from the console view (as long as your `trigger-file` is named `.reloadtrigger`). For IntelliJ IDEA, you can follow the [instructions in their documentation](https://www.jetbrains.com/help/idea/spring-boot.html#application-update-policies).
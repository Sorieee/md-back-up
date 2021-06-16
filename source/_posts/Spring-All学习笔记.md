---
title: Spring All学习笔记
date: 2020-10-16 21:14:23
tags: [Spring, Sping Cloud]
---

https://github.com/wuyouzhuguli/SpringAll



# 1.开启Spring Boot

在idea中创建Spring Boot项目。

![](https://pic.imgdb.cn/item/5f899ee81cd1bbb86bb36d57.jpg)

创建一个叫MySpringStudy的项目。

![](https://pic.imgdb.cn/item/5f899f421cd1bbb86bb3a6f6.jpg)

![](https://pic.imgdb.cn/item/5f899f581cd1bbb86bb3b6b2.jpg)

项目结构如下：

![](https://pic.imgdb.cn/item/5f899f971cd1bbb86bb3dd4d.jpg)

删除src后，再按类似的流程创建ch1helloworld

## 简单演示

简单写一个Hello World。

```java
package top.sorie.ch1helloworld;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class Ch1helloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(Ch1helloworldApplication.class, args);
    }

    @RequestMapping("/")
    public String hellWorld() {
        return "Hello World";
    }
}

```

结果报错了：

java.lang.NoSuchFieldError: defaultInstance

是重复引入了包的问题

```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
 <dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-web</artifactId>
     <version>5.2.8.RELEASE</version>
</dependency>
```

选择删除后面一个，就ok了。

![](https://pic.imgdb.cn/item/5f89a2c41cd1bbb86bb58ef0.jpg)

在postman中访问，发现POST和GET都可以。

![](https://pic.imgdb.cn/item/5f89a3071cd1bbb86bb5b7a1.jpg)

发现如果是用`@RequestMapping("/")`, 使用OPTIONS查找支持的方法如下：

![](https://pic.imgdb.cn/item/5f89a3531cd1bbb86bb5e4b7.jpg)

## 打包发布

点击Project Structure

![](https://pic.imgdb.cn/item/5f89a3ec1cd1bbb86bb642c4.jpg)

点开Artifacts

![](https://pic.imgdb.cn/item/5f89a40e1cd1bbb86bb65707.jpg)

然后添加一个jar

![](https://pic.imgdb.cn/item/5f89a73c1cd1bbb86bb89165.jpg)

选择类后

![](https://pic.imgdb.cn/item/5f89a76e1cd1bbb86bb8a02f.jpg)

点ok后

![](https://pic.imgdb.cn/item/5f89a7811cd1bbb86bb8a663.jpg)

然后选择构建

![](https://pic.imgdb.cn/item/5f89a83a1cd1bbb86bb8f692.jpg)

![](https://pic.imgdb.cn/item/5f89a8551cd1bbb86bb9222c.jpg)

然后生成jar包，cd到对应目录下，执行命令

```
java -jar myspringstudy.jar
```

**报错：**

**myspringstudy.jar中没有主清单属性**

接下来换个方式

![](https://pic.imgdb.cn/item/5f89a97e1cd1bbb86bb9d4b6.jpg)

在cd到对应目录重新执行

```
java -jar ch1helloworld-0.0.1-SNAPSHOT.jar
```

启动成功, 访问

http://localhost:8080/

## 聊聊Pom.xml

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
            <artifactId>spring-boot-starter-web</artifactId>
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

spring-boot-starter-paren指定了该项目是一个Spring Boot项目，提供了诸多默认的依赖。

ctrl点击spring-boot-parent版本(如果没有就重启一下idea)，然后再点击spring-boot-dependencies。

![](https://pic.imgdb.cn/item/5f8b05ae1cd1bbb86b1aeaab.jpg)

该pom定义了诸多其他包的版本。

![](https://pic.imgdb.cn/item/5f8b05e41cd1bbb86b1af91f.jpg)



但是并非所有的properties都会被启用，其启用与否取决于是否配置了相应的starter。

**spring-boot-starter-web**

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-web默认集成了一些依赖，如果不需要就可以替换掉，比如。

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

**spring-boot-maven-plugin**

提供了

1.把项目打包成一个可执行的超级jar(uber-JAR), 包括把应用程序的所有依赖打入JAR文件内，并为JAR添加一个描述文件，其中的内容能让你用`java -jar`来运行应用程序。

2.搜索`public static void main()`方法来标记为可运行类。

# 2.Spring Boot的一些基础配置

## 定制Banner

Spring Boot启动会有一个默认启动图案。如果想替换，在src/main/resources目录下新建banner.txt，然后将自己图案复制进去即可。ASCII图案可以通过http://www.network-science.de/ascii/一键生成。

banner也可以关闭，在properties文件中

```properties
spring.main.banner-mode=off
```

## 全局配置文件

在src/main/resources目录下，Spring Boot提供了一个名为application.properties或application.yml的全局配置文件，可对一些默认配置的配置值进行修改。

```properties
blog.name=sorie 
```

定义一个BlogProperties，通过@Value来加载配置文件中的属性。

```java
@Component
public class BlogProperties {
    @Value("${blog.name}")
    private String name;
}
```

属性非常多的情况下也可以使用@ConfigurationProperties来指定前缀。

```java
@ConfigurationProperties(prefix="blog")
public class ConfigBean {
    private String name;
    private String title;
    // get,set略
}
```

**属性间的引用**

```properties
blog.name=sorie
blog.url=http://${blog.name}.top
```

**自定义配置文件**

```properties
test.name=KangKang
test.age=25
```



```java
@Configuration
@ConfigurationProperties(prefix="test")
@PropertySource("classpath:test.properties")
@Component
public class TestConfigBean {
    private String name;
    private int age;
    // get,set略
}
```

注解`@PropertySource("classpath:test.properties")`指明了使用哪个配置文件。要使用该配置Bean，同样也需要在入口类里使用注解`@EnableConfigurationProperties({TestConfigBean.class})`来启用该配置。

### 问题2.1

如果不使用`@EnableConfigurationProperties({TestConfigBean.class})`，好像也没问题。

## 通过命令行设置属性值

在运行Spring Boot jar文件时，可以使用命令`java -jar xxx.jar --server.port=8081`来改变端口的值。

如果不想项目配置被命令行修改

```java
public static void main(String[] args) {
    SpringApplicationBuilder builder = new SpringApplicationBuilder(Ch1helloworldApplication.class);
    builder.addCommandLineProperties(false);
    builder.run(args);
}
```

## 使用XML配置

虽然Spring Boot并不推荐我们继续使用xml配置，但如果出现不得不使用xml配置的情况，Spring Boot允许我们在入口类里通过注解`@ImportResource({"classpath:some-application.xml"})`来引入xml配置文件。

## Profile配置

Profile用来针对不同的环境下使用不同的配置文件，多环境配置文件必须以`application-{profile}.properties`的格式命，其中`{profile}`为环境标识。

至于哪个具体的配置文件会被加载，需要在application.properties文件中通过`spring.profiles.active`属性来设置，其值对应`{profile}`值。

如：`spring.profiles.active=dev`就会加载application-dev.properties配置文件内容。可以在运行jar文件的时候使用命令`java -jar xxx.jar --spring.profiles.active={profile}`切换不同的环境配置。

# 3. Spring Boot中使用MyBatis

先创建一个基础的Spring boot项目

## mybatis-spring-boot-starter

在pom中引入

```
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

具体引用那个版本，查看

http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

查看`mybatis-spring-boot-starter`引用的包

![](https://pic.imgdb.cn/item/5f8c5bf91cd1bbb86b6ee102.jpg)

## MySql

在POM中引入

```xml
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```



## Druid数据源

Druid是一个关系型数据库连接池，是阿里巴巴的一个开源项目，地址：https://github.com/alibaba/druid。Druid不但提供连接池的功能，还提供监控功能，可以实时查看数据库连接池和SQL查询的工作情况。

### 引入依赖

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```

pom文件

```yml
server:
  servlet:
    context-path: /web
mybatis:
  # type-aliases扫描路径
  # type-aliases-package:
  # mapper xml实现扫描路径
  mapper-locations: classpath:mapper/*.xml
  property:
    order: BEFORE
spring:
  datasource:
    druid:
      # 数据库访问配置, 使用druid数据源
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/springstudydb?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
      username: root
      password: cquisse
      # 连接池配置
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 连接等待超时时间
      max-wait: 30000
      # 配置检测可以关闭的空闲连接间隔时间
      time-between-eviction-runs-millis: 60000
      # 配置连接在池中的最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: select '1' from dual
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-open-prepared-statements: 20
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙
      filters: stat,wall
      # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
      aop-patterns: com.springboot.servie.*


      # WebStatFilter配置
      web-stat-filter:
        enabled: true
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤的格式
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      # StatViewServlet配置
      stat-view-servlet:
        enabled: true
        # 访问路径为/druid时，跳转到StatViewServlet
        url-pattern: /druid/*
        # 是否能够重置数据
        reset-enable: false
        # 需要账号密码才能访问控制台
        login-username: druid
        login-password: druid123
        # IP白名单
        # allow: 127.0.0.1
        #　IP黑名单（共同存在时，deny优先于allow）
        # deny: 192.168.1.218

      # 配置StatFilter
      filter:
        stat:
          log-slow-sql: true
```

## 使用MyBatis

创建表

```sql
CREATE TABLE springstudydb.`student` (
`SNO`  varchar(3),
`SNAME` varchar(9),
`SSEX` VARCHAR(1),
PRIMARY KEY (`SNO`)
);
INSERT INTO springstudydb.STUDENT VALUES ('001', 'KangKang', 'M ');
INSERT INTO springstudydb.STUDENT VALUES ('002', 'Mike', 'M ');
INSERT INTO springstudydb.STUDENT VALUES ('003', 'Jane', 'F ');
```

创建对应实体

```java
public class Student implements Serializable{
    private static final long serialVersionUID = -339516038496531943L;
    private String sno;
    private String name;
    private String sex;
    // get,set略
}
```

创建studentMapper

```java
import org.apache.ibatis.annotations.*;
import org.springframework.stereotype.Component;
import top.sorie.ch2mybatis.entity.Student;

@Component
@Mapper
public interface StudentMapper {
    @Insert("insert into student(sno,sname,ssex) values(#{sno},#{name},#{sex})")
    int add(Student student);

    @Update("update student set sname=#{name},ssex=#{sex} where sno=#{sno}")
    int update(Student student);

    @Delete("delete from student where sno=#{sno}")
    int deleteBysno(String sno);

    @Select("select * from student where sno=#{sno}")
    @Results(id = "student",value= {
            @Result(property = "sno", column = "sno", javaType = String.class),
            @Result(property = "name", column = "sname", javaType = String.class),
            @Result(property = "sex", column = "ssex", javaType = String.class)
    })
    Student queryStudentBySno(String sno);
}
```

简单的语句只需要使用@Insert、@Update、@Delete、@Select这4个注解即可，动态SQL语句需要使用@InsertProvider、@UpdateProvider、@DeleteProvider、@SelectProvider等注解。具体可参考MyBatis官方文档：http://www.mybatis.org/mybatis-3/zh/java-api.html。

## 使用xml方式

使用xml方式需要在application.yml中进行一些额外的配置

```yml
mybatis:
  # type-aliases扫描路径
  # type-aliases-package:
  # mapper xml实现扫描路径
  mapper-locations: classpath:mapper/*.xml
  property:
    order: BEFORE
```

## 测试

编写Service以及Controller

```
public interface StudentService {
    int add(Student student);
    int update(Student student);
    int deleteBysno(String sno);
    Student queryStudentBySno(String sno);
}
```

实现类：

```java
@Service("studentService")
public class StudentServiceImp implements StudentService{
    @Autowired
    private StudentMapper studentMapper;
    
    @Override
    public int add(Student student) {
        return this.studentMapper.add(student);
    }
    
    @Override
    public int update(Student student) {
        return this.studentMapper.update(student);
    }
    
    @Override
    public int deleteBysno(String sno) {
        return this.studentMapper.deleteBysno(sno);
    }
    
    @Override
    public Student queryStudentBySno(String sno) {
        return this.studentMapper.queryStudentBySno(sno);
    }
}
   
```

controller

```java
@RestController
public class TestController {

    @Autowired
    private StudentService studentService;

    @RequestMapping( value = "/querystudent", method = RequestMethod.GET)
    public Student queryStudentBySno(String sno) {
        return this.studentService.queryStudentBySno(sno);
    }
}
```

访问：http://localhost:8080/web/querystudent?sno=001

![](https://pic.imgdb.cn/item/5f8dc15b1cd1bbb86bb7b3b2.jpg)

然后登陆druid管理平台 http://localhost:8080/web/druid

![](https://pic.imgdb.cn/item/5f8dc1d71cd1bbb86bb7cfd3.jpg)

输入账号密码就可以看到监控后台

![](https://pic.imgdb.cn/item/5f8dc21f1cd1bbb86bb7e2d8.jpg)

切换到sql监控可以看到刚才执行的sql

![](https://pic.imgdb.cn/item/5f8dc23f1cd1bbb86bb7eaff.jpg)


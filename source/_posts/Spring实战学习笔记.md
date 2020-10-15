---
title: Spring实战学习笔记
date: 2020-07-20 21:01:10
tags: [Spring,Spring实战]
---

学习书籍：Spring实战第5版

源代码下载位置：https://github.com/habuma/spring-in-action-5-samples

https://www.manning.com/books/spring-in-action-fifth-edition

# 第1部分 Spring基础

## 第1章 Spring起步

本章内容：

* Spring 和 Spring Boot 的 必备 知识

* 初始化Spring项目
* Spring生态系统概览

### 1.1 什么是Spring

​	Spring的核心是提供了一个容器(Container)，通常称为Spring应用上下文(Spring application context)，它们或创建和管理应用组件。这些组件也可以称为bean。

​	把bean装配在一起的行为是通过一种基于依赖注入的(dependency Injection, DI)的模式实现的。

​	组件不会在去创建它所依赖的组件并管理它们的生命周期，通过依赖注入，用容器来管理和维护所有组件，并且将其注入到需要它们的bean中。

​	通常通过构造器参数和属性访问来实现。

​	![](https://pic.imgdb.cn/item/5f15a89e14195aa5944f626a.jpg)

​	历史上，装配的方式是试用一个或多个XML文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="knight" class="sia.knights.BraveKnight">
  <constructor-arg ref="quest" />
</bean>

<bean id="quest" class="sia.knights.SlayDragonQuest">
  <constructor-arg value="#{T(System).out}" />
</bean>

</beans>

```

​	在最近的Spring版本中，基于Java的配置更为常见。

```java
@Configuration
public class KnightConfig {

  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }
  
  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }

}
```

​	@Configuration注解会告知Spring这是一个配置类，会为Spring应用上下文提供Bean。

​	这个配置类的方法试用@Bean注解进行了编著，表明这些方法所返回的对象会以bean的形式添加到上下文中。

​	相对于XML的配置方式，基于Java的配置会带来多项额外收益，比如更强的类型安全性以及更好的重构能力。

不过不管是哪种方式，不能自动配置的时候才是重要的。

​	自动配置起源于自动装配(Autowiring)和组件扫描(component scanning)。扫描可以自动发现组件，创建成上下文中的Bean。又通过自动装配技术，注入到所依赖的其他bean中。

​	随着Spring Boot的引入，自动配置的能力大大增强，远超上面所述。最广为人知的增强方法就是自动配置(autoconfiguration)，Spring Boot能够基于类路径中的条目、环境变量和其他因素合理猜测需要配置的组件并将它们装配在一起。

### 1.2 初始化Spring应用

​	学会如何使用Spring Initializr初始化应用。

​	使用它的几个方式：

* 通过地址https://start.spring.io/的Web应用；
* 在命令行中使用curl命令；
* 在IntelliJ IDEA中创建新项目；
* 在NetBeans中创建新项目。

作者展示的是在Spring Tool Suite中使用Spring Initializr。

我则使用IDEA创建项目。

#### 1.2.1 使用IDEA初始化Spring项目

点击新建项目

![](https://pic.imgdb.cn/item/5f15abb214195aa594508be5.jpg)



选择Spring Initializr

![](https://pic.imgdb.cn/item/5f15abd014195aa594509eec.jpg)



填写Poject Metadata

![](https://pic.imgdb.cn/item/5f15ac2a14195aa59450c684.jpg)



选择依赖，此处只勾选Lombok

![](https://pic.imgdb.cn/item/5f15ac4614195aa59450d543.jpg)



如果必要，在这里可以选择Spring boot的版本

![](https://pic.imgdb.cn/item/5f15ac7a14195aa59450ef8d.jpg)



最后一步，确认项目名字和存放地址。

![](https://pic.imgdb.cn/item/5f15ac9714195aa59450fb91.jpg)



#### 1.2.2 确认Spring项目结果

目录如下

![](https://pic.imgdb.cn/item/5f15acc514195aa594510c4f.jpg)

应用代码放在了src/main/java中

测试代码放在了src/test/java中

非Java的资源放在了src/main/resources中

注意几点

* mvnw和mvnw.cmd：这是Maven包装器脚本，借助这些脚本即使你的机器上没有安装Maven也可以构建构建项目。
* pom.xml：这是Maven构建规范，随后会深入介绍该文件
* SiaApplication：这是主类，它会启动该项目。
* applicatin.properties:这个文件起初是空的，提供给我们指定配置属性的地方。详细在第5章
* static&templates: 与作者不同我这里没有这两个文件夹，是用来存放静态内容和Thymleaf模板的。
* SiaApplicationTests：简单测试类，保证上下文可以成功加载。

##### 探索构建规范

pom.xml如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>cn.sorie</groupId>
    <artifactId>sia</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sia</name>
    <description>Spring in action Study Source Code</description>
	//默认打包为jar
    
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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

​	应用构建默认会构建为jar文件。可能觉得有点奇怪。打包成jar文件是基于云思维做出的选择。因为不是所有的云平台都可以运行war。

​	第2章会详细谅解如何构建war文件。

​	在依赖中注意starter这个单词，Spring Boot starter依赖的特别之处在于它们本身并不包含库代码，而是传递性地拉取其他的库。

​	有三个好处：

* 构建文件显著减小并更易于管理
* 可以更具它们所提供的功能来思考依赖而不是库文件名称。
* 不必担心版本问题，只需要关心使用的哪个版本的Spring Boot就可以了。



​	最后，构建规范害包含了一个Spring Boot插件。提供了重要功能。

* 提供了一个Maven goal，允许我们使用maven来允许引用
* 确保所有库都包含在可执行的jar文件中
* 在jar中生成一个manifest文件，将引导类生命为可执行jar的主类



​	打开主类看一下

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SiaApplication {

    public static void main(String[] args) {
        SpringApplication.run(SiaApplication.class, args);
    }

}
```

​	只有很少代码

​	@SpringBootApplication是一个组合注解，组合了3个注解。

* @SpringBootConfiguration：将该类声明为配置类。其实上是@Configuration注解的特殊形式。
* @EnableAutoConfiguration：启用Spring Boot的自动配置。自动配置我们会用到的组件
* @ComponentScan：启用组件扫描。发现@Component，@Controller，@Service这样的组件。



​	还有一个比较重要的地方是main方法，调用了SpringApplication的run方法。后者会真正执行应用的引导过程，就是创建Spring应用上下文。参数一个是配置类，另一个是命令行参数。

##### 测试应用

打开测试类，修改代码如下

```java
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
class SiaApplicationTests {

    @Test
    void contextLoads() {
    }

}

```

​	@SpringBootTest会告诉Junit在启动测试的时候要加上Spring Boot的功能。

​	SpringRunner是SpringJUnit4ClassRunner的别名。在Spring 4.3中引入，一面溢出对特定JUnit版本的关联。

​	这个代码就是测试是否可以成功启动Spring应用上下文。

​	下面开始添加一些自定义的代码。

### 1.3 编写Spring应用

​	添加一个主页

* 一个控制器类，用来处理主页相关的请求；
* 一个视图模板，用来定义主页看起来是什么样子



​	还会有一个测试类



#### 1.3.1 处理Web请求

​	Spring自带一个强大的Web框架，名为Spring MVC。核心是控制器理念。

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller            // <1>
public class HomeController {

  @GetMapping("/")     // <2>
  public String home() {
    return "home";     // <3>
  }

}
```

​	@Controller没有做额外的事情，主要目的是让组件扫描将这个类识别为一个组件。

​	实际上@Controller，@Component,@Service,@Repository等作用都是完全相同的，但是语义不同。

#### 1.3.2 定义视图

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" 
      xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Taco Cloud</title>
  </head>
  
  <body>
    <h1>Welcome to...</h1>
    <img th:src="@{/images/TacoCloud.png}"/>
  </body>
</html>
```

#### 1.3.3 测试控制器

```java
@RunWith(SpringRunner.class)
@WebMvcTest(HomeController.class)   // <1>
public class HomeControllerTest {

  @Autowired
  private MockMvc mockMvc;   // <2>

  @Test
  public void testHomePage() throws Exception {
    mockMvc.perform(get("/"))    // <3>
    
      .andExpect(status().isOk())  // <4>
      
      .andExpect(view().name("home"))  // <5>
      
      .andExpect(content().string(           // <6>
          containsString("Welcome to...")));  
  }

}
```

#### 1.3.4 构建与运行应用

​	如何启动略。

​	Spring Boot应用的习惯做法是将所有它需要的东西都放到一起，没有必要将其部署到某种应用服务器中。Tomcat是应用的一部分。

​	接下来快速浏览一下DevTools的特性。

#### 1.3.5 了解Spring Boot DevTools

* 代码变更后应用会自动重启；
* 当前面向浏览器的资源发生变化，会自动刷新浏览器；
* 自动禁用模板缓存
* 如果使用了H2数据库的话，内置了H2控制台。



​	因为仅用于开发，可以很智能地在生产环境中把自己禁用掉。



##### 应用自动重启 

​	Devtools会监控变更，有变化会自动重启应用。

​	更准确的是，它运行的时候，程序被加载到JVM两个独立的类加载器中。其中一个加载Java代码、属性文件和main路径下几乎所有内容。另外一个加载器加载依赖的库。

​	检测到变更时，只会重新加载包含项目代码的类加载器，减少应用启动时间。

​	缺点是自动重启无法反应依赖项的变化，变更依赖无法让变更生效。

##### 浏览器自动刷新和禁用模板缓存

​	默认情况下Thymeleaf和FreeMarker这样的模板方案会缓存解析结果。生产环境比较好，但是开发环境不太友好。

​	所以禁用后刷新一下就可以看到结果，但是刷新都懒得点时，DevTools可以提供一些功能。

​	它会自动启动一个LiveReload服务器，它与LiveReload浏览器插件结合起来，可以在静态文件发生变化时，自动刷新浏览器。

##### 内置H2控制台

​	如果使用H2数据库进行开发，DevTools会自动启动H2控制台。



#### 1.3.6 回顾一下

* 使用Spring Initializr创建初始化项目结构
* 编写控制器处理针对主页的请求；
* 定义一个视图模板来渲染主页；
* 编写了一个测试类来验证工作符合预期。



​	早pom文件中声明对Web和Thymeleaf starter的依赖。这两个依赖会传递引入大量其他依赖，包括：

* Spring MVC框架
* 嵌入式的Tomcat
* Thymeleaf和Thymeleaf布局语言；



​	还引入了Spring Boot的自动配置库。自动配置会探测到这些苦，完成如下功能。

* 在上下文中配置bean以启动Spring MVC
* 在上下文中配置嵌入式的tomcat服务器；
* 配置Thymeleaf视图解析器，以便于使用Thymeleaf模板渲染Spring MVC视图。

### 1.4 俯瞰Spring风景线

​	想了解整体情况，只需要查看 Initializr表单的一堆复选框列表即可，有100多个可选的依赖项。

#### 1.4.1 Spring 核心框架

​	Spring核心框架是Spring中一切基础。它提供了核心容器和依赖注入框架，还提供了一些其他重要的特性。

​	比如可以创建REST API，数据持久化的基础支持。

​	最新的Spring版本还添加了反应式风格变成的支持。

#### 1.4.2 Spring Boot

​	除了starter依赖和自动配置还提供了大量其他有用的特性：

	* Actuator能够洞察应用运行时的内部工作状态，包括指标、线程dump信息、应用监控状况以及应用可用的环境属性；
	* 灵活的环节属性规范；
	* 在核心框架的测试辅助功能上提供了对测试的额外支持。



​	除此之外，还提供了基于Groovy脚本的编程模型，称为Spring Boot命令行接口(Command-Line Interface, CLI)。



#### 1.4.3 Spring data

​	它提供了非常令人惊叹的功能：将repository定义为非常简单的Java接口，在定义驱动存储和检索数据的方法是使用一种命名约定即可。

​	可以处理多种不同类型数据库：关系型数据库(JPA),文档数据库(mongo),图数据库(Noe4j)等。

#### 1.4.4 Spring Security

​	安全框架：

	* 身份验证；
	* 授权；
	* API安全性。



​	范围太大，这本书无法深入。



#### 1.4.5 Spring Integration和Spring Batch

​	Spring Integration解决实时集成问题，Spring Batch解决批量集成问题。

#### 1.4.6 Spring Cloud

​	微服务。

​	建议阅读《Spring Microservices in Action》一书。



### 1.5 小结

* Spring旨在简化开发
* Spring Boot 构建在Spring之上，通过简化依赖管理、自动配置和运行时洞察，使Spring更加易懂
* Spring应用程序可以使用Spring Initializr进行初始化。
* Spring上下文中，组件可以使用Java或XML显示生命，也可以通过组件扫描发现。

## 第2章 开发Web应用

本章内容：

* 浏览器中展示模型数据
* 处理和校验表单输入
* 选择视图模板库

### 2.1 展现信息

​	在Spring Web中，获取和处理数据是控制器的任务，将数据渲染到HTML展示时视图的任务。

​	需要构建如下组件：

* 用来定义taco配料属性的领域类。
* 用来获取配料信息并将其床底至视图的SpringMVC控制类。
* 用来在用户的浏览器中配置配料列表的视图模板

![](https://pic.imgdb.cn/item/5f16effd14195aa594e58acc.jpg)



​	本章只关注Spring的Web框架，数据库在第3张讲解。

#### 2.1.1 构建领域类

```java
@Data
@RequiredArgsConstructor
public class Ingredient {
  
  private final String id;
  private final String name;
  private final Type type;
  
  public static enum Type {
    WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
  }

}
```

​	使用了Lombok，不需要自定义getter, setter, equals(), hashCode(), toString()等方法。

添加依赖即可，idea也要添加lombok插件。

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <!-- <scope>compileOnly</scope> -->
</dependency>
```

#### 2.1.2 创建控制器类

* 处理路径为"/design"的HTTP GET请求
* 构建配料的列表
* 处理请求，并将配料数据传递给要渲染为HTML的视图模板，发送给发起请求的Web浏览器。

```java
@Slf4j
@Controller
@RequestMapping("/design")
public class DesignTacoController {

//end::head[]

    @ModelAttribute
    public void addIngredientsToModel(Model model) {
        List<Ingredient> ingredients = Arrays.asList(
                new Ingredient("FLTO", "Flour Tortilla", Ingredient.Type.WRAP),
                new Ingredient("COTO", "Corn Tortilla", Ingredient.Type.WRAP),
                new Ingredient("GRBF", "Ground Beef", Ingredient.Type.PROTEIN),
                new Ingredient("CARN", "Carnitas", Ingredient.Type.PROTEIN),
                new Ingredient("TMTO", "Diced Tomatoes", Ingredient.Type.VEGGIES),
                new Ingredient("LETC", "Lettuce", Ingredient.Type.VEGGIES),
                new Ingredient("CHED", "Cheddar", Ingredient.Type.CHEESE),
                new Ingredient("JACK", "Monterrey Jack", Ingredient.Type.CHEESE),
                new Ingredient("SLSA", "Salsa", Ingredient.Type.SAUCE),
                new Ingredient("SRCR", "Sour Cream", Ingredient.Type.SAUCE)
        );

        Ingredient.Type[] types = Ingredient.Type.values();
        for (Ingredient.Type type : types) {
            model.addAttribute(type.toString().toLowerCase(),
                    filterByType(ingredients, type));
        }
    }

    //tag::showDesignForm[]
    @GetMapping
    public String showDesignForm(Model model) {
        model.addAttribute("design", new Taco());
        return "design";
    }

//end::showDesignForm[]

/*
//tag::processDesign[]
  @PostMapping
  public String processDesign(Design design) {
    // Save the taco design...
    // We'll do this in chapter 3
    log.info("Processing design: " + design);

    return "redirect:/orders/current";
  }

//end::processDesign[]
 */

    //tag::processDesignValidated[]
    @PostMapping
    public String processDesign(@Valid @ModelAttribute("design") Taco design, Errors errors, Model model) {
        if (errors.hasErrors()) {
            return "design";
        }

        // Save the taco design...
        // We'll do this in chapter 3
        log.info("Processing design: " + design);

        return "redirect:/orders/current";
    }

//end::processDesignValidated[]

    //tag::filterByType[]
    private List<Ingredient> filterByType(
            List<Ingredient> ingredients, Ingredient.Type type) {
        return ingredients
                .stream()
                .filter(x -> x.getType().equals(type))
                .collect(Collectors.toList());
    }

//end::filterByType[]
// tag::foot[]
}
// end::foot[]
```

​	注意注解。首先是@Slf4j，是Lombok提供的注解，会在类自动生成一个SLF4J的Logger和下面代码效果一样

```java
private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(DesignTacoController.class);
```

​	然后是@Controller，用于识别这个类是控制器。Spring会发现它并自动创建一个DesignTacoController实例，并将该实例作为Spring应用上下文中的bean。

​	还有@RequestMapping注解。可以指定该控制器处理的请求类型。在该例中处理类型都是以"/design"开头的请求。

##### 处理GET请求

​	修饰showDesignForm方法的@GetMapping注解对类级别的@RequestMapping进行了细化。

​	@GetMapping是一个比较新的注解，在Spring4.3引入，效果类似于

```java
@RequestMapping(
    method = {RequestMethod.GET}
)
```

​	显然@GetMapping更加简洁。

![](https://pic.imgdb.cn/item/5f16f4f214195aa594e801e3.jpg)

>**让正确的事情变更更容易**
>
>@RequestMapping(method = {RequestMethod.GET})让开发人员容易懒惰地忽略掉method属性。
>
>作者喜欢在类级别上使用@RequestMapping，具体的方法上使用@GetMapping、@PostMapping等。

#### 2.1.3 设计视图

​	使用ThymeLeaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

​	在运行时，Spring Boot的自动配置功能会发现ThymeLeaf在类路径中，因此会为Spring MVC创建支撑ThymeLeaf视图的bean。

```html
<span th:text="${ingredient.name}">INGREDIENT</span><br/>
```

th:text是Thymeleaf命名空间中的属性，它会执行替换过程,${}会告诉它要使用某个请求属性。

​	不做过多介绍。

```html
<!-- tag::all[] -->
<!-- tag::head[] -->
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Taco Cloud</title>
    <link rel="stylesheet" th:href="@{/styles.css}" />
  </head>

  <body>
    <h1>Design your taco!</h1>
    <img th:src="@{/images/TacoCloud.png}"/>

<!-- tag::formTag[] -->
    <form method="POST" th:object="${design}">
    <!-- end::all[] -->

    <span class="validationError"
          th:if="${#fields.hasErrors('ingredients')}"
          th:errors="*{ingredients}">Ingredient Error</span>

    <!-- tag::all[] -->
    <div class="grid">
<!-- end::formTag[] -->
<!-- end::head[] -->
      <div class="ingredient-group" id="wraps">
<!-- tag::designateWrap[] -->
      <h3>Designate your wrap:</h3>
      <div th:each="ingredient : ${wrap}">
        <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
        <span th:text="${ingredient.name}">INGREDIENT</span><br/>
      </div>
<!-- end::designateWrap[] -->
      </div>

     <div class="ingredient-group" id="proteins">
      <h3>Pick your protein:</h3>
      <div th:each="ingredient : ${protein}">
        <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
        <span th:text="${ingredient.name}">INGREDIENT</span><br/>
      </div>
      </div>

     <div class="ingredient-group" id="cheeses">
      <h3>Choose your cheese:</h3>
      <div th:each="ingredient : ${cheese}">
        <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
        <span th:text="${ingredient.name}">INGREDIENT</span><br/>
      </div>
      </div>

     <div class="ingredient-group" id="veggies">
      <h3>Determine your veggies:</h3>
      <div th:each="ingredient : ${veggies}">
        <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
        <span th:text="${ingredient.name}">INGREDIENT</span><br/>
      </div>
      </div>

     <div class="ingredient-group" id="sauces">
      <h3>Select your sauce:</h3>
      <div th:each="ingredient : ${sauce}">
        <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
        <span th:text="${ingredient.name}">INGREDIENT</span><br/>
      </div>
      </div>
      </div>

      <div>


      <h3>Name your taco creation:</h3>
      <input type="text" th:field="*{name}"/>
      <!-- end::all[] -->
      <span th:text="${#fields.hasErrors('name')}">XXX</span>
      <span class="validationError"
            th:if="${#fields.hasErrors('name')}"
            th:errors="*{name}">Name Error</span>
      <!-- tag::all[] -->
      <br/>

      <button>Submit your taco</button>
      </div>
<!-- tag::closeFormTag[] -->
    </form>
<!-- end::closeFormTag[] -->
  </body>
</html>
<!-- end::all[] -->
```

效果如下：

![](https://pic.imgdb.cn/item/5f16f7a514195aa594e8e427.jpg)

### 2.2 处理表单提交

​	form标签method被设置为POST。意味着在提交的时候会调用"/design"的POST方法，把所有表单中的数据发送到服务端。

​	有一个POST方法来处理请求

```java
@PostMapping
public String processDesign(@Valid @ModelAttribute("design") Taco design, Errors errors, Model model) {
    if (errors.hasErrors()) {
        return "design";
    }

    // Save the taco design...
    // We'll do this in chapter 3
    log.info("Processing design: " + design);

    return "redirect:/orders/current";
}
```

当表单提交时，输入会绑定到Taco对象上。

然后创建一个OrderController来将taco快递过去

```java
@Slf4j
@Controller
@RequestMapping("/orders")
public class OrderController {
  
//end::baseClass[]
//tag::orderForm[]
  @GetMapping("/current")
  public String orderForm(Model model) {
    model.addAttribute("order", new Order());
    return "orderForm";
  }
//end::orderForm[]

/*
//tag::handlePost[]
  @PostMapping
  public String processOrder(Order order) {
    log.info("Order submitted: " + order);
    return "redirect:/";
  }
//end::handlePost[]
*/
  
//tag::handlePostWithValidation[]
  @PostMapping
  public String processOrder(@Valid Order order, Errors errors) {
    if (errors.hasErrors()) {
      return "orderForm";
    }
    
    log.info("Order submitted: " + order);
    return "redirect:/";
  }
//end::handlePostWithValidation[]
  
//tag::baseClass[]
  
}
//end::baseClass[]
```

视图为

```html
<!-- tag::allButValidation[] -->
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
  <head>
    <title>Taco Cloud</title>
    <link rel="stylesheet" th:href="@{/styles.css}" />
  </head>

  <body>

    <form method="POST" th:action="@{/orders}" th:object="${order}">
      <h1>Order your taco creations!</h1>

      <img th:src="@{/images/TacoCloud.png}"/>
      <a th:href="@{/design}" id="another">Design another taco</a><br/>

      <div th:if="${#fields.hasErrors()}">
        <span class="validationError">
        Please correct the problems below and resubmit.
        </span>
      </div>

      <h3>Deliver my taco masterpieces to...</h3>
      <label for="name">Name: </label>
      <input type="text" th:field="*{name}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('name')}"
            th:errors="*{name}">Name Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <label for="street">Street address: </label>
      <input type="text" th:field="*{street}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('street')}"
            th:errors="*{street}">Street Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <label for="city">City: </label>
      <input type="text" th:field="*{city}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('city')}"
            th:errors="*{city}">City Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <label for="state">State: </label>
      <input type="text" th:field="*{state}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('state')}"
            th:errors="*{state}">State Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <label for="zip">Zip code: </label>
      <input type="text" th:field="*{zip}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('zip')}"
            th:errors="*{zip}">Zip Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <h3>Here's how I'll pay...</h3>
<!-- tag::validatedField[] -->
      <label for="ccNumber">Credit Card #: </label>
      <input type="text" th:field="*{ccNumber}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('ccNumber')}"
            th:errors="*{ccNumber}">CC Num Error</span>
<!-- tag::allButValidation[] -->
<!-- end::validatedField[] -->
      <br/>

      <label for="ccExpiration">Expiration: </label>
      <input type="text" th:field="*{ccExpiration}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('ccExpiration')}"
            th:errors="*{ccExpiration}">CC Num Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <label for="ccCVV">CVV: </label>
      <input type="text" th:field="*{ccCVV}"/>
<!-- end::allButValidation[] -->
      <span class="validationError"
            th:if="${#fields.hasErrors('ccCVV')}"
            th:errors="*{ccCVV}">CC Num Error</span>
<!-- tag::allButValidation[] -->
      <br/>

      <input type="submit" value="Submit order"/>
    </form>

  </body>
</html>
<!-- end::allButValidation[] -->
```



### 2.3 校验表单输入

javax和hibernate的校验

```java
import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;

import org.hibernate.validator.constraints.CreditCardNumber;

import lombok.Data;

@Data
public class Order {

  //end::allButValidation[]
  @NotBlank(message="Name is required")
  //tag::allButValidation[]
  private String name;
  //end::allButValidation[]

  @NotBlank(message="Street is required")
  //tag::allButValidation[]
  private String street;
  //end::allButValidation[]

  @NotBlank(message="City is required")
  //tag::allButValidation[]
  private String city;
  //end::allButValidation[]

  @NotBlank(message="State is required")
  //tag::allButValidation[]
  private String state;
  //end::allButValidation[]

  @NotBlank(message="Zip code is required")
  //tag::allButValidation[]
  private String zip;
  //end::allButValidation[]

  @CreditCardNumber(message="Not a valid credit card number")
  //tag::allButValidation[]
  private String ccNumber;
  //end::allButValidation[]

  @Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
           message="Must be formatted MM/YY")
  //tag::allButValidation[]
  private String ccExpiration;
  //end::allButValidation[]

  @Digits(integer=3, fraction=0, message="Invalid CVV")
  //tag::allButValidation[]
  private String ccCVV;

}
//end::allButValidation[]
//end::all[]
```



#### 2.3.1 声明校验规则

略

#### 2.3.2 在表单绑定时执行校验

需要修改控制器

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors) {
  if (errors.hasErrors()) {
    return "orderForm";
  }
  
  log.info("Order submitted: " + order);
  return "redirect:/";
}
```

有错误就反馈到页面上

#### 2.3.3 展示校验错误

```java
<span class="validationError"
      th:if="${#fields.hasErrors('street')}"
      th:errors="*{street}">Street Error</span>
```

### 2.4 使用视图控制器

编写三个Controller

* 都是用@Controller注解；
* 除了HomeController，其他Controller都是用了@RequestMapping注解；
* 都有一个或者多个带@GetMapping或@PostMapping注解的方法



​	大部分控制器遵循这个标准。

​	如果一个控制器无需填充说明或处理输入，有另外方式可以定义控制器。

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("home");
  }

}
```

也可以用主类去继承这个接口。

### 2.5 选择视图模板

![](https://pic.imgdb.cn/item/5f1851b314195aa594804081.jpg)

![](https://pic.imgdb.cn/item/5f1851c214195aa5948048d3.jpg)

#### 缓存模板

​	默认情况，模板只有在第一次使用的时候解析一次，解析结果会被后续请求所使用。开发不方便，可以禁用掉。

```properties
#application.properties
spring.thymeleaf.cache=false
```

![](https://pic.imgdb.cn/item/5f18523214195aa5948098cf.jpg)



或者使用Spring DevTools，它的其中一个功能就是禁止模板缓存。

### 2.6 小结

* Spring MVC是一个强大的Web框架；
* Spring MVC基于注解；
* 大多请求会返回一个视图的逻辑名称，请求会转发到这样的视图上；
* Spring MVC支持校验，通过JavaBean ValidationAPI和Validation API实现(如Hibernate Validator)完成的；
* 对于没有模型数据和逻辑处理的HTTP GET请求，可以使用视图控制器处理；
* 除了Thymeleaf，Spring还支持各种视图方案。

## 第3章 使用数据

本章内容：

* 使用Spring的JdbcTempldate；
* 使用SimpleJdbcInsert插入数据；
* 使用Spring Data声明JPA repository。

### 3.1 使用JDBC读取和写入数据

​	在处理关系型数据时，有多种可选方案，最常见的是JDBC和JPA。Spring都支持两种的抽象形式，让使用更加容易。

​	对JDBC的支持归功于JDBCTempldate类。

​	普通使用JDBC很麻烦，代码这里就省去了。

​	如果是使用JDBCTemplate代码就如下

![](https://pic.imgdb.cn/item/5f18576c14195aa59484f030.jpg)

#### 3.1.1 调整领域对象以适应持久化

```java
import lombok.Data;
import lombok.RequiredArgsConstructor;

@Data
@RequiredArgsConstructor
public class Ingredient {
  
  private final String id;
  private final String name;
  private final Type type;
  
  public static enum Type {
    WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
  }

}
```



```java
@Data
public class Order {
  
  private Long id;
  
  private Date placedAt;
  
//end::newFields[]

  @NotBlank(message="Delivery name is required")
  private String deliveryName;
  
  @NotBlank(message="Street is required")
  private String deliveryStreet;
  
  @NotBlank(message="City is required")
  private String deliveryCity;
  
  @NotBlank(message="State is required")
  private String deliveryState;
  
  @NotBlank(message="Zip code is required")
  private String deliveryZip;

  @CreditCardNumber(message="Not a valid credit card number")
  private String ccNumber;
  
  @Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
           message="Must be formatted MM/YY")
  private String ccExpiration;

  @Digits(integer=3, fraction=0, message="Invalid CVV")
  private String ccCVV;

  private List<Taco> tacos = new ArrayList<>();
  
  public void addDesign(Taco design) {
    this.tacos.add(design);
  }
  
  /*
// tag::newFields[]
  ...

// end::newFields[]
   */
//tag::newFields[]
}
//end::newFields[]
```



```java
@Data
public class Taco {

  private Long id;
  
  private Date createdAt;

//end::newFields[]

  @NotNull
  @Size(min=5, message="Name must be at least 5 characters long")
  private String name;
  
  @Size(min=1, message="You must choose at least 1 ingredient")
  private List<Ingredient> ingredients;

  /*
//tag::newFields[]
   ...
   
//end::newFields[]
   */
//tag::newFields[]
}
//end::newFields[]
```

#### 3.1.2 使用JdbcTempldate

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

嵌入式的数据库h2

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

##### 定义JDBC repository

```java
public interface IngredientRepository {

  Iterable<Ingredient> findAll();
  
  Ingredient findById(String id);
  
  Ingredient save(Ingredient ingredient);
  
}
```

用JDBC实现

```java
@Repository
public class JdbcIngredientRepository
    implements IngredientRepository {

  //tag::jdbcTemplate[]
  private JdbcTemplate jdbc;
  
  //end::jdbcTemplate[]

  @Autowired
  public JdbcIngredientRepository(JdbcTemplate jdbc) {
    this.jdbc = jdbc;
  }
//end::classShell[]

  //tag::finders[]
  @Override
  public Iterable<Ingredient> findAll() {
    return jdbc.query("select id, name, type from Ingredient",
        this::mapRowToIngredient);
  }

  // tag::findOne[]
  @Override
  public Ingredient findById(String id) {
    return jdbc.queryForObject(
        "select id, name, type from Ingredient where id=?",
        this::mapRowToIngredient, id);
  }
  
  // end::findOne[]
  
  //end::finders[]

  /*
  //tag::preJava8RowMapper[]
  @Override
  public Ingredient findOne(String id) {
    return jdbc.queryForObject(
        "select id, name, type from Ingredient where id=?",
        new RowMapper<Ingredient>() {
          public Ingredient mapRow(ResultSet rs, int rowNum) 
              throws SQLException {
            return new Ingredient(
                rs.getString("id"), 
                rs.getString("name"),
                Ingredient.Type.valueOf(rs.getString("type")));
          };
        }, id);
  }
  //end::preJava8RowMapper[]
   */
  
  //tag::save[]
  @Override
  public Ingredient save(Ingredient ingredient) {
    jdbc.update(
        "insert into Ingredient (id, name, type) values (?, ?, ?)",
        ingredient.getId(), 
        ingredient.getName(),
        ingredient.getType().toString());
    return ingredient;
  }
  //end::save[]

  // tag::findOne[]
  //tag::finders[]
  private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
      throws SQLException {
    return new Ingredient(
        rs.getString("id"), 
        rs.getString("name"),
        Ingredient.Type.valueOf(rs.getString("type")));
  }
  //end::finders[]
  // end::findOne[]

  
  /*
//tag::classShell[]

  ...
//end::classShell[]
   */
//tag::classShell[]

}
//end::classShell[]
```

##### 插入一行数据

```java
  @Override
  public Ingredient save(Ingredient ingredient) {
    jdbc.update(
        "insert into Ingredient (id, name, type) values (?, ?, ?)",
        ingredient.getId(), 
        ingredient.getName(),
        ingredient.getType().toString());
    return ingredient;
  }
```

将其运用到Controller中

```java
@Controller
@RequestMapping("/design")
@SessionAttributes("order")
public class DesignTacoController {
  
//end::classShell[]

//tag::bothRepoProperties[]
//tag::ingredientRepoProperty[]
  private final IngredientRepository ingredientRepo;
  
//end::ingredientRepoProperty[]
  private TacoRepository designRepo;

//end::bothRepoProperties[]
  
  /*
// tag::ingredientRepoOnlyCtor[]
  @Autowired
  public DesignTacoController(IngredientRepository ingredientRepo) {
    this.ingredientRepo = ingredientRepo;
  }
// end::ingredientRepoOnlyCtor[]
   */

  //tag::bothRepoCtor[]
  @Autowired
  public DesignTacoController(
        IngredientRepository ingredientRepo, 
        TacoRepository designRepo) {
    this.ingredientRepo = ingredientRepo;
    this.designRepo = designRepo;
  }

  //end::bothRepoCtor[]
  
  // tag::modelAttributes[]
  @ModelAttribute(name = "order")
  public Order order() {
    return new Order();
  }
  
  @ModelAttribute(name = "taco")
  public Taco taco() {
    return new Taco();
  }

  // end::modelAttributes[]
  // tag::showDesignForm[]
  
  @GetMapping
  public String showDesignForm(Model model) {
    List<Ingredient> ingredients = new ArrayList<>();
    ingredientRepo.findAll().forEach(i -> ingredients.add(i));
    
    Ingredient.Type[] types = Ingredient.Type.values();
    for (Ingredient.Type type : types) {
      model.addAttribute(type.toString().toLowerCase(), 
          filterByType(ingredients, type));      
    }

    return "design";
  }
//end::showDesignForm[]

  //tag::processDesign[]
  @PostMapping
  public String processDesign(
      @Valid Taco design, Errors errors, 
      @ModelAttribute Order order) {

    if (errors.hasErrors()) {
      return "design";
    }

    Taco saved = designRepo.save(design);
    order.addDesign(saved);

    return "redirect:/orders/current";
  }
  //end::processDesign[]
  
  private List<Ingredient> filterByType(
      List<Ingredient> ingredients, Ingredient.Type type) {
    return ingredients
              .stream()
              .filter(x -> x.getType().equals(type))
              .collect(Collectors.toList());
  }

  /*
//tag::classShell[]

  ...

//end::classShell[]
   */
//tag::classShell[]

}
//end::classShell[]
```

#### 3.1.3 定义模式和加载数据

![](https://pic.imgdb.cn/item/5f185baf14195aa594888aca.jpg)

schema.sql

```sql
create table if not exists Ingredient (
  id varchar(4) not null,
  name varchar(25) not null,
  type varchar(10) not null
);

create table if not exists Taco (
  id identity,
  name varchar(50) not null,
  createdAt timestamp not null
);

create table if not exists Taco_Ingredients (
  taco bigint not null,
  ingredient varchar(4) not null
);

alter table Taco_Ingredients
    add foreign key (taco) references Taco(id);
alter table Taco_Ingredients
    add foreign key (ingredient) references Ingredient(id);

create table if not exists Taco_Order (
   id identity,
   deliveryName varchar(50) not null,
   deliveryStreet varchar(50) not null,
   deliveryCity varchar(50) not null,
   deliveryState varchar(2) not null,
   deliveryZip varchar(10) not null,
   ccNumber varchar(16) not null,
   ccExpiration varchar(5) not null,
   ccCVV varchar(3) not null,
    placedAt timestamp not null
);

create table if not exists Taco_Order_Tacos (
   tacoOrder bigint not null,
   taco bigint not null
);

alter table Taco_Order_Tacos
    add foreign key (tacoOrder) references Taco_Order(id);
alter table Taco_Order_Tacos
    add foreign key (taco) references Taco(id);
```

问题是这些定义放在什么地方，实际上在resource表下面就可以，在应用启动会基于数据库执行这个文件的SQL,保存为schema.sql

然后还有data.sql

```java
delete from Taco_Order_Tacos;
delete from Taco_Ingredients;
delete from Taco;
delete from Taco_Order;

delete from Ingredient;
insert into Ingredient (id, name, type) 
                values ('FLTO', 'Flour Tortilla', 'WRAP');
insert into Ingredient (id, name, type) 
                values ('COTO', 'Corn Tortilla', 'WRAP');
insert into Ingredient (id, name, type) 
                values ('GRBF', 'Ground Beef', 'PROTEIN');
insert into Ingredient (id, name, type) 
                values ('CARN', 'Carnitas', 'PROTEIN');
insert into Ingredient (id, name, type) 
                values ('TMTO', 'Diced Tomatoes', 'VEGGIES');
insert into Ingredient (id, name, type) 
                values ('LETC', 'Lettuce', 'VEGGIES');
insert into Ingredient (id, name, type) 
                values ('CHED', 'Cheddar', 'CHEESE');
insert into Ingredient (id, name, type) 
                values ('JACK', 'Monterrey Jack', 'CHEESE');
insert into Ingredient (id, name, type) 
                values ('SLSA', 'Salsa', 'SAUCE');
insert into Ingredient (id, name, type) 
                values ('SRCR', 'Sour Cream', 'SAUCE');
```

### 3.2 使用Spring DataJpa 持久化数据

* Spring Data JPA：基于关系型数据可以进行JPA持久化；
* Spring Data MongoDB：持久化到Mongo文档数据库；
* Spring Data Neo4j：持久化到Neo4j图数据库；
* Spring Data Redis：持久化到 Redis Key-Value存储；
* Spring Data Cassandra：持久化到Cassandra数据库。



​	Spring Data提供了一项有趣且最有用的特性：基于repository规范接口自动生成repository的功能。



#### 3.2.1 添加Spring Data JPA到项目中

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

不仅会引入jpa，还会传递性地将Hibernate作为JPA实现引入进来。如果不想用Hibernate，比如想用EclipseLink来替代：

![](https://pic.imgdb.cn/item/5f199e6e14195aa594669c09.jpg)



​	可能还会涉及到其他变更，具体参考JPA文档。



#### 3.2.2 将领域类标准为实体

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@Entity
public class Ingredient {
  
  @Id
  private final String id;
  private final String name;
  private final Type type;
  
  public static enum Type {
    WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
  }

}
```



taco类，可以用注解让其新增的数据自动生成id

```java
@Data
@Entity
public class Taco {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;

  @NotNull
  @Size(min=5, message="Name must be at least 5 characters long")
  private String name;

  private Date createdAt;

  @ManyToMany(targetEntity=Ingredient.class)
  @Size(min=1, message="You must choose at least 1 ingredient")
  private List<Ingredient> ingredients = new ArrayList<>();
  
  @PrePersist
  void createdAt() {
    this.createdAt = new Date();
  }
}

```



order

```java
@Data
@Entity
@Table(name="Taco_Order")
public class Order implements Serializable {

  private static final long serialVersionUID = 1L;
  
  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  
  private Date placedAt;
  
//end::allButDetailProperties[]
  @NotBlank(message="Delivery name is required")
  private String deliveryName;
  
  @NotBlank(message="Street is required")
  private String deliveryStreet;
  
  @NotBlank(message="City is required")
  private String deliveryCity;
  
  @NotBlank(message="State is required")
  private String deliveryState;
  
  @NotBlank(message="Zip code is required")
  private String deliveryZip;

  @CreditCardNumber(message="Not a valid credit card number")
  private String ccNumber;
  
  @Pattern(regexp="^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
           message="Must be formatted MM/YY")
  private String ccExpiration;

  @Digits(integer=3, fraction=0, message="Invalid CVV")
  private String ccCVV;

  /*
  //tag::allButDetailProperties[]
  ...
  
  //end::allButDetailProperties[]
   */
  
//tag::allButDetailProperties[]
  @ManyToMany(targetEntity=Taco.class)
  private List<Taco> tacos = new ArrayList<>();
  
  public void addDesign(Taco design) {
    this.tacos.add(design);
  }
  
  @PrePersist
  void placedAt() {
    this.placedAt = new Date();
  }
  
}
//end::allButDetailProperties[]
```

#### 3.2.3 声明JPA repository

```java
import org.springframework.data.repository.CrudRepository;

import tacos.Ingredient;

public interface IngredientRepository 
         extends CrudRepository<Ingredient, String> {

}
```



```java
import org.springframework.data.repository.CrudRepository;

import tacos.Order;

public interface OrderRepository 
         extends CrudRepository<Order, Long> {

}
```



```java
import org.springframework.data.repository.CrudRepository;

import tacos.Taco;

public interface TacoRepository 
         extends CrudRepository<Taco, Long> {

}
```



#### 3.2.4 自定义JPA repository

​	比如获取投递到指定邮编的订单,只需要在repository中声明：

```java
List<Order> findByDeliveryZip(String deliveryZip);
```

​	创建repository实现时，会检查repository接口的所有方法，解析名称，并基于持久化对象推测方法意图。本质上就是定义了一组小型的领域特定语言(DSL),持久化的细节都是用过repository方法的签名来描述的。

​	更复杂样例：

```java
List< Order> readOrdersByDeliveryZipAndPlacedAtBetween( String deliveryZip, Date startDate, Date endDate);
```

![](https://pic.imgdb.cn/item/5f19a0e714195aa594689d7c.jpg)



​	除了Equals和Between之外，Spring Data方法签名还支持如下操作符：

* IsAfter、After、IsGreaterThan、GreaterThan
* IsGreaterThanEqual、GreaterThan
* IsBefore、Before、IsLessThan、LessThan
* IsLessThanEqual、LessThanEqual
* IsBetween、Between
* IsNull、Null
* IsNotNull、NotNull
* IsIn、In
* IsNotIn、NotIn
* IsStartingWith、StartingWith、StartsWith
* IsEndingWith、EndingWith、EndsWith
* IsContaining、Containing、Contains
* IsLike、Like
* IsNotLike、NotLike
* IsNot、Not
* IgnoringCase、IgnoresCase

作为IgnoringCase、IgnoresCase的替代方案还可以加上AllIgnoreingCase或者AllIgnoresCase，它会忽略所有String比较的大小写。

对于更复杂的查询，可以添加@Query注解

![](https://pic.imgdb.cn/item/5f19a39d14195aa59469f9b3.jpg)



### 3.3 小结

* Spring的JdbcTempdate能够极大地简化JDBC的使用。
* 在我们需要知道数据库所生产的ID值时，可以组合使用PreparedStatementCreator和KeyHolder
* 为了简化数据的插入，可以使用SimpleJdbcInsert
* Spring Data JPA可以极大地简化JPA持久化，只需要编写Repository接口即可

## 第4章 保护Spring

本章内容：

* 自动配置Spring Security
* 设置自定义的用户存储
* 自定义登录页
* 防范CSRF攻击
* 知道用户是谁



​	信息是最重要的东西，安全性是绝大多数系统的一个重要切面。

### 4.1 启用Spring Security

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <scope>test</scope>
</dependency>
```



​	添加后启动项目，应用将会弹出一个HTTP basic认证对话框并提示你进行认证。想通过认证需要用户名和密码。用户名为user，密码在日志中

![](https://pic.imgdb.cn/item/5f1d783514195aa594bd5139.jpg)



​	得到了如下的安全特性：

* 所有的HTTP请求路径都需要认证；
* 不需要特定的角色和权限；
* 没有登录页面；
* 认证过程是通过HTTP Basic认证对话框实现的；
* 系统只有一个用户，用户名是user



### 4.2 配置Spring Security



```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

}
```



​	它提供了一个简单的登录页

![](https://pic.imgdb.cn/item/5f1d793b14195aa594be47fc.jpg)





​	Spring Security为配置用户存储提供了多个可选方案：

* 基于内存的用户存储；
* 基于JDBC的用户存储；
* 以LDAP作为后台的用户存储；
* 自定义用户详情服务。



​	不管使用哪种用户存储，可以通过覆盖WebSecurityConfigurerAdapter中的configure方法来进行配置。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
}
```



#### 4.2.1 基于内存的用户存储



```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  
  auth
    .inMemoryAuthentication()
      .withUser("buzz")
        .password("infinity")
        .authorities("ROLE_USER")
      .and()
      .withUser("woody")
        .password("bullseye")
        .authorities("ROLE_USER");
  
}
```



#### 4.2.2 基于JDBC的用户存储

```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  
  auth
    .jdbcAuthentication()
      .dataSource(dataSource);
  
}
```



##### 重写默认的用户查询功能

Spring Security内部的查询用户SQL:

![](https://pic.imgdb.cn/item/5f1d7c5d14195aa594c12964.jpg)



自定义查询

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
          "select username, password, enabled from Users " +
          "where username=?")
      .authoritiesByUsernameQuery(
          "select username, authority from UserAuthorities " +
          "where username=?");
  
}
```

##### 使用转码后的密码

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  
  auth
    .jdbcAuthentication()
      .dataSource(dataSource)
      .usersByUsernameQuery(
          "select username, password, enabled from Users " +
          "where username=?")
      .authoritiesByUsernameQuery(
          "select username, authority from UserAuthorities " +
          "where username=?")
      .passwordEncoder(new StandardPasswordEncoder("53cr3t");
  
}
```



加密模块包括了多个实现：

* BCryptPasswordEncoder：使用bcrypt强哈希加密；
* NoOpPasswordEncoder：不进行任何转码；
* Pbkdf2PasswordEncoder：使用PDKDF2加密；
* SCryptPasswordEncoder：使用scrypt哈希加密；
* StandardPasswordEncoder：使用SHA-256哈希加密。



​	不管使用哪一个转码器，数据库中的密码永远不会解码。登录时的策略与之相反，输入的密码按相同算法进行转码，然后和数据库做对比。



#### 4.2.3 以LDAP作为后端的用户存储

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}");
}
```

userSearchFilter和groupSearchFilter 用来基础LDAP查询提供过滤条件，分别用于搜索用户和组。默认情况下，用户和组的基础查询都是空的，表明搜索会在LDAP层级结构的根开始。可以改变这个默认行为：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase(" ou= groups")
      .groupSearchFilter(" member={ 0}");
}
```



##### 配置密码比对

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase(" ou= groups")
      .groupSearchFilter(" member={ 0}")
      .passwordCompare();
}
```

​	默认情况下，表单中提供的密码将会与LDAP条目中的userPassword属性进行比对。

​	可以通过passwordAttribute（）方法来声明密码属性的名称:



```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase("ou= groups")
      .groupSearchFilter("member={ 0}")
      .passwordCompare()
      .passwordEncoder(new BCryptPasswordEncoder())
      .passwordAttribute("passcode");
}
```



##### 引用远程的LDAP服务器



```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase("ou= groups")
      .groupSearchFilter("member={ 0}")
      .passwordCompare()
      .passwordEncoder(new BCryptPasswordEncoder())
      .passwordAttribute("passcode")
      .contextSource()
      .url(" ldap:// tacocloud. com: 389/ dc= tacocloud, dc= com");
}

```



##### 配置嵌入式的LDAP服务器

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase("ou= groups")
      .groupSearchFilter("member={ 0}")
      .passwordCompare()
      .passwordEncoder(new BCryptPasswordEncoder())
      .passwordAttribute("passcode")
      .contextSource()
      .url(" ldap:// tacocloud. com: 389/ dc= tacocloud, dc= com")
      .contextSource()
      .root(" dc= tacocloud, dc= com");
}

```

​	LDAP服务器启动时，尝试在类路径下寻找LDIF文件来加载数据。LDIF(LDAP Data Interchange Format, LDAP数据交换格式)是以文本文件展示LDAP数据的标准方式。每条记录可以有一行或多行，每行包含一个name:value配对信息。记录之间用空行进行分割。

​	如果不想从整个根路径下搜索，可以通过ldif()方法来明确指定加载那个LDIF文件

```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {
  auth
    .ldapAuthentication()
      .userSearchFilter("(uid={0})")
      .groupSearchFilter("member={0}")
      .groupSearchBase("ou= groups")
      .groupSearchFilter("member={ 0}")
      .passwordCompare()
      .passwordEncoder(new BCryptPasswordEncoder())
      .passwordAttribute("passcode")
      .contextSource()
      .url(" ldap:// tacocloud. com: 389/ dc= tacocloud, dc= com")
      .contextSource()
      .root(" dc= tacocloud, dc= com")
      .ldif(" classpath: users. ldif");
}

```

如下就是一个ldif文件

![](https://pic.imgdb.cn/item/5f1d830914195aa594c89d61.jpg)

![](https://pic.imgdb.cn/item/5f1d831514195aa594c8ad8b.jpg)



### 4.2.4 自定义用户认证

##### 定义用户领域对象和持久化

```java
@Entity
@Data
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@RequiredArgsConstructor
public class User implements UserDetails {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  
  private final String username;
  private final String password;
  private final String fullname;
  private final String street;
  private final String city;
  private final String state;
  private final String zip;
  private final String phoneNumber;
  
  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
  }

  @Override
  public boolean isAccountNonExpired() {
    return true;
  }

  @Override
  public boolean isAccountNonLocked() {
    return true;
  }

  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }

  @Override
  public boolean isEnabled() {
    return true;
  }

}
```

实现了UserDetails接口



定义Repository

```java
public interface UserRepository extends CrudRepository<User, Long> {

  User findByUsername(String username);
  
}
```



##### 创建用户详情服务

```java
@Service
public class UserRepositoryUserDetailsService 
        implements UserDetailsService {

  private UserRepository userRepo;

  @Autowired
  public UserRepositoryUserDetailsService(UserRepository userRepo) {
    this.userRepo = userRepo;
  }
  
  @Override
  public UserDetails loadUserByUsername(String username)
      throws UsernameNotFoundException {
    User user = userRepo.findByUsername(username);
    if (user != null) {
      return user;
    }
    throw new UsernameNotFoundException(
                    "User '" + username + "' not found");
  }

}
```



在Security配置中配置用户详情服务



```java
@Override
protected void configure(AuthenticationManagerBuilder auth)
    throws Exception {

  auth
    .userDetailsService(userDetailsService)
    .passwordEncoder(encoder());
  
}
```

##### 注册用户

略



### 4.3 保护Web请求

配置安全性规则

* 为某个请求提供服务前，先满足特定条件
* 配置自定义的登录页
* 支持用户退出应用
* 预防跨站请求伪造

#### 4.3.1 保护请求

![](https://pic.imgdb.cn/item/5f20498214195aa594320c52.jpg)





![](https://pic.imgdb.cn/item/5f2048ad14195aa594313130.jpg)

![](https://pic.imgdb.cn/item/5f2048d414195aa59431598a.jpg)



还支持access方法来扩展

![](https://pic.imgdb.cn/item/5f20491f14195aa59431a303.jpg)

![](https://pic.imgdb.cn/item/5f20492a14195aa59431ad1a.jpg)

```java
  http
      .authorizeRequests()
        .antMatchers("/design", "/orders")
          .access("hasRole('ROLE_USER')")
        .antMatchers("/", "/**").access("permitAll")
```



#### 4.3.2 自定义登录页

```java
.and()
  .formLogin()
    .loginPage("/login")
```

视图控制器

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

  // tag::customLoginViewController[]
  @Override
  public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("home");
    registry.addViewController("/abc").setViewName("home");
    registry.addViewController("/login");
  }
  // end::customLoginViewController[]

}
```

html略

自定义路径和输入域的名称

![](https://pic.imgdb.cn/item/5f204cec14195aa5943663e8.jpg)



指定默认的成功页。

![](https://pic.imgdb.cn/item/5f204d2814195aa59436a08d.jpg)



#### 4.3.3 退出

```java
.and()
  .logout()
    .logoutSuccessUrl("/")
```



#### 4.3.4 防止跨站请求伪造

​	为了防止跨域请求伪造，可以在展现表单的时候生成一个CSRF token，并且放在隐藏域中，临时存储起来。提交表单时和其他表单数据一起发送至服务端与最初的token进行对比。匹配才允许处理。

​	最好不要禁用，如果需要禁用

```
.and()
.csrf()
.disable()
```

### 4.4 了解用户是谁

常用方式如下：

* 注入principal对象到控制器方法中
* 注入Authentication对象到控制器方法中
* 使用SecurityContextHolder来获取安全上下文；
* 使用@AuthenticationPrinciple注解来标注方法。

### 4.5 小结

略

## 第5章 使用配置属性

本章内容：

* 细粒度的自动配置bean
* 将配置属性应用到组件上
* 使用Spring Profile



​	Spring Boot提供了配置属性(configuration property)的方法。

### 5.1 细粒度的自动配置

​	在Spring中有两种（但相关）的配置：

* bean装配：声明在Spring应用上下文中创建哪些应用组件以及它们之间如何相互注入的配置。
* 属性注入：设置Spring应用上下文中bean的值的配置。



![](https://pic.imgdb.cn/item/5f2183a914195aa5949aa72d.jpg)



#### 5.1.1 理解Spring的环境抽象

​	Spring环境会拉取多个属性源：

* JVM系统属性；
* 操作系统环境变量；
* 命令行参数
* 应用配置文件

![](https://pic.imgdb.cn/item/5f21841914195aa5949b18fa.jpg)



​	比如希望Servlet用另一个端口监听请求，而不再使用8080。在application.properties中

```
server.port=9090
```

或者application.yml

```
server:
	port: 9090
```

如果喜欢在外部配置该属性，可以在命令行参数添加

```
$ java -jar tarcloud-0.0.5-SNAPSHOT.jar --server.port=9090
```

如果希望应用始终在一个特定的端口启动

```
$ export SERVER_PORT=9090
```

#### 5.1.2 配置数据源

​	就是配置使用什么数据库或者数据源。

​	此处略。

#### 5.1.3 配置嵌入式服务器

​	如果将server.port设置为0不会真的在0这个端口启动，而是任选一个可用端口。保证不会和硬件编码的端口号冲突。

​	底层服务器的配置不仅仅局限于一个端口，比如常用给的一项设置就是让它处理HTTPS请求。为了实现这一点，首先要用JDK的keytool命令行工具生成keystore

![](https://pic.imgdb.cn/item/5f2188ff14195aa5949df844.jpg)

​	然后设置一些属性，以便在嵌入式服务器中启动HTTPS。但是不如以下方便：

![](https://pic.imgdb.cn/item/5f21894514195aa5949e2125.jpg)

#### 5.1.4 配置日志

​	Spring Boot默认通过Logback配置日志，以Info级别写入到控制台。

​	为了控制日志的配置，可以在类路径的根目录创建一个logback.xml文件。具体怎么配置此处略。

​	不过借助Spring Boot的配置属性功能更加方便。

![](https://pic.imgdb.cn/item/5f218a3c14195aa5949e9f39.jpg)

​	具体有哪些属性的配置此处略，用到再看。

#### 5.1.5 使用特定的属性值

​	比如想创建一个名为greeting.welcome的属性，来源于另外一个属性。

![](https://pic.imgdb.cn/item/5f218b5a14195aa5949f26ca.jpg)



### 5.2 创建自己的配置属性

​	比如要添加一个pageSize的属性在Controller中

![](https://pic.imgdb.cn/item/5f218caf14195aa5949fc990.jpg)



需要注解@ConfigurationProperties

然后在yml中或者在环境变量中，设置该属性。

![](https://pic.imgdb.cn/item/5f218ce314195aa5949fe00e.jpg)



#### 5.2.1 定义配置属性的持有者

​	@ConfigurationProperties通常会放到一种特定类型的Bean中

```java
@Component
@ConfigurationProperties(prefix="taco.orders")
@Data
// end::notValidated[]
@Validated
//tag::notValidated[]
public class OrderProps {
  
//end::notValidated[]
  @Min(value=5, message="must be between 5 and 25")
  @Max(value=25, message="must be between 5 and 25")
//tag::notValidated[]
  private int pageSize = 20;

}
```

然后注入到Controller中即可



#### 5.2.2 声明配置属性元数据



​	缺少元数据，会出现警告。

![](https://pic.imgdb.cn/item/5f22c67514195aa594300dc5.jpg)

​	创建自定义配置属性的元数据，需要在WEB-INF下创建一个additionnal-spring-configuration-metadata.json文件。

![](https://pic.imgdb.cn/item/5f22c70014195aa594304c81.jpg)

![](https://pic.imgdb.cn/item/5f22c70f14195aa5943051de.jpg)

​	利用ide的一些快捷功能，可以快速创建元数据文件。

### 5.3  使用profile进行配置

​	应用部署到不同环境时，数据库的细节在开发环境和质量保证环境中可能就不相同。



#### 5.3.1  定义特定profile的属性

文件名称要遵守如下的约定：

​	application-{profile名}.yml

​	比如application-prod.yml



#### 5.3.2 激活profile

​	在application.yml这样设置

![](https://pic.imgdb.cn/item/5f22cbf014195aa594323f88.jpg)

​	这样设置比较糟糕。

​	在生产环境可以这样设置

![](https://pic.imgdb.cn/item/5f22cc2a14195aa594325955.jpg)



​	如果想激活多个profile

![](https://pic.imgdb.cn/item/5f22cc4314195aa59432631b.jpg)

#### 5.3.3 使用profile条件化地创建bean

​	在Bean上使用@Profile注解

​	

```
@Profile("dev")

@Profile({"dev", "qa"})

@Profile("! prod")
```



# 第2部分 Spring集成

## 第6章 创建REST服务

​	本章内容：

* 在Spring MVC中定义REST端点
* 启用超链接REST资源
* 自动化基于repository的REST端点

### 6.1  编写RESTful控制器

##### 6.1.1 从服务器中检索数据

​	用@RestController修饰Controller。

​	它会告诉Spring，所有处理器方法的返回值都要直接写入响应体中，而不是将值放入模型中并传递给一个视图以便进行渲染。

​	如果不加该注解，那么需要为每个方法都添加@ResponseBody注解。

​	另外一个方案是返回ResponseEntity对象。

##### 6.1.2 发送数据到服务端

略

##### 6.1.3 在服务器上更新数据

略

##### 6.1.4 删除服务器上的数据

略

### 6.2 启用超媒体

​	超媒体作为应用状态引起（HATEOAS）是一种创建自描述API的方式。

​	如果客户端想要某个数据，URL就要以http://或者https://前缀然后添加主机名。

​	如果启用了超媒体功能，API将会描述自己的URL，减轻客户端对其硬编码的痛苦。

​	![](https://pic.imgdb.cn/item/5f22d1b914195aa594352ac8.jpg)



​	每个元素都包含了一个self连接，来引用该资源。整个列表还有一个recents属性，来引用该API自身。

​	如果要启用超媒体功能，添加如下依赖。

![](https://pic.imgdb.cn/item/5f22d24c14195aa59435b5eb.jpg)



##### 6.2.1 添加超链接

​	Spring HATEOAS提供了两个主要类型来表示超链接资源：Resource和Resources。

​	![](https://pic.imgdb.cn/item/5f22d29f14195aa59435e928.jpg)



这么写，会返回如下json片段

![](https://pic.imgdb.cn/item/5f22d2c714195aa59435fb8d.jpg)



​	但是直接传递localhost比较糟糕。使用构建者ControllerLinkBuilder效果更好，它可以自动探知主机名是什么。

​	![](https://pic.imgdb.cn/item/5f22d31814195aa594361cac.jpg)

或者

![](https://pic.imgdb.cn/item/5f22d33f14195aa594362c92.jpg)

![](https://pic.imgdb.cn/item/5f22d36514195aa594363c30.jpg)

##### 6.2.2 创建资源装配器

​	将资源装配定义为一个装配的工具类。

​	代码略



##### 6.2.3 命名嵌套式的关联关系

​	@Relation注解可以帮助我们消除JSON字段名和Java代码定义的资源集合之间的耦合。

​	![](https://pic.imgdb.cn/item/5f22d4ab14195aa59436bf5f.jpg)

​	

​	可以指定在Resource对象中引用TacoResource对象列表时它应该被命名为tacos，如果是单个引用，名称将会是taco。

### 6.3 启用数据后端服务

​	Spring Data Rest可以为Spring Data创建的Repository自动生成REST API。

​	![](https://pic.imgdb.cn/item/5f22d54414195aa59436faa6.jpg)

​	似懂非懂，但是感觉用不上。

#### 6.3.1 调整资源路径和关系名称

​	![](https://pic.imgdb.cn/item/5f22d8af14195aa594385d91.jpg)

#### 6.3.2 分页和排序

略

#### 6.3.3  添加自定义的端点

​	@RepositoryRestController

#### 6.3.4 为Spring Data端点添加自定义的超链接

略



## 第7章 消费REST服务

​	本章内容：

* 使用RestTempldate消费REST API
* 使用Traverson导航超媒体API



​	Spring可以采用多种方式来消费REST API，包括:

* RestTemplate: Spring核心框架提供的简单、同步REST客户端
* Traverson：Spring HATEOAS提供的支持超链接、同步的REST客户端，其灵感来源于同名的JavaScript库。
* WebClient：Spring 5所引入的反应式、异步REST客户端。

### 7.1 使用RestTemplate消费Rest端点



![](https://pic.imgdb.cn/item/5f2432e614195aa594b76c30.jpg)

![](https://pic.imgdb.cn/item/5f24331714195aa594b77b11.jpg)

![](https://pic.imgdb.cn/item/5f24332914195aa594b780b8.jpg)



​	要是用RestTemplate可以new一个实例或者注入到Bean中。

![](https://pic.imgdb.cn/item/5f24337214195aa594b79cc8.jpg)

#### 7.1.1 GET资源

可变参数列表

![](https://pic.imgdb.cn/item/5f24360614195aa594b8b611.jpg)



MAP指定变量

![](https://pic.imgdb.cn/item/5f2436b614195aa594b8eaca.jpg)

![](https://pic.imgdb.cn/item/5f2436c314195aa594b8efec.jpg)



UriComponentBuilder

![](https://pic.imgdb.cn/item/5f2436d814195aa594b8f87b.jpg)



GetForEntity和GetForObject很像

![](https://pic.imgdb.cn/item/5f24372714195aa594b915ba.jpg)

#### 7.1.2 PUT资源

与getForObject以及getForEntity类似。

方法是put。

#### 7.1.3 DELETE资源

方法是delete，使用方法同上。

#### 7.1.4 POST资源

postForEntity

postForObject

postForLocation

最后不是返回资源本身而是新创建资源的URI。

### 7.2  使用Traverson导航REST API

略



### 第8章 发送异步消息

略




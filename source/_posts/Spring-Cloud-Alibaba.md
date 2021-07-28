![](https://pic.imgdb.cn/item/60bed7a7844ef46bb2884323.jpg)title: Spring Cloud Alibaba
date: 2021-06-07 11:18:14
tags:

# 微服务解决方案之Spring Cloud

## 什么是Spring Cloud

​	Spring Cloud提供了一些可以让开发者快速构建微服务应用的工具，比如配置管理、服务发现、熔断、智能路由等，这些服务可以在任何分布式环境下很好地工作。Spring Cloud主要致力于解决如下问题：

* Distributed/versioned configuration，分布式及版化配置。
* Service registration and discovery，服务注册与发现。
* Routing，服务路由。
* Service-to-service calls，服务调用。
* Load balancing，负载均衡。
* Circuit Breakers，断路器。
* Global locks，全局锁。
* Leadership election and cluster state，Leader选举及集群状态。
* Distributed messaging，分布式消息。

### Spring Cloud Alibaba

* Sentinel，流量控制和服务降级。
* Nacos，服务注册与发现。
* Nacos，分布式配置中心。
* RocketMQ，消息驱动。
* Seate，分布式事务。
* Dubbo，RPC通信。
* OSS，阿里云对象存储（收费的云服务）。

# Spring Cloud的核心之Spring Boot

## 重新认识Spring Boot

**Ioc**

​	IoC（控制反转）实际上就是把对象的生命周期托管到Spring容器中，而反转是指对象的获取方式被反转了，这个概念可能不是很好理解，咱们通过两张图来了解一下IoC的作用。图3-1表示的是传统意义上对象的创建方式，客户端类如果需要用到User及UserDetail，需要通过new来构建，这种方式会使代码之间的耦合度非常高。
​	当使用Spring IoC容器之后，客户端类不需要再通过new来创建这些对象，如图3-2所示，在图3-1的基础上增加了IoC容器后，客户端类获得User及UserDetail对象实例时，不再通过new来构建，而是直接从IoC容器中获得。那么Spring IoC容器中的对象是什么时候构建的呢？在早期的Spring中，主要通过XML的方式来定义Bean，Spring会解析XML文件，把定义的Bean装载到IoC容器中。

![](https://pic.imgdb.cn/item/60bde8ce844ef46bb2a2b429.jpg)

**DI**

​	DI（Dependency Inject），也就是依赖注入，简单理解就是IoC容器在运行期间，动态地把某种依赖关系注入组件中。为了彻底搞明白DI的概念，我们继续看一下图3-2。在Spring配置文件中描述了两个Bean，分别是User和UserDetail，这两个对象从配置文件上来看是没有任何关系的，但实际上从类的关系图来看，它们之间存在聚合关系。如果我们希望这个聚合关系在IoC容器中自动完成注入，也就是像这段代码一样，通过user.getUserDetail来获得UserDetail实例，该怎么做呢？

![](https://pic.imgdb.cn/item/60bde923844ef46bb2a6b7ed.jpg)

### 装配升级变化

* XML配置形式的变化
  * 使用JavaConfig形式之后，只需要使用＠Configuration注解即可，它等同于XML的配置形式：
* Bean装载方式的变化
  * 基于JavaConfig的配置形式，可以通过＠Bean注解来将一个对象注入IoC容器中，默认情况下采用方法名称作为该Bean的id。
* 依赖注入的变化
* 其他配置的变化
  * ＠ComponentScan对应XML形式的`<context：component-scan base-package=""/>`，它会扫描指定包路径下带有＠Service、＠Repository、＠Controller、＠Component等注解的类，将这些类装载到IoC容器。
  * ＠Import对应XML形式的`<import resource=""/>`，导入其他的配置文件。

### Spring Boot的价值

**约定优于配置**

​	约定优于配置（Convention Over Configuration）是一种软件设计范式，目的在于减少配置的数量或者降低理解难度，从而提升开发效率。需要注意的是，它并不是一种新的思想，实际上从我们开始接触Java依赖，就会发现很多地方都有这种思想的体现。比如，数据库中表名的设计对应到Java中实体类的名字，就是一种约定，我们可以从这个实体类的名字知道它对应数据库中哪张表。再比如，每个公司都会有自己的开发规范，开发者按照开发规范可以在一定程度上减少Bug的数量，增加可读性和可维护性。

约定优于配置的思想主要体现在以下方面（包括但不限于）：

* Maven目录结构的约定。
* Spring Boot默认的配置文件及配置文件中配置属性的约定。
* 对于Spring MVC的依赖，自动依赖内置的Tomcat容器。
* 对于Starter组件自动完成装配。

**Spring Boot的核心**

* Starter组件，提供开箱即用的组件。
* 自动装配，自动根据上下文完成Bean的装配。
* Actuator，Spring Boot应用的监控。
* Spring Boot CLI，基于命令行工具快速构建Spring Boot应用。

## Spring Boot自动装配的原理

### 自动装配的实现

​	自动装配在Spring Boot中是通过＠EnableAutoConfiguration注解来开启的，这个注解的声明在启动类注解＠SpringBootApplication内。

​	进入＠SpringBootApplication注解，可以看到＠EnableAutoConfiguration注解的声明。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
...
}
```

​	Spring 3.1版本就已经支持＠Enable注解了，它的主要作用把相关组件的Bean装配到IoC容器中。＠Enable注解对JavaConfig的进一步完善，为使用Spring Framework的开发者减少了配置代码量，降低了使用的难度。比如常见的＠Enable注解有＠EnableWebMvc、＠EnableScheduling等。
在前面的章节中讲过，如果基于JavaConfig的形式来完成Bean的装载，则必须要使用＠Configuration注解及＠Bean注解。而＠Enable本质上就是针对这两个注解的封装，所以大家如果仔细关注过这些注解，就不难发现这些注解中都会携带一个＠Import注解，比如＠EnableScheduling：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)
@Documented
public @interface EnableScheduling {

}

```

​	使用＠Enable注解后，Spring会解析到＠Import导入的配置类，从而根据这个配置类中的描述来实现Bean的装配。

### EnableAutoConfiguration

​	进入＠EnableAutoConfiguration注解里，可以看到除＠Import注解之外，还多了一个＠AutoConfigurationPackage注解（它的作用是把使用了该注解的类所在的包及子包下所有组件扫描到Spring IoC容器中）。并且，＠Import注解中导入的并不是一个Configuration的配置类，而是一个AutoConfigurationImportSelector类。从这一点来看，它就和其他的＠Enable注解有很大的不同。

### AutoConfigurationImportSelector

​	AutoConfigurationImportSelector实现了ImportSelector，它只有一个selectImports抽象方法，并且返回一个String数组，在这个数组中可以指定需要装配到IoC容器的类，当在＠Import中导入一个ImportSelector的实现类之后，会把该实现类中返回的Class名称都装载到IoC容器中。

```java
public interface ImportSelector {

	@Nullable
	default Predicate<String> getExclusionFilter() {
		return null;
	}

}
```

​	和＠Configuration不同的是，ImportSelector可以实现批量装配，并且还可以通过逻辑处理来实现Bean的选择性装配，也就是可以根据上下文来决定哪些类能够被IoC容器初始化。接下来通过一个简单的例子带大家了解ImportSelector的使用。

![](https://pic.imgdb.cn/item/60bdecae844ef46bb2d36d3d.jpg)

![](https://pic.imgdb.cn/item/60bdecc7844ef46bb2d497bc.jpg)

​	这种实现方式相比＠Import（*Configuration.class）的好处在于装配的灵活性，还可以实现批量。比如GpImportSelector还可以直接在String数组中定义多个Configuration类，由于一个配置类代表的是某一个技术组件中批量的Bean声明，所以在自动装配这个过程中只需要扫描到指定路径下对应的配置类即可。

### 自动装配原理分析

​	定位到AutoConfigurationImportSelector中的selectImports方法，它是ImportSelector接口的实现，这个方法中主要有两个功能：

* AutoConfigurationMetadataLoader.loadMetadata从META-INF/spring-autoconfigure-metadata.properties中加载自动装配的条件元数据，简单来说就是只有满足条件的Bean才能够进行装配。
* 收集所有符合条件的配置类autoConfigurationEntry.getConfigurations（），完成自动装配。

```java
	@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

​	需要注意的是，在AutoConfigurationImportSelector中不执行selectImports方法，而是通过ConfigurationClassPostProcessor中的processConfigBeanDefinitions方法来扫描和注册所有配置类的Bean，最终还是会调用getAutoConfigurationEntry方法获得所有需要自动装配的配置类。
我们重点分析一下配置类的收集方法getAutoConfigurationEntry，结合之前Starter的作用不难猜测到，这个方法应该会扫描指定路径下的文件解析得到需要装配的配置类，而这里面用到了SpringFactoriesLoader，这块内容后续随着代码的展开再来讲解。简单分析一下这段代码，它主要做几件事情：

* getAttributes获得＠EnableAutoConfiguration注解中的属性exclude、excludeName等。
* getCandidateConfigurations获得所有自动装配的配置类，后续会重点分析。
* removeDuplicates去除重复的配置项。
* getExclusions根据＠EnableAutoConfiguration注解中配置的exclude等属性，把不需要自动装配的配置类移除。
* fireAutoConfigurationImportEvents广播事件。
* 最后返回经过多层判断和过滤之后的配置类集合。

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

​	它先获得所有的配置类，通过去重、exclude排除等操作，得到最终需要实现自动装配的配置类。这里需要重点关注的是getCandidateConfigurations，它是获得配置类最核心的方法。

```java
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

​	这里用到了SpringFactoriesLoader，它是Spring内部提供的一种约定俗成的加载方式，类似于Java中的SPI。简单来说，它会扫描classpath下的META-INF/spring.factories文件，spring.factories文件中的数据以Key=Value形式存储，而SpringFactoriesLoader.loadFactoryNames会根据Key得到对应的value值。因此，在这个场景中，Key对应为EnableAutoConfiguration，Value是多个配置类，也就是getCandidateConfigurations方法所返回的值。

![](https://pic.imgdb.cn/item/60bdef33844ef46bb2f2fa44.jpg)

​	打开RabbitAutoConfiguration，可以看到，它就是一个基于JavaConfig形式的配置类。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
    ...
}
```

​	除了基本的＠Configuration注解，还有一个＠ConditionalOnClass注解，这个条件控制机制在这里的用途是，判断classpath下是否存在RabbitTemplate和Channel这两个类，如果是，则把当前配置类注册到IoC容器。另外，＠EnableConfigurationProperties是属性配置，也就是说我们可以按照约定在application.properties中配置RabbitMQ的参数，而这些配置会加载到RabbitProperties中。实际上，这些东西都是Spring本身就有的功能。
​	至此，自动装配的原理基本上就分析完了，简单来总结一下核心过程：

* 通过＠Import（AutoConfigurationImportSelector）实现配置类的导入，但是这里并不是传统意义上的单个配置类装配。
* AutoConfigurationImportSelector类实现了ImportSelector接口，重写了方法selectImports，它用于实现选择性批量配置类的装配。
* 通过Spring提供的SpringFactoriesLoader机制，扫描classpath路径下的META-INF/spring.factories，读取需要实现自动装配的配置类。
* 通过条件筛选的方式，把不符合条件的配置类移除，最终完成自动装配。

### ＠Conditional条件装配

​	＠Conditional是Spring Framework提供的一个核心注解，这个注解的作用是提供自动装配的条件约束，一般与＠Configuration和＠Bean配合使用。
​	简单来说，Spring在解析＠Configuration配置类时，如果该配置类增加了＠Conditional注解，那么会根据该注解配置的条件来决定是否要实现Bean的装配。

#### ＠Conditional的使用

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

   /**
    * All {@link Condition} classes that must {@linkplain Condition#matches match}
    * in order for the component to be registered.
    */
   Class<? extends Condition>[] value();

}
```

​	Condition是一个函数式接口，提供了matches方法，它主要提供一个条件匹配规则，返回true表示可以注入Bean，反之则不注入。

我们接下来基于＠Conditional实现一个条件装配的案例。

* 自定义一个Condition，逻辑很简单，如果当前操作系统是Windows，则返回true，否则返回false。

![](https://pic.imgdb.cn/item/60be0fb9844ef46bb2a5ff26.jpg)

* 创建一个配置类，装载一个BeanClass（自定义的一个类)。

![](https://pic.imgdb.cn/item/60be101f844ef46bb2ab7a7d.jpg)

​	在BeanClass的bean声明方法中增加＠Conditional（GpCondition.class），其中具体的条件是我们自定义的GpCondition类。

#### Spring Boot中的＠Conditional

* ConditionalOnBean/ConditionalOnMissingBean：容器中存在某个类或者不存在某个Bean时进行Bean装载。
* ConditionalOnClass/ConditionalOnMissingClass：classpath下存在指定类或者不存在指定类时进行Bean装载。
* ConditionalOnCloudPlatform：只有运行在指定的云平台上才加载指定的Bean。
* ConditionalOnExpression：基于SpEl表达式的条件判断。
* ConditionalOnJava：只有运行指定版本的Java才会加载Bean。
* ConditionalOnJndi：只有指定的资源通过JNDI加载后才加载Bean。
* ConditionalOnWebApplication/ConditionalOnNotWebApplication：如果是Web应用或者不是Web应用，才加载指定的Bean。
* ConditionalOnProperty：系统中指定的对应的属性是否有对应的值。
* ConditionalOnResource：要加载的Bean依赖指定资源是否存在于classpath中。
* ConditionalOnSingleCandidate：只有在确定了给定Bean类的单个候选项时才会加载Bean。



​	这些注解只需要添加到＠Configuration配置类的类级别或者方法级别，然后根据每个注解的作用来传参就行。下面演示几种注解类的使用。

![](https://pic.imgdb.cn/item/60be14cf844ef46bb2eceaa4.jpg)

​	在application.properties或application.yml文件中当gp.bean.enable=true时才会加载ConditionConfig这个Bean，如果没有匹配上也会加载，因为matchIfMissing=true，默认值是false。

![](https://pic.imgdb.cn/item/60be15de844ef46bb2fc0b6d.jpg)

​	当Spring IoC容器中存在GpBean时，才会加载ConditionConfig。

### spring-autoconfigure-metadata

​	除了＠Conditional注解类，在Spring Boot中还提供了spring-autoconfigure-metadata.properties文件来实现批量自动装配条件配置。

​	它的作用和＠Conditional是一样的，只是将这些条件配置放在了配置文件中。下面这段配置来自spring-boot-autoconfigure.jar包中的/META-INF/spring-autoconfigure-metadata.properties文件。

![](https://pic.imgdb.cn/item/60be1770844ef46bb2140f44.jpg)

​	同样，这种形式也是“约定优于配置”的体现，通过这种配置化的方式来实现条件过滤必须要遵循两个条件：

* 配置文件的路径和名称必须是/META-INF/spring-autoconfigure-metadata.properties。
* 配置文件中key的配置格式：自动配置类的类全路径名.条件=值。

​	这种配置方法的好处在于，它可以有效地降低Spring Boot的启动时间，通过这种过滤方式可以减少配置类的加载数量，因为这个过滤发生在配置类的装载之前，所以它可以降低Spring Boot启动时装载Bean的耗时。

## 手写实现一个Starter

* 涉及相关组件的Jar包依赖。
* 自动实现Bean的装配。
* 自动声明并且加载application.properties文件中的属性配置。

### 起名规范

* 官方命名的格式为：spring-boot-starter-模块名称，比如spring-boot-starter-web。
* 自定义命名格式为：模块名称-spring-boot-starter，比如mybatis-spring-boot-starter。

### 实现基于Redis的Starter

* 创建一个工程，命名为redis-spring-boot-starter。
* 添加Jar包依赖，Redisson提供了在Java中操作Redis的功能，并且基于Redis的特性封装了很多可直接使用的场景，比如分布式锁。

![](https://pic.imgdb.cn/item/60be1a0a844ef46bb23e4cbb.jpg)

* 定义属性类，实现在application.properties中配置Redis的连接参数，由于只是一个简单版本的Demo，所以只简单定义了一些必要参数。另外＠ConfigurationProperties这个注解的作用是把当前类中的属性和配置文件（properties/yml）中的配置进行绑定，并且前缀是gp.redisson。

![](https://pic.imgdb.cn/item/60be1a2d844ef46bb240a547.jpg)

* 定义需要自动装配的配置类，主要就是把RedissonClient装配到IoC容器，值得注意的是＠ConditionalOnClass，它表示一个条件，在当前场景中表示的是：在classpath下存在Redisson这个类的时候，RedissonAutoConfiguration才会实现自动装配。另外，这里只演示了一种单机的配置模式，除此之外，Redisson还支持集群、主从、哨兵等模式的配置，大家有兴趣的话可以基于当前案例去扩展，建议使用config.fromYAML方式，直接加载配置完成不同模式的初始化，这会比根据不同模式的判断来实现配置化的方式更加简单。

![](https://pic.imgdb.cn/item/60be1a5e844ef46bb2440328.jpg)

* 在resources下创建META-INF/spring.factories文件，使得Spring Boot程序可以扫描到该文件完成自动装配，key和value对应如下：

![](https://pic.imgdb.cn/item/60be1a7e844ef46bb2463aa8.jpg)

#  微服务架构下的服务治理

* 如何协调线上运行的服务，以及保障服务的高可用性。
* 如何根据不同服务的访问情况来合理地调控服务器资源，提高机器的利用率。
* 线上出现故障时，如何动态地对故障业务做降级、流量控制等。
* 如何动态地更新服务中的配置信息，比如限流阈值、降级开关等。
* 如何实现大规模服务集群所带来的服务地址的管理和服务上下线的动态感知。

## 如何理解Apache Dubbo

​	Apache Dubbo是一个分布式服务框架，主要实现多个系统之间的高性能、透明化调用，简单来说它就是一个RPC框架，但是和普通的RPC框架不同的是，它提供了服务治理功能，比如服务注册、监控、路由、容错等。

​	促使Apache Dubbo框架产生的原因有两个：

* 在大规模服务化之后，服务越来越多，服务消费者在调用服务提供者的服务时，需要在配置文件中维护服务提供者的URL地址，当服务提供者出现故障或者动态扩容时，所有相关的服务消费者都需要更新本地配置的URL地址，这种维护成本非常高。这个时候，实现服务的上下动态线感知及服务地址的动态维护就显得非常重要了。
* 随着用户的访问量增大，后端服务为了支撑更大的访问量，会通过增加服务器来扩容。但是，哪些服务要扩容，哪些服务要缩容，需要一个判断依据，也就是说需要知道每个服务的调用量及响应时间，这个时候，就需要有一种监控手段，使用监控的数据作为容量规划的参考值，从而实现根据不同服务的访问情况来合理地调控服务器资源，提高机器的利用率。

![](https://pic.imgdb.cn/item/60beb870844ef46bb2991589.jpg)

## Apache Dubbo实现远程通信

​	创建两个普通的Maven工程，分别为order-service和user-service，代表订单服务和用户服务，这两个服务之间在实际业务场景中会存在相互依赖的情况，比如订单服务中的某个功能可能需要查询用户信息时，就需要调用用户服务指定的接口来完成。

**user-service的实现流程**

* 在user-service服务中定义了两个模块，分别为user-api和user-provider，前者用来定义当前服务对外提供的接口，这个模块会部署到Maven的远程私服上，便于服务调用者依赖；后者是针对这个接口的实现，该实现会独立部署在服务器上。

![](https://pic.imgdb.cn/item/60beb940844ef46bb2a49d3e.jpg)

* 在user-api中定义一个接口，执行mvn install将其打包成Jar包安装到本地仓库，本地环境的其他项目就可以找到该依赖，当然，如果自己搭建了私服，可以通过mvn deploy发布。

![](https://pic.imgdb.cn/item/60beb955844ef46bb2a5ccdd.jpg)

* 在user-provider中编写实现，这里需要注意的是，user-provider中需要用到user-api中定义的IUserService接口，所以需要先添加user-api的maven dependency依赖。

![](https://pic.imgdb.cn/item/60beb972844ef46bb2a768f0.jpg)

* 添加Dubbo依赖

![](https://pic.imgdb.cn/item/60beb986844ef46bb2a88be7.jpg)

* 创建配置文件resources/META-INF/spring/user-provider.xml，把服务发布到网络上，让其他进程可以访问。因为Dubbo采用了Spring配置的扩展来实现透明化的服务发布和服务消费，所以它的配置基本上和以往通过XML形式描述Bean差不多。
  * dubbo:application用来描述提供方的应用信息，比如应用名称、维护人、版本等，其中应用名称是必填项。开发者或者运维人员可以通过监控平台查看这些信息来更快速地定位和解决问题。
  * dubbo:registry 配置注册中心的地址，如果不需要注册中心，可以设置为N/A。Dubbo支持多种注册中心，比如ZooKeeper、Nacos等。
  * dubbo:service 描述需要发布的服务接口，也就是这个接口可供本网络上的其他进程访问。interface表示定义的接口，ref表示这个接口的实现。

![](https://pic.imgdb.cn/item/60beb9e8844ef46bb2adf910.jpg)

* 加载Spring的XML文件，可以通过ClassPathXmlApplicationContext来完成加载启动的过程，也可以通过Main.main（args）来启动。两者在本质上没有区别，只是Dubbo做了一层封装，简化了开发者的使用。

![](https://pic.imgdb.cn/item/60beba06844ef46bb2afa5ec.jpg)

* 启动之后，可以在控制台的日志中看到如下信息，说明服务已经发布成功，而且还打印了Dubbo发布的地址dubbo://192.168.13.1:20880/com.gupaoedu.book.dubbo.IUserService，这个地址是一个远程通信地址，服务调用者可以基于该地址来访问该服务完成远程通信的流程。

![](https://pic.imgdb.cn/item/60beba22844ef46bb2b134a9.jpg)

**order-service的实现流程**

* 添加user-api和Dubbo的Maven依赖，前者是用户访问IUserService接口的方法，后者通过远程代理完成远程通信过程。

![](https://pic.imgdb.cn/item/60bebb99844ef46bb2c64b8b.jpg)

* 在resources/META-INF/spring/consumer.xml中配置远程服务的引用，主要关注一下dubbo：reference这个配置，它会生成一个针对当前interface的远程服务的代理，指向的远程服务地址是user-service发布的Dubbo协议的URL地址。

![](https://pic.imgdb.cn/item/60bebbf3844ef46bb2cb3ddc.jpg)

* 加载Spring配置文件，使用方式和本地Bean一样，通过从IoC容器中获取一个实例对象进行调用，需要注意的是，这里的IUserService返回的是一个代理对象，它的底层会基于网络通信来实现远程服务的调用。

##  Spring Boot集成Apache Dubbo

**服务提供方开发流程**

* 创建一个普通的Maven工程springboot-provider，并创建两个模块：sample-api和sample-provider，其中sample-provider模块是一个Spring Boot工程。
* 在sample-api模块中定义一个接口，并且通过mvn install安装到本地私服。

![](https://pic.imgdb.cn/item/60bebd9d844ef46bb2e36ff0.jpg)

* 在sample-provider中引入以下依赖，其中dubbo-spring-boot-starter是Apache Dubbo官方提供的开箱即用的组件。

![](https://pic.imgdb.cn/item/60bebdb5844ef46bb2e4df23.jpg)

* 在sample-provider中实现IHelloService，并且使用Dubbo中提供的＠Service注解发布服务。

![](https://pic.imgdb.cn/item/60bebdeb844ef46bb2e8452d.jpg)

* 在application.properties文件中添加Dubbo服务的配置信息，配置元素在前面的章节中讲过，只是换了一种配置形式。

![](https://pic.imgdb.cn/item/60bebe06844ef46bb2e9ed15.jpg)

* 启动Spring Boot，需要注意的是，需要在启动方法上添加＠DubboComponentScan注解，它的作用和Spring Framework提供的＠ComponetScan一样，只不过这里扫描的是Dubbo中提供的＠Service注解。

![](https://pic.imgdb.cn/item/60bebe3e844ef46bb2ed6a95.jpg)

* 创建一个Spring Boot项目springboot-consumer，添加Jar包依赖。

![](https://pic.imgdb.cn/item/60bebe60844ef46bb2ef9cf6.jpg)

* 在application.properties中配置项目名称。

```properties
dubbo.application.name=spring-boot-consumer
```

​	在Spring Boot启动类中，使用Dubbo提供的＠Reference注解来获得一个远程代理对象。

![](https://pic.imgdb.cn/item/60bebe9c844ef46bb2f345b4.jpg)

​	另外，官方还提供了Dubbo-Spring-Boot-Actuator模块，可以实现针对Dubbo服务的健康检查；还可以通过Endpoints实现Dubbo服务信息的查询和控制等，为生产环境中对Dubbo服务的监控提供了很好的支持。

## 快速上手ZooKeeper

### ZooKeeper安装

略

### ZooKeeper的数据结构

​	ZooKeeper树中的每个节点被称为Znode，Znode维护了一个stat状态信息，其中包含数据变化的时间和版本等。并且每个Znode可以设置一个value值，ZooKeeper并不用于通用的数据库或者大容量的对象存储，它只是管理和协调有关的数据，所以value的数据大小不建议设置得非常大，较大的数据会带来更大的网络开销。

​	ZooKeeper上的每个节点的数据都是允许读和写的，读表示获得指定Znode上的value数据，写表示修改指定Znode上的value数据。另外，节点的创建规则和文件系统中文件的创建规则类似，必须要按照层级创建。举个简单的例子，如果需要创建/node/node1/node1-1，那么必须先创建/node/node1这两个层级节点。

![](https://pic.imgdb.cn/item/60bebf4f844ef46bb2fe7b4d.jpg)

### ZooKeeper特性

​	节点类型分为

* 持久化节点，节点的数据会持久化到磁盘。
* 临时节点，节点的生命周期和创建该节点的客户端的生命周期保持一致，一旦该客户端的会话结束，则该客户端所创建的临时节点会被自动删除。
* 有序节点，在创建的节点后面会增加一个递增的序列，该序列在同一级父节点之下是唯一的。需要注意的是，持久化节点或者临时节点也是可以设置为有序节点的，也就是持久化有序节点或者临时有序节点。



​	在3.5.3版本之后，又增加了两种节点类型，分别是：

* 容器节点，当容器节点下的最后一个子节点被删除时，容器节点就会被自动删除。
* TTL节点，针对持久化节点或者持久化有序节点，我们可以设置一个存活时间，如果在存活时间之内该节点没有任何修改并且没有任何子节点，它就会被自动删除。

### Watcher机制

​	ZooKeeper提供了一种针对Znode的订阅/通知机制，也就是当Znode节点状态发生变化时或者ZooKeeper客户端连接状态发生变化时，会触发事件通知。这个机制在服务注册与发现中，针对服务调用者及时感知到服务提供者的变化提供了非常好的解决方案。

​	在ZooKeeper提供的Java API中，提供了三种机制来针对Znode进行注册监听，分别是：

* getData（），用于获取指定节点的value信息，并且可以注册监听，当监听的节点进行创建、修改、删除操作时，会触发相应的事件通知。
* getChildren（），用于获取指定节点的所有子节点，并且允许注册监听，当监听节点的子节点进行创建、修改、删除操作时，触发相应的事件通知。
* exists（），用于判断指定节点是否存在，同样可以注册针对指定节点的监听，监听的时间类型和getData（）相同。



​	Watcher事件的触发都是一次性的，比如客户端通过getData（'/node'，true）注册监听，如果/node节点发生数据修改，那么该客户端会收到一个修改事件通知，但是/node再次发生变化时，客户端无法收到Watcher事件，为了解决这个问题，客户端必须在收到的事件回调中再次注册事件。

### 常见应用场景分析

**分布式锁**

​	如果使用ZooKeeper实现分布式锁达到排他的目的，只需要用到节点的特性：临时节点，以及同级节点的唯一性。

**获得锁的过程**

​	在获得排他锁时，所有客户端可以去ZooKeeper服务器上/Exclusive_Locks节点下创建一个临时节点/lock。ZooKeeper基于同级节点的唯一性，会保证所有客户端中只有一个客户端能创建成功，创建成功的客户端获得了排他锁，没有获得锁的客户端就需要通过Watcher机制监听/Exclusive_Locks节点下子节点的变更事件，用于实时监听/lock节点的变化情况以做出反应

**释放锁的过程**

​	在获得锁的过程中，我们定义的锁节点/lock为临时节点，那么在以下两种情况下会触发锁的释放。

* 获得锁的客户端因为异常断开了和服务端的连接，基于临时节点的特性，/lock节点会被自动删除。
* 获得锁的客户端执行完业务逻辑之后，主动删除了创建的/lock节点。



​	当/lock节点被删除之后，ZooKeeper服务器会通知所有监听了/Exclusive_Locks子节点变化的客户端。这些客户端收到通知后，再次发起创建/lock节点的操作来获得排他锁。

**Master选举**

* 同一级节点不能重复创建一个已经存在的节点，这个有点类似于分布式锁的实现场景，其实Master选举的场景也是如此。假设集群中有3个节点，需要选举出Master，那么这三个节点同时去ZooKeeper服务器上创建一个临时节点/master-election，由于节点的特性，只会有一个客户端创建成功，创建成功的客户端所在的机器就成了Master。同时，其他没有创建成功的客户端，针对该节点注册Watcher事件，用于监控当前的Master机器是否存活，一旦发现Master“挂了”，也就是/master-election节点被删除了，那么其他的客户端将会重新发起Master选举操作。
* 利用临时有序节点的特性来实现。所有参与选举的客户端在ZooKeeper服务器的/master节点下创建一个临时有序节点，编号最小的节点表示Master，后续的节点可以监听前一个节点的删除事件，用于触发重新选举，如图4-3所示，client01、client02、client03三个节点去ZooKeeper Server的/master节点下创建临时有序节点，编号最小的节点client01表示Master节点，client02和client03会分别通过Watcher机制监听比自己编号小的一个节点，比如client03会监听client01-0000000001节点的删除事件、client02会监听client-03-0000000002节点的删除事件，一旦最小的节点被删除，那么在图4-3这个场景中，client-03就会被选举为Master。

![](https://pic.imgdb.cn/item/60bec0ac844ef46bb214fa9a.jpg)

## Apache Dubbo集成ZooKeeper实现服务注册

​	大规模服务化之后，在远程RPC通信过程中，会遇到两个比较尖锐的问题：

* 服务动态上下线感知。
* 负载均衡。

**服务动态上下线感知**

​	服务调用者要感知到服务提供者上下线的变化。

**负载均衡**

​	当服务提供者是由多个节点组成的集群环境时，服务调用者需要通过负载均衡算法来动态选择一台目标服务器进行远程通信。

![](https://pic.imgdb.cn/item/60bec173844ef46bb22220c8.jpg)

* 在springboot-provider项目的sample-provider模块中添加ZooKeeper相关依赖，其中curator-framework和curator-recipes是ZooKeeper的开源客户端。

![](https://pic.imgdb.cn/item/60bec1ab844ef46bb225dff9.jpg)

* 修改application.properties文件，修改dubbo.registry.address的地址为ZooKeeper服务器的地址，表示当前Dubbo服务需要注册到ZooKeeper上。

![](https://pic.imgdb.cn/item/60bec1f6844ef46bb22ae982.jpg)

* 服务调用方只需要修改application.properties，设置Dubbo服务注册中心的地址即可，当Dubbo调用方发起远程调用时，会去注册中心获取目标服务的URL地址以完成最终通信。

![](https://pic.imgdb.cn/item/60bec20b844ef46bb22c4f9d.jpg)

### ZooKeeper注册中心的实现原理

​	当Dubbo服务启动时，会去Zookeeper服务器上的/dubbo/com.gupaoedu.book.dubbo.IHelloService/providers目录下创建当前服务的URL，其中com.gupaoedu.book.dubbo.IHelloService是发布服务的接口全路径名称，providers表示服务提供者的类型，dubbo://ip:port表示该服务发布的协议类型及访问地址。其中，URL是临时节点，其他皆为持久化节点。在这里使用临时节点的好处在于，如果注册该节点的服务器下线了，那么这个服务器的URL地址就会从ZooKeeper服务器上被移除。

![](https://pic.imgdb.cn/item/60bec286844ef46bb2344bec.jpg)

​	Dubbo服务消费者启动时，会对/dubbo/com.gupaoedu.book.dubbo.IHelloService/providers节点下的子节点注册Watcher监听，这样便可以感知到服务提供方节点的上下线变化，从而防止请求发送到已经下线的服务器造成访问失败。同时，服务消费者会在dubbo/com.gupaoedu.book.dubbo.IHelloService/consumers下写入自己的URL，这样做的目的是可以在监控平台上看到某个Dubbo服务正在被哪些服务调用。最重要的是，Dubbo服务的消费者如果需要调用IHelloService服务，那么它会先去/dubbo/com.gupaoedu.book.dubbo.IHelloService/providers路径下获得所有该服务的提供方URL列表，然后通过负载均衡算法计算出一个地址进行远程访问。

​	整体来看，服务注册和动态感知的功能用到了ZooKeeper中的临时节点、持久化节点、Watcher等，回过头看前面分析的ZooKeeper的应用场景可以发现，几乎所有的场景都是基于这些来完成的。另外，不得不提的是，Dubbo还可以针对不同的情况来实现以下功能。

* 基于临时节点的特性，当服务提供者宕机或者下线时，注册中心会自动删除该服务提供者的信息。
* 注册中心重启时，Dubbo能够自动恢复注册数据及订阅请求。
* 为了保证节点操作的安全性，ZooKeeper提供了ACL权限控制，在Dubbo中可以通过dubbo.registry.username/dubbo.registry.password设置节点的验证信息。
* 注册中心默认的根节点是/dubbo，如果需要针对不同环境设置不同的根节点，可以使用dubbo.registry.group修改根节点名称。

## 实战Dubbo Spring Cloud

### 实现Dubbo服务提供方

​	spring-cloud-dubbo-sample-api、spring-cloud-dubbo-sample-provider。其中spring-cloud-dubbo-sample-api是一个普通的Maven工程，spring-cloud-dubbo-sample-provider是一个Spring Boot工程。

​	Dubbo官方推荐的做法是把服务接口打成Jar包发布到仓库上。服务调用者可以依赖该Jar包，通过接口调用方式完成远程通信。对于服务提供者来说，也需要依赖该Jar包完成接口的实现。

![](https://pic.imgdb.cn/item/60bec3e3844ef46bb24a9570.jpg)

* 在spring-cloud-dubbo-sample-api中声明接口，并执行mvn install将Jar包安装到本地仓库。

![](https://pic.imgdb.cn/item/60bec3f8844ef46bb24be87c.jpg)

* 在spring-cloud-dubbo-sample-provider中添加依赖。

![](https://pic.imgdb.cn/item/60bec42f844ef46bb24f6409.jpg)

​	依赖说明

* spring-cloud-starter：Spring Cloud核心包。
* spring-cloud-dubbo-sample-api：API接口声明。
* spring-cloud-starter-dubbo：引入Spring Cloud Alibaba Dubbo。
* spring-cloud-starter-zookeeper-discovery：基于ZooKeeper实现服务注册发现的artifactId。



​	需要注意的是，上述依赖的artifact没有指定版本，所以需要在父pom中显式声明dependencyManagement。

![](https://pic.imgdb.cn/item/60bec484844ef46bb254b79d.jpg)

* 在spring-cloud-dubbo-sample-provider中创建接口的实现类HelloServiceImpl，其中＠Service是Dubbo服务的注解，表示当前服务会发布为一个远程服务。

![](https://pic.imgdb.cn/item/60bec4a8844ef46bb256e397.jpg)

* 在application.properties中配置Dubbo相关的信息。

![](https://pic.imgdb.cn/item/60bec4d6844ef46bb259a4eb.jpg)

​	其中spring.cloud.zookeeper.discovery.register=true表示服务是否需要注册到注册中心。spring.cloud.zookeeper.connect-string表示ZooKeeper的连接字符串。

* 在启动类中声明＠DubboComponentScan注解，并启动服务。

![](https://pic.imgdb.cn/item/60bec4f4844ef46bb25b759c.jpg)

​	＠DubboComponentScan扫描当前注解所在的包路径下的＠org.apache.dubbo.config.annotation.Service注解，实现服务的发布。发布完成之后，就可以在ZooKeeper服务器上看一个/services/${project-name}`节点，这个节点中保存了服务提供方相关的地址信息。

### 实现Dubbo服务调用方

* 创建一个名为spring-cloud-dubbo-consumer的Spring Boot工程，添加如下依赖，与服务提供方所依赖的配置没什么区别。为了演示需要，增加了spring-boot-starter-web组件，表示这是一个Web项目。

![](https://pic.imgdb.cn/item/60bec555844ef46bb26159a1.jpg)

* 在application.properties文件中添加Dubbo相关配置信息。

![](https://pic.imgdb.cn/item/60bec5cc844ef46bb268ac8a.jpg)

​	配置信息和spring-cloud-dubbo-sample项目的配置信息差不多，有两个配置需要单独说明一下：

* spring.cloud.zookeeper.discovery.register=false表示当前服务不需要注册到ZooKeeper上，默认为true。
* dubbo.cloud-subscribed-services表示服务调用者订阅的服务提供方的应用名称列表，如果有多个应用名称，可以通过“，”分割开，默认值为“*”，不推荐使用默认值。当dubbo.cloud.subscribed-services为默认值时，控制台的日志中会输入一段警告信息。
* 创建HelloController类，暴露一个/say服务，来消费Dubbo服务提供者的IHelloService服务。

![](https://pic.imgdb.cn/item/60bec609844ef46bb26c5718.jpg)

## Apache Dubbo的高级应用

* 支持多种协议的服务发布，默认是dubbo://，还可以支持rest://、webservice://、thrift://等。
* 支持多种不同的注册中心，如Nacos、ZooKeeper、Redis，未来还将会支持Consul、Eureka、Etcd等。
* 支持多种序列化技术，如avro、fst、fastjson、hessian2、kryo等。

### 集群容错

**容错模式**

​	Dubbo默认提供了6种容错模式，默认为Failover Cluster。如果这6种容错模式不能满足你的实际需求，还可以自行扩展。这也是Dubbo的强大之处，几乎所有的功能都提供了插拔式的扩展。

* **Failover Cluster**，失败自动切换。当服务调用失败后，会切换到集群中的其他机器进行重试，默认重试次数为2，通过属性retries=2可以修改次数，但是重试次数增加会带来更长的响应延迟。这种容错模式通常用于读操作，因为事务型操作会带来数据重复问题。
* **Failfast Cluster**，快速失败。当服务调用失败后，立即报错，也就是只发起一次调用。通常用于一些幂等的写操作，比如新增数据，因为当服务调用失败时，很可能这个请求已经在服务器端处理成功，只是因为网络延迟导致响应失败，为了避免在结果不确定的情况下导致数据重复插入的问题，可以使用这种容错机制。
* **Failsafe Cluster**，失败安全。也就是出现异常时，直接忽略异常。
* **Failback Cluster**，失败后自动回复。服务调用出现异常时，在后台记录这条失败的请求定时重发。这种模式适合用于消息通知操作，保证这个请求一定发送成功。
* **Forking Cluster**，并行调用集群中的多个服务，只要其中一个成功就返回。可以通过forks=2来设置最大并行数。
* **Broadcast Cluster**，广播调用所有的服务提供者，任意一个服务报错则表示服务调用失败。这种机制通常用于通知所有的服务提供者更新缓存或者本地资源信息。

**配置方式**

​	配置方式非常简单，只需要在指定服务的＠Service注解上增加一个参数即可。注意，在没有特殊说明的情况下，后续代码都是基于前面的Dubbo Spring Cloud的代码进行改造的。在＠Service注解中增加cluster="failfast"参数，表示当前服务的容错方式为快速失败。

![](https://pic.imgdb.cn/item/60bec793844ef46bb28472c3.jpg)

​	在实际应用中，查询语句容错策略建议使用默认的Failover Cluster，而增删改操作建议使用Failfast Cluster或者使用Failover Cluster（retries="0"）策略，防止出现数据重复添加等其他问题！建议在设计接口的时候把查询接口方法单独做成一个接口提供查询。

### 负载均衡

​	负载均衡应该不是一个陌生的概念，在访问量较大的情况下，我们会通过水平扩容的方式增加多个节点来平衡请求的流量，从而提升服务的整体性能。简单来说，如果一个服务节点的TPS是100，那么如果增加到5个节点的集群，意味着整个集群的TPS可以达到500。

​	当服务调用者面对5个节点组成的服务提供方集群时，请求应该分发到集群中的哪个节点，取决于负载均衡算法，通过该算法可以让每个服务器节点获得适合自己处理能力的负载。负载均衡可以分为硬件负载均衡和软件负载均衡，硬件负载均衡比较常见的就是F5，软件负载均衡目前比较主流的是Nginx。

​	Dubbo中提供了4种负载均衡策略，默认负载均衡策略是random。同样，如果这4种策略不能满足实际需求，我们可以基于Dubbo中的SPI机制来扩展。

* Random LoadBalance，随机算法。可以针对性能较好的服务器设置较大的权重值，权重值越大，随机的概率也会越大。
* RoundRobin LoadBalance，轮询。按照公约后的权重设置轮询比例。
* LeastActive LoadBalance，最少活跃调用。处理较慢的节点将会收到更少的请求。
* ConsistentHash LoadBalance，一致性Hash。相同参数的请求总是发送到同一个服务提供者。

**配置方式**

![](https://pic.imgdb.cn/item/60bec80e844ef46bb28c18c2.jpg)

### 服务降级

​	服务降级是一种系统保护策略，当服务器访问压力较大时，可以根据当前业务情况对不重要的服务进行降级，以保证核心服务的正常运行。所谓的降级，就是把一些非必要的功能在流量较大的时间段暂时关闭，比如在双11大促时，淘宝会把查看历史订单、商品评论等功能关闭，从而释放更多的资源来保障大部分用户能够正常完成交易。

降级有多个层面的分类：

* 按照是否自动化可分为自动降级和人工降级。
* 按照功能可分为读服务降级和写服务降级。



​	人工降级一般具有一定的前置性，比如在电商大促之前，暂时关闭某些非核心服务，如评价、推荐等。而自动降级更多的来自于系统出现某些异常的时候自动触发“兜底的流畅”，比如：

* 故障降级，调用的远程服务“挂了”，网络故障或者RPC服务返回异常。这类情况在业务允许的情况下可以通过设置兜底数据响应给客户端。
* 限流降级，不管是什么类型的系统，它所支撑的流量是有限的，为了保护系统不被压垮，在系统中会针对核心业务进行限流。当请求流量达到阈值时，后续的请求会被拦截，这类请求可以进入排队系统，比如12306。也可以直接返回降级页面，比如返回“活动太火爆，请稍候再来”页面。



​	Dubbo提供了一种Mock配置来实现服务降级，也就是说当服务提供方出现网络异常无法访问时，客户端不抛出异常，而是通过降级配置返回兜底数据，操作步骤如下：

* 在spring-cloud-dubbo-consumer项目中创建MockHelloService类，这个类只需要实现自动降级的接口即可，然后重写接口中的抽象方法实现本地数据的返回。

![](https://pic.imgdb.cn/item/60bece18844ef46bb2ee9911.jpg)

* 在HelloController类中修改＠Reference注解增加Mock参数。其中设置了属性cluster="failfast"，因为默认的容错策略会发起两次重试，等待的时间较长。

![](https://pic.imgdb.cn/item/60bece3a844ef46bb2f0e806.jpg)

* 在不启动Dubbo服务端或者服务端的返回值超过默认的超时时间时，访问/say接口得到的结构就是MockHelloService中返回的数据。

### 主机绑定规则

​	主机绑定表示的是Dubbo服务对外发布的IP地址，默认情况下Dubbo会按照以下顺序来查找并绑定主机IP地址：

* 查找环境变量中DUBBO_IP_TO_BIND属性配置的IP地址。
* 查找dubbo.protocol.host属性配置的IP地址，默认是空，如果没有配置或者IP地址不合法，则继续往下查找。
* 通过LocalHost.getHostAddress获取本机IP地址，如果获取失败，则继续往下查找。
* 如果配置了注册中心的地址，则使用Socket通信连接到注册中心的地址后，使用for循环通过socket.getLocalAddress（）.getHostAddress（）扫描各个网卡获取网卡IP地址。



​	任意一个步骤检测到合法的IP地址，便会将其返回作为对外暴露的服务IP地址。需要注意的是，获取的IP地址并不是写入注册中心的地址，默认情况下，写入注册中心的IP地址优先选择环境变量中DUBBO_IP_TO_REGISTRY属性配置的IP地址。在这个属性没有配置的情况下，才会选取前面获得的IP地址并写入注册中心。

​	使用默认的主机绑定规则，可能会存在获取的IP地址不正确的情况，导致服务消费者与注册中心上拿到的URL地址进行通信。因为Dubbo检测本地IP地址的策略是先调用LocalHost.getHostAddress，这个方法的原理是通过获取本机的hostname映射IP地址，如果它指向的是一个错误的IP地址，那么这个错误的地址将会作为服务发布的地址注册到ZooKeeper节点上，虽然Dubbo服务能够正常启动，但是服务消费者却无法正常调用。按照Dubbo中IP地址的查找规则，如果遇到这种情况，可以使用很多种方式来解决：

* 在/etc/hosts中配置机器名对应正确的IP地址映射。
* 在环境变量中添加DUBBO_IP_TO_BIND或者DUBBO_IP_TO_REGISTRY属性，Value值为绑定的主机地址。
* 通过dubbo.protocol.host设置主机地址。



​	除获取绑定主机IP地址外，对外发布的端口也是需要注意的，Dubbo框架中针对不同的协议都提供了默认的端口号：

* Dubbo协议的默认端口号是20880。
* Webservice协议的默认端口号是80。



​	在实际使用过程中，建议指定一个端口号，避免和其他Dubbo服务的端口产生冲突。

## Apache Dubbo核心源码分析

​	Apache Dubbo的源码相对来说还是比较容易理解的，只需要理解几个点：

* SPI机制。
* 自适应扩展点。
* IoC和AOP。
* Dubbo如何与Spring集成。

### 源码构建

**源码下载**

​	在编写本书时，Dubbo的最新版本是2.7.5，所以我们分析的源码以这个版本作为基础。

https://github.com/apache/dubbo/releases

**源码构建**

​	Dubbo源码构建依赖于Maven及JDK，Maven版本要求2.2.1以上、JDK版本要求1.5以上。
​	进入Dubbo源码根目录下，执行mvn install-Dmaven.test.skip命令来做一次构建。

**IDE支持**

​	针对不同的开发工具，可以通过以下命令来生成IDE工程。

* IntelliJ IDEA:mvn idea:idea.
* Eclipse:mvn eclipse:eclipse.

### Dubbo的核心之SPI

​	在Dubbo的源码中，很多地方会存下面这样三种代码，分别是自适应扩展点、指定名称的扩展点、激活扩展点：

![](https://pic.imgdb.cn/item/60bed488844ef46bb258b22b.jpg)

​	这种扩展点实际上就是Dubbo中的SPI机制。关于SPI，不知道大家是否还有印象，我们在分析Spring Boot自动装配的时候提到过SpringFactoriesLoader，它也是一种SPI机制。实际上，这两者的实现思想是类似的。

#### Java SPI扩展点实现

​	SPI全称是Service Provider Interface，原本是JDK内置的一种服务提供发现机制，它主要用来做服务的扩展实现。SPI机制在很多场景中都有运用，比如数据库连接，JDK提供了java.sql.Driver接口，这个驱动类在JDK中并没有实现，而是由不同的数据库厂商来实现，比如Oracle、MySQL这些数据库驱动包都会实现这个接口，然后JDK利用SPI机制从classpath下找到相应的驱动来获得指定数据库的连接。这种插拔式的扩展加载方式，也同样遵循一定的协议约定。比如所有的扩展点必须要放在resources/META-INF/services目录下，SPI机制会默认扫描这个路径下的属性文件以完成加载。

* 创建一个普通的Maven工程Driver，定义一个接口。这个接口只是一个规范，并没有实现，由第三方厂商来提供实现。

![](https://pic.imgdb.cn/item/60bed4d6844ef46bb25d636f.jpg)

* 创建另一个普通的Maven工程Mysql-Driver，添加Driver的Maven依赖。

![](https://pic.imgdb.cn/item/60bed4f0844ef46bb25ef1d0.jpg)

* 创建MysqlDriver，实现Driver接口，这个接口表示一个第三方的扩展实现。

![](https://pic.imgdb.cn/item/60bed505844ef46bb2603c8b.jpg)

* 在resources/META-INF/services目录下创建一个以Driver接口全路径名命名的文件com.gupaoedu.book.spi.Driver，在里面填写这个Driver的实现类扩展。

```
com.gupaoedu.book.spi.MysqlDriver
```

* 创建一个测试类，使用ServiceLoader加载对应的扩展点。从结果来看，MysqlDriver这个扩展点被加载并且输出了相应的内容。

![](https://pic.imgdb.cn/item/60bed55f844ef46bb2659e00.jpg)

#### Dubbo自定义协议扩展点

​	Dubbo或者SpringFactoriesLoader并没有使用JDK内置的SPI机制，只是利用了SPI的思想根据实际情况做了一些优化和调整。Dubbo SPI的相关逻辑被封装在了ExtensionLoader类中，通过ExtensionLoader我们可以加载指定的实现类。

​	Dubbo的SPI扩展有两个规则：

* 和JDK内置的SPI一样，需要在resources目录下创建任一目录结构：META-INF/dubbo、META-INF/dubbo/internal、META-INF/services，在对应的目录下创建以接口全路径名命名的文件，Dubbo会去这三个目录下加载相应扩展点。
* 文件内容和JDK内置的SPI不一样，内容是一种Key和Value形式的数据。Key是一个字符串，Value是一个对应扩展点的实现，这样的方式可以按照需要加载指定的实现类。



​	实现步骤如下:

* 在一个依赖了Dubbo框架的工程中，创建一个扩展点及一个实现。其中，扩展点需要声明＠SPI注解。

![](https://pic.imgdb.cn/item/60bed6b8844ef46bb27a4a0f.jpg)

* 在resources/META-INF/dubbo目录下创建以SPI借口命名的文件com.gupaoedu.book.dubbo.spi.Driver。

![](https://pic.imgdb.cn/item/60bed6f1844ef46bb27daeea.jpg)

* 创建测试类，使用ExtensionLoader.getExtensionLoader.getExtension（"mysqlDriver"）获得指定名称的扩展点实现。

![](https://pic.imgdb.cn/item/60bed706844ef46bb27ee655.jpg)

#### Dubbo SPI扩展点源码分析(todo)



# 服务注册与发现

![](https://pic.imgdb.cn/item/60bed7a7844ef46bb2884323.jpg)

​	服务消费者要去调用多个服务提供者组成的集群。首先，服务消费者需要在本地配置文件中维护服务提供者集群的每个节点的请求地址。其次，服务提供者集群中如果某个节点下线或者宕机，服务消费者的本地配置中需要同步删除这个节点的请求地址，防止请求发送到已宕机的节点上造成请求失败。为了解决这类的问题，就需要引入服务注册中心，它主要有以下功能：

* 服务地址的管理。
* 服务注册。
* 服务动态感知。



​	能够实现这类功能的组件很多，比如ZooKeeper、Eureka、Consul、Etcd、Nacos等。ZooKeeper在第4章中介绍过，在这一章中主要介绍Alibaba的Nacos。

## 什么是Alibaba Nacos

​	Nacos致力于解决微服务中的统一配置、服务注册与发现等问题。它提供了一组简单易用的特性集，帮助开发者快速实现动态服务发现、服务配置、服务元数据及流量管理。
​	Nacos的关键特性如下。

**服务发现和服务健康监测**

​	Nacos支持基于DNS和基于RPC的服务发现。服务提供者使用原生SDK、OpenAPI或一个独立的Agent TODO注册Service后，服务消费者可以使用DNS或HTTP&API查找和发现服务。
​	Nacos提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求。Nacos支持传输层（PING或TCP）和应用层（如HTTP、MySQL、用户自定义）的健康检查。对于复杂的云环境和网络拓扑环境中（如VPC、边缘网络等）服务的健康检查，Nacos提供了agent上报和服务端主动检测两种健康检查模式。Nacos还提供了统一的健康检查仪表盘，帮助用户根据健康状态管理服务的可用性及流量。

**动态配置服务**

​	业务服务一般都会维护一个本地配置文件，然后把一些常量配置到这个文件中。这种方式在某些场景中会存在问题，比如配置需要变更时要重新部署应用。而动态配置服务可以以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置，可以使配置管理变得更加高效和敏捷。配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。

​	另外，Nacos提供了一个简洁易用的UI（控制台样例Demo）帮助用户管理所有服务和应用的配置。Nacos还提供了包括配置版本跟踪、金丝雀发布、一键回滚配置及客户端配置更新状态跟踪在内的一系列开箱即用的配置管理特性，帮助用户更安全地在生产环境中管理配置变更，降低配置变更带来的风险。

**动态DNS服务**

​	动态DNS服务支持权重路由，让开发者更容易地实现中间层负载均衡、更灵活的路由策略、流量控制，以及数据中心内网的简单DNS解析服务。

**服务及其元数据管理**

​	Nacos可以使开发者从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的SLA及最重要的metrics统计数据。

## Nacos的基本使用

### Nacos安装

​	Nacos支持三种部署模式，分别是单机、集群和多集群。需要注意的是，Nacos依赖Java环境，并且要求使用JDK 1.8以上版本。

​	Nacos的安装方式有两种，一种是源码安装，另一种直接是使用已经编译好的安装包。由于后续需要分析Nacos源码，所以选择第一种安装方式。

* 在https://github.com/alibaba/nacos/releases上下载当前Nacos的最新版本（1.1.4）。

  ```
  https://gitee.com/mirrors/Nacos.git
  ```

* 解压进入根目录，执行mvn -P release-nacos clean install -U构建，构建之后会创建一个distribution目录。

* 执行cd distribution/target/nacos-server-$version/nacos/bin。

```sh
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
```



* 执行sh startup.sh -m standalone，启动服务。
* 服务启动之后，可以通过http://127.0.0.1:8848/nacos访问Nacos的控制台。控制台主要用于增强对服务列表、健康状态管理、服务治理、分布式配置管理等方面的管控能力，可以进一步帮助开发者降低管理微服务应用架构的成本。

### Nacos服务注册发现相关API说明

​	Nacos提供了SDK及Open API的方式来完成服务注册与发现等操作，由于服务端只提供了REST接口，所以SDK本质上是对于HTTP请求的封装。下面简单列一下和服务注册相关的核心接口。

**注册实例**

​	将服务地址信息注册到Nacos Server：

![](https://pic.imgdb.cn/item/60bf6b22844ef46bb2bfbc2e.jpg)

参数说明如下。

* serviceName：服务名称。

* ip：服务实例IP。

* port：服务实例Port。

* clusterName：集群名称，表示该服务实例属于哪个集群。

* instance：实例属性，实际上就是把上面这些参数封装成一个对象。

  

调用方式：

![](https://pic.imgdb.cn/item/60bf6b78844ef46bb2c5d979.jpg)

**获取全部实例**

​	根据服务名称从Nacos Server上获取所有服务实例：

![](https://pic.imgdb.cn/item/60bf6b97844ef46bb2c7fad4.jpg)

​	参数说明如下。

* serviceName：服务名称。
* cluster：集群列表，可以传递多个值。



​	调用方式：

![](https://pic.imgdb.cn/item/60bf6c98844ef46bb2d9691e.jpg)

**监听服务**

​	监听服务是指监听指定服务下的实例变化。在前面的分析中我们知道，客户端从Nacos Server上获取的实例必须是健康的，否则会造成客户端请求失败。监听服务机制可以让客户端及时感知服务提供者实例的变化。

![](https://pic.imgdb.cn/item/60bf6cd3844ef46bb2dd6427.jpg)

​	参数说明如下。

* EventListener：当服务提供者实例发生上、下线时，会触发一个事件回调。



​	需要注意的是，监听服务的Open API也访问/nacos/v1/ns/instance/list，具体的原理会在源码分析中讲解。
服务监听有两种方式：

* 第一种是客户端调用/nacos/v1/ns/instance/list定时轮询。
* 第二种是基于DatagramSocket的UDP协议，实现服务端的主动推送。

### Nacos集成Spring Boot实现服务注册与发现

​	本节通过Spring Boot集成Nacos实现一个简单的服务注册与发现功能。

* 创建一个Spring Boot工程spring-boot-nacos-discovery。
* 添加Maven依赖。

![](https://pic.imgdb.cn/item/60bf6d46844ef46bb2e55579.jpg)

* 创建DiscoveryController类，通过＠NacosInjected注入Nacos的NamingService，并提供discovery方法，可以根据服务名称获得注册到Nacos上的服务地址。

![](https://pic.imgdb.cn/item/60bf6da9844ef46bb2ec3524.jpg)

* 在application.properties中添加Nacos服务地址的配置。

![](https://pic.imgdb.cn/item/60bf6dc4844ef46bb2ee0a79.jpg)

* 启动SpringBootNacosDiscoveryApplication，调用curl http://127.0.0.1:8080/discovery?serviceName=example去Nacos服务器上查询服务名称example所对应的地址信息，此时由于Nacos Server并没有example的服务实例，返回一个空的JSON数组[]。
* 接着，通过Nacos提供的Open API，向Nacos Server注册一个名字为example的服务。

![](https://pic.imgdb.cn/item/60bf6de7844ef46bb2f0869a.jpg)

* 再次访问curl http://127.0.0.1:8080/discovery?serviceName=example，将返回以下信息。

![](https://pic.imgdb.cn/item/60bf6e09844ef46bb2f2f6ab.jpg)

​	通过Spring Boot集成Nacos实现服务注册与发现的案例，相信大家对Nacos已经有了一个初步的认识。

## Nacos的高可用部署

​	在分布式架构中，任何中间件或者应用都不允许单点存在，所以开源组件一般都会自己支持高可用集群解决方案。如图5-2所示，Nacos提供了类似于ZooKeeper的的集群架构，包含一个Leader节点和多个Follower节点。和ZooKeeper不同的是，它的数据一致性算法采用的是Raft，同样采用了该算法的中间件有Redis Sentinel的Leader选举、Etcd等。

![](https://pic.imgdb.cn/item/60bf6ff4844ef46bb217018c.jpg)

### 安装环境要求

​	请确保在环境中安装使用：

* 64 bit OS Linux/UNIX/Mac，推荐使用Linux系统。
* 64 bit JDK 1.8及以上，下载并配置。
* Maven 3.2.x及以上，下载并配置。
* 3个或3个以上Nacos节点才能构成集群。
* MySQL数据库。

### 集群配置

​	在conf目录下包含以下文件。

* application.properties：Spring Boot项目默认的配置文件。
* cluster.conf.example：集群配置样例文件。
* nacos-mysql.sql：MySQL数据库脚本。Nacos支持Derby和MySQL两种持久化机制，默认采用Derby数据库。如果采用MySQL，需要运行该脚本创建数据库和表。
* nacos-logback.xml：Nacos日志配置文件。



​	配置Nacos集群需要用到cluster.conf文件，我们可以直接重命名提供的example文件，修改该配置信息如下：

​	![](https://pic.imgdb.cn/item/60bf7333844ef46bb25a3dda.jpg)

​	这3台机器中的cluster.conf配置保持一致。由于这3台机器之间需要彼此通信，所以在部署的时候需要防火墙对外开放8848端口。

### 配置MySQL数据库

​	Derby数据库是一种文件类型的数据库，在使用时会存在一定的局限性。比如它无法支持多用户同时操作，在数据量大、连接数多的情况下会产生大量连接的积压。所以在生产环境中，可以用MySQL替换。

* 执行nacos-mysql.sql初始化。
* 分别修改3台机器中${NACOS_HOM}\conf下的application.properties文件，增加MySQL的配置。

![](https://pic.imgdb.cn/item/60bf774d844ef46bb2ae7b60.jpg)

### 启动Nacos服务

​	分别进入3台机器的bin目录，执行sh startup.sh或者startup.cmd-m cluster命令启动服务。服务启动成功之后，在${NACOS_HOME}\logs\start.out下可以获得如下日志，表示服务启动成功。

![](https://pic.imgdb.cn/item/60bf7803844ef46bb2be4575.jpg)

​	通过http://$NACOS_CLUSTER_IP:8848/nacos访问Nacos控制台，在“节点列表”下可以看到如图5-3所示的信息，表示当前集群由哪些节点组成及节点的状态。

![](https://pic.imgdb.cn/item/60bf7824844ef46bb2c10b80.jpg)

## Dubbo使用Nacos实现注册中心

* 创建一个普通Maven项目spring-boot-dubbo-nacos-sample，添加两个模块：nacos-sample-api和nacos-sample-provider。其中，nacos-sample-provider是一个Spring Boot工程。
* 在nacos-sample-api中声明接口。

![](https://pic.imgdb.cn/item/60bf79aa844ef46bb2e5ade4.jpg)

* 在nacos-sample-provider中添加依赖。

![](https://pic.imgdb.cn/item/60bf7a0a844ef46bb2efab19.jpg)

上述依赖包的简单说明如下：

* dubbo-spring-boot-starter，Dubbo的Starter组件，添加Dubbo依赖。
* nacos-discovery-spring-boot-starter，Nacos的Starter组件。
* nacos-sample-api，接口定义类的依赖。

创建HelloServiceImpl类，实现IHelloService接口。

![](https://pic.imgdb.cn/item/60bf7d4f844ef46bb239f587.jpg)

修改application.properties配置。仅将dubbo.registry.address中配置的协议改成了nacos://127.0.0.1:8848，基于Nacos协议。

![](https://pic.imgdb.cn/item/60bf7d63844ef46bb23bc94b.jpg)

 运行Spring Boot启动类，注意需要声明DubboComponentScan。

![](https://pic.imgdb.cn/item/60bf7d77844ef46bb23d8ba2.jpg)

​	服务启动成功之后，访问Nacos控制台，进入“服务管理”→“服务列表”，如图5-4所示，可以看到所有注册在Nacos上的服务。

![](https://pic.imgdb.cn/item/60bf7d9d844ef46bb240e2d1.jpg)

​	在图5-4所示界面中，在“操作”列单击“详情”，可以看到IHelloService下所有服务提供者的实例元数据，如图5-5所示。

​	服务的消费过程和第4章中演示的例子没有太大的区别，不再重复讲解，大家可以基于前面的代码进行改造。

![](https://pic.imgdb.cn/item/60bf7db4844ef46bb24300dc.jpg)

## Spring Cloud Alibaba Nacos Discovery

​	Nacos作为Spring Cloud Alibaba中服务注册与发现的核心组件，可以很好地帮助开发者将服务自动注册到Nacos服务端，并且能够动态感知和刷新某个服务实例的服务列表。使用Spring Cloud Alibaba Nacos Discovery可以基于Spring Cloud规范快速接入Nacos，实现服务注册与发现功能。

### 服务端开发

​	创建一个普通的Maven项目spring-cloud-nacos-sample，在项目中添加两个模块：

* spring-cloud-nacos-sample-api，暴露服务接口，作为服务提供者及服务消费者的接口契约。
* spring-cloud-nacos-sample-provider，项目类型为Spring Cloud，它是接口的实现。



​	项目的创建方式和类型与前面所演示的步骤一样。为了避免大家在实践的时候出现错误，笔者会将完整的过程再讲一遍，服务提供方的操作步骤如下。

* 在spring-cloud-nacos-sample-api项目中定义一个接口IHelloService。

![](https://pic.imgdb.cn/item/60bf7e8f844ef46bb2569ac5.jpg)

*  在		项目的pom.xml文件中添加相关依赖包。

![](https://pic.imgdb.cn/item/60bf7eae844ef46bb2595803.jpg)

![](https://pic.imgdb.cn/item/60bf7ebf844ef46bb25ae592.jpg)

* spring-cloud-starter：Spring Cloud核心包。
* spring-cloud-starter-dubbo：引入Spring Cloud Alibaba Dubbo。
* spring-cloud-dubbo-sample-api：API的接口声明。
* spring-cloud-alibaba-nacos-discovery：基于Nacos的服务注册与发现。



​	需要注意的是，在笔者所使用的版本中，spring-cloud-starter传递依赖的spring-cloud-context版本为2.2.1.RELEASE。这个版本的包存在兼容问题，会导致如下错误：

![](https://pic.imgdb.cn/item/60bf7efc844ef46bb26078b2.jpg)

​	所以我们通过exclusion排除了依赖，并且引入了2.1.1.RELEASE版本来解决。

​	需要注意的是，上述依赖的artifact没有指定版本，所以需要在父pom中显式地声明dependencyManagement：

![](https://pic.imgdb.cn/item/60bf7f22844ef46bb2641937.jpg)

* 在spring-cloud-nacos-sample-provider中创建接口的实现类HelloServiceImpl，其中＠Service是Dubbo服务的注解，表示当前服务会发布成一个远程服务。

![](https://pic.imgdb.cn/item/60bf7f47844ef46bb2678f31.jpg)

* 在application.properties中提供Dubbo及Nacos的配置，用于声明Dubbo服务暴露的网络端口和协议，以及服务注册的地址信息，完整的配置如下。

![](https://pic.imgdb.cn/item/60bf7f77844ef46bb26c0618.jpg)

​	以上配置的简单说明如下。

* dubbo.scan.base-packages：功能等同于＠DubboComponentScan，指定Dubbo服务实现类的扫描包路径。
* dubbo.registry.address：Dubbo服务注册中心的配置地址，它的值spring-cloud://localhost表示挂载到Spring Cloud注册中心，不配置的话会提示没有配置注册中心的错误。
* spring.cloud.nacos.discovery.server-addr：Nacos服务注册中心的地址。
* 启动服务。

![](https://pic.imgdb.cn/item/60bf7fb7844ef46bb271e0a0.jpg)

​	按照以上步骤开发完成之后，进入Nacos控制台的“服务管理”→“服务列表”，如果看到如图5-6所示的界面，说明服务已经发布成功了。

![](https://pic.imgdb.cn/item/60bf7fcb844ef46bb273c55c.jpg)

​	单击“详情”，会看到如图5-7所示的信息。

![](https://pic.imgdb.cn/item/60bf8016844ef46bb27a9944.jpg)

​	细心的读者会发现，基于Spring Cloud Alibaba Nacos Discovery实现服务注册时，元数据中发布的服务接口是con.alibaba.cloud.dubbo.service.DubboMetadataService。那么消费者要怎么去找到IHelloService呢？别急，进入Nacos控制台的“配置列表”，可以看到如图5-8所示的配置信息。实际上这里把发布的接口信息存储到了配置中心，并且建立了映射关系，从而使得消费者在访问服务的时候能够找到目标接口进行调用。至此，服务端便全部开发完了，接下来我们开始消费端的开发。

![](https://pic.imgdb.cn/item/60bf8033844ef46bb27d4f91.jpg)

### 消费端开发

* 创建一个Spring Boot项目spring-cloud-nacos-consumer。
* 添加相关Maven依赖。

![](https://pic.imgdb.cn/item/60bf8184844ef46bb29d660d.jpg)

​	上述依赖包和服务提供者的没什么区别，为了演示效果需要，增加了spring-boot-starter-web依赖。

* 在application.properties中添加配置信息。

![](https://pic.imgdb.cn/item/60bf8203844ef46bb2aa4230.jpg)

这些配置前面都讲过，就不重复解释了。

* 定义HelloController，用于测试Dubbo服务的访问。

![](https://pic.imgdb.cn/item/60bf8227844ef46bb2addea2.jpg)

* 启动服务

![](https://pic.imgdb.cn/item/60bf824b844ef46bb2b17431.jpg)

![](https://pic.imgdb.cn/item/60bf8258844ef46bb2b2b915.jpg)

## Nacos实现原理分析(todo)



# Nacos实现统一配置管理

* 配置的动态更新：在实际应用中会有动态更新配置的需求，比如修改服务连接地址、限流的配置等。在传统模式下，需要手动修改配置文件并且重启应用才能生效，这种方式效率太低，重启也会导致服务暂时不可用。
* 配置集中式管理：在微服务架构中，某些核心服务为了保证高性能会部署上百个节点，如果在每个节点中都维护一个配置文件，一旦配置文件中的某个属性需要修改，可想而知，工作量是巨大的。
*  配置内容的安全性和权限：配置文件随着源代码统一提交到代码库中，容易造成生产环境配置信息的数据泄露。
* 不同部署环境下配置的管理：前面提到过通过profile机制来管理不同环境下的配置，这种方式对于日常维护来说比较烦琐。

## Nacos集成Spring Boot实现统一配置管理

### 项目准备

* 创建一个Spring Boot工程spring-boot-nacos-config。
* 添加Nacos Config的Jar包依赖。

![](https://pic.imgdb.cn/item/60c0ae78844ef46bb270ac91.jpg)

* 在application.properties中添加Nacos Server的地址。

![](https://pic.imgdb.cn/item/60c0ae8e844ef46bb271eb6f.jpg)

* 创建NacosConfigController类，用于从Nacos Server动态读取配置。

![](https://pic.imgdb.cn/item/60c0aebb844ef46bb27477c9.jpg)

* ＠NacosPropertySource：用于加载dataId为example的配置源，autoRefreshed表示开启自动更新。
* ＠NacosValue：设置属性的值，其中info表示key，而Local Hello World代表默认值。也就是说，如果key不存在，则使用默认值。这是一种高可用的策略，在实际应用中，我们需要尽可能考虑到在配置中心不可用的情况况下如何保证服务的可用性。

### 启动Nacos Server

​	直接进入${NACOS_HOME}\bin目录，执行sh startup.sh启动Nacos Server即可。

### 创建配置

创建配置有两种方式：

* 在Nacos控制台上创建。
* 使用Open API方式创建。



**控制台**

​	通过http://ip:8848/nacos控制台进入“配置管理”→“配置列表”，单击“创建”按钮进入如图6-1所示界面。

![](https://pic.imgdb.cn/item/60c0af20844ef46bb27a4164.jpg)

* Data ID：表示Nacos中某个配置集的ID，通常用于组织划分系统的配置集。
* Group：表示配置所属的分组。
* 配置格式：当前配置内容所遵循的格式。

**Open API创建方式**

![](https://pic.imgdb.cn/item/60c0af44844ef46bb27c6611.jpg)

### 启动测试

![](https://pic.imgdb.cn/item/60c0b076844ef46bb28f0253.jpg)

## Spring Cloud Alibaba Nacos Config

### Nacos Config的基本应用

* 创建Spring Boot项目，添加spring-cloud-starter依赖。
* 添加Jar包依赖。

![](https://pic.imgdb.cn/item/60c0b373844ef46bb2bfbe6f.jpg)

* 创建bootstrap.properties文件，并在bootstrap.properties中添加Nacos Server的连接地址。

![](https://pic.imgdb.cn/item/60c0b384844ef46bb2c0e97b.jpg)

 	配置说明：

* spring.cloud.nacos.config.prefix表示Nacos配置中心上的Data ID的前缀。
* spring.cloud.nacos.config.server-addr设置Nacos配置中心的地址。如果地址是域名，配置的方式应该是域名：port，即便监听的端口是80，也需要将80端口带上。



​	需要注意，这些配置项是需要放在bootstrap.properties文件中的。在Spring Boot中有两种上下文配置，一种是bootstrap，另外一种是application。bootstrap是应用程序的父上下文，也就是说bootstrap加载优先于application。由于在加载远程配置之前，需要读取Nacos配置中心的服务地址信息，所以Nacos服务地址等属性配置需要放在bootstrap.properties文件中。

* 在Nacos Console中创建如下配置。

![](https://pic.imgdb.cn/item/60c0b537844ef46bb2e08a8e.jpg)

* 在启动类中，读取配置中心的数据。

![](https://pic.imgdb.cn/item/60c0b551844ef46bb2e27cde.jpg)

### 动态更新配置

​	配置中心必然需要支持配置的动态更新，也就是在配置中心上修改配置的值之后，应用程序需要感知值的变化。下面我们通过一段代码来演示动态更新的实现：

![](https://pic.imgdb.cn/item/60c0b572844ef46bb2e4d523.jpg)

![](https://pic.imgdb.cn/item/60c0b589844ef46bb2e67d1b.jpg)

###  基于Data ID配置YAML的文件扩展名

​	Spring Cloud Alibaba Nacos Config从Nacos Config Server中加载配置时，会匹配Data ID。在Spring Cloud Nacos的实现中，Data ID默认规则是${prefix}-${spring.profile.active}.${file-extension}。

* 在默认情况下，会去Nacos服务器上加载Data ID以${spring.applicationname}.${file-extension：properties}为前缀的基础配置。比如在6.3.1节演示的代码中，我们在bootstrap.properties文件中配置了属性spring.application.name=spring-cloud-nacos-config-sample，在不通过spring.cloud.nacos.config.prefix指定Data ID前缀时，默认会读取Nacos Config Server中Data ID为spring-cloud-nacos-config-sample.properties的配置信息。
* 如果明确指定了spring.cloud.nacos.config.prefix=example属性，则会加载Data ID=example的配置。
* spring.profile.active表示多环境支持，在后续的章节中会详细说明。

在实际应用中，如果大家用的是YAML格式的配置，Nacos Config也提供了YAML配置格式的支持，执行步骤如下。

* 在bootstrap.properties中声明spring.cloud.nacos.config.file-extension=yaml。
* 在Nacos控制台上增加如下配置。

![](https://pic.imgdb.cn/item/60c0b839844ef46bb2171ce4.jpg)

### 不同环境的配置切换

​	在Spring Boot中，可以基于spring.profiles.active实现不同环境下的配置切换，这在实际工作中用得比较多。很多公司都会有开发环境、测试环境、预生产环境、生产环境等，服务部署在不同的环境下，有一些配置是不同的，所以我们希望能够通过一个属性非常方便地指定当前应用部署的环境，并根据不同的环境加载对应的配置。基于Spring Boot项目的多环境支持配置步骤如下。

*  在resources目录下根据不同环境创建不同的配置。
  * application-dev.properties
  * application-test.properties
  *  application-prod.properties
*  定义一个application.properties默认配置，在该配置中通过spring.profiles.active=${env}来指定当前使用哪个环境的配置，如果${env}的值为prod，表示使用application-prod.properties。也可以通过设置VM options=-Dspring.profiles.active=prod来指定使用的环境配置。

* 在bootstrap.properties中声明spring.profiles.active=prod。需要注意的是，必须要在bootstrap.properties中声明。
* 在Nacos控制台上新增两个Data ID的配置项。
  * spring-cloud-nacos-config-sample-test.properties，配置内容为info=test。
  * spring-cloud-nacos-config-sample-prod.properties，配置内容为info=prod env：Hello。

![](https://pic.imgdb.cn/item/60c0b90c844ef46bb2263c4e.jpg)

### Nacos Config自定义Namespace和Group

​	在前面的章节中使用Nacos Config时都采用默认的Namespace：public和Group：DEFAULT_GROUP，从名字我们基本能够猜测到它们的作用。我们看一下如图6-2所示的Nacos提供的数据模型，它的数据模型Key是由三元组来进行唯一确定的。

​	其中Namespace用于解决多环境及多租户数据的隔离问题，比如在多套不同的环境下，可以根据指定的环境创建不同的Namespace，实现多环境的隔离，或者在多用户的场景中，每个用户可以维护自己的Namespace，实现每个用户的配置数据和注册数据的隔离。需要注意的是，在不同的Namespace下，可以存在相同的Group或DataId。

![](https://pic.imgdb.cn/item/60c0b948844ef46bb22a7956.jpg)

* Group是Nacos中用来实现Data ID分组管理的机制，从图6-2可以看出，它可以实现不同Service/DataId的隔离。对于Group的用法，其实没有固定的规定，比如它可以实现不同环境下的DataId的分组，也可以实现不同应用或者组件下使用相同配置类型的分组，比如database_url。
* 官方的建议是，通过Namespace来区分不同的环境，而Group可以专注在业务层面的数据分组。最重要的还是提前做好规划，对Namespace和Group进行基本的定调，避免使用上的混乱。



​	了解了Namespace和Group的概念之后，下面讲一下Spring Cloud Alibaba Nacos Config如何实现自定义Namespace和Group。

**Namespace**

* 在Nacos控制台的“命名空间”下，创建一个命名空间，如图6-3所示。
* 在bootstrap.properties中添加如下配置：

![](https://pic.imgdb.cn/item/60c0b981844ef46bb22e6cd4.jpg)

​	ee6d2c78-003b-4151-9984-63b1b40111a0对应的是Namespace中命名空间的ID，这个值可以在如图6-3所示的界面获取。

![](https://pic.imgdb.cn/item/60c0ba95844ef46bb2427fd9.jpg)

**Group**

​	Group不需要提前创建，只需要在创建的时候指定，配置方法如下。

* 在Nacos控制台的“新建配置”界面中指定配置所属的Group，如图6-4所示。
* 在bootstrap.properties中添加如下配置即可：

![](https://pic.imgdb.cn/item/60c0babf844ef46bb2458b9e.jpg)

**Data ID**

​	Data ID是Nacos中某个配置集的ID，它通常用于组织划分系统的配置集。在前面的示例中我们都是通过配置文件的名字来进行配置的划分的，也可以通过Java包的全路径来划分，主要取决于Data ID的使用维度。

![](https://pic.imgdb.cn/item/60c0bad4844ef46bb2470f88.jpg)

​	Spring Cloud Alibaba Nacos Config同样支持自动以Data ID配置。

![](https://pic.imgdb.cn/item/60c0baeb844ef46bb248d271.jpg)

* spring.cloud.nacos.config.ext-config[n]支持多个Data ID的扩展配置，包含三个属性：data-id、group、refresh。
* spring.cloud.nacos.config.ext-config[n].data-id指定Nacos Config的Data ID。
* spring.cloud.nacos.config.ext-config[n].group指定Data ID所在的组。
* spring.cloud.nacos.config.ext-config[n].refresh控制Data ID在配置发生变更时是否动态刷新，以感知最新的配置值。默认是false，也就是不会实现动态刷新。

在使用过程中，有两个注意点：

* spring.cloud.nacos.config.ext-config[n].data-id的值必须要带文件的扩展名，可以支持properties、yaml、json等。
* spring.cloud.nacos.config.ext-config[n].data-id配置多个Data ID时，n的值越大，优先级越高。



​	通过自定义扩展的Data Id配置，既可以解决多个应用的配置共享问题，在支持一个应用有多个配置文件的情况。需要注意的是，在ext-config和${spring.application.name}.${file-extension：properties}都存在的情况下，优先级高的是后者

# 基于Sentinel的微服务限流及熔断

## 服务限流的作用及实现

* 在Nginx层添加限流模块限制平均访问速度。
* 通过设置数据库连接池、线程池的大小来限制总的并发数。
* 通过Guava提供的Ratelimiter限制接口的访问速度。
* TCP通信协议中的流量整形。

### 计数算法

​	计数器算法是一种比较简单的限流实现算法，在指定周期内累加访问次数，当访问次数达到设定的阈值时，触发限流策略，当进入下一个时间周期时进行访问次数的清零。

​	这种算法存在一个临界问题，如图7-3所示，在第一分钟的0：58和第二分钟的1：02这个时间段内，分别出现了100个请求，整体来看就会出现4秒内总的请求量达到200，超出了设置的阈值。

### 滑动窗口算法

​	简单来说，滑动窗口算法的原理是在固定窗口中分割出多个小时间窗口，分别在每个小时间窗口中记录访问次数，然后根据时间将窗口往前滑动并删除过期的小时间窗口。最终只需要统计滑动窗口范围内的所有小时间窗口总的计数即可。
如图7-4所示，我们将一分钟拆分为4个小时间窗口，每个小时间窗口最多能够处理25个请求。并且通过虚线框表示滑动窗口的大小（当前窗口的大小是2，也就是在这个窗口内最多能

![](https://pic.imgdb.cn/item/60c837f2844ef46bb2f608c2.jpg)

​	够处理50个请求）。同时滑动窗口会随着时间往前移动，比如前面15s结束之后，窗口会滑动到15s～45s这个范围，然后在新的窗口中重新统计数据。这种方式很好地解决了固定窗口算法的临界值问题。
​	Sentinel就是采用滑动窗口算法来实现限流的，后续在源码分析部分会再讲到。

### 令牌桶限流算法

​	令牌桶是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。对于每一个请求，都需要从令牌桶中获得一个令牌，如果没有获得令牌，则需要触发限流策略。

​	![](https://pic.imgdb.cn/item/60c8385a844ef46bb2fcbde6.jpg)

​	假设令牌生成速度是每秒10个，也就等同于QPS=10，此时在请求获取令牌的时候，会存在三种情况：

* 请求速度大于令牌生成速度：那么令牌会很快被取完，后续再进来的请求会被限流。
* 请求速度等于令牌生成速度：此时流量处于平稳状态。
* 请求速度小于令牌生成速度：说明此时系统的并发数并不高，请求能被正常处理。



​	由于令牌桶有固定的大小，当请求速度小于令牌生成速度时，令牌桶会被填满。所以令牌桶能够处理突发流量，也就是在短时间内新增的流量系统能够正常处理，这是令牌桶的特性。

### 漏桶限流算法

​	漏桶限流算法的主要作用是控制数据注入网络的速度，平滑网络上的突发流量。

​	漏桶限流算法的原理如图7-6所示，在漏桶算法内部同样维护一个容器，这个容器会以恒定速度出水，不管上面的水流速度多快，漏桶水滴的流出速度始终保持不变。实际上消息中间件就使用了漏桶限流的思想，不管生产者的请求量有多大，消息的处理能力取决于消费者。

![](https://pic.imgdb.cn/item/60c8a9a1844ef46bb2269359.jpg)

在漏桶限流算法中，存在以下几种可能的情况：

* 请求速度大于漏桶流出水滴的速度：也就是请求数超出当前服务所能处理的极限，将会触发限流策略。
* 请求速度小于或者等于漏桶流出水滴的速度，也就是服务端的处理能力正好满足客户端的请求量，将正常执行。

## 服务熔断与降级

​	服务熔断是指当某个服务提供者无法正常为服务调用者提供服务时，比如请求超时、服务异常等，为了防止整个系统出现雪崩效应，暂时将出现故障的接口隔离出来，断绝与外部接口的联系，当触发熔断之后，后续一段时间内该服务调用者的请求都会直接失败，直到目标服务恢复正常。

​	服务降级需要有一个参考指标，一般来说有以下几种常见方案：

* 平均响应时间：比如1s内持续进入5个请求，对应时刻的平均响应时间均超超过阈值，那么接下来在一个固定的时间窗口内，对这个方法的访问都会自动熔断。
* 异常比例：当某个方法每秒调用所获得的异常总数的比例超过设定的阈值时，该资源会自动进入降级状态，也就是在接下来的一个固定时间窗口中，对这个方法的调用都会自动返回。
* 异常数量：和异常比例类似，当某个方法在指定时间窗口内获得的异常数量超过阈值时会触发熔断。

## 分布式限流框架Sentinel

### Sentinel的特性

* 丰富的应用场景：几乎涵盖所有的应用场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制等。
* 实时监控：Sentinel提供了实时监控功能。开发者可以在控制台中看到接入应用的单台机器秒级数据，甚至500台以下规模的集群汇总运行情况。
* 开源生态支持：Sentinel提供开箱即用的与其他开源框架/库的整合，例如与Spring Cloud、Dubbo、gRPC的整合。开发者只需要引入相应的依赖并进行简单的配置即可快速接入Sentinel。
* SPI扩展点支持：Sentinel提供了SPI扩展点支持，开发者可以通过扩展点来定制化限流规则，动态数据源适配等需求。

![](https://pic.imgdb.cn/item/60c8abdd844ef46bb2453996.jpg)

### Sentinel的组成

* 核心库（Java客户端）：不依赖任何框架/库，能够运行于所有Java运行时环境，同时对Dubbo、Spring Cloud等框架也有较好的支持。
* 控制台（Dashboard）：基于Spring Boot开发，打包后可以直接运行，不需要额外的Tomcat等应用容器。

### Sentinel Dashboard的部署

Sentinel Dashboard的安装步骤如下。

* 在GitHub中Sentinel的源码仓库中：
  * 直接下载源码通过mvn clean package自己构建。
  * 直接在Release页面下载已经构建好的Jar。
* 通过以下命令启动控制台：
  * -Dserver.port：指定Sentinel控制台的访问端口，默认是8080。
  * -Dcsp.sentinel.dashboard.server：指定Sentinel Dashboard控制台的IP地址和端口，这里进行设置的目的是把自己的限流数据暴露到监控平台。
  * -Dproject.name：设置项目名称。

## Sentinel的基本应用

​	使用Sentinel的核心库来实现限流，主要分以下几个步骤。

* 定义资源。
* 定义限流规则。
* 检验规则是否生效。

### Sentinel实现限流

​	首先，引入Sentinel的核心库：

![](https://pic.imgdb.cn/item/60c8ae01844ef46bb265171c.jpg)

​	然后，定义一个普通的业务方法：

![](https://pic.imgdb.cn/item/60c8b051844ef46bb287a81b.jpg)

​	在doSomething方法中，通过使用Sentinel中的SphU.entry（"doSomething"）定义一个资源来实现流控的逻辑，它表示当请求进入doSomething方法时，需要进行限流判断。如果抛出BlockException异常，则表示触发了限流。

​	接着，针对该保护的资源定义限流规则：

![](https://pic.imgdb.cn/item/60c8aeed844ef46bb2735042.jpg)

​	针对资源doSomething，通过initFlowRules设置限流规则，其中参数的含义如下。

* Grade：限流阈值类型，QPS模式（1）或并发线程数模式（0）。
* count：限流阈值。
* resource：设置需要保护的资源。这个资源的名称必须和SphU.entry中使用的名称保持一致。



​	上述代码的意思是，针对doSomething方法，每秒最多允许通过20个请求，也就是QPS为20。

![](https://pic.imgdb.cn/item/60c8af4a844ef46bb278be05.jpg)

### 资源的定义方式

​	除此之外，还可以通过返回布尔值的方式来定义资源，代码如下：

![](https://pic.imgdb.cn/item/60c8b076844ef46bb289dbed.jpg)

​	在这种方式中，需要注意资源使用完之后要调用Sph0.exit（），否则会导致调用链记录异常，抛出ErrorEntryFreeException异常。
​	Sentinel还可以使用＠SentinelResource支持注解的方式来定义资源，具体实现方式如下：

![](https://pic.imgdb.cn/item/60c8b140844ef46bb295bac0.jpg)

### Sentinel资源保护规则

​	Sentinel支持多种保护规则：流量控制规则、熔断降级规则、系统保护规则、来源访问控制规则、热点参数规则。

​	限流规则在前面的案例中简单使用过，先通过FlowRule来定义限流规则，然后通过FlowRuleManager.loadRules来加载规则列表。完整的限流规则设置代码如下：

![](https://pic.imgdb.cn/item/60c9ef81844ef46bb24f0f59.jpg)

其中，FlowRule部分属性的含义说明如下。

* limitApp：是否需要针对调用来源进行限流，默认是default，即不区分调用来源。
* strategy：调用关系限流策略——直接、链路、关联。
* controlBehavior：流控行为，包括直接拒绝、排队等待、慢启动模式，默认是直接拒绝。
* clusterMode：是否是集群限流，默认为否。

#### 基于并发数和QPS的流量控制

​	Sentinel流量控制统计有两种类型，通过grade属性来控制：

* 并发线程数（FLOW_GRADE_THREAD）。
* QPS(FLOW_GRADE_QPS).

**并发线程数**

​	并发线程数限流用来保护业务线程不被耗尽。Sentinel并发线程数限流就是统计当前请求的上下文线程数量，如果超出阈值，新的请求就会被拒绝。

**QPS**

​	QPS（Queries Per Second）表示每秒的查询数，也就是一台服务器每秒能够响应的查询次数。当QPS达到限流的阈值时，就会触发限流策略。

#### QPS流量控制行为

​	当QPS超过阈值时，就会触发流量控制行为，这种行为是通过controlBehavior来设置的，它包含：

* 直接拒绝（RuleConstant.CONTROL_BEHAVIOR_DEFAULT）；
* Warm Up（RuleConstant.CONTROL_BEHAVIOR_WARM_UP），冷启动（预热）；
* 匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）；
* 冷启动+匀速排队（RuleConstant.CONTROL_BEHAVIOR_WARM_UP_RATE_LIMITER）。

**直接拒绝**

​	直接拒绝是默认的流量控制方式，也就是请求流量超出阈值时，直接抛出一个FlowException。

**Warm Up**

​	Warm Up是一种冷启动（预热）方式。当流量突然增大时，也就意味着系统从空闲状态突然切换到繁忙状态，有可能会瞬间把系统压垮。当我们希望请求处理的数量逐步递增，并在一个预期时间之后达到允许处理请求的最大值时，Warm Up就可以达到这个目的。
​	如图7-10所示，当前系统所能够处理的最大并发数是480，首先，在最下面标记的位置，系统一直处于空闲状态，接着请求量突然直线升高。这个时候系统并不是直接将QPS拉到最大值，而是在一定时间内逐步增加阈值，而中间这段时间就是一个系统逐步预热的过程。

**匀速排队**

​	![](https://pic.imgdb.cn/item/60c9f0a0844ef46bb25bc871.jpg)

#### 调用关系流量策略

​	调用关系包括调用方和被调用方，一个方法又可能会调用其他方法，形成一个调用链。所谓的调用关系流量策略，就是根据不同的调用维度来触发流量控制。

* 根据调用方限流。
* 根据调用链路入口限流。
* 具有关系的资源流量控制（关联流量控制）。

**调用方限流**

​	所谓调用方限流，就是根据请求来源进行流量控制，我们需要设置limitApp属性来设置来源信息，它有三个选项。

* default：表示不区分调用者，也就是任何访问调用者的请求都会进行限流统计。
* {some_origin_name}：设置特定的调用者，只有来自这个调用者的请求才会进行流量统计和控制。
* other：表示针对除{some_origin_name}外的其他调用者进行流量控制。



​	由于同一个资源可以配置多条规则，如果多个规则设置的limitApp不一样，那么规则的生效顺序为：{some_origin_name}→other→default。

**根据调用链路入口限流**

​	一个被限流保护的方法，可能来自不同的调用链路。比如针对资源nodeA，入口Entrance1和入口Entrance2都调用了资源nodeA，那么Sentinel允许只根据某个入口来进行流量统计。比如我们针对nodeA资源，设置针对Entrance1入口的调用才会统计请求次数。它在一定程度上有点类似于调用方限流。

![](https://pic.imgdb.cn/item/60c9f121844ef46bb261cfd3.jpg)

**关联流量控制**

​	当两个资源之间存在依赖关系或者资源争抢时，我们就说这两个资源存在关联。这两个存在依赖关系的资源在执行时可能会因为某一个资源执行操作过于频繁而影响另外一个资源的执行效率，所以关联流量控制（流控）就是限制其中一个资源的执行流量。

### Sentinel实现服务熔断

​	Sentinel实现服务熔断操作的配置和限流类似，不同之处在于限流采用的是FlowRule，而熔断中采用的是DegradeRule，配置代码如下：

![](https://pic.imgdb.cn/item/60c9f177844ef46bb265d633.jpg)

其中，有几个属性说明如下。

* grade：熔断策略，支持秒级RT、秒级异常比例、分钟级异常数。默认是秒级RT。
* timeWindow：熔断降级的时间窗口，单位为s。也就是触发熔断降级之后多长时间内自动熔断。
* rtSlowRequestAmount：在RT模式下，1s内持续多少个请求的平均RT超出阈值后触发熔断，默认值是5。
* minRequestAmount：触发的异常熔断最小请求数，请求数小于该值时即使异常比例超出阈值也不会触发熔断，默认值是5。



​	Sentinel提供三种熔断策略，对于不同策略，参数的含义也不相同。

* 平均响应时间（RuleConstant.DEGRADE_GRADE_RT）：如果1s内持续进入5个请求，对应的平均响应时间都超过了阈值（count，单位为ms），那么在接下来的时间窗口（timeWindow，单位为s）内，对这个方法的调用都会自动熔断，抛出DegradeException。
  * Sentinel默认统计的RT上限是4900ms，如果超出此阈值都会算作4900ms，如果需要修改，则通过启动参数-Dcsp.sentinel.statistic.max.rt=xxx来配置。
* 异常比例（RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO）：如果每秒资源数≥minRequestAmount（默认值为5），并且每秒的异常总数占总通过量的比例超过阈值count （count的取值范围是[0.0，1.0]，代表0%～100%），则资源将进入降级状态。同样，在接下来的timeWindow之内，对这个方法的调用都会自动触发熔断。
* 异常数（RuleConstant.DEGRADE_GRADE_EXCEPTION_COUNT）：当资源最近一分钟的异常数目超过阈值之后，会触发熔断。需要注意的是，如果timeWindow小于60s，则结束熔断状态后仍然可能再进入熔断状态。

## Spring Cloud集成Sentinel实践

​	Spring Cloud Alibaba默认为Sentinel整合了Servlet、RestTemplate、FeignClient和Spring WebFlux。它不仅补全了Hystrix在Servlet和RestTemplate这一块的空白，而且还完全兼容了Hystrix在FeignClient中限流降级的用法，并支持灵活配置和调整流控规则。

### Sentinel接入Spring Cloud

* 创建一个基于Spring Boot的项目，并集成Greenwich.SR2版本的Spring Cloud依赖。
* 添加Sentinel依赖包。

​	![](https://pic.imgdb.cn/item/60c9f281844ef46bb272090a.jpg)

* 创建一个REST接口，并通过＠SentinelResource配置限流保护资源。

![](https://pic.imgdb.cn/item/60c9f29d844ef46bb27355d4.jpg)

在上述代码中，配置限流资源有几种情况：

* Sentinel starter在默认情况下会为所有的HTTP服务提供限流埋点，所以如果只想对HTTP服务进行限流，那么只需要添加依赖即可，不需要修改任何代码。
* 如果想要对特定的方法进行限流或者降级，则需要通过＠SentinalResouce注解来实现限流资源的定义。
* 可以通过SphU.entry（）方法来配置资源。
* 手动配置流控规则，可以借助Sentinel的InitFunc SPI扩展接口来实现，只需要实现自己的InitFunc接口，并在init方法中编写规则加载的逻辑即可。

![](https://pic.imgdb.cn/item/60c9f2e7844ef46bb276be3f.jpg)

​	SPI是扩展点机制，如果需要被Sentinel加载，那么还要在resource目录下创建META-INF/services/com.alibaba.csp.sentinel.init.InitFunc文件，文件内容就是自定义扩展点的全路径。

![](https://pic.imgdb.cn/item/60c9f308844ef46bb2784252.jpg)

​	按照上述配置好之后，在初次访问任意资源的时候，Sentinel就会自动加载hello资源的流控规则。

* 启动服务后，访问http://localhost:8080/say方法，当访问频率超过设定阈值时，就会触发限流。

### 基于Sentinel Dashboard来实现流控配置

​	基于Sentinel Dashboard来配置流控规则，可以实现流控规则的动态配置，执行步骤如下。

* 启动Sentinel Dashboard。
* 在application.yml中增加如下配置。

![](https://pic.imgdb.cn/item/60c9f342844ef46bb27aea1c.jpg)

​	spring.cloud.sentinel.transport.dashboard指向的是Sentinel Dashboard的服务器地址，可以实现流控数据的监控和流控规则的分发。

* 提供一个REST接口，代码如下。

![](https://pic.imgdb.cn/item/60c9f367844ef46bb27c91db.jpg)

​	此处不需要添加任何资源埋点，在默认情况下Sentinel Starter会对所有HTTP请求进行限流。

* 启动服务后，此时访问http://localhost:8080/dash，由于没有配置流控规则，所以不存在限流行为。

至此，Spring Cloud集成Sentinel就配置完成了，接下来，进入Sentinel Dashboard来实现流控规则的配置。

* 访问http://192.168.216.128：7777/进入Sentinel Dashboard。
*  进入spring.application.name对应的菜单，访问“簇点链路”，如图7-12所示，在该列表下可以看到/dash这个REST接口的资源名称。
* 针对/dash这个资源名称，可以在图7-12中最右边的操作栏单击“流控”按钮设置流控规则，如图7-13所示。

![](https://pic.imgdb.cn/item/60c9f39c844ef46bb27efb9e.jpg)

​	新增规则中的所有配置信息，实际就是FlowRule中对应的属性配置，为了演示示效果，把单机阈值设置为1。

* 新增完成之后，再次访问http://localhost:8080/dash接口，当QPS超过1时，就可以看到限流的效果，并获得如下输出：

![](https://pic.imgdb.cn/item/60c9f427844ef46bb2854cd6.jpg)

### 自定义URL限流异常

​	在默认情况下，URL触发限流后会直接返回。

​	在实际应用中，大都采用JSON格式的数据，所以如果希望修改触发限流之后的返回结果形式，则可以通过自定义限流异常来处理，实现UrlBlockHandler并且重写blocked方法：

![](https://pic.imgdb.cn/item/60c9f47c844ef46bb2894666.jpg)

### URL资源清洗

​	Sentinel中HTTP服务的限流默认由Sentinel-Web-Servlet包中的CommonFilter来实现，从代码中可以看到，这个Filter会把每个不同的URL都作为不同的资源来处理。

​	在下面这段代码中，提供了一个携带{id}参数的REST风格API，对于每一个不同的{id}，URL也都不一样，所以在默认情况下Sentinel会把所有的URL当作资源来进行流控。

![](https://pic.imgdb.cn/item/60c9f4db844ef46bb28dd06f.jpg)

这会导致两个问题：

* 限流统计不准确，实际需求是控制clean方法总的QPS，结果统计的是每个URL的QPS。
* 导致Sentinel中资源数量过多，默认资源数量的阈值是6000，对于多出的资源规则将不会生效。



​	针对这个问题可以通过UrlCleaner接口来实现资源清洗，也就是对于/clean/{id}这个URL，我们可以统一归集到/clean/*资源下，具体配置代码如下，实现UrlCleaner接口，并重写clean方法即可。

![](https://pic.imgdb.cn/item/60c9f52a844ef46bb291abe9.jpg)

## Sentinel集成Nacos实现动态流控规则

​	当然，我们还需要对定义的资源设置流控规则，前面演示了两种方式：

* 通过FlowRuleManager.loadRules（List rules）手动加载流控规则。
* 在Sentinel Dashboard上针对资源动态创建流控规则。



​	针对第一种设置方式，如果接入Sentinel Dashboard，那么同样支持动态修改流控规则。但是，这里会存在一个问题，基于Sentinel Dashboard所配置的流控规则都是保存在内存中的，一旦应用重启，这些规则都会被清除。为了解决这个问题，Sentinel提供了动态数据源支持。

​	目前，Sentinel支持Consul、ZooKeeper、Redis、Nacos、Apollo、etcd等数据源的扩展。下面通过一个案例演示Spring Cloud Sentinel集成Nacos实现动态流控规则，配置步骤如下。

* 添加Nacos数据源的依赖包。

![](https://pic.imgdb.cn/item/60c9f621844ef46bb29dc7f6.jpg)

*  创建一个REST接口，用于测试。

![](https://pic.imgdb.cn/item/60c9f637844ef46bb29ed9f9.jpg)

* 在application.yml中添加数据源配置。

![](https://pic.imgdb.cn/item/60c9f64b844ef46bb29fde59.jpg)

​	部分配置说明如下。

* datasource：目前支持redis、apollo、zk、file、nacos，选择什么类型的数据源就配置相应的key即可。
* data-id：可以设置成${spring.application.name}，方便区分不同应用的配置。
* rule-type：表示数据源中规则属于哪种类型，如flow、degrade、param-flow、gw-flow等。
* data-type：指配置项的内容格式，Spring Cloud Alibaba Sentinel提供了JSON和XML两种格式。如果需要自定义，则可以将值配置为custom，并配置converter-class指向converter类。
*  登录Nacos控制台，创建流控配置规则，配置信息如图7-14所示。

![](https://pic.imgdb.cn/item/60c9f693844ef46bb2a35ff4.jpg)

* 最后，登录Sentinel Dashboard，找到执行项目名称菜单下的“流控控规则”，就可以看到在Nacos上所配置的流控规则已经被加载了，如图7-15所示。

![](https://pic.imgdb.cn/item/60c9f6ae844ef46bb2a4c2a1.jpg)

* 当我们在Nacos的控制台上修改流控规则后，可以同步在Sentinel Dashboard上看到流控规则的变化。

通过上述配置整合之后，接口流控规则的动态修改就存在于以下两个地方：

* Sentinel Dashboard.
* Nacos控制台。



​	那么问题就来了，在Nacos控制台上修改流控规则，虽然可以同步到Sentinel Dashboard，但是Nacos此时应该作为一个流控规则的持久化平台，所以正常的操作过程应该是开发者在Sentinel Dashboard上修改流控规则后同步到Nacos上，遗憾的是，目前Sentinel Dashboard不支持该功能。

​	所以，Nacos名义上是“Datasource”，但实际上充当的仍然是配置中心的角色，开发者可以在Nacos控制台上动态修改流控规则并实现规则同步。在实际开发中，很难避免在不清楚情况的前提下，部分开发者通过Sentinel Dashboard来管理流控规则，部分开发者通过Nacos来管理流控规则，这将会导致非常严重的问题。

​	如图7-16所示，Nacos在此处扮演的角色应该是一个“Datasource”，所以笔者强烈建议大家不要在Nacos上修改流控规则，因为这种修改的危险系数很高，毕竟Nacos的UI并不是专门负责流控规则维护的。

![](https://pic.imgdb.cn/item/60c9f702844ef46bb2a901ff.jpg)

​	这也就意味着流控规则的管理应该集中在Sentinel Dashboard上，接下来的问题就很简单了，我们需要实现Sentinel Dashboard来动态维护流控规则并同步到Nacos上，目前官方还没有提供支持，但是大家可以自己来实现。

## Sentinel Dashboard集成Nacos实现规则同步

​	Sentinel Dashboard的“流控规则”下的所有操作，都会调用Sentinel-Dashboard源码中的FlowControllerV1类，这个类中包含流控规则本地化的CRUD操作。

​	另外，在com.alibaba.csp.sentinel.dashboard.controller.v2包下存在一个FlowControllerV2类，这个类同样提供流控规则的CRUD，和V1版本不同的是，它可以实现指定数据源的规则拉取和发布，部分代码如下：

![](https://pic.imgdb.cn/item/60c9f772844ef46bb2ae99bf.jpg)

### Sentinel Dashboard源码修改

​	修改Sentinel Dashboard的源码，具体的实现步骤如下。

* 在GitHub中下载Sentinel Dashboard 1.7.1的源码。
* 使用IDEA工具打开${Sentinel_home}/sentinel-dashboard工程。
*  在pom.xml中把sentinel-datasource-nacos依赖的`<scope>`注释掉。

修改resources/app/scripts/directives/sidebar/sidebar.html文件中的下面这段代码，将dashboard.flowV1改成dashboard.flow，也就是去掉V1。

![](https://pic.imgdb.cn/item/60c9f7fb844ef46bb2b55be2.jpg)

​	修改之后，会调用FlowControllerV2中的接口。

*  在com.alibaba.csp.sentinel.dashboard.rule包中创建一个nacos包，并创建一个类用来加载外部化配置。

![](https://pic.imgdb.cn/item/60c9f819844ef46bb2b6f6cd.jpg)

*  创建一个Nacos配置类NacosConfiguration。

![](https://pic.imgdb.cn/item/60c9f855844ef46bb2ba1ce1.jpg)

* 注入Converter转换器，将FlowRuleEntity转化成FlowRule，以及反向转化。
* 注入Nacos配置服务ConfigService。
* 创建一个常量类NacosConstants，分别表示默认的GROUP_ID和DATA_ID的后缀。

![](https://pic.imgdb.cn/item/60c9f882844ef46bb2bc7f0f.jpg)

* 实现动态从Nacos配置中心获取流控规则。

![](https://pic.imgdb.cn/item/60c9f8d2844ef46bb2c0d01e.jpg)

![](https://pic.imgdb.cn/item/60c9f8dc844ef46bb2c15af0.jpg)

​	在第5章中讲过Nacos配置中心，所以这段代码不难理解。主要是通过ConfigServic.getConfig方法从Nacos Config Server中读取指定配置信息，并通过converter转化为FlowRule规则。

* 创建一个流控规则发布类，在Sentinel Dashboard上修改完配置之后，需要调用该发布方法将数据持久化到Nacos中。

![](https://pic.imgdb.cn/item/60c9f919844ef46bb2c4a0b3.jpg)

* 修改FlowControllerV2类，将上面配置的两个类注入进来，表示规则的拉取和规则的发布统一用我们前面自定义的两个实例。

![](https://pic.imgdb.cn/item/60c9f92e844ef46bb2c5c113.jpg)

*  在application.properties文件中添加Nacos服务端的配置信息。

![](https://pic.imgdb.cn/item/60c9f93e844ef46bb2c6ad3a.jpg)

* 使用以下命令将代码打包成一个fat jar，根据前面介绍的操作方法启动服务。

![](https://pic.imgdb.cn/item/60c9f966844ef46bb2c8db28.jpg)

### Sentinel Dashboard规则数据同步

​	对于应用程序来说，需要改动的地方比较少，只要注意配置文件中data-id的命名要以-sentinel-flow结尾即可，因为在Sentinel Dashboard中我们写了一个固定的后缀。

![](https://pic.imgdb.cn/item/60c9f998844ef46bb2cb93ea.jpg)

​	后续的测试过程就比较简单了。

* 直接登录Sentinel Dashboard，进入“流控规则”，然后针对指定的资源创建流控规则。
*  进入Nacos控制台，就可以看到如图7-17所示的配置列表。

![](https://pic.imgdb.cn/item/60c9f9b2844ef46bb2ccffee.jpg)

## Dubbo集成Sentinel实现限流

​	Sentinel提供了与Dubbo整合的模块Sentinel Apache Dubbo Adapter，可以针对服务提供方和服务消费方进行流控，在使用的时候，只需要添加以下依赖。

![](https://pic.imgdb.cn/item/60c9f9e7844ef46bb2cfed77.jpg)

​	添加好该依赖之后，Dubbo服务中的接口和方法（包括服务端和消费端）就会成为Sentinel中的资源，只需针对指定资源配置流控规则就可以实现Sentinel流控功能。
​	Sentinel Apache Dubbo Adapter实现限流的核心原理是基于Dubbo的SPI机制实现Filter扩展，Dubbo的Filter机制是专门为服务提供方和服务消费方调用过程进行拦截设计的，每次执行远程方法，该拦截都会被执行。

​	同时，Sentinel Apache Dubbo Adapter还可以自定义开启或者关闭某个过滤器（Filter）的功能，下面这段代码表示关闭消费端的过滤器。

![](https://pic.imgdb.cn/item/60c9fa04844ef46bb2d19289.jpg)

### Dubbo服务接入Sentinel Dashboard

​	spring-cloud-starter-alibaba-sentinel目前无法支持Dubbo服务的限流，所以针对Dubbo服务的限流只能使用sentinel-apache-dubbo-adapter。这个适配组件并没有自动接入Sentinel Dashboard，需要通过以下步骤来进行接入。

* 引入sentinel-transport-simple-http依赖，这个依赖可以上报应用相关信息到控制台。

![](https://pic.imgdb.cn/item/60c9fa2f844ef46bb2d3e189.jpg)

* 添加启动参数。

![](https://pic.imgdb.cn/item/60c9fa3f844ef46bb2d4cdde.jpg)

参数配置说明如下。

* -Djava.net.preferIPv4Stack：表示只支持IPv4。
* -Dcsp.sentinel.api.port：客户端的port，用于上报应用的信息。
* -Dcsp.sentinel.dashboard.server：Sentinel Dashboard地址。
* -Dproject.name：应用名称，会在Sentinel Dashboard右侧展示。
* 登录Sentinel Dashboard之后，进入“簇点链路”，就可以看到如图7-18所示的资源信息。

![](https://pic.imgdb.cn/item/60c9fa83844ef46bb2d864de.jpg)

​	需要注意的是，限流可以通过服务接口或服务方法设置。

* 服务接口：resourceName为接口的全限定名，在图7-18中的体现为com.gupaoedu.sentinel.dubbo.IHelloService。
* 服务方法：resourceName为接口全限定名：方法名，如com.gupaoedu.sentinel.dubbo.IHelloService：sayHello（）。

### Dubbo服务限流规则配置

​	Dubbo限流规则同样可以通过以下集中方式来实现。

* Sentinel Dashboard.
* FlowRuleManager.loadRules(List rules).



​	前面我们讲过基于Sentinel Dashboard来实现流控规则配置，最终持久化到Nacos中。然而规则的持久化机制在Spring Cloud Sentinel中是自动实现的，在Sentinel Apache Dubbo Adapter组件中并没有实现该功能。下面演示一下在Dubbo服务中如何实现规则的持久化。

* 添加sentinel-datasource-nacos的依赖。

![](https://pic.imgdb.cn/item/60c9facb844ef46bb2dc4b83.jpg)

* 通过Sentinel提供的InitFunc扩展点，实现Nacos数据源的配置。

![](https://pic.imgdb.cn/item/60c9fade844ef46bb2dd4b24.jpg)

​	NacosDataSourceInitFunc要实现自动加载，需要在resource目录下的META-INF/services中创建一个名称为com.alibaba.csp.sentinel.init.InitFunc的文件，文件内容为NacosDataSourceInitFunc的全路径。

![](https://pic.imgdb.cn/item/60c9fafa844ef46bb2decc3b.jpg)

*  访问Sentinel Dashboard，在针对某个资源创建流控规则时，这个规则会同步保存到Nacos配置中心。而当Nacos配置中心发生变化时，会触发事件机制通知Dubbo应用重新加载流控规则。

## Sentinel热点限流

​	热点数据表示经常访问的数据，在有些场景中我们希望针对这些访问频次非常高的数据进行限流，比如针对一段时间内频繁访问的用户IP地址进行限流，或者针对频繁访问的某个用户ID进行限流。

​	Sentinel提供了热点参数限流的策略，它是一种特殊的限流，在普通限流的基础上对同一个受保护的资源区根据请求中的参数分别处理，该策略只对包含热点参数的资源调用生效。热点限流在以下场景中使用较多。

* 服务网关层：例如防止网络爬虫和恶意攻击，一种常用方法就是限制爬虫的IP地址，客户端IP地址就是一种热点参数。
* 写数据的服务：例如业务系统提供写数据的服务，数据会写入数据库之类的存储系统。存储系统的底层会加锁写磁盘上的文件，部分存储系统会将某一类数据写入同一个文件。如果底层写同一文件，会出现抢占锁的情况，导致出现大量超时和失败。出现这种情况时一般有两种解决办法：修改存储设计、对热点参数限流。



​	Sentinel通过LRU策略结合滑动窗口机制来实现热点参数的统计，其中，LRU策略可以统计单位时间内最常访问的热点数据，滑动窗口机制可以协助统计每个参数的QPS。

​	如图7-19所示，Sentinel会根据请求的参数来判断哪些是热点参数，然后通过热点参数限流规则，将QSP超过设定阈值的请求阻塞。

![](https://pic.imgdb.cn/item/60c9fbd9844ef46bb2eabd2f.jpg)

### 热点参数限流的使用

​	引入热点参数限流依赖包sentinel-parameter-flow-control。

![](https://pic.imgdb.cn/item/60c9fbf5844ef46bb2ec233c.jpg)

​	接下来，创建一个REST接口，并定义限流埋点，此处针对参数id配置热点限流规则。

![](https://pic.imgdb.cn/item/60c9fc0c844ef46bb2ed5345.jpg)

​		针对不同的热点参数，需要通过SphU.entry（resourceName，EntryType.IN，1，id）方法设置，其最后一个参数是一个数组，有多个热点参数时就按照次序依次传入，该配置表示后续会针对该参数进行热点限流。

​	下面针对上述资源sayHelo设置热点参数限流规则，通过ParamFlowRuleManager.loadRules方法加载热点参数规则。

![](https://pic.imgdb.cn/item/60c9fc2d844ef46bb2ef0858.jpg)

​	通过测试工具或者快速刷新浏览器来测试热点参数限流。如图7-20所示，访问Sentinel Dashboard，进入“实时监控”来查看限流的效果。

![](https://pic.imgdb.cn/item/60c9fc53844ef46bb2f1157f.jpg)

###  ＠SentinelResource热点参数限流

​	如果是通过＠SentinelResource注解来定义资源的，当注解所配置的方法上有参数时，Sentinel会把这些参数传入Sphu.entry（res，args）。比如下面这段代码，会把id这个参数作为热点参数进行限流。

![](https://pic.imgdb.cn/item/60c9fc81844ef46bb2f37b01.jpg)

### 热点参数规则说明

​	热点参数规则是通过ParamFlowRule来配置的，它的大部分属性和FlowRule类似，下面针对ParamFlowRule特定的属性进行简单说明。

* durationInSec：统计窗口时间长度，单位为秒。
* maxQueueingTimeMS：最长排队等待时长，只有当流控行为controlBehavior设置为匀速排队模式时生效。
* paramIdx：热点参数的索引，属于必填项，它对应的是SphU.entry（xxx，args）中的参数索引位置。
* paramFlowItemList：针对指定参数值单独设置限流阈值，不受count阈值值的限制。

# 分布式事务

​	所谓的数据库事务是指作为单个逻辑工作单元执行的多个数据库操作，要么同时成功，要么同时失败，它必须满足ACID特性，即：

* 原子性（Atomicity）：事务必须是原子工作单元，不可继续分割，要么全部成功，要么全部失败。
* 一致性（Consistency）：事务完成时，所有的数据都必须保持一致。
*  隔离性（Isolation）：由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。
* 持久性（Durability）：事务执行完成之后，它对于系统的影响是永久性的。

## 分布式事务问题的理论模型

​	分布式事务问题也叫分布式数据一致性问题，简单来说就是如何在分布式场景中保证多个节点数据的一致性。分布式事务产生的核心原因在于存储资源的分布性。在实际应用中，我们应该尽可能地从设计层面去避免分布式事务的问题，因为任何一种解决方案都会增加系统的复杂度。接下来我们了解一下分布式事务问题的常见解决方案。

### X/Open分布式事务模型

​	X/Open DTP（X/Open Distributed Transaction Processing Reference Model）是X/Open这个组织定义的一套分布式事务的标准。这个标准提出了使用两阶段提交（2PC，Two-Phase-Commit）来保证分布式事务的完整性。如图8-2所示，X/Open DTP中包含以下三种角色。

* AP：Application，表示应用程序。
* RM：Resource Manager，表示资源管理器，比如数据库
* TM：Transaction Manager，表示事务管理器，一般指事务协调者，负责协调和管理事务，提供AP编程接口或管理RM。可以理解为Spring中提供的Transaction Manager。

![](https://pic.imgdb.cn/item/60e6602f5132923bf89f8d55.jpg)

​	图8-2所展示的角色和关系与本地事务的原理基本相同，唯一不同的在于，如果此时RM代表数据库，那么TM需要能够管理多个数据库的事务，大致实现步骤如下：

* 配置TM，把多个RM注册到TM，相当于TM注册RM作为数据源。
* AP从TM管理的RM中获取连接，如果RM是数据库则获取JDBC连接。
* AP向TM发起一个全局事务，生成全局事务ID（XID），XID会通知各个RM。
* AP通过第二步获得的连接直接操作RM完成数据操作。这时，AP在每次操作时会把XID传递给RM。
* AP结束全局事务，TM会通知各个RM全局事务结束。
* 根据各个RM的事务执行结果，执行提交或者回滚操作。



​	在原本的单机事务下，会存在跨库事务的可见性问题，导致无法实现多节点事务的全局可控。

​	要注意的是，TM和多个RM之间的事务控制，是基于XA协议（XA Specification）来完成的。XA协议是X/Open提出的分布式事务处理规范，也是分布式事务处理的工业标准，它定义了xa_和ax_系列的函数原型及功能描述、约束等。目前Oracle、MySQL、DB2都实现了XA接口，所以它们都可以作为RM。

![](https://pic.imgdb.cn/item/60e661b45132923bf8a7b25e.jpg)

### 两阶段提交协议

​	会涉及两个阶段的提交，第一阶段是事务的准备阶段，第二阶段是事务的提交或者回滚阶段。这两个阶段都是由事务管理器发起的。两阶段提交协议的执行流程如下。

* 准备阶段：事务管理器（TM）通知资源管理器（RM）准备分支事务，记录事务日志，并告知事务管理器的准备结果。
* 提交/回滚阶段：如果所有的资源管理器（RM）在准备阶段都明确返回成功，则事务管理器（TM）向所有的资源管理器（RM）发起事务提交指令完成数据的变更。反之，如果任何一个资源管理器（RM）明确返回失败，则事务管理器（TM）会向所有资源管理器（RM）发送事务回滚指令。完整的执行流程如图8-4所示。



​	两阶段提交将一个事务的处理过程分为投票和执行两个阶段，它的优点在于充分考虑到了分布式系统的不可靠因素，并且采用非常简单的方式（两阶段提交）就把由于系统不可靠而导致事务提交失败的概率降到最小。当然，它也并不是完美的，存在以下缺点。

* 步阻塞：从图8-4的执行流程来看，所有参与者（RM）都是事务阻塞型的，对于任何一次指令都必须要有明确的响应才能继续进行下一步，否则会处于阻塞状态，占用的资源一直被锁定。
* 过于保守：任何一个节点失败都会导致数据回滚。
* 事务协调者的单点故障：如果协调者在第二阶段出现了故障，那么其他的参与者（RM）会一直处于锁定状态。
* “脑裂”导致数据不一致问题：在第二阶段中，事务协调者向所有参与者（RM）发送commit请求后，发生局部网络异常导致只有一部分参与者（RM）接收到了commit请求，这部分参与者（RM）收到请求后会执行commit操作，但是未收到commit请求的节点由于事务无法提交，导致数据出现不一致问题。

![](https://pic.imgdb.cn/item/60e662b35132923bf8ad2154.jpg)

### 三阶段提交协议

​	三阶段提交协议是两阶段提交协议的改进版本，它利用超时机制解决了同步阻塞的问题，三阶段提交协议的具体描述如下。

* CanCommit（询问阶段）：事务协调者向参与者发送事务执行请求，询问是否可以完成指令，参与者只需要回答是或者不是即可，不需要做真正的事务操作，这个阶段会有超时中止机制。
* PreCommit（准备阶段）：事务协调者会根据参与者的反馈结果决定是否继续执行，如果在询问阶段所有参与者都返回可以执行操作，则事务协调者会向所有参与者发送PreCommit请求，参与者收到请求后写redo和undo日志，执行事务操作但是不提交事务，然后返回ACK响应等待事务协调者的下一步通知。如果在询问阶段任意参与者返回不能执行操作的结果，那么事务协调者会向所有参与者发送事务中断请求。
* DoCommit（提交或回滚阶段）：这个阶段也会存在两种结果，仍然根据上一步骤的执行结果来决定DoCommit的执行方式。如果每个参与者在PreCommit阶段都返回成功，那么事务协调者会向所有参与者发起事务提交指令。反之，如果参与者中的任一参与者返回失败，那么事务协调者就会发起中止指令来回滚事务。



​	三阶段提交协议的时序图如图8-5所示。

![](https://pic.imgdb.cn/item/60e6639e5132923bf8b2152d.jpg)

三阶段提交协议和两阶段提交协议相比有一些不同点：

* 增加了一个CanCommit阶段，用于询问所有参与者是否可以执行事务操作并且响应，它的好处是，可以尽早发现无法执行操作而中止后续的行
* 准备阶段之后，事务协调者和参与者都引入了超时机制，一旦超时，事务协调者和参与者会继续提交事务，并且认为处于成功状态，因为在这种情况下事务默认为成功的可能性比较大。

### CAP定理和BASE理论

​	实际上，一旦超时，在三阶段提交协议下仍然可能出现数据不一致的情况，当然概率是比较小的。另外，最大的好处就是基于超时机制来避免资源的永久锁定。需要注意的是，不管是两阶段提交协议还是三阶段提交协议，都是数据一致性解决方案的实现，我们可以在实际应用中灵活调整。比如ZooKeeper集群中的数据一致性，就用到了优化版的两阶段提交协议，优化的地方在于，它不需要所有参与者在第一阶段返回成功才能提交事务，而是利用少数服从多数的投票机制来完成数据的提交或者回滚。

​	前面提到的两阶段提交和三阶段提交是XA协议解决分布式数据一致性问题的基本原理，但是这两种方案为了保证数据的强一致性，降低了可用性。实际上这里涉及分布式事务的两个理论模型。

**CAP定理**

​	CAP定理，又叫布鲁尔定理。简单来说它是指在分布式系统中不可能同时满足一致性（C：Consistency）、可用性（A：Availability）、分区容错性（P：Partition Tolerance）这三个基本需求，最多同时满足两个。

* C：数据在多个副本中要保持强一致，比如前面说的分布式数据一致性问题。
* A：系统对外提供的服务必须一直处于可用状态，在任何故障下，客户端都能在合理的时间内获得服务端的非错误响应。
* P：在分布式系统中遇到任何网络分区故障，系统仍然能够正常对外提供服务。



​	CAP定理证明，在分布式系统中，要么满足CP，要么满足AP，不可能实现CAP或者CA。原因是网络通信并不是绝对可靠的，比如网络延时、网络异常等都会导致系统故障。而在分布式系统中，即便出现网络故障也需要保证系统仍然能够正常对外提供服务，所以在分布式系统中Partition Tolerance是必然存在的，也就是需要满足分区容错性。

​	如果是CA或者CAP这种情况，相当于网络百分之百可靠，否则当出现网络分区的情况时，为了保证数据的一致性，必须拒绝客户端的请求。但是如果拒绝了请求，就无法满足A，所以在分布式系统中不可能选择CA，因此只能有AP或者CP两种选择。

* AP：对于AP来说，相当于放弃了强一致性，实现最终的一致，这是很多互联网公司解决分布式数据一致性问题的主要选择。
* CP：放弃了高可用性，实现强一致性。前面提到的两阶段提交和三阶段提交都采用这种方案。可能导致的问题是用户完成一个操作会等待较长的时间。

**BASE理论**

​	BASE理论是由于CAP中一致性和可用性不可兼得而衍生出来的一种新的思想，BASE理论的核心思想是通过牺牲数据的强一致性来获得高可用性。它有如下三个特性。

* Basically Available（基本可用）：分布式系统在出现故障时，允许损失一部分功能的可用性，保证核心功能的可用。
* Soft State（软状态）：允许系统中的数据存在中间状态，这个状态不影响系统的可用性，也就是允许系统中不同节点的数据副本之间的同步存在延时。
* Eventually Consistent（最终一致性）：中间状态的数据在经过一段时间之后，会达到一个最终的数据一致性。

​	BASE理论并没有要求数据的强一致，而是允许数据在一段时间内是不一致的，但是数据最终会在某个时间点实现一致。在互联网产品中，大部分都会采用BASE理论来实现数据的一致，因为产品的可用性对于用户来说更加重要。

## 分布式事务问题的常见解决方案

​	对于数据一致性问题有AP和CP两种方案，但是在电商领域等互联网场景下，基于CP的强一致性方案在数据库性能和系统处理能力上会存在一定的瓶颈。所以在互联网场景中更多采用柔性事务，所谓的柔性事务是遵循BASE理论来实现的事务模型，它有两个特性：基本可用、柔性状态。在本节中主要基于柔性事务模型来分析互联网产品中分布式事务的常见解决方案。

### TCC补偿型方案

​	TCC（Try-Confirm-Cancel）是一种比较成熟的分布式数据一致性解决方案，它实际上是把一个完整的业务拆分为如下三个步骤。

* Try：这个阶段主要是对数据的校验或者资源的预留。
* Confirm：确认真正执行的任务，只操作Try阶段预留的资源。
* Cancel：取消执行，释放Try阶段预留的资源。



​	其实TCC是一种两阶段提交的思想，第一阶段通过Try进行准备工作，第二阶段Confirm/Cancel表示Try阶段操作的确认和回滚。在分布式事务场景中，每个服务实现TCC之后，就作为其中的一个资源，参与到整个分布式事务中。然后主业务服务在第一阶段中分别调用所有TCC服务的Try方法。最后根据第一个阶段的执行情况来决定对第二阶段的Confirm或者Cancel。TCC执行流程如图8-6所示。

![](https://pic.imgdb.cn/item/60e66b115132923bf8da9dbc.jpg)

​	需要注意的是，TCC服务支持接口调用失败发起重试，所以TCC暴露的接口都需要满足幂等性。

### 基于可靠性消息的最终一致性方案

​	基于可靠性消息的最终一致性是互联网公司比较常用的分布式数据一致性解决方案，它主要利用消息中间件（Kafka、RocketMQ或RabbitMQ）的可靠性机制来实现数据一致性的投递。

![](https://pic.imgdb.cn/item/60e66bd35132923bf8de9fec.jpg)

​	在图8-7的解决方案中，我们不难发现一些问题，就是支付服务的本地事务与发送消息这个操作的原子性问题，具体描述如下。

* 先发送消息，再执行数据库事务，在这种情况下可能会出现消息发送成功但是本地事务更新失败的情况，仍然会导致数据不一致的问题

![](https://pic.imgdb.cn/item/60e66bf15132923bf8df3db6.jpg)

* 先执行数据库事务操作，再发送消息，在这种情况下可能会出现MQ响应超时导致异常，从而将本地事务回滚，但消息可能已经发送成功了，也会存在数据不一致的问题。

![](https://pic.imgdb.cn/item/60e66c0d5132923bf8dfd5dd.jpg)

​	以上问题也有很多成熟的解决方案，以RocketMQ为例，它提供了事务消息模型，如图8-8所示，具体的执行逻辑如下：

* 生产者发送一个事务消息到消息队列上，消息队列只记录这条消息的数据，此时消费者无法消费这条消息。
* 生产者执行具体的业务逻辑，完成本地事务的操作。
* 接着生产者根据本地事务的执行结果发送一条确认消息给消息队列服务器，如果本地事务执行成功，则发送一个Commit消息，表示在第一步中发送的消息可以被消费，否则，消息队列服务器会把第一步存储的消息删除。
* 消息队列服务器上存储的消息被生产者确认之后，消费者就可以消费这条消息，消息消费完成之后发送一个确认标识给消息队列服务器，表示该消息投递成功。

![](https://pic.imgdb.cn/item/60e66efa5132923bf8ef793b.jpg)

​	在RocketMQ事务消息模型中，事务是由生产者来完成的，消费者不需要考虑，因为消息队列可靠性投递机制的存在，如果消费者没有签收该消息，那么消息队列服务器会重复投递，从而实现生产者的本地数据和消费者的本地数据在消息队列的机制下达到最终一致。

​	不难发现，在RocketMQ的事务消息模型中最核心的机制应该是事务回查，实际上查询模式在很多类似的场景中都可以应用。在分布式系统中，由于网络通信的存在，服务之间的远程通信除成功和失败两种结果外，还存在一种未知状态，比如网络超时。服务提供者可以提供一个查询接口向外部输出操作的执行状态，服务调用方可以通过调用该接口得知之前操作的结果并进行相应的处理。

### 最大努力通知型

​	最大努力通知型和基于可靠性消息的最终一致性方案的实现是类似的，它是一种比较简单的柔性事务解决方案，也比较适用于对数据一致性要求不高的场景，最典型的使用场景是支付宝支付结果通知，实现流程如图8-9所示。

![](https://pic.imgdb.cn/item/60e670625132923bf8f6c8fd.jpg)

下面站在商户的角度来分析最大努力通知型的处理过程。

* 商户先创建一个支付订单，然后调用支付宝发起支付请求。
* 支付宝唤醒支付页面完成支付操作，支付宝同样会针对该商户创建一个支付交易，并且根据用户的支付结果记录支付状态。
* 针对这个订单，在理想状态下支付宝的交易状态和商户的交易状态会在通知完成后达到最终一致。但是由于网络的不确定性，支付结果通知可能会失败或者丢失，导致商户端的支付订单的状态是未知的。所以最大努力通知型的作用就体现了，如果商户端在收到支付结果通知后没有返回一个“SUCCESS”状态码，那么这个支付结果回调请求会以衰减重试机制（逐步拉大通知的间隔）继续触发，比如1min、5min、10min、30min……直到达到最大通知次数。如果达到指定次数后商户还没有返回确认状态，怎么处理呢？
* 支付宝提供了一个交易结果查询接口，可以根据这个支付订单号去支付宝查询支付状态，然后根据返回的结果来更新商户的支付订单状态，这个过程可以通过定时器来触发，也可以通过人工对账来触发。



​	从上述分析可以发现，所谓的最大努力通知，就是在商户端如果没有返回一个消息确认时，支付宝会不断地进行重试，直到收到一个消息确认或者达到最大重试次数。

## 分布式事务框架Seata

​	Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。它提供了AT、TCC、Saga和XA事务模式，为开发者提供了一站式的分布式事务解决方案。其中TCC和XA我们前面分析过，AT和Saga这两种事务模式是什么呢？下面先来简单介绍一下这两种事务模式。

### AT模式

​	AT模式是Seata最主推的分布式事务解决方案，它是基于XA演进而来的一种分布式事务模式，所以它同样分为三大模块，分别是TM、RM和TC，其中TM和RM作为Seata的客户端与业务系统集成，TC作为Seata的服务器独立部署。TM表示事务管理器（Transaction Manager），它负责向TC注册一个全局事务，并生成一个全局唯一的XID。在AT模式下，每个数据库资源被当作一个RM（Resource Manager），在业务层面通过JDBC标准的接口访问RM时，Seata会对所有请求进行拦截。每个本地事务进行提交时，RM都会向TC（Transaction Coordinator，事务协调器）注册一个分支事务。Seata的AT事务模式如图8-10所示。

![](https://pic.imgdb.cn/item/60e672c45132923bf8034b66.jpg)

* TM向TC注册全局事务，并生成全局唯一的XID。
* RM向TC注册分支事务，并将其纳入该XID对应的全局事务范围。
* RM向TC汇报资源的准备状态。
* TC汇总所有事务参与者的执行状态，决定分布式事务是全部回滚还是提交。
* TC通知所有RM提交/回滚事务。



​	AT模式和XA一样，也是一个两阶提交事务模型，不过和XA相比，做了很多优化，笔者会在后续的章节中重点分析AT模式的实现原理。

### Saga模式

​	Saga模式又称为长事务解决方案，它是由普林斯顿大学的Hector Garcia-Molina和Kenneth Salem提出的，主要描述的是在没有两阶段提交的情况下如何解决分布式事务问题。其核心思想是：把一个业务流程中的长事务拆分为多个本地短事务，业务流程中的每个参与者都提交真实的提交给该本地短事务，当其中一个参与者事务执行失败，则通过补偿机制补偿前面已经成功的参与者。

​	如图8-11所示，Saga由一系列sub-transaction Ti组成，每个Ti都有对应的补偿动作Ci，补偿动作用于撤销Ti造成的数据变更结果。它和TCC相比，少了Try这个预留动作，每一个Ti操作都真实地影响到数据库。

![](https://pic.imgdb.cn/item/60e673e95132923bf8091f7b.jpg)

* T1，T2，T3，...，Ti：这种方式表示所有事务都正常执行。
* T1，T2，...，Tj，Cj，...，C2，C1（其中0<j<i）：这种方式表示执行到Tj事务时出现异常，通过补充操作撤销之前所有成功的sub-transaction。

另外，Saga提供了以下两种补偿恢复方式。

* 向后恢复，也就是上面提到的第二种工作模式，如果任一子事务执行失败，则把之前执行的结果逐一撤销。
* 向前恢复，也就是不进行补偿，而是对失败的事务进行重试，这种方式比较适合于事务必须要执行成功的场景。



​	不管是向后恢复还是向前恢复，都可能出现失败的情况，在最坏的情况下只能人工干预处理。

#### Saga的优劣势

​	和XA或者TCC相比，它的优势包括：一阶段直接提交本地事务；没有锁等待，性能较高；在事件驱动的模式下，短事务可以异步执行；补偿机制的实现比较简单。

​	缺点是Saga并不提供原子性和隔离性支持，隔离性的影响是比较大的，比如用户购买一个商品后系统赠送一张优惠券，如果用户已经把优惠券使用了，那么事务如果出现异常要回滚时就会出现问题。

#### Saga的实现方式

​	电商平台下单的流程是一个典型的长事务场景，根据Saga模式的定义，先将长事务拆分成多个本地短事务，每个服务的本地事务按照执行顺序逐一提交，一旦其中一个服务的事务出现异常，则采用补偿的方式逐一撤回。这一过程的实现会涉及Saga的的协调模式，它有两种常用的协调模式。

* 事件/编排式：把Saga的决策和执行顺序逻辑分布在Saga的每一个参与者中，它们通过交换事件的方式来进行沟通。
* 命令/协同式：把Saga的决策和执行顺序逻辑集中在一个Saga控制类中，它以命令/回复的方式与每项服务进行通信，告诉它们应该执行哪些操作。



![](https://pic.imgdb.cn/item/60e6752e5132923bf80fd605.jpg)

**事件/编排式**

​	在基于事件的编排模式中，第一个服务执行完一个本地事务之后，发送一个事件。这个事件会被一个或者多个服务监听，监听到事件的服务再执行本地事务并发布新的事件，此后一直延续这种事件触发模式，直到该业务流程中最后一个服务的本地事务执行结束，才意味着整个分布式长事务也执行结束，如图8-13所示。

![](https://pic.imgdb.cn/item/60e675515132923bf8109311.jpg)

​	这个流程看起来很复杂，但是却是比较常见的解决方案，下面简单描述一下具体的步骤。

* 订单服务创建新的订单，把订单状态设置为待支付，并发布一个ORDER_CREATE_EVENT事件。
* 库存服务监听到ORDER_CREATE_EVENT事件后，执行本地的库存冻结方法，如果执行成功，则发布一个ORDER_PREPARED_EVENT事件。
* 支付服务监听ORDER_PREPARED_EVENT事件后，执行账户扣款方法，并发布PAY_ORDER_EVENT事件。
* 最后，积分服务监听PAY_ORDER_EVENT事件，增加账户积分，并更新订单状态为成功。

**命令/协同式**

​	命令/协同式需要定义一个Saga协调器，负责告诉每一个参与者该做什么，Saga协调器以命令/回复的方式与每项服务进行通信，如图8-14所示。

![](https://pic.imgdb.cn/item/60ed93455132923bf82dcdb5.jpg)

命令/协同式的实现步骤如下：

* 订单服务首先创建一个订单，然后创建一个订单Saga协调器，启动订单事务。
* Saga协调器向库存服务发送冻结库存命令，库存服务通过Order Saga Reply Queue回复执行结果。
* 接着，Saga协调器继续向支付服务发起账户扣款命令，支付服务通过Order Saga Reply Queue回复执行结果。
* 最后，Saga协调器向积分服务发起增加积分命令，积分服务回复执行结果。

## Seata的安装

​	Seata是一个需要独立部署的中间件，除直接部署外，它还支持多种部署方式，比如Docker、Kubernetes、Helm。本节主要讲解直接安装的方式。

* 在Seata官网下载1.0.0版本的安装包，1.0.0是笔者写作时最新的发布版本。
* 进入${SEATA_HOME}\bin目录，根据系统类型执行相应的启动脚本，在Linux/Max下的执行命令如下。

```
sh seata-server.sh
```

seata-server.sh支持设置启动参数，完整的参数列表如表8-1所示。

![](https://pic.imgdb.cn/item/60ed93ae5132923bf8308206.jpg)

### file存储模式

​	Server端存储模式（store.mode）有file、db两种（后续将引入Raft实现Seata的高可用机制），file存储模式无须改动，直接启动即可。

​	file存储模式为单机模式，全局事务会话信息持久化在本地文件${SEATA_HOME}\bin\sessionStore\root.data中，性能较高，启动命令如下：

![](https://pic.imgdb.cn/item/60ed9a035132923bf85cbfb0.jpg)

### db存储模式

​	db存储模式为高可用模式，全局事务会话信息通过db共享，性能相对差一些。操作步骤如下。

* 创建表结构。Seata全局事务会话信息由全局事务、分支事务、全局锁构成，对应表globaltable、branchtable、lock_table。



```mysql
- -------------------------------- The script used when storeMode is 'db' --------------------------------
-- the table to store GlobalSession data
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store BranchSession data
CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- the table to store lock data
CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(128),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

* 设置事务日志存储方式，进入${SEATA_HOME}\conf\file.conf，修改store.mode="db"。

* 修改数据库连接：

![](https://pic.imgdb.cn/item/60ed9ac25132923bf8625b83.jpg)

* 启动seata-server：

![](https://pic.imgdb.cn/item/60ed9af95132923bf8640dda.jpg)

参数说明如下。

-h：注册到注册中心的IP地址，Seata-Server可以把自己注册到注册中心，支持Nacos、Eureka、Redis、ZooKeeper、Consul、Etcd3、Sofa。

-p：Server RPC监听端口。

-m：全局事务会话信息存储模式，包括file、db，优先读取启动参数。

-n：Server node，有多个Server时，需区分各自节点，用于生成不同区间的transactionId，以免冲突。

### Seata服务端配置中心说明

在${SEATA_HOME}\conf目录下有两个配置文件，分别是registry.conf和file.conf。

#### registry.conf配置说明

**registry**

registry.conf中包含两项配置：registry、config，完整的配置内容如下。

![](https://pic.imgdb.cn/item/60ed9bb25132923bf8699a0b.jpg)

![](https://pic.imgdb.cn/item/60ed9bc25132923bf86a155d.jpg)

![](https://pic.imgdb.cn/item/60ed9bd45132923bf86aa44b.jpg)

​	registry表示配置Seata服务注册的地址，支持目前市面上所有主流的注册中心组件。它的配置非常简单，通过type指定注册中心的类型，然后根据指定的类型配置对应的服务地址信息，比如当type=nacos时，则匹配到Nacos的配置项如下。

![](https://pic.imgdb.cn/item/60ed9bef5132923bf86b736d.jpg)

**config**

​	config配置用于配置Seata服务端的配置文件地址，也就是可以通过config配置指定Seata服务端的配置信息的加载位置，它支持从远程配置中心读取和本地文件读取两种方式。如果配置为远程配置中心，可以使用type指定，配置形式和registry相同。

![](https://pic.imgdb.cn/item/60ed9c0a5132923bf86c46d7.jpg)

### file.conf配置说明

​	file.conf存储的是Seata服务端的配置信息，完整配置如下。它包含transport、server、metrics，分别表示协议配置、服务端配置、监控。

![](https://pic.imgdb.cn/item/60ed9c525132923bf86e8709.jpg)

![](https://pic.imgdb.cn/item/60ed9c5e5132923bf86ee299.jpg)

![](https://pic.imgdb.cn/item/60ed9c6f5132923bf86f719b.jpg)

​	Seata服务端启动时会加载file.conf中的配置参数，这些参数读者不需要记，只需要知道这些参数可以优化即可，在Seata官网上对参数有非常详细的说明。

#### 从配置中心加载配置

​	从前面的分析过程中我们知道，Seata服务在启动时可以将自己注册到注册中心上，并且file.conf文件中的配置同样可以保存在配置中心，接下来我们尝试把配置信息存储到Nacos上。

**将配置上传到Nacos**

​	在GitHub的官方代码托管平台下载Seata的源码，在源码包中有一个script文件夹（目前只在源码包中存在），目录结构如下：

* client：存放客户端的SQL脚本，参数配置。
* config-center：各个配置中心参数导入脚本，config.txt（包含server和client）为通用参数文件。
* server：服务端数据库脚本及各个容器配置。



​	进入config-center目录，包含config.txt和不同配置中心的目录（该目录下包含shell脚本和py脚本）。其中config.txt存放的是Seata客户端和服务端的所有配置信息。

​	在config-center\nacos目录下，执行如下脚本：

![](https://pic.imgdb.cn/item/60ed9d2b5132923bf8755122.jpg)

​	该脚本的作用是把config.txt中的配置信息上传到Nacos配置中心。由于config.txt中提供的是默认配置，在实际使用时可以先修改该文件中的内容，再执行上传操作（当然，也可以上传完成之后在Nacos控制台上根据实际需求修改对应的配置项）。

![](https://pic.imgdb.cn/item/60ed9d485132923bf876314f.jpg)

**Seata服务端修改配置加载位置**

​	进入${SEATA_HOME}\conf目录，修改registry.conf文件中的config段，完整配置如下：

![](https://pic.imgdb.cn/item/60ed9dba5132923bf879d0b4.jpg)

## AT模式Dubbo集成Seata

​	本节中，我们仍然通过一个电商平台的购买逻辑，基于Dubbo集成Seata实现一个分布式事务的解决方案。在整个业务流程中，会涉及如下三个服务。

* 订单服务：用于创建订单。
* 账户服务：从账户中扣除余额。
* 库存服务：扣减指定商品的库存数量。

![](https://pic.imgdb.cn/item/60ed9ed15132923bf882d544.jpg)

### 项目准备

​	使用第5章构建的基于Spring Boot+Nacos+Dubbo的项目结构，分别构建以下服务。

* sample-order-service，订单服务。
* sample-repo-service，库存服务。
* sample-account-service，账户服务。
* sample-seata-common，公共服务组件。
* sample-rest-web，提供统一业务的REST接口服务。



​	其中sample-order-service、sample-repo-service、sample-account-service是基于Spring Boot+Dubbo构建的微服务，sample-rest-web提供统一的业务服务入口，sample-seata-common提供公共组件。

> 注意，在当前演示的项目中用到了Nacos，所以需要提前启动Nacos服务。

### 数据库准备

​	创建三个数据库：seata_order、seata_repo、seata_account，并分别在这三个数据库中创建对应的业务表。

![](https://pic.imgdb.cn/item/60ed9fb55132923bf88a7bdb.jpg)

![](https://pic.imgdb.cn/item/60ed9fc25132923bf88aee7a.jpg)

### 核心方法说明

​	sample-order-service、sample-repo-service、sample-account-service这三个服务需要集成MyBatis，用于和数据库交互，集成的过程比较简单，笔者不做过多阐述。

**sample-account-service**

​	账户服务提供余额扣减的功能，具体代码如下。

> 注意，这个案例只是为了演示分布式事务的场景，并没有考虑到高并发情况下的数据安全问题。

![](https://pic.imgdb.cn/item/60eda0aa5132923bf892e499.jpg)

**sample-order-service**

​	订单服务负责创建订单，并且在创建订单之前先基于Dubbo协议调用账户服务的资金扣减接口。

![](https://pic.imgdb.cn/item/60eda0c75132923bf893ecba.jpg)

![](https://pic.imgdb.cn/item/60eda0d55132923bf89471c8.jpg)

**sample-repo-service**

​	库存服务提供库存扣减功能，同样这里也没有处理高并发场景下的性能及安全问题。

![](https://pic.imgdb.cn/item/60eda1005132923bf895f3ab.jpg)

**sample-rest-web**

​	sample-rest-web是基于Spring Boot的Web项目，主要用于对外提供以业务为维度的REST接口，它会分别调用库存服务和订单服务，实现库存扣减及订单创建功能。

![](https://pic.imgdb.cn/item/60eda1255132923bf8974436.jpg)

![](https://pic.imgdb.cn/item/60eda1385132923bf897f475.jpg)

### 项目启动顺序及访问

​	这几个项目彼此之间存在依赖关系，服务与服务之间的依赖可以参考图8-16，服务的启动顺序为：

* sample-seata-common为公共组件，需要先通过mvn。
* install安装到本地仓库，给其他服务依赖。
* 接下来启动账户服务sample-account-service，它会被订单服务调用。
* 启动订单服务sample-order-service。
* 启动库存服务sample-repo-service。
* 启动sample-rest-web，它作为REST的业务入口，最后启动。



​	通过如下curl命令进行整体下单流程的测试，并监控数据库表中对应数据的变化，确保整个调用链路是正常的。

![](https://pic.imgdb.cn/item/60eda1ac5132923bf89c0f5a.jpg)

###  整合Seata实现分布式事务

​	在上述流程中，假设库存扣减成功了，但是在创建订单时如果由于账户资金不足导致失败，就会出现数据不一致的场景。按照正常的流程来说，被扣减的库存需要加回去，这就是一个分布式事务的场景。接下来我们在项目中整合Seata来解决该问题。

#### 添加Seata Jar包依赖

![](https://pic.imgdb.cn/item/60eda2025132923bf89f15b1.jpg)

#### 添加Seata配置项目

​	同样分别在4个项目中的application.yml文件中添加Seata的配置项，具体配置明细如下。

![](https://pic.imgdb.cn/item/60eda21e5132923bf8a01118.jpg)

![](https://pic.imgdb.cn/item/60eda22f5132923bf8a0a7af.jpg)

![](https://pic.imgdb.cn/item/60eda2415132923bf8a147f3.jpg)

上述配置中有几个配置项需要注意：

* seata.support.spring.datasource-autoproxy：true属性表示数据源自动代理开关，在sample-order-service、sample-account-service、sample-repo-service中设置为true，在sample-rest-web中设置为false，因为该项目并没有访问数据源，不需要代理。
* 如果注册中心为file，seata.service.grouplist需要填写Seata服务端连接地址。在默认情况下，注册中心配置为file，如果需要从注册中心上进行服务发现，可以增加如下配置。

![](https://pic.imgdb.cn/item/60eda3245132923bf8a95a65.jpg)

​	tx-service-group表示指定服务所属的事务分组，如果没有指定，默认使用spring.application.name加上字符串-seata-service-group。需要注意这两项配置必须要配置一项，否则会报错。

#### 添加回滚日志表

​	分别在3个数据库seata-account、seata-repo、seata-order中添加一张回滚日志表，用于记录每个数据库表操作的回滚日志，当某个服务的事务出现异常时会根据该日志进行回滚。

![](https://pic.imgdb.cn/item/60eda3475132923bf8aaa076.jpg)

#### sample-rest-web增加全局事务控制

​	修改sample-rest-web工程的RestOrderServiceImpl，做两件事情：

* 增加＠GlobalTransactional全局事务注解。
* 模拟一个异常处理，当商品编号等于某个指定的值时抛出异常，触发整个事务的回滚。

![](https://pic.imgdb.cn/item/60eda3685132923bf8abd510.jpg)

![](https://pic.imgdb.cn/item/60eda3755132923bf8ac52ec.jpg)

#### 启动服务进行测试

![](https://pic.imgdb.cn/item/60eda38b5132923bf8ad2402.jpg)

​	从异常触发的位置来看，如果没有引入分布式事务，那么即便出现了异常，由于库存扣减、订单创建、账户资金扣减等操作都已经生效，所以数据无法被回滚。在引Seata之后，在异常出现后会触发各个事务分支的数据回滚，保证数据的正确性，如果配置正常，在3个Dubbo服务的控制台中会获得如下输出，完成事务回滚操作。

![](https://pic.imgdb.cn/item/60eda3a45132923bf8ae0b2f.jpg)

## Spring Cloud Alibaba Seata

### Spring Cloud项目准备

​	构建4个项目，实现逻辑及核心代码与8.5节完全一致，只增加了Greenwich.SR2版本的Spring Cloud依赖。项目名称如下：

* spring-cloud-seata-account.
* spring-cloud-seata-repo.
* spring-cloud-seata-order.
* spring-cloud-seata-rest.

### 集成Spring Cloud Alibaba Seata

​	在上述的4个服务中分别集成Spring Cloud Alibaba Seata，步骤如下。

* 添加依赖包。

![](https://pic.imgdb.cn/item/60eda3fc5132923bf8b13484.jpg)

* spring-cloud-alibaba-seata不支持yml形式，所以只能使用file.conf和registry.conf文件来描述客户端的配置信息。可以直接将${SEATA_HOME}\conf目录下的这两个文件复制到项目的resource目录中。同样，如果希望从配置中心加载这些配置项，在registry.conf中指定配置中心地址即可。file.conf完整配置项如下。

![](https://pic.imgdb.cn/item/60eda4115132923bf8b1f7ac.jpg)

![](https://pic.imgdb.cn/item/60eda4225132923bf8b2963a.jpg)

![](https://pic.imgdb.cn/item/60eda42e5132923bf8b307b7.jpg)

​	在上述配置中，vgroup_mapping.${txServiceGroup}="default"表示事务群组，其中${txServiceGroup}表示事务服务分组，它的值要设置为spring.cloud.alibaba.seata.tx-service-group或者spring.application.name+"seata.tx-service-group"。

​	在上述配置中，vgroup_mapping.${txServiceGroup}="default"表示事务群组，其中${txServiceGroup}表示事务服务分组，它的值要设置为spring.cloud.alibaba.seata.tx-service-group或者spring.application.name+"seata.tx-service-group"。

* 在spring-cloud-seata-account、spring-cloud-seata-repo、spring-cloud-seata-order这3个服务中添加一个配置类SeataAutoConfig，主要做两件事：
  * 配置数据源代理DataSourceProxy。
  * 初始化GlobalTransactionScanner，装载到Spring IoC容器。

![](https://pic.imgdb.cn/item/60eda4855132923bf8b6343f.jpg)

![](https://pic.imgdb.cn/item/60eda4955132923bf8b6cd71.jpg)

![](https://pic.imgdb.cn/item/60eda4a65132923bf8b77079.jpg)

​	在8.5节演示的过程中是不存在上述这个配置类的，原因是seata-spring-boot-starter主动完成了这些功能，并且Seata自动实现了数据源的代理。而这里演示的过程是通过手动配置来完成的。其中，GlobalTransactionScanner中的两个参数分别是applicationId（应用名称）和txServiceGroup（事务分组），事务分组会在后续的章节中详细说明。

![](https://pic.imgdb.cn/item/60eda4b85132923bf8b81b52.jpg)

​	需要注意的是，2.1.1.RELEASE版本内嵌的seata-all版本是0.9.0，所以它无法和seata-spring-boot-starter兼容。

​	如果采用上述自定义配置类SeataAutoConfig，需要在＠SpringBootApplication注解内exclude掉spring-cloud-alibaba-seata内的GlobalTransactionAutoConfiguration，否则两个配置类会产生冲突。

​	@SpringBootApplication(exclude=GlobalTransactionAutoConfiguration.class)

* spring-cloud-seata-rest项目中的配置类如下，由于它并没有关联数据源，所以只需要装载GlobalTransactionScanner即可，它主要自动扫描包含GlobalTransactional注解的代码逻辑。

![](https://pic.imgdb.cn/item/60eda4f15132923bf8ba36e1.jpg)

​	至此，基于Spring Cloud生态下的Seata框架整合就配置完成了。实际上，由于Spring Cloud并没有提供分布式事务处理的标准，所以它不像配置中心那样插拔式地集成各种主流的解决方案。Spring Cloud Alibaba Seata本质上还是基于Spring Boot自动装配来集成的，在没有提供标准化配置的情况下只能根据不同的分布式事务框架进行配置和整合。

### 关于事务分组的说明

​	在Seata Client端的file.conf配置中有一个属性vgroup_mapping，它表示事务分组映射，是Seata的资源逻辑，类似于服务实例，它的主要作用是根据分组来获取Seata Server的服务实例。

**服务分组的工作机制**

首先，在应用程序中需要配置事务分组，也就是使用GlobalTransactionScanner构造方法中的txServiceGroup参数，这个参数有如下几种赋值方式。

* 默认情况下，获取spring.application.name的值+"-seata-service-group"。
* 在Spring Cloud Alibaba Seata中，可以使用spring.cloud.alibaba.seata.tx-service-group赋值。
* 在Seata-Spring-Boot-Starter中，可以使用seata.tx-service-group赋值。



​	然后，Seata客户端会根据应用程序的txServiceGroup去指定位置（file.conf或者远程配置中心）查找service.vgroup_mapping.${txServiceGroup}对应的配置值，该值代表TC集群（Seata Server）的名称。

​	最后，程序会根据集群名称去配置中心或者file.conf中获得对应的服务列表，也就是clusterName.grouplist对应的TC集群真实的服务列表。实现原理如图8-17所示，具体步骤描述如下。

* 获取事务分组spring-cloud-seata-repo配置的值Agroup。
* 拿到事务分组的值Agroup，拼接成service.vgroup_mapping.Agroup，去配置中心查找集群名，得到default。
* 拼接service.default.grouplist，查找集群名对应的Seata Server服务地址：192.168.1.1：8091。

![](https://pic.imgdb.cn/item/60eda5525132923bf8bdc15f.jpg)

**思考事务分组设计**

​	通过上述分析可以发现，在客户端获取服务器地址并没有直接采用服务名称，而是增加了一层事务分组映射到集群的配置。这样做的好处在于，事务分组可以作为资源的逻辑隔离单位，当某个集群出现故障时，可以把故障缩减到服务级别，实现快速故障转移，只需要切换对应的分组即可。

# RocketMQ分布式消息通信

## 什么是RocketMQ

​	RocketMQ是一个低延迟、高可靠、可伸缩、易于使用的分布式消息中间件（也称消息队列），经过阿里巴巴多年双11的验证，是由阿里巴巴开源捐献给Apache的顶级项目。

### RocketMQ的应用场景

* 削峰填谷：诸如秒杀、抢红包、企业开门红等大型活动皆会带来较高的流量脉冲，很可能因没做相应的保护而导致系统超负荷甚至崩溃，或因限制太过导致请求大量失败而影响用户体验，RocketMQ可提供削峰填谷的服务来解决这些问题。
* 异步解耦：交易系统作为淘宝/天猫主站最核心的系统，每笔交易订单数据的产生会引起几百个下游业务系统的关注，包括物流、购物车、积分、流计算分析等，整体业务系统庞大而且复杂，RocketMQ可实现异步通信和应用解耦，确保主站业务的连续性。
* 顺序收发：细数一下，日常需要保证顺序的应用场景非常多，例如证券交易过程中的时间优先原则，交易系统中的订单创建、支付、退款等流程，航班中的旅客登机消息处理等。与先进先出（First In First Out，缩写FIFO）原理类似，RocketMQ提供的顺序消息即保证消息的FIFO。
* 分布式事务一致性：交易系统、红包等场景需要确保数据的最终一致性，大量引入RocketMQ的分布式事务，既可以实现系统之间的解耦，又可以保证最终的数据一致性。
* 大数据分析：数据在“流动”中产生价值，传统数据分析大都基于批量计算模型，无法做到实时的数据分析，利用RocketMQ与流式计算引擎相结合，可以很方便地实现对业务数据进行实时分析。
* 分布式缓存同步：天猫双11大促，各个分会场琳琅满目的商品需要实时感知价格的变化，大量并发访问会导致会场页面响应时间长，集中式缓存因为带宽瓶颈限制商品变更的访问流量，通过RocketMQ构建分布式缓存，可实时通知商品数据的变化。

### RocketMQ的安装

略

### RocketMQ如何发送消息

​	Spring Cloud Alibaba已集成RocketMQ，使用Spring Cloud Stream对RocketMQ发送和接收消息。

* 第1步，在pom.xml中引入Jar包。

![](https://pic.imgdb.cn/item/60f544465132923bf8f749b1.jpg)

* 第2步，配置application.properties。

![](https://pic.imgdb.cn/item/60f544e25132923bf8fa6b2f.jpg)

​	name-server指定RocketMQ的NameServer地址，将指定名称为output的Binding消息发送到TopicTest。

* 第3步，使用Binder发送消息。

![](https://pic.imgdb.cn/item/60f545105132923bf8fb53c3.jpg)

​	＠EnableBinding（{Source.class}）表示绑定配置文件中名称为output的消息通道Binding，Source类中定义的消息通道名称为output。发送HTTP请求http://localhost:8081/send?msg=tcever将消息发送到RocketMQ中。

​	在实际开发场景中会存在多个发送消息通道，可以自定义消息通道的名称，参考Source类自定义一个接口，修改通道名称和相关配置即可。

![](https://pic.imgdb.cn/item/60f5456b5132923bf8fd11c2.jpg)

​	到此，就可以添加一个自定义发送消息通道，使用orderOutput消息发送到TopicOrder中了。

### RocketMQ如何消费消息

​	RocketMQ消费消息的步骤如下。

* 第1步，pom.xml中引入Jar包。

![](https://pic.imgdb.cn/item/60f545b55132923bf8fe8764.jpg)

* 第2步，配置application.properties。

![](https://pic.imgdb.cn/item/60f545ce5132923bf8ff0079.jpg)

​	name-server指定RocketMQ的NameServer地址，destination指定Topic名称，指定名称为input的Binding接收TopicTest的消息。

* 第3步，定义消息监听。

![](https://pic.imgdb.cn/item/60f5462e5132923bf800dca0.jpg)

​	＠EnableBinding（{Sink.class}）表示绑定配置文件中名称为input的消息通道Binding，Sink类中定义的消息通道的名称为input，＠StreamListener表示定义一个消息监听器，接收RocketMQ中的消息。

​	在实际开发场景中同样会存在多个接收消息通道，可以自定义消息通道的名称，参考Sink类自定义一个接口，修改通道名称和相关配置即可。

![](https://pic.imgdb.cn/item/60f546905132923bf802b866.jpg)

![](https://pic.imgdb.cn/item/60f546aa5132923bf803352e.jpg)

##  Spring Cloud Alibaba RocketMQ

​	Spring Cloud Stream是Spring Cloud体系内的一个框架，用于构建与共享消息传递系统连接的高度可伸缩的事件驱动微服务，其目的是简化消息业务在Spring Cloud应用程序中的开发。

​	Spring Cloud Stream的架构图如图9-1所示，应用程序通过Spring Cloud Stream注入的输入通道inputs和输出通道outputs与消息中间件Middleware通信，消息通道通过特定的中间件绑定器Binder实现连接到外部代理。

![](https://pic.imgdb.cn/item/60f547bb5132923bf8086c26.jpg)

​	Spring Cloud Stream的实现基于发布/订阅机制，核心由四部分构成：Spring Framework中的Spring Messaging和Spring Integration，以及Spring Cloud Stream中的Binders和Bindings。

​	**Spring Messaging**：Spring Framework中的统一消息编程模型，其核心对象如下。

* Message：消息对象，包含消息头Header和消息体Payload。
* MessageChannel：消息通道接口，用于接收消息，提供send方法将消息发送至消息通道。
* MessageHandler：消息处理器接口，用于处理消息逻辑。



​	**Spring Integration**：Spring Framework中用于支持企业集成的一种扩展机制，作用是提供一个简单的模型来构建企业集成解决方案，对Spring Messaging进行了扩展。

* MessageDispatcher：消息分发接口，用于分发消息和添加删除消息处理器。
* MessageRouter：消息路由接口，定义默认的输出消息通道。
* Filter：消息的过滤注解，用于配置消息过滤表达式。
* Aggregator：消息的聚合注解，用于将多条消息聚合成一条。
* Splitter：消息的分割，用于将一条消息拆分成多条。



**Binders**：目标绑定器，负责与外部消息中间件系统集成的组件。

**Bindings**：外部消息中间件系统与应用程序提供的消息生产者和消费者（由Binders创建）之间的桥梁。

​	Spring Cloud Stream官方提供了Kafka Binder和RabbitMQ Binder，用于集成Kafka和RabbitMQ，Spring Cloud Alibaba中加入了RocketMQ Binder，用于将RocketMQ集成到Spring Cloud Stream。

### Spring Cloud Alibaba RocketMQ架构图

​	Spring Cloud Alibaba RocketMQ的架构图如图9-2所示，总体分为四部分。

![](https://pic.imgdb.cn/item/60f5487f5132923bf80c25b3.jpg)



* MessageChannel（output）：消息通道，用于发送消息，Spring Cloud Stream的标准接口。
* MessageChannel（input）：消息通道，用于订阅消息，Spring Cloud Stream的标准接口。
* Binder bindProducer：目标绑定器，将发送通道发过来的消息发送到RocketMQ消息服务器，由Spring Cloud Alibaba团队按照Spring Cloud Stream的标准协议实现。
* Binder bindConsumer：目标绑定器，将接收到RocketMQ消息服务器的消息推送给订阅通道，由Spring Cloud Alibaba团队按照Spring Cloud Stream的标准协议实现。
  后面将以代码为例，通过源码深入分析Spring Cloud Alibaba RocketMQ。

### Spring Cloud Stream消息发送流程

​	Spring Cloud Stream消息发送流程如图9-3所示，包括发送、订阅、分发、委派、消息处理等，具体实现如下。

![](https://pic.imgdb.cn/item/60f548e55132923bf80e0e5f.jpg)



* 在业务代码中调用MessageChannel接口的Send（）方法，例如source.output（）.send（message）。

![](https://pic.imgdb.cn/item/60f5492f5132923bf80f7a64.jpg)

​	AbstractMessageChannel是消息通道的基本实现类，提供发送消息和接收消息的公用方法。

![](https://pic.imgdb.cn/item/60f5495b5132923bf8104a2e.jpg)

* 消息发送到AbstractSubscribableChannel类实现的doSend（）方法如下。

![](https://pic.imgdb.cn/item/60f549cf5132923bf81272be.jpg)

* 通过消息分发类MessageDispatcher把消息分发给MessageHandler。

![](https://pic.imgdb.cn/item/60f549e55132923bf812e013.jpg)

* 通过消息分发类MessageDispatcher把消息分发给MessageHandler。

![](https://pic.imgdb.cn/item/60f549fb5132923bf8134630.jpg)



​	AbstractSubscribableChannel的实现类DirectChannel得到MessageDispatcher的实现类UnicastingDispatcher。

![](https://pic.imgdb.cn/item/60f54a1b5132923bf813df89.jpg)

调用dispatch（）方法把消息分发给各个MessageHandler。

![](https://pic.imgdb.cn/item/60f54a385132923bf8146bc5.jpg)

![](https://pic.imgdb.cn/item/60f54a495132923bf814bda5.jpg)

​	遍历所有MessageHandler，调用handleMessage（）处理消息。

![](https://pic.imgdb.cn/item/60f54a985132923bf816285a.jpg)

​	查看MessageHandler是从哪里来的，也就是handlers列表中的MessageHandler是如何添加的。

![](https://pic.imgdb.cn/item/60f54ae05132923bf817859e.jpg)

* AbstractMessageChannelBinder在初始化Binding时，会创建并初始化SendingHandler，调用subscribe（）添加到handlers列表。

![](https://pic.imgdb.cn/item/60f54bbb5132923bf81b7d08.jpg)

![](https://pic.imgdb.cn/item/60f54bd05132923bf81bdb6c.jpg)

### RocketMQ Binder集成消息发送

​	AbstractMessageChannelBinder类提供了创建MessageHandler的规范，createProducerMessageHandler方法在初始化Binder的时候会加载。

![](https://pic.imgdb.cn/item/60f54bf75132923bf81c8d40.jpg)

​	RocketMQMessageChannelBinder类根据规范完成RocketMQMessageHandler的创建和初始化，RocketMQMessageHandler是消息处理器MessageHandler的具体实现，RocketMQMessageHandler在RocketMQBinder中的作用是转化消息格式并发送消息。

![](https://pic.imgdb.cn/item/60f54c275132923bf81d7213.jpg)

![](https://pic.imgdb.cn/item/60f54c425132923bf81deaf0.jpg)

​	RocketMQMessageHandler中持有RocketMQTemplate对象，RocketMQTemplate是对RocketMQ客户端API的封装，Spring Boot中已经支持RocketMQTemplate，Spring Cloud Stream对其兼容。

​	DefaultMQProducer是由RocketMQ客户端提供的API，发送消息到RocketMQ消息服务器都是由它来完成的。

![](https://pic.imgdb.cn/item/60f54c7a5132923bf81ee363.jpg)

​	RocketMQMessageHandler是消息发送的处理逻辑，解析Message对象头中的参数，调用RocketMQTemplate中不同的发送消息接口。

![](https://pic.imgdb.cn/item/60f54ca65132923bf81fa159.jpg)

![](https://pic.imgdb.cn/item/60f54cb45132923bf81fee79.jpg)

​	发送普通消息、事务消息、定时消息还是顺序消息，由Message对象的消息头Header中的属性决定，在业务代码创建Message对象时设置。

### RocketMQ Binder集成消息订阅

​	AbstractMessageChannelBinder类中提供了创建MessageProducer的协议，在初始化Binder的时候会加载createConsumerEndpoint方法。

![](https://pic.imgdb.cn/item/60f54cd95132923bf8209704.jpg)

​	同样，由RocketMQMessageChannelBinder类根据协议完成RocketMQInboundChannelAdapter的创建和初始化。

​	![](https://pic.imgdb.cn/item/60f54d1a5132923bf821b5c7.jpg)

​	RocketMQInboundChannelAdapter是适配器，需要适配Spring Framework中的重试和回调机制，它在RocketMQ Binder中的作用是订阅消息并转化消息格式。RocketMQListenerBindingContainer是对RocketMQ客户端API的封装，适配器中持有它的对象。

![](https://pic.imgdb.cn/item/60f54d395132923bf8224adf.jpg)

![](https://pic.imgdb.cn/item/60f54d695132923bf823296f.jpg)

![](https://pic.imgdb.cn/item/60f54d9d5132923bf824089d.jpg)

​	RocketMQ提供了两种消费模式：顺序消费和并发消费。RocketMQ客户端API中顺序消费的默认监听器是DefaultMessageListenerOrderly类，并发消费的默认监听器是DefaultMessageListenerConcurrently类。无论哪种消费模式，监听器收到消息后都会回调RocketMQListener。

![](https://pic.imgdb.cn/item/60f54db55132923bf8247c9f.jpg)

​	RocketMQListener也是Spring Boot中已支持的RocketMQ组件，Spring Cloud Stream对其兼容。

​	在适配器RocketMQInboundChannelAdapter中创建和初始化RocketMQListener的实现类。

![](https://pic.imgdb.cn/item/60f54e715132923bf827c4d9.jpg)

​	DefaultMessageListenerOrderly对象在收到RocketMQ消息后，会先回调BindingRocketMQListener的onMessage方法，再调用RocketMQInboundChannelAdapter父类中的sendMessage方法将消息发送到DirectChannel。

### Spring Cloud Stream消息订阅流程

​	在Spring Cloud Stream中接收消息和发送消息的消息模型是一致的，Binder中接收到的消息先发送到MessageChannel，由订阅的MessageChannel通过Dispatcher转发到对应的MessageHandler进行处理。

​	Spring Cloud Stream消息接收流程如图9-4所示。

![](https://pic.imgdb.cn/item/60f54f085132923bf82a66fc.jpg)

​	RocketMQInboundChannelAdapter调用sendMessage（）发送消息。

![](https://pic.imgdb.cn/item/60f54f5f5132923bf82bddc6.jpg)

​	getOutputChannel（）得到的MessageChannel是在初始化RocketMQ Binder时传入的DirectChannel，对应例子中的Input通道。

​	MessagingTemplate继承了GenericMessagingTemplate类，实际执行了doSend（）方法发送消息。

​	![](https://pic.imgdb.cn/item/60f54f7a5132923bf82c5713.jpg)

​	由于MessageChannel的实例是DirectChannel对象，就复用了前面讲Spring Cloud Stream消息发送流程中提到的流程，通过消息分发类MessageDispatcher把消息分发给MessageHandler。

​	DirectChannel对应的消息处理器是StreamListenerMessageHandler，在消息处理器中回调使用了＠StreamListener注解的业务方法。

![](https://pic.imgdb.cn/item/60f54f915132923bf82cbca3.jpg)

​	InvocableHandlerMethod中持有BeanFactory、Method、MethodParameter等对象，使用Java反射机制完成回调。那么，StreamListenerMessageHandler是怎么和使用＠StreamListener注解的业务方法关联上的呢？

![](https://pic.imgdb.cn/item/60f54fa65132923bf82d15f8.jpg)\

​	在Spring容器管理的所有单例对象初始化完成之后，遍历StreamListenerHandlerMethodMapping，进行StreamListenerMessageHandler和InvocableHandlerMethod的创建和初始化。

​	从类名看显而易见，StreamListenerHandlerMethodMapping保存了StreamListener和HandlerMethod的映射关系。根据代码逐渐往上找，创建映射关系也是在StreamListenerAnnotationBeanPostProcessor类中完成的。

![](https://pic.imgdb.cn/item/60f54fc15132923bf82d8c0d.jpg)

​	StreamListenerAnnotationBeanPostProcessor找到所有使用＠StreamListener的Method，并创建StreamListenerHandlerMethodMapping对象，将映射关系保存到集合中。

![](https://pic.imgdb.cn/item/60f54fdb5132923bf82dfd11.jpg)

![](https://pic.imgdb.cn/item/60f54ff35132923bf82e6279.jpg)

看到MethodMapping，大家是否会想起Spring MVC中的HandlerMapping？Spring中许多模块的技术原理是相同的，在具体功能实现上会有一些差异。

到此，Spring Cloud Stream RocketMQ的相关知识介绍完了，其他内容不再展开，总结一下前面的内容：

* Spring Cloud Stream提供了简单易用的消息编程模型，内部基于发布/订阅模型实现。
* Spring Cloud Stream的Binder提供标准协议，不同的消息中间件都可以按照标准协议接入。
* Binder提供bindConsumer和bindProducer接口协议，分别用于构造生产者和消费者。



​	除了使用Spring Cloud Stream的消息模型来使用RocketMQ的消息功能，还可以使用Spring Boot中集成的RocketMQ组件，Spring Cloud Alibaba对其做了兼容，例如常见的RocketMQTemplate，相关资料在网上非常多，读者可自行查阅。

​	接下来笔者将为大家重点讲解RocketMQ的架构设计、RocketMQ中常见的功能和场景、在Spring Cloud Stream中如何使用RocketMQ，并深入讲解RocketMQ的技术原理。

## RocketMQ集群管理

​	在分布式服务SOA架构中，任何中间件或者应用都不允许单点存在，服务发现机制是必备的。服务实例有多个，且数量是动态变化的。注册中心会提供服务管理能力，服务调用方在注册中心获取服务提供者的信息，从而进行远程调用。

​	下面介绍RocketMQ的整体架构设计、集群管理，涉及RocketMQ中一些重要的概念。

### 整体架构设计

​	说到RocketMQ的架构设计，不得不说一下它与Kafka的渊源。Kafka是一款高性能的消息中间件，在大数据场景中经常使用，但由于Kafka不支持消费失败重试、定时消息、事务消息，顺序消息也有明显缺陷，难以支撑淘宝交易、订单、充值等复杂业务场景。淘宝中间件团队参考Kafka重新设计并用Java编写了RocketMQ，因此在RocketMQ中会有一些概念和Kafka相似。

​	常见的消息中间件Kafka、RabbitMQ、RocketMQ等都基于发布/订阅机制，消息发送者（Producer）把消息发送到消息服务器，消息消费者（Consumer）从消息服务器订阅感兴趣的消息。这个过程中消息发送者和消息消费者是客户端，消息服务器是服务端，客户端与服务端双方都需要通过注册中心感知对方的存在。

​	RocketMQ部署架构主要分为四部分，如图9-5所示。

* Producer：消息发布的角色，主要负责把消息发送到Broker，支持分布式集群方式部署。
* Consumer：消息消费者的角色，主要负责从Broker订阅消息消费，支持分布式集群方式部署。
* Broker：消息存储的角色，主要负责消息的存储、投递和查询，以及服务高可用保证，支持分布式集群方式部署。
* NameServer：服务管理的角色，主要负责管理Broker集群的路由信息，支持分布式集群方式部署。

![](https://pic.imgdb.cn/item/60f5536b5132923bf83d10af.jpg)

​	NameServer是一个非常简单的Topic路由注册中心，其角色类似于Dubbo中依赖的ZooKeeper，支持Broker的动态注册与发现。主要包括如下两个功能。

* 服务注册：NameServer接收Broker集群的注册信息，保存下来作为路由信息的基本数据，并提供心跳检测机制，检查Broker是否还存活。
* 路由信息管理：NameServer保存了Broker集群的路由信息，用于提供给客户端查询Broker的队列信息。Producer和Consumer通过NameServer可以知道Broker集群的路由信息，从而进行消息的投递和消费。

### 基本概念

* Message：消息，系统所传输信息的物理载体，生产和消费数据的最小单位。每条消息必须属于一个Topic，RocketMQ中每条消息拥有唯一的MessageID，且可以携带具有业务标识的Key。
* Topic：主题，表示一类消息的集合，每个主题都包含若干条消息，每条消息都只能属于一个主题，Topic是RocketMQ进行消息订阅的基本单位。
* Queue：消息队列，组成Topic的最小单元。默认情况下一个Topic会对应多个Queue，Topic是逻辑概念，Queue是物理存储，在Consumer消费Topic消息时底层实际则拉取Queue的消息。
* Tag：为消息设置的标志，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费的处理逻辑，实现更好的扩展性。
* UserProperties：用户自定义的属性集合，属于Message的一部分。
* ProducerGroup：同一类Producer的集合，这类Producer发送同一类消息且发送逻辑一致。如果发送的是事务消息且原始生产者在发送之后崩溃，则Broker服务器会联系同一生产者组的其他生产者实例以提交或回溯消费。
* ConsumerGroup：同一类Consumer的集合，这类Consumer通常消费同一类消息且消费逻辑一致。消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易。要注意的是，消费者组的消费者实例必须订阅完全相同的Topic。

### 为什么放弃ZooKeeper而选择NameServer

​	在Kafka中的服务注册与发现通常是用ZooKeeper来完成的，RocketMQ早期也使用了ZooKeeper做集群的管理，但后来放弃了转而使用自己开发的NameServer。说到这里，大家可能会有个疑问，这些能力ZooKeeper早就有了，为什么要重复“造轮子”自己再写一个服务注册中心呢？带着这个疑问我们先来看一下两者部署拓扑图的对比。

​	在Kafka中，Topic是逻辑概念，分区（Partition）是物理概念。1个Topic可以设置多个分区，每个分区可以设置多个副本（Replication），即有1个Master分区、多个Slave分区。

​	Kafka的部署拓扑图如图9-6所示。

![](https://pic.imgdb.cn/item/60f553cb5132923bf83ea08e.jpg)

​	例如，搭建3个Broker构成一个集群，创建一个Topic取名为TopicA，分区是3个，副本数是2个。在图9-6中part表示分区，M表示Master，S表示Slave。在Kafka中消息只能发送到Master分区中，消息发送给Topic时会发送到具体某个分区。如果发送给part0就只会发送到Broker0这个实例，再由Broker0同步到Broker1和Broker2中的part0副本中；如果发送给part1就只会发送到Broker1这个实例，再由Broker1同步到Broker0和Broker2中的part1副本中。

​	在RocketMQ中，Topic也是逻辑概念，队列（Queue）是物理概念（对应Kafka中的分区）。1个Topic可以设置多个队列，每个队列也可以有多个副本，即有1个Master队列、多个Slave队列。

​	RocketMQ的部署拓扑图如图9-7所示。

![](https://pic.imgdb.cn/item/60f56d6b5132923bf8982156.jpg)

​	为了方便对比，同样创建了一个Topic取名为TopicA，队列是3个，副本数也是2个，但构成Broker集群的实例有9个。

​	Kafka与RocketMQ两者在概念上相似，但又有明显的差异。

*  在Kafka中，Master和Slave在同一台Broker机器上，Broker机器上有多个分区，每个分区的Master/Slave身份是在运行过程中选举出来的。Broker机器具有双重身份，既有Master分区，也有Slave分区。
* 在RocketMQ中，Master和Slave不在同一台Broker机器上，每台Broker机器不是Master就是Slave，Broker的Master/Slave身份是在Broker的配置文件中预先定义好的，在Broker启动之前就已经决定了。



​	这个差异的影响在哪呢？Kafka的Master/Slave需要通过ZooKeeper选举出来，而RocketMQ不需要。问题就在这个选举上，ZooKeeper具备选举功能，选举机制的原理就是少数服从多数，那么ZooKeeper的选举机制必须由ZooKeeper集群中的多个实例共同完成。ZooKeeper集群中的多个实例必须相互通信，如果实例数很多，网络通信就会变得非常复杂且低效。

​	NameServer的设计目标是让网络通信变简单，从而使性能得到极大的提升。为了避免单点故障，NameServer也必须以集群的方式部署，但集群中各实例间相互不进行网络通信。NameServer是无状态的，可以任意部署多个实例。Broker向每一台NameServer注册自己的路由信息，因此每一个NameServer实例都保存一份完整的路由信息。NameServer与每台Broker机器保持长连接，间隔30s从路由注册表中将故障机器移除。NameServer为了降低实现的复杂度，并不会立刻通知客户端的Producer和Consumer。

​	集群环境下实例很多，偶尔会出现各种各样的问题，以下几种场景需要大家思考。

* 当某个NameServer因宕机或网络问题下线了，Broker如何同步路由信息？由于Broker会连接NameServer集群的每个实例，Broker仍然可以向其他NameServer同步其路由信息，Produce和Consumer仍然可以动态感知Broker的路由信息。
* NameServer如果检测到Broker宕机，并没有立即通知Producer和Consumer，Producer将消息发送到故障的Broker怎么办？Consumer从Broker订阅消息失败怎么办？



​	RocketMQ作者为了简化NameServer的设计，这两个问题都是在客户端中解决的，具体将在后续“高可用设计”的章节中解答。

* 由于NameServer集群中的实例相互不通信，在某个时间点不同NameServer实例保存的路由注册信息可能不一致。

​	这对发送消息和消费消息并不会有什么影响，原理和上一个问题是一样的，从这里能看出NameServer是CAP中的AP架构。

## 如何实现顺序消息

### 顺序消息的使用场景

​	日常中需要保证顺序的应用场景非常多，也就是顺序消息和使用场景非常多。例如交易系统中的订单创建、支付、退款等流程，先创建订单才能支付，支付完成的订单才能退款，这需要保证先进先出（First In First Out，缩写FIFO）。又例如数据库的BinLog消息，数据库执行新增语句、修改语句，BinLog消息的顺序也必须保证是新增消息、修改消息。

### 如何发送和消费顺序消息

我们使用RocketMQ顺序消息来模拟一下订单的场景，顺序消息分为两部分：顺序发送（发消息）、顺序消费（收消息）。

* 第1步，顺序发消息。

![](https://pic.imgdb.cn/item/60f56f285132923bf89da4ff.jpg)

​	为了简化代码，模拟按顺序依次发送创建、支付、退款消息到TopicTest。

​	这里发送顺序消息的代码，相比9.1.3节中发送普通消息的代码，修改了两处地方：

* 在spring.propeties配置文件中指定producer.sync=true，默认是异步发送，此处改为同步发送。
*  MessageBuilder设置Header信息头，表示这是一条顺序消息，将消息固定地发送到第0个消息队列。



* 第2步，顺序收消息。

![](https://pic.imgdb.cn/item/60f56f655132923bf89e6848.jpg)

​	相比9.1.4节中的消费普通消息，仅修改了spring.properties配置文件consumer.orderly=true，默认是并发消费，此处改成顺序消费。

​	程序运行之后查看控制台的日志输出，也是按顺序打印出来的。

![](https://pic.imgdb.cn/item/60f56f985132923bf89f0679.jpg)

### 顺序发送的技术原理

​	RocketMQ的顺序消息分2种情况：局部有序和全局有序。前面的例子是局部有序场景。

* 局部有序：指发送同一个队列的消息有序，可以在发送消息时指定队列，在消费消息时也按顺序消费。例如同一个订单ID的消息要保证有序，不同订单ID的消息没有约束，相互不影响，不同订单ID之间的消息是并行的。
* 全局有序：设置Topic只有1个队列可以实现全局有序，创建Topic时手动设置。此类场景极少，性能差，通常不推荐使用。



​	RocketMQ中消息发送有三种方式：同步、异步、单向。

* 同步：发送网络请求后会同步等待Broker服务器的返回结果，支持发送失败重试，适用于较重要的消息通知场景。
* 异步：异步发送网络请求，不会阻塞当前线程，不支持失败重试，适用于对响应时间要求更高的场景。
* 单向：单向发送原理和异步一致，但不支持回调。适用于响应时间非常短、对可靠性要求不高的场景，例如日志收集。



​	顺序消息发送的原理很简单，同一类消息发送到相同的队列即可。为了保证先发送的消息先存储到消息队列，必须使用同步发送的方式，否则可能出现先发的消息后到消息队列，此时消息就乱序了。

​	RocketMQ核心源码如下。

![](https://pic.imgdb.cn/item/60f56ff15132923bf8a01e79.jpg)

​	选择队列的过程由messageQueueSelector和hashKey在实现类SelectMessageQueueByHash中完成。

![](https://pic.imgdb.cn/item/60f570065132923bf8a06109.jpg)

* 根据hashKey计算hash值，hashKey是我们前面例子中的订单ID，因此相同订单ID的hash值相同。
* 用hash值和队列数mqs.size（）取模，得到一个索引值，结果小于队列数。
* 根据索引值从队列列表中取出一个队列mqs.get（value），hash值相同则队列相同。



​	在队列列表的获取过程中，由Producer从NameServer根据Topic查询Broker列表，缓存在本地内存中，以便下次从缓存中读取。

### 普通发送的技术原理

​	RocketMQ中除顺序消息外，还支持事务消息和延迟消息，非这三种特殊的消息我们称为普通消息以便区分。日常开发过程中最常用的是普通消息，这是因为最常用的场景是系统间的异步解耦和流量的削峰填谷，这些场景下尽量保证消息高性能收发即可。

​	从普通消息与顺序消息的对比来看，普通消息在发送时选择消息队列的策略不同。普通消息发送选择队列有两种机制：轮询机制和故障规避机制（也称故障延迟机制）。默认使用轮询机制，一个Topic有多个队列，轮询选择其中一个队列。

​	轮询机制的原理是路由信息TopicPublishInfo中维护了一个计数器sendWhichQueue，每发送一次消息需要查询一次路由，计算器就进行“+1”，通过计数器的值index与队列的数量取模计算来实现轮询算法。

![](https://pic.imgdb.cn/item/60f570b15132923bf8a27d13.jpg)

![](https://pic.imgdb.cn/item/60f570cd5132923bf8a2d683.jpg)

### 顺序消费的技术原理

​	RocketMQ支持两种消息模式：集群消费（Clustering）和广播消费（Broadcasting）。两者的区别是，在广播消费模式下每条消息会被ConsumerGroup的每个Consumer消费，在集群消费模式下每条消息只会被ConsumerGroup的一个Consumer消费。

​	多数场景都使用集群消费，消息每消费一次代表一次业务处理，集群消费表示每条消息由业务应用集群中任意一个服务实例来处理。少数场景使用广播消费，例如数据发生变化，更新业务应用集群中每个服务的本地缓存，这就需要一条消息整个集群都消费一次，默认是集群消费，消息模式是前提条件，我们下面也仅分析这种模式下的情况。

​	顺序消费也称为有序消费，原理是同一个消息队列只允许Consumer中的一个消费线程拉取消费。Consumer中有个消费线程池，多个线程会同时消费消息。在顺序消费的场景下消费线程请求到Broker时会先申请独占锁，获得锁的请求则允许消费。

![](https://pic.imgdb.cn/item/60f571135132923bf8a3ab77.jpg)

![](https://pic.imgdb.cn/item/60f571215132923bf8a3d80d.jpg)

​	消息消费成功后，会向Broker提交消费进度，更新消费位点信息，避免下次拉取到已消费的消息。顺序消费中如果消费线程在监听器中进行业务处理时抛出异常，则不会提交消费进度，消费进度会阻塞在当前这条消息，并不会继续消费该队列中后续的消息，从而保证顺序消费。在顺序消费的场景下，特别需要注意对异常的处理，如果重试也失败，会一直阻塞在当前消息，直到超出最大重试次数，从而在很长一段时间内无法消费后续消息造成队列消息堆积。

### 并发消费的技术原理

​	RocketMQ支持两种消费方式：顺序消费和并发消费。并发消费是默认的消费方式，日常开发过程中最常用的方式，除了顺序消费就是并发消费。

​	并发消费也称为乱序消费，其原理是同一个消息队列提供给Consumer中的多个消费线程拉取消费。Consumer中会维护一个消费线程池，多个消费线程可以并发去同一个消息队列中拉取消息进行消费。如果某个消费线程在监听器中进行业务处理时抛出异常，当前消费线程拉取的消息会进行重试，不影响其他消费线程和消息队列的消费进度，消费成功的线程正常提交消费进度。并发消费相比顺序消费没有资源争抢上锁的过程，消费消息的速度比顺序消费要快很多。

### 消息的幂等性

​	说到消息消费不得不提消息的幂等性，业务代码中通常收到一条消息进行一次业务逻辑处理，如果一条相同消息被重复收到几次，是否会导致业务重复处理？Consumer能否不重复接收消息？

​	RocketMQ不保证消息不被重复消费，如果业务对消费重复非常敏感，必须要在业务层面进行幂等性处理，具体实现可以通过分布式锁来完成。

​	在所有消息系统中消费消息有三种模式：at-most-once（最多一次）、at-least-once（最少一次）和exactly-only-once（精确仅一次），分布式消息系统都是在三者间取平衡，前两者是可行的并且被广泛使用的。

* at-most-once：消息投递后不论消费是否成功，不会再重复投递，有可能会导致消息未被消费，RocketMQ未使用该方式。
* at-least-once：消息投递后，消费完成后，向服务器返回ACK（消费确认机制），没有消费则一定不会返回ACK消息。由于网络异常、客户端重启等原因，服务器未能收到客户端返回的ACK，服务器则会再次投递，这就会导致可能重复消费，RocketMQ通过ACK来确保消息至少被消费一次。
* exactly-only-once：必须下面两个条件都满足，才能认为消息是“Exactly Only Once”。①发送消息阶段，不允许发送重复的消息；②消费消息阶段，不允许消费重复的消息。在分布式系统环境下，如果要实现该模式，巨大的开销不可避免。RocketMQ为了追求高性能，并不保证此特性，无法避免消息重复，由业务上进行幂等性处理。

## 如何实现事务消息

### 事务消息的使用场景

​	事务消息的使用场景很多，例如，在电商系统中用户下单后新增了订单记录，对应的商品库存需要减少，怎么保证新增订单后商品库存减少呢？又例如红包业务，张三给李四发红包，张三的账户余额需要扣减，李四的账户余额需要增加，怎么保证张三账户扣钱后李四账户加钱呢？

​	此类问题都是事务问题，可以简单理解为：一个表的数据更新后，如何保证另外一个表的数据也更新成功。如果使用同一个数据库实例，那么问题很简单，可以使用本地事务来解决，Spring的＠Transactional注解就支持。

​	但实际场景并不这么简单，互联网应用的流量大，系统规模通常也比较大，会存在许多数据库实例、分库分表等。我们需要修改的表往往不在同一个数据库实例或同一个数据库中，此时就不能使用本地事务来解决了，需要用到分布式事务。RocketMQ的一大特点就是支持事务消息，支持一些分布式事务场景，下面我们看RocketMQ事务消息的具体用法。

### 如何发送事务消息

我们使用RocketMQ事务消息来模拟下单减库存的场景，为了简化代码，部分非核心的实现仅用注释说明。

* 第1步，发送订单的事务消息，预提交。

![](https://pic.imgdb.cn/item/60f572535132923bf8a7a27d.jpg)

​	Order对象保存了订单信息，随机生成一个ID作为消息的事务ID，定义了一个名为OrderTransactionGroup的事务组，用于下一步接收本地事务的监听。

​	此时消息已经发送到Broker中，但还未投递出去，Consumer暂时还不能消费这条消息。

*  第2步，执行订单信息入库的事务操作，提交或回滚事务消息。

![](https://pic.imgdb.cn/item/60f5728c5132923bf8a8601b.jpg)

![](https://pic.imgdb.cn/item/60f572d85132923bf8a9536e.jpg)

​	实现RocketMQLocalTransactionListener接口，使用＠RocketMQTransactionListener注解用于接收本地事务的监听，txProducerGroup是事务组名称，和前面定义的OrderTransactionGroup保持一致。RocketMQLocalTransactionListener接口有两个实现方法。

* executeLocalTransaction：执行本地事务，在第1步中消息发送成功会回调执行，一旦事务提交成功，下游应用的Consumer能收到该消息，在这里demo的本地事务就是保存订单信息入库。
* checkLocalTransaction：检查本地事务执行状态，如果executeLocalTransaction方法中返回的状态是未知UNKNOWN或者未返回状态，默认会在预处理发送的1分钟后由Broker通知Producer检查本地事务，在Producer中回调本地事务监听器中的checkLocalTransaction方法。检查本地事务时，可以根据事务ID查询本地事务的状态，再返回具体事务状态给Broker。



* 第3步，消费订单消息。

![](https://pic.imgdb.cn/item/60f573215132923bf8aa4574.jpg)

​	消费事务消息与消费普通消息的代码是一样的，无须做任何修改。

### 事务消息的技术原理

​	RocketMQ采用了2PC的方案来提交事务消息。第一阶段Producer向Broker发送预处理消息（也称半消息），此时消息还未被投递出去，Consumer不能消费；第二阶段Producer向Broker发送提交或回滚消息。具体流程如下：

* 发送预处理消息成功后，开始执行本地事务。
* 如果本地事务执行成功，发送提交请求提交事务消息，消息会投递给Consumer，如图9-8所示。

![](https://pic.imgdb.cn/item/60f5739b5132923bf8abe09f.jpg)

* 如果本地事务执行失败，发送回滚请求回滚事务消息，消息不会投递给Consumer，如图9-9所示。
* 如果本地事务状态未知，网络故障或Producer宕机，Broker未收到二次确认的消息。由Broker端发送请求给Producer进行消息回查，确认提交或回滚。如果消息状态一直未被确认，需要人工介入处理，如图9-10所示。

![](https://pic.imgdb.cn/item/60f573b85132923bf8ac4745.jpg)

![](https://pic.imgdb.cn/item/60f573ce5132923bf8ac8f57.jpg)

## 高性能设计

​	经过阿里巴巴多年双11验证，RocketMQ在稳定的基础上一直保持着非常高的性能，这是诸多企业在消息中间件方面选择使用RocketMQ的重要原因。RocketMQ的高性能设计体现在三个方面：数据存储设计、动态伸缩的能力、消息实时投递。数据存储设计包括顺序写盘、消费队列设计、消息跳跃读、数据零拷贝。动态伸缩的能力包括消息队列扩容、Broker集群扩容。

​	RocketMQ以高吞吐量著称，这主要得益于其数据存储方式的设计。数据存储的核心由两部分组成：CommitLog数据存储文件和ConsumeQueue消费队列文件。Producer将消息发送到Broker服务器，Broker会把所有消息都存储在CommitLog文件中，再由CommitLog转发到ConsumeQueue文件提供给各个Consumer消费。、

​	RocketMQ存储设计如图9-11所示。

![](https://pic.imgdb.cn/item/60f574405132923bf8ae1614.jpg)

### 顺序写盘

​	顺序写盘指写磁盘上的文件采用顺序写的方式，在解释为什么要顺序写盘之前，我们先简单了解一下磁盘读写的过程。一次磁盘请求（读或写）完成过程由三个动作组成：寻道、旋转延迟、数据传输。

* 寻道：磁头移动定位到指定磁道，时间很长，是指找到数据在哪个地方。
* 旋转延迟：等待指定扇区旋转至磁头下，机械硬盘和每分钟多少转有关系，时间很短。
* 数据传输：数据通过系统总线从磁盘传送到内存，时间很短。



​	磁盘读写最慢的动作是寻道，缩短寻道时间就能有效提升磁盘的读写速度，最优的方式就是不用寻道。随机写会导致磁头不停地更换磁道，时间都花在寻道上了，顺序写几乎不用换磁道，或者寻道时间很短。

​	CommitLog文件是负责存储消息数据的文件，所有Topic的消息都会先存在{ROCKETMQ_HOME}/store/commitlog文件夹下的文件中，消息数据写入CommitLog文件是加锁串行追加写入。

​	RocketMQ为了保证消息发送的高吞吐量，使用单个文件存储所有Topic的消息，从而保证消息存储是完全的磁盘顺序写，但这样给文件读取（消费消息）带来了困难。

​	当消息到达CommitLog文件后，会通过线程异步几乎实时地将消息转发给消费队列文件。每个CommitLog文件的默认大小是1GB，写满1GB再写新的文件，大量数据I/O都在顺序写同一个CommitLog文件。文件名按照该文件起始的总的字节偏移量offset命名，文件名固定长度为20位，不足20位前面补0，如图9-12所示。

* 第一个文件起始偏移量是0，即文件名是00000000000000000000。
* 第二个文件起始偏移量是1024×1024×1024=1073741824（1GB=1073741824 B），即文件名是0000000001073741824。

![](https://pic.imgdb.cn/item/60f5752c5132923bf8b1450c.jpg)

文件名这样设计的目的是在消费消息时能够根据偏移量offset快速定位到消息存储在某个CommitLog文件，从而加快消息的检索速度。

* 消息数据文件中每条消息数据的具体格式如表9-1所示。



![](https://pic.imgdb.cn/item/60f5754b5132923bf8b1b33d.jpg)

### 消费队列设计

​	消费Broker中存储消息的实际工作就是读取文件，但消息数据文件中所有Topic的消息数据是混合在一起的，消费消息时是区分Topic消费的，这就导致如果消费时也读取CommitLog文件会使得消费消息的性能差、吞吐量低。为了解决消息数据文件顺序写难以读取的问题，RocketMQ中设计了消费队列ConsumeQueue。

​	ConsumeQueue负责存储消费队列文件，在消息写入CommitLog文件时，会异步转发到ConsumeQueue文件，然后提供给Consumer消费。ConsumeQueue文件中并不存储具体的消息数据，只存CommitLog的偏移量offset、消息大小size、消息Tag Hashcode，如表9-2所示。

![](https://pic.imgdb.cn/item/60f575955132923bf8b2b44d.jpg)

​	每个Topic在某个Broker下对应多个队列Queue，默认是4个消费队列Queue。每一条记录的大小是20B，默认一个文件存储30万个记录，文件名同样也按照字节偏移量offset命名，文件名固定长度为20位，不足20位前面补0。

* 第一个文件起始偏移量是0，文件名是00000000000000000000，与CommitLog文件一致。
* 第二个文件起始偏移量是20×30w=6000000，第二个文件名是00000000000006000000。



​	在集群模式下，Broker会记录客户端对每个消费队列的消费偏移量，定位到ConsumeQueue里相应的记录，并通过CommitLog的Offset定位到CommitLog文件里的该条消息，如图9-13所示。

![](https://pic.imgdb.cn/item/60f575fd5132923bf8b41bb2.jpg)

### 消息跳跃读取

​	消费Broker中存储消息的实际工作就是读取文件，消息队列文件是一种数据结构上的设计。前面讲了磁盘顺序读写和消息队列文件的设计，为了高性能读数据，除此之外，还使用了操作系统中的Page Cache机制。RocketMQ读取消息依赖操作系统的PageCache，PageCache命中率越高则读性能越高，操作系统会尽量预读数据，使应用直接访问磁盘的概率降低。消息队列文件的读取流程如下。

* 检查要读的数据是否在上次预读的Cache中。
* 如果没有命中Cache，操作系统从磁盘中读取对应的数据页，并将该数据页之后的连续几页一起读入Cache中，再将应用需要的数据返回给应用，这种方式称为跳跃读取。
* 如果命中Cache，上次缓存的数据有效，操作系统认为在顺序读盘，则继续扩大缓存的数据范围，将之前缓存的数据页之后的几页数据再读取到Cache中。



​	在计算机系统中，CPU、RAM、DISK的速度不相同，按速度高低排列为：CPU>RAM>DISK。CPU与RAM之间、RAM与DISK之间的速度和容量差异是指数级的。为了在速度和容量上折中，在CPU与RAM之间使用CPU Cache以提高访存速度，在RAM与磁盘之间，操作系统使用Page Cache提高系统对文件的访问速度。

### 数据零拷贝

​	在网络通信过程中，通常情况下对文件的读写要多经历一次数据拷贝，例如写文件数据要从用户态拷贝到内核态，再由内核态写入物理文件。所谓零拷贝，指的是用户态与内核态之间不存在拷贝。

​	RocketMQ中的文件读写主要通过Java NIO中的MappedByteBuffer来进行文件映射。利用了Java NIO中的FileChannel模型，可以直接将物理文件映射到缓冲区的PageCache，少了一次数据拷贝过程，提高了读写速度。

### 动态伸缩能力

​	着业务的增长，线上流量会出现快速增长，经常出现的情况是已有的服务器集群能力不足以支撑现有的流量，此时就需要增加服务器（扩容）。例如大促、秒杀等活动，流量上涨持续一段时间后又会回归到正常情况，为了避免服务器资源（成本）浪费，此时就需要减少服务器（缩容），这些场景会用到RocketMQ中的动态伸缩能力。

​	动态伸缩（水平扩容）能力是分布式应用很重要的能力，RocketMQ中的动态伸缩能力主要体现在消息队列扩容和集群扩容两个方面，需要根据实际场景进行选择。

* 消息队列扩容/缩容：一个Consumer实例可以同时消费多个消息队列中的消息。如果一个Topic的消息量特别大，但Broker集群水位压力还是很低，就可以对该Topic的消息队列进行扩容，Topic的消息队列数跟消费速度成正比。消息队列数在创建Topic时可以指定，也可以在运行中修改。相反，如果一个Topic的消息量特别小，但该Topic的消息队列数很多，则可以对该Topic消息队列缩容。
* Broker集群扩容/缩容：同样如果一个Topic的消息量特别大，但Broker集群水位很高，此时就需要对Broker机器扩容。扩容方式很简单，直接加机器部署Broker即可。新的Broker启动后会向NameServer注册，Producer和Consumer通过NameServer发现新Broker并更新路由信息。相反，如果Broker集群水位很低，则可以适当减少Broker服务器来节约成本。

### 消息实时投递

​	消息的高性能还体现在消息发送到存储之后能否立即被客户端消费，这涉及消息的实时投递，Consumer消费消息的实时性与获取消息的方式有很大关系。

​	任何一款消息中间件都会有两种获取消息的方式：Push推模式和Pull拉模式。这两种模式各有优缺点，并适用于不同的场景。

* Push推模式：当消息发送到服务端时，由服务端主动推送给客户端Consumer。优点是客户端Consumer能实时地接收到新的消息数据。也有两个缺点。缺点1是如果Consumer消费一条消息耗时很长，消费推送速度大于消费速度时，Consumer消费不过来会出现缓冲区溢出；缺点2则是一个Topic往往会对应多个ConsumerGroup，服务端一条消息会产生多次推送，可能会对服务端造成压力。
* Pull拉模式：由客户端Consumer主动发送请求，每间隔一段时间轮询去服务端拉取消息。优点是Consumer可以根据当前消费速度选择合适的时机触发拉取。缺点则是拉取的间隔时间不好控制。间隔时间如果很长，会导致消息消费不及时，服务端容易积压消息；间隔时间如果很短，服务端收到的消息少，会导致Consumer可能多数拉取请求都是无效的（拿不到消息），从而浪费网络资源和服务端资源。



​	这两种获取消息方式的缺点都很明显，单一的方式难以应对复杂的消费场景，所以RocketMQ中提供了一种推/拉结合的长轮询机制来平衡推/拉模式各自的缺点。

​	长轮询本质上是对普通pull模式的优化，即还是以客户端Consumer轮询的方式主动发送拉取请求到服务端Broker，Broker如果检测到有新的消息就立即返回Consumer，但如果没有新消息则暂时不返回任何信息，挂起当前请求缓存到本地，Broker后台有个线程去检查挂起请求，等到新消息产生时再返回Consumer。平常使用的DefaultMQPushConsumer的实现就是推、拉结合的，既能解决资源浪费问题，也能解决消费不及时问题。

## 高可用设计

​	计算机系统的可用性用平均无故障时间来度量，系统的可用性越高，则平均无故障时间越长。高可用性也是分布式中间件的重要特性，RocketMQ的高可用设计主要有四个方面的体现。

* 消息发送的高可用：在消息发送时可能会遇到网络问题、Broker宕机等情况，而NameServer检测Broker是有延迟的，虽然NameServer每间隔10秒会扫描所有Broker信息，但要Broker的最后心跳时间超过120秒以上才认为该Broker不可用，所以Producer不能及时感知Broker下线。如果在这期间消息一直发送失败，那么消息发送失败率会很高，这在业务上是无法接受的。这里大家可能会有一个疑问，为什么NameServer不及时检查Broker和通知Producer？这是因为那样做会使网络通信和架构设计变得非常复杂，而NameServer的设计初衷就是尽可能简单，所以这块的高可用方案在Producer中来实现。RocketMQ采用了一些发送端的高可用方案，来解决发送失败的问题，其中最重要的两个设计是重试机制与故障延迟机制。
* 消息存储的高可用：在RocketMQ中消息存储的高可用体现在发送成功的消息不能丢、Broker不能发生单点故障，出现Broker异常宕机、操作系统Crash、机房断电或断网等情况保证数据不丢。RocketMQ主要通过消息持久化（也称刷盘）、主从复制、读写分离机制来保证。
* 消息消费的高可用：实际业务场景中无法避免消费消息失败的情况，可能由于网络原因导致，也可能由于业务逻辑错误导致。但无论发生任何情况，即使消息一直消费失败，也不能丢失消息数据。RocketMQ主要通过消费重试机制和消息ACK机制来保证。
* 集群管理的高可用：集群管理的高可用主要体现在NameServer的设计上，当部分NameServer节点宕机时不会有什么糟糕的影响，只剩一个NameServer节点RocketMQ集群也能正常运行，即使NameServer全部宕机，也不影响已经运行的Broker、Producer和Consumer。

### 消息发送重试机制

​	重试机制比较简单，在消息发送出现异常时会尝试再次发送，默认最多重试三次。重试机制仅支持同步发送方式，不支持异步和单向发送方式。根据发送失败的异常类型处理策略略有不同，如果是网络异常RemotingException和客户端异常MQClientException会重试，而Broker服务端异常MQBrokerException和线程中断异常InterruptedException则不会再重试，且抛出异常。

![](https://pic.imgdb.cn/item/60f577845132923bf8b98c18.jpg)

### 故障规避机制

​	在介绍NameServer时提到，NameServer为了简化和客户端通信，发现Broker故障时并不会立即通知客户端。故障规避机制用来解决当Broker出现故障，Producer不能及时感知而导致消息发送失败的问题。默认是不开启的，如果在开启的情况下，消息发送失败的时候会将失败的Broker暂时排除在队列选择列表外。规避时间是衰减的，如果Broker一直不可用，会被NameServer检测到并在Producer更新路由信息时进行剔除。

​	在选择查找路由时，选择消息队列的关键步骤如下。

* 先按轮询算法选择一个消息队列。
* 从故障列表判断该消息队列是否可用。

![](https://pic.imgdb.cn/item/60f577b95132923bf8ba4cdd.jpg)

![](https://pic.imgdb.cn/item/60f577cf5132923bf8baa0a9.jpg)

​	判断消息队列是否可用有两个步骤。

* 判断其是否在故障列表中，不在故障列表中代表可用。
* 在故障列表faultItemTable中还需判断当前时间是否大于等于故障规避的开始时间startTimestamp，使用这个时间判断是因为通常故障时间是有限制的，Broker宕机之后会有相关运维去恢复。

![](https://pic.imgdb.cn/item/60f577f75132923bf8bb4728.jpg)

​	这部分重点在于故障机器FaultItem在什么场景下进入故障列表faultItemTable中，相信大家应该也能猜到，消息发送失败时就可能是机器故障了。回顾一下前面重试机制中的消息发送代码，可以看到两种情况会调用updateFaultItem（），消息发送结束后和发送出现异常时。

​	只有在开启故障规避机制时才会更新故障机器信息，根据isolation计算故障周期时长，故障时长duration的单位是毫秒。

![](https://pic.imgdb.cn/item/60f578145132923bf8bbb27f.jpg)

​	currentLatency代表响应时间，computeNotAvailableDuration（）根据响应时间来计算故障周期时长，响应时间越长则故障周期也越长。网络异常、Broker异常、客户端异常都是固定响应时长30秒，所以它们的故障周期时长为10分钟。而消息发送成功，或线程中断异常响应时长在100毫秒内，则故障周期时长为0。

![](https://pic.imgdb.cn/item/60f5783d5132923bf8bc44a0.jpg)

​	FaultItem存储了Broker名称、响应时长、故障规避开始时间，最重要的是故障规避开始时间，会用来判断Queue是否可用。故障周期时长为0就代表了没有故障。当响应时长超过100毫秒时代表Broker可能机器出现问题，也会进入故障列表，网络异常、Broker异常、客户端异常则设定故障周期为10分钟。

### 同步刷盘与异步刷盘

​	刷盘是指消息数据发送到Broker之后，写入磁盘中做持久化，保障在Broker出现故障重启时数据不会丢失。RocketMQ提供了两种刷盘机制：同步刷盘和异步刷盘，在性能和使用场景上有有明显区别。

**同步刷盘**

​	在同步刷盘的模式下，当消息写到内存后，会等待数据写到磁盘的CommitLog文件。具体实现源码在CommitLog＃handleDiskFlush中，GroupCommitRequest是刷盘任务，提交刷盘任务后，会在刷盘队列中等待刷盘，而刷盘线程GroupCommitService每间隔10毫秒写一批数据到磁盘。为什么不直接写呢？主要原因是磁盘I/O压力大、写入性能低，每间隔10毫秒写一次可以提升磁盘I/O效率和写入性能。

![](https://pic.imgdb.cn/item/60f578785132923bf8bd1c66.jpg)

![](https://pic.imgdb.cn/item/60f578885132923bf8bd57c2.jpg)

​	GroupCommitService是刷盘线程，内部有两个刷盘任务列表，在执行service.putRequest（request）时仅提交刷盘任务到任务列表，request.waitForFlush会同步等待GroupCommitService将任务列表中的任务刷盘完成。

![](https://pic.imgdb.cn/item/60f578c95132923bf8be3d83.jpg)

​	这里有两个队列读写分离，requestsWrite是写队列，用于保存添加进来的刷盘任务，requestsRead是读队列，在刷盘之前会把写队列的数据放到读队列。

![](https://pic.imgdb.cn/item/60f578e15132923bf8be94e7.jpg)

​	刷盘的时候依次读取requestsRead中的数据写入磁盘，写入完成后清空requestsRead。读写分离设计的目的是在刷盘时不影响任务提交到列表。

![](https://pic.imgdb.cn/item/60f578f75132923bf8bedf7e.jpg)

​	mappedFileQueue.flush（0）是刷盘操作，通过MappedFile映射的CommitLog文件写入磁盘。

![](https://pic.imgdb.cn/item/60f5790c5132923bf8bf2688.jpg)

**异步刷盘**

​	RocketMQ默认采用异步刷盘，异步刷盘又有两种策略：开启缓冲池和不开启缓冲池。

![](https://pic.imgdb.cn/item/60f5792a5132923bf8bf980a.jpg)

​	不开启缓冲池：默认不开启，刷盘线程FlushRealTimeService会每间隔500毫秒尝试去刷盘。这间隔500毫秒仅仅是尝试，实际去刷盘还得满足一些前提条件，即距离上次刷盘时间超过10秒，或者写入内存的数据超过4页（16KB），这样即使服务器宕机，丢失的数据也是在10秒内的或大小在16KB以内的。

![](https://pic.imgdb.cn/item/60f579445132923bf8bff59c.jpg)

![](https://pic.imgdb.cn/item/60f579535132923bf8c02b99.jpg)

​	开启缓冲池：RocketMQ会申请一块和CommitLog文件相同大小的堆外内存用来做缓冲池，数据会先写入缓冲池，提交线程CommitRealTimeService也每间隔500毫秒尝试提交到文件通道等待刷盘，刷盘最终还由FlushRealTimeService来完成，和不开启缓冲池的处理一致。使用缓冲池的目的是多条消息合并写入，从而提高I/O性能。

![](https://pic.imgdb.cn/item/60f579715132923bf8c097c0.jpg)

![](https://pic.imgdb.cn/item/60f579835132923bf8c0d7b6.jpg)

![](https://pic.imgdb.cn/item/60f579905132923bf8c10c84.jpg)

### 主从复制

​	RocketMQ为了提高消息消费的高可用性，避免Broker发生单点故障引起存储在Broker上的消息无法及时消费，同时避免单个机器上硬盘坏损出现消息数据丢失。RocketMQ采用Broker数据主从复制机制，当消息发送到Master服务器后会将消息同步到Slave服务器，如果Master服务器宕机，消息消费者还可以继续从Slave拉取消息。

​	消息从Master服务器复制到Slave服务器上，有两种复制方式：同步复制SYNC_MASTER和异步复制ASYNC_MASTER。通过配置文件${ROCKETMQ_HOME}/conf/broker.conf里的brokerRole参数进行设置。

* 同步复制：Master服务器和Slave服务器都写成功后才返回给客户端写成功的状态。优点是如果Master服务器出现故障，Slave服务器上有全部数据的备份，很容易恢复到Master服务器。缺点是由于多了一个同步等待的步骤，会增加数据写入延迟，并且降低系统的吞吐量。
*  异步复制：仅Master服务器写成功即可返回给客户端写成功的状态。优点刚好是同步复制的缺点，由于没有那一次同步等待的步骤，服务器的延迟较低且吞吐量较高。缺点显而易见，如果Master服务器出现故障，有些数据因为没有被写入Slave服务器，未同步的数据有可能会丢失。



​	在实际应用中需要结合业务场景，合理设置刷盘方式和主从复制方式。不建议使用SYNC_FLUSH同步它刷盘方式，因为它会频繁地触发写磁盘操作，性能下降很明显。高性能是RocketMQ的一个明显特点，因此放弃性能是不合适的选择。通常可以把Master和Slave设置成ASYNC_FLUSH异步刷盘、SYNC_MASTER同步复制，这样即使有一台服务器出故障，仍然可以保证数据不丢失。

### 读写分离

​	读写分离机制也是高性能、高可用架构中常见的设计。例如MySQL也实现了读写分离机制，Client只能从Master服务器写数据，可以从Master服务器和Slave服务器都读数据。RocketMQ的设计也是如此，但在实现方式上又有一些区别。

​	RocketMQ的Consumer在拉取消息时，Broker会判断Master服务器的消息堆积量以决定Consumer是否从Slave服务器拉取消息消费。默认一开始从Master服务器上拉取消息，如果Master服务器的消息堆积超过了物理内存的40%，则会在返回给Consumer的消息结果里告知Consumer，下次需要从其他Slave服务器上拉取消息。

### 消费重试机制

​	实际业务场景中无法避免消费消息失败的情况，消费失败可能是因为业务处理中调用远程服务网络问题失败，不代表消息一定不能被消费，通过重试可以解决。在介绍RocketMQ的消费重试机制之前，先介绍一下“重试队列”和“死信队列”。

*  重试队列：在Consumer由于业务异常导致消费消息失败时，将消费失败的消息重新发送给Broker保存在重试队列，这样设计的原因是不能影响整体消费进度又必须防止消费失败的消息丢失。重试队列的消息存在一个单独的Topic中，不在原消息的Topic中，Consumer自动订阅该Topic。重试队列的Topic名称格式为“%RETRY%+consumerGroup”，每个业务Topic都会有多个ConsumerGroup，每个ConsumerGroup消费失败的情况都不一样，因此各对应一个重试队列的Topic。
*  死信队列：由于业务逻辑Bug等原因，导致Consumer对部分消息长时间消费重试一直失败，为了保证这部分消息不丢失，同时不能阻塞其他能重试消费成功的消息，超过最大重试消费次数之后的消息会进入死信队列。消息进入死信队列之后就不再自动消费，需要人工干预处理。死信队列也存在一个单独的Topic中，名称格式为“%DLQ%+consumerGroup”，原理和重试队列一致。



​	通常故障恢复需要一定的时间，如果不间断地重试，重试又失败的情况会占用并浪费资源，所以RocketMQ的消费重试机制采用时间衰减的方式，使用了自身定时消费的能力。首次在10秒后重试消费，如果消费成功则不再重试，如果消费失败则继续重试消费，第二次在30秒后重试消费，依此类推，每次重试的间隔时间都会加长，直到超出最大重试次数（默认为16次），则进入死信队列不再重试。重试消费过程中的间隔时间使用了定时消息，重试的消息数据并非直接写入重试队列，而是先写入定时消息队列，再通过定时消息的功能转发到重试队列。

​	RocketMQ支持定时消息（也称延迟消息），延迟消息是指消息发送之后，等待指定的延迟时间后再进行消费。除了支持消费重试机制，延迟消息也适用于一些处理异步任务的场景。例如调用某个服务，调用结果需要异步在1分钟内返回，此时就可以发送一个延迟消息，延迟时间为1分钟，等1分钟后收到该消息去查询上次的调用结果是否返回。

​	RocketMQ不支持任意时间精确的延迟消息，仅支持1s、5s、10s、30s、1min、2min、3min、4min、5min、6min、7min、8min、9min、10min、20min、30min、1h、2h。

### ACK机制

​	在实际业务场景中，业务应用在消费消息的过程中偶尔会出现一些异常情况，例如程序发布导致的重启，或网络突然出现问题，此时正在进行业务处理的消息可能消费完了，也可能业务逻辑执行到一半没有消费完，那么如何去识别这些情况呢？这就需要消息的ACK机制。

​	广播模式的消费进度保存在客户端本地，集群模式的消费进度保存在Broker上。集群模式中RocketMQ中采用ACK机制确保消息一定被消费。在消息投递过程中，不是消息从Broker发送到Consumer就算消费成功了，需要Consumer明确给Broker返回消费成功状态才算。如果从Broker发送到Consumer后，已经完成了业务处理，但在给Broker返回消费成功状态之前，Consumer发生宕机或断电、断网等情况，Broker未收到反馈，则不会保存消费进度。Consumer重启之后，消息会重新投递，此时也会出现重复消费的场景，前面讲过消息幂等性需要业务自行保证。

### Broker集群部署

​	Broker集群部署是消息存储高可用的基本保障，最直接的表现是Broker出现单机故障或重启时，不会影响RocketMQ整体的服务能力。RocketMQ中Broker有四种不同的集群搭建方式。

* 单Master模式：单Master模式仅部署一台Broker机器，属于非集群模式，9.1.2节中的样例就采用了单Master模式。这种方式存在单点故障的风险，一旦Broker重启或者宕机，会导致整个服务不可用。不建议线上环境使用，仅可以用于本地测试。

* 多Master模式：一个集群全部都是Master机器，没有Slave机器，属于不配置主从复制的场景，例如2个Master或者3个Master。也不建议线上环境使用，这种模式的优缺点如下。

  * 优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复，由于RAID10磁盘非常可靠，消息也不会丢失（异步刷盘丢失少量消息，同步刷盘一条不丢），性能最高。
  * 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息的实时性会受到影响。这个缺点是致命的，消息实时性受到影响意味着一段时间内部分消费不可用，违背系统的可用性原则。

* 异步复制的多Master多Slave模式：每个Master配置一个Slave，有多对Master-Slave，主从复制采用异步复制方式，主备有短暂消息延迟（毫秒级），这种模式的优缺点如下。

  * 优点：即使磁盘损坏，消息丢失非常少，且消息实时性不会受影响，同时Master宕机后，消费者仍然可以从Slave消费，而且此过程对应用透明，不需要人工干预，性能同多Master模式几乎一样。
  * 缺点：在Master宕机且磁盘损坏的情况下可能会丢失少量消息。出现这种场景的概率很小，有风险但是很低。

* 同步复制的多Master多Slave模式：每个Master配置一个Slave，有多对Master-Slave，主从复制采用同步复制方式，即只有主备都写成功，才向应用返回成功。线上推荐使用异步刷盘+同步复制的多Master多Slave模式，这种模式的优缺点如下。

  * 优点：数据与服务都无单点故障，在Master宕机的情况下，消息无延迟，服务可用性与数据可用性都非常高。
  * 缺点：性能比异步复制模式略低（大约低10%左右），发送单个消息的RT会略高。

  


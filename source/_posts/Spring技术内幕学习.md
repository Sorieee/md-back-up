https://blog.csdn.net/zp357252539/article/details/80566937

```java
maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }

maven { url 'https://maven.aliyun.com/nexus/content/repositories/jcenter' }
```

# Spring Boot refresh

```java
ServletWebServerApplicationContext.refresh
    -> AbstractApplicationContext.refresh
    	-> AbstractApplicationContext.obtainFreshBeanFactory
    	-> GenericApplicationContext.refreshBeanFactory
```

```
BeanDefinitionReaderUtils.registerBeanDefinition
```

Bean容器

```java
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);    
```

先把BeanDefinition放到beanDefinitionMap中



# 2. Spring Framework的核心：IoC容器的实现

## 2.3IoC容器初始化过程

​	简单来说，IoC容器的初始化是由前面介绍的refresh()方法来启动的，这个方法标志着IoC容器的正式启动。具体来说，这个启动包括BeanDefinition的Resouce定位、载入和注册三个基本过程。如果我们了解如何编程式地使用IoC容器，就可以清楚地看到Resource定位和载入过程的接口调用。在下面的内容里，我们将会详细分析这三个过程的实现。

​	第一个过程是Resource定位过程。这个Resource定位指的是BeanDefinition的资源定位，它由ResourceLoader通过统一的Resource接口来完成，这个Resource对各种形式的BeanDefinition的使用都提供了统一接口。对于这些BeanDefinition的存在形式，相信大家都不会感到陌生。比如，在文件系统中的Bean定义信息可以使用FileSystemResource来进行抽象；在类路径中的Bean定义信息可以使用前面提到的ClassPathResource来使用，等等。这个定位过程类似于容器寻找数据的过程，就像用水桶装水先要把水找到一样。

​	第二个过程是BeanDefinition的载入。这个载入过程是把用户定义好的Bean表示成IoC容器内部的数据结构，而这个容器内部的数据结构就是BeanDefinition。下面介绍这个数据结构的详细定义。具体来说，这个BeanDefinition实际上就是POJO对象在IoC容器中的抽象，通过这个BeanDefinition定义的数据结构，使IoC容器能够方便地对POJO对象也就是Bean进行管理。在下面的章节中，我们会对这个载入的过程进行详细的分析，使大家对整个过程有比较清楚的了解。

​	第三个过程是向IoC容器注册这些BeanDefinition的过程。这个过程是通过调用BeanDefinitionRegistry接口的实现来完成的。这个注册过程把载入过程中解析得到的BeanDefinition向IoC容器进行注册。通过分析，我们可以看到，在IoC容器内部将BeanDefinition注入到一个HashMap中去，IoC容器就是通过这个HashMap来持有这些BeanDefinition数据的。

​	值得注意的是，这里谈的是IoC容器初始化过程，在这个过程中，一般不包含Bean依赖注入的实现。在Spring IoC的设计中，Bean定义的载入和依赖注入是两个独立的过程。依赖注入一般发生在应用第一次通过getBean向容器索取Bean的时候。但有一个例外值得注意，在使用IoC容器时有一个预实例化的配置，通过这个预实例化的配置（具体来说，可以通过为Bean定义信息中的lazyinit属性），用户可以对容器初始化过程作一个微小的控制，从而改变这个被设置了lazyinit属性的Bean的依赖注入过程。举例来说，如果我们对某个Bean设置了lazyinit属性，那么这个Bean的依赖注入在IoC容器初始化时就预先完成了，而不需要等到整个初始化完成以后，第一次使用getBean时才会触发。

### 2.3.1　BeanDefinition的Resource定位

​	以编程的方式使用DefaultListableBeanFactory时，首先定义一个Resource来定位容器使用的BeanDefinition。这时使用的是ClassPathResource，这意味着Spring会在类路径中去寻找以文件形式存在的BeanDefinition信息。

```java
ClassPathResource res=new ClassPathResource("beans.xml");
```

​	这里定义的Resource并不能由DefaultListableBeanFactory直接使用，Spring通过BeanDefinitionReader来对这些信息进行处理。在这里，我们也可以看到使用ApplicationContext相对于直接使用DefaultListableBeanFactory的好处。因为在ApplicationContext中，Spring已经为我们提供了一系列加载不同Resource的读取器的实现，而DefaultListableBeanFactory只是一个纯粹的IoC容器，需要为它配置特定的读取器才能完成这些功能。当然，有利就有弊，使用DefaultListableBeanFactory这种更底层的容器，能提高定制IoC容器的灵活性。

### 2.3.2　BeanDefinition的载入和解析

![](https://pic.imgdb.cn/item/61c6d03e2ab3f51d919971a9.jpg)

​	进入到AbstractRefreshableApplicationContext的refreshBeanFactory()方法中，在这个方法中创建了BeanFactory。在创建IoC容器前，如果已经有容器存在，那么需要把已有的容器销毁和关闭，保证在refresh以后使用的是新建立起来的IoC容器。这么看来，这个refresh非常像重启动容器，就像重启动计算机那样。在建立好当前的IoC容器以后，开始了对容器的初始化过程，比如BeanDefinition的载入，具体的交互过程如图2-10所示。

​	可以从AbstractRefreshableApplicationContext的refreshBeanFactory方法开始，了解这个Bean定义信息载入的过程。

### 2.3.3 BeanDefinition在IoC容器中的注册

```java
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```



![](https://pic.imgdb.cn/item/61c6da9c2ab3f51d919f10f0.jpg)

## 2.4 IoC容器的依赖注入

​	上面对IoC容器的初始化过程进行了详细的分析，这个初始化过程完成的主要工作是在IoC容器中建立BeanDefinition数据映射。在此过程中并没有看到IoC容器对Bean依赖关系进行注入，接下来分析一下IoC容器是怎样对Bean的依赖关系进行注入的。

​	假设当前IoC容器已经载入了用户定义的Bean信息，开始分析依赖注入的原理。首先，注意到依赖注入的过程是用户第一次向IoC容器索要Bean时触发的，当然也有例外，也就是我们可以在BeanDefinition信息中通过控制lazy-init属性来让容器完成对Bean的预实例化。这个预实例化实际上也是一个完成依赖注入的过程，但它是在初始化的过程中完成的，稍后我们会详细分析这个预实例化的处理。当用户向IoC容器索要Bean时，如果读者还有印象，那么一定还记得在基本的IoC容器接口BeanFactory中，有一个getBean的接口定义，这个接口的实现就是触发依赖注入发生的地方。为了进一步了解这个依赖注入过程的实现，下面从DefaultListableBeanFactory的基类AbstractBeanFactory入手去看看getBean的实现，如代码清单2-22所示。

```java
// Bean 的依赖Bean
	private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap<>(64);
// Bean被哪些Bean依赖了
	private final Map<String, Set<String>> dependenciesForBeanMap = new ConcurrentHashMap<>(64);
```

![](https://pic.imgdb.cn/item/61c6e6212ab3f51d91a7133c.jpg)

​	重点来说，getBean是依赖注入的起点，之后会调用createBean，下面通过createBean代码来了解这个实现过程。在这个过程中，Bean对象会依据BeanDefinition定义的要求生成。在AbstractAutowireCapableBeanFactory中实现了这个createBean，createBean不但生成了需要的Bean，还对Bean初始化进行了处理，比如实现了在BeanDefinition中的init-method属性定义，Bean后置处理器等。具体的过程如代码清单2-23所示。

​	这里用CGLIB对Bean进行实例化。CGLIB是一个常用的字节码生成器的类库，它提供了一系列的API来提供生成和转换Java的字节码的功能。在Spring AOP中也使用CGLIB对Java的字节码进行增强。在IoC容器中，要了解怎样使用CGLIB来生成Bean对象，需要看一下SimpleInstantiationStrategy类。这个Strategy是Spring用来生成Bean对象的默认类，它提供了两种实例化Java对象的方法，一种是通过BeanUtils，它使用了JVM的反射功能，一种是通过前面提到的CGLIB来生成，如代码清单2-25所示。

## 2.5　容器其他相关特性的设计与实现

### 2.5.1　ApplicationContext和Bean的初始化及销毁

​	对于BeanFactory，特别是ApplicationContext，容器自身也有一个初始化和销毁关闭的过程。下面详细看看在这两个过程中，应用上下文完成了什么，可以让我们更多地理解应用上下文的工作，容器初始化和关闭过程可以简要地通过图2-16来表现。

![](https://pic.imgdb.cn/item/61d440ad2ab3f51d917d8087.jpg)

​	从图中可以看到，对ApplicationContext启动的过程是在AbstractApplicationContext中实现的。在使用应用上下文时需要做一些准备工作，这些准备工作在prepareBeanFactory()方法中实现。在这个方法中，为容器配置了ClassLoader、PropertyEditor和BeanPost-Processor等，从而为容器的启动做好了必要的准备工作。

IoC的Bean的生命周期

* Bean实例的创建。

* 为Bean实例设置属性。
* 调用Bean的初始化方法。
* 应用可以通过IoC容器使用Bean。
* 当容器关闭时，调用Bean的销毁方法。

Bean的初始化方法调用是在以下的initializeBean方法中实现的：

### 2.5.4　BeanPostProcessor的实现

​	BeanPostProcessor是使用IoC容器时经常会遇到的一个特性，这个Bean的后置处理器是一个监听器，它可以监听容器触发的事件。将它向IoC容器注册后，容器中管理的Bean具备了接收IoC容器事件回调的能力。	BeanPostProcessor的使用非常简单，只需要通过设计一个具体的后置处理器来实现。同时，这个具体的后置处理器需要实现接口类BeanPostProcessor，然后设置到XML的Bean配置文件中。这个BeanPostProcessor是一个接口类，它有两个接口方法，一个是postProcessBeforeInitialization，在Bean的初始化前提供回调入口；一个是postProcessAfterInitialization，在Bean的初始化后提供回调入口，这两个回调的触发都是和容器管理Bean的生命周期相关的。这两个回调方法的参数都是一样的，分别是Bean的实例化对象和Bean的名字。BeanPostProcessor为具体的处理提供基本的回调输入，如代码清单2-33所示。

![](https://pic.imgdb.cn/item/61d44f542ab3f51d918992fd.jpg)

​	PostProcessBeforeInitialization是在populateBean完成之后被调用的。从BeanPostProcessor中的一个回调接口入手，对另一个回调接口postProcessAfterInitialization方法的调用，实际上也是在同一个地方封装完成的，这个地方就是populateBean方法中的initializeBean调用。关于这一点，读者会在接下来的分析中了解得很清楚。在前面对IoC的依赖注入进行分析时，对这个populateBean有过分析，这个方法实际上完成了Bean的依赖注入。在容器中建立Bean的依赖关系，是容器功能实现的一个很重要的部分。节选doCreateBean中的代码就可以看到postProcessBeforeInitialization调用和populateBean调用的关系，如下所示。

```java
Object exposedObject=bean;
try{
	populateBean(beanName,mbd,instanceWrapper);
/*在完成对Bean的生成和依赖注入以后，开始对Bean进行初始化，这个初始化过程包含了对后置处理器postProcessBeforeInitialization的回调*/
    exposedObject=initializeBean(
        beanName,exposedObject,mbd);
}
```

### 2.5.5 autowiring（自动依赖装配）的实现

​	在前面对IoC容器实现原理的分析中，一直是通过BeanDefinition的属性值和构造函数以显式的方式对Bean的依赖关系进行管理的。在Spring中，相对这种显式的依赖管理方式，IoC容器还提供了自动依赖装配的方式，为应用使用容器提供更大的方便。在自动装配中，不需要对Bean属性做显式的依赖关系声明，只需要配置好autowiring属性，IoC容器会根据这个属性的配置，使用反射自动查找属性的类型或者名字，然后基于属性的类型或名字来自动匹配IoC容器中的Bean，从而自动地完成依赖注入。

​	从autowiring使用上可以知道，这个autowiring属性在对Bean属性进行依赖注入时起作用。对Bean属性依赖注入的实现原理，在前面已经做过分析。回顾那部分内容，不难发现，对autowirng属性进行处理，从而完成对Bean属性的自动依赖装配，是在populateBean中实现的。节选AbstractAutowireCapableBeanFactory的populateBean方法中与autowiring实现相关的部分，可以清楚地看到这个特性在容器中实现的入口。也就是说，对属性autowiring的处理是populateBean处理过程的一个部分。在populateBean的实现中，在处理一般的Bean之前，先对autowiring属性进行处理。如果当前的Bean配置了autowire_by_name和autowire_by_type属性，那么调用相应的autowireByName方法和autowireByType方法。这两个方法很巧妙地应用了IoC容器的特性。例如，对于autowire_by_name，它首先通过反射机制从当前Bean中得到需要注入的属性名，然后使用这个属性名向容器申请与之同名的Bean，这样实际又触发了另一个Bean的生成和依赖注入的过程。实现过程如代码清单2-35所示。

### 2.5.7　Bean对IoC容器的感知

* BeanNameAware，可以在Bean中得到它在IoC容器中的Bean实例名称。
* BeanFactoryAware，可以在Bean中得到Bean所在的IoC容器，从而直接在Bean中使用IoC容器的服务。
* ApplicationContextAware，可以在Bean中得到Bean所在的应用上下文，从而直接在Bean中使用应用上下文的服务。
* MessageSourceAware，在Bean中可以得到消息源。
* ApplicationEventPublisherAware，在Bean中可以得到应用上下文的事件发布器，从而可以在Bean中发布应用上下文的事件。
* ResourceLoaderAware，在Bean中可以得到ResourceLoader，从而在Bean中使用ResourceLoader加载外部对应的Resource资源。

# 3. Spring AOP的实现

## 3.1　Spring AOP概述

### 3.1.1　AOP概念回顾

​	AspectJ：源代码和字节码级别的编织器，用户需要使用不同于Java的新语言。

​	AspectWerkz：AOP框架，使用字节码动态编织器和XML配置。

​	JBoss-AOP：基于拦截器和元数据的AOP框架，运行在JBoss应用服务器上。以及在AOP中用到的一些相关的技术实现：
​	BCEL（Byte-Code Engineering Library）：Java字节码操作类库，具体的信息可以参见项目网站http://jakarta.apache.org/bcel/。
​	Javassist：Java字节码操作类库，JBoss的一个子项目，项目信息可以参见项目网站http://jboss.org/javassist/。

![](https://pic.imgdb.cn/item/61d6da952ab3f51d914b2ea4.jpg)

​	AOP联盟定义的AOP体系结构把与AOP相关的概念大致分为由高到低、从使用到实现的三个层次。从上往下，最高层是语言和开发环境，在这个环境中可以看到几个重要的概念：“基础”（base）可以视为待增强对象或者说目标对象；“切面”（aspect）通常包含对于基础的增强应用；“配置”（configuration）可以看成是一种编织，通过在AOP体系中提供这个配置环境，可以把基础和切面结合起来，从而完成切面对目标对象的编织实现。

​	在分析中，以ProxyFactoryBean和ProxyFactory为例进行说明。

### 3.1.2　Advice通知

​	Advice（通知）定义在连接点做什么，为切面增强提供织入接口。在Spring AOP中，它主要描述Spring AOP围绕方法调用而注入的切面行为。Advice是AOP联盟定义的一个接口，具体的接口定义在org.aopalliance.aop.Advice中。在Spring AOP的实现中，使用了这个统一接口，并通过这个接口，为AOP切面增强的织入功能做了更多的细化和扩展，比如提供了更具体的通知类型，如BeforeAdvice、AfterAdvice、ThrowsAdvice等。作为Spring AOP定义的接口类，具体的切面增强可以通过这些接口集成到AOP框架中去发挥作用。

![](https://pic.imgdb.cn/item/61d706ff2ab3f51d916ef814.jpg)

在BeforeAdvice的继承关系中，定义了为待增强的目标方法设置的前置增强接口MethodBeforeAdvice，使用这个前置接口需要实现一个回调函数：

```java
void before(Method method,Object[]args,Object target)throws Throwable;
```

​	作为回调函数，before方法的实现在Advice中被配置到目标方法后，会在调用目标方法时被回调。具体的调用参数有：Method对象，这个参数是目标方法的反射对象；Object[]对象数组，这个对象数组中包含目标方法的输入参数。以CountingBeforeAdvice为例来说明BeforeAdvice的具体使用，CountingBeforeAdvice是接口MethodBeforeAdvice的具体实现，如代码清单3-1所示。可以看到，它的实现比较简单，完成的工作是统计被调用的方法次数。作为切面增强实现，它会根据调用方法的方法名进行统计，把统计结果根据方法名和调用次数作为键值对放入一个map中。

​	在Advice的实现体系中，Spring还提供了AfterAdvice这种通知类型，它的类接口关系如图3-3所示。

![](https://pic.imgdb.cn/item/61d70a862ab3f51d91728193.jpg)

​	了解了BeforeAdvice和AfterAdvice，在Spring AOP中，还可以看到另外一种Advice通知类型，那就是ThrowsAdvice，它的类层次关系如图3-4所示。

![](https://pic.imgdb.cn/item/61d70ad42ab3f51d9172b6f1.jpg)

### 3.1.3　Pointcut切点

​	Pointcut（切点）决定Advice通知应该作用于哪个连接点，也就是说通过Pointcut来定义需要增强的方法的集合，这些集合的选取可以按照一定的规则来完成。在这种情况下，Pointcut通常意味着标识方法，例如，这些需要增强的地方可以由某个正则表达式进行标识，或根据某个方法名进行匹配等。

![](https://pic.imgdb.cn/item/61d70ce32ab3f51d91742e72.jpg)

​	从源代码实现上同样可以得到相应的Spring AOP的Pointcut设计，如图3-6所示。

![](https://pic.imgdb.cn/item/61d70d022ab3f51d917445a3.jpg)

​	在Pointcut的基本接口定义中可以看到，需要返回一个MethodMatcher。对于Point的匹配判断功能，具体是由这个返回的MethodMatcher来完成的，也就是说，由这个MethodMatcher来判断是否需要对当前方法调用进行增强，或者是否需要对当前调用方法应用配置好的Advice通知。在Pointcut的类继承关系中，以正则表达式切点JdkRegexpMethodPointcut的实现原理为例，来具体了解切点Pointcut的工作原理。JdkRegexpMethodPointcut类完成通过正则表达式对方法名进行匹配的功能。在JdkRegexpMethodPointcut的基类StaticMethod-MatcherPointcut的实现中可以看到，设置MethodMatcher为StaticMethodMatcher，同时JdkRegexpMethodPointcut也是这个MethodMatcher的子类，它的类层次关系如图3-7所示。

![](https://pic.imgdb.cn/item/61d70d1d2ab3f51d91745832.jpg)

![](https://pic.imgdb.cn/item/61d70d352ab3f51d91746a48.jpg)

图　3-8　对JdkRegexpMethodPointcut的matches方法的调用关系

​	在JdkRegexpMethodPointcut中，通过JDK来实现正则表达式的匹配，这在代码清单3-5中可以看得很清楚。如果要详细了解使用JDK的正则表达式匹配功能，可以参考JDK的API文档。接下来看看其他的Pointcut。

​	在Spring AOP中，还提供了其他的MethodPointcut，比如通过方法名匹配进行Advice匹配的NameMatchMethodPointcut。它的matches方法实现很简单，匹配的条件是方法名相同或者方法名相匹配。

### 3.1.4　Advisor通知器

​	完成对目标方法的切面增强设计（Advice）和关注点的设计（Pointcut）以后，需要一个对象把它们结合起来，完成这个作用的就是Advisor（通知器）。通过Advisor，可以定义应该使用哪个通知并在哪个关注点使用它，也就是说通过Advisor，把Advice和Pointcut结合起来，这个结合为使用IoC容器配置AOP应用，或者说即开即用地使用AOP基础设施，提供了便利。在Spring AOP中，我们以一个Advisor的实现（DefaultPointcutAdvisor）为例，来了解Advisor的工作原理。在DefaultPointcutAdvisor中，有两个属性，分别是advice和pointcut。通过这两个属性，可以分别配置Advice和Pointcut。

## 3.2　Spring AOP的设计与实现

### 3.2.1　JVM的动态代理特性

​	前面已经介绍了横切关注点的一些概念，以及它们在Spring中的具体设计和实现。具体来说，在Spring AOP实现中，使用的核心技术是动态代理，而这种动态代理实际上是JDK的一个特性（在JDK 1.3以上的版本里，实现了动态代理模式）。通过JDK的动态代理特性，可以为任意Java对象创建代理对象，对于具体使用来说，这个特性是通过Java Reflection API来完成的。在了解具体的Java Reflection之前，先简要地复习一下Proxy模式，其静态类图如图3-9所示。

​	在图3-9中，可以看到有一个RealSubject，这个对象是目标对象，而在代理模式的设计中，会设计一个接口和目标对象一致的代理对象Proxy，它们都实现了接口Subject的request方法。在这种情况下，对目标对象的request的调用，往往就被代理对象“浑水摸鱼”给拦截了，通过这种拦截，为目标对象的方法操作做了铺垫，所以称之为代理模式。了解了如图3-10所示的调用关系，就可以清楚地了解这里的过程。

![](https://pic.imgdb.cn/item/61d710962ab3f51d9176ea2a.jpg)

![](https://pic.imgdb.cn/item/61d710a32ab3f51d9176f4cc.jpg)

​	如果在JDK中实现，需要实现下面所示的InvocationHandler接口：

```java
public interface InvocationHandler{
	public Object invoke(Object proxy,Method method,Object[]args)throws Throwable;
}
```

​	在这个接口方法中，只声明了一个invoke方法，这个invoke方法的第一个参数是代理对象实例，第二个参数是Method方法对象，代表的是当前Proxy被调用的方法，最后一个参数是被调用的方法中的参数。通过这些信息，在invoke方法实现中，已经可以了解Proxy对象的调用背景了。至于怎样让invoke方法和Proxy挂上钩，熟悉Proxy用法的读者都知道，只要在实现通过调用Proxy.newIntance方法生成具体Proxy对象时把InvocationHandler设置到参数里面就可以了，剩下的由Java虚拟机来完成。

### 3.2.2　Spring AOP的设计分析

​	在Spring AOP中，虽然对于AOP的使用者来说，只需要配置相关的Bean定义即可，但仔细分析Spring AOP的内部设计可以看到，为了让AOP起作用，需要完成一系列过程，比如，需要为目标对象建立代理对象，这个代理对象可以通过使用JDK的Proxy来完成，也可以通过第三方的类生成器CGLIB来完成。然后，还需要启动代理对象的拦截器来完成各种横切面的织入，这一系列的织入设计是通过一系列Adapter来实现的。通过一系列Adapter的设计，可以把AOP的横切面设计和Proxy模式有机地结合起来，从而实现在AOP中定义好的各种织入方式。具体的设计实现可以参考后面的内容，这里只是简要介绍一下。

### 3.3　建立AopProxy代理对象

![](https://pic.imgdb.cn/item/61d9053c2ab3f51d91ba04c7.jpg)

​	在这个类继承关系中，可以看到完成AOP应用的类，比如AspectJProxyFactory、ProxyFactory和ProxyFactoryBean，它们都在同一个类的继承体系下，都是ProxyConfig、AdvisedSupport和ProxyCreatorSupport的子类。作为共同基类，可以将ProxyConfig看成是一个数据基类，这个数据基类为ProxyFactoryBean这样的子类提供了配置属性；在另一个基类AdvisedSupport的实现中，封装了AOP对通知和通知器的相关操作，这些操作对于不同的AOP的代理对象的生成都是一样的，但对于具体的AOP代理对象的创建，AdvisedSupport把它交给它的子类们去完成；对于ProxyCreatorSupport，可以将它看成是其子类创建AOP代理对象的一个辅助类。通过继承以上提到的基类的功能实现，具体的AOP代理对象的生成，根据不同的需要，分别由ProxyFactoryBean、AspectJProxyFactory和ProxyFactory来完成。对于需要使用AspectJ的AOP应用，AspectJProxyFactory起到集成Spring和AspectJ的作用；对于使用Spring AOP的应用，ProxyFactoryBean和ProxyFactoy都提供了AOP功能的封装，只是使用ProxyFactoryBean，可以在IoC容器中完成声明式配置，而使用ProxyFactory，则需要编程式地使用Spring AOP的功能；对于它们是如何封装实现AOP功能的，会在本章小结中给出详细的分析，在这里，通过这些类层次关系的介绍，先给读者留下一个大致的印象。Proxy-FactoryBean相关的类层次关系如图3-12所示。

![](https://pic.imgdb.cn/item/61d908362ab3f51d91bc6759.jpg)

### 3.3.2　配置ProxyFactoryBean

1）定义使用的通知器Advisor，这个通知器应该作为一个Bean来定义。很重要的一点是，这个通知器的实现定义了需要对目标对象进行增强的切面行为，也就是Advice通知。
	2）定义ProxyFactoryBean，把它作为另一个Bean来定义，它是封装AOP功能的主要类。在配置ProxyFactoryBean时，需要设定与AOP实现相关的重要属性，比如proxyInterface、interceptorNames和target等。从属性名称可以看出，interceptorNames属性的值往往设置为需要定义的通知器，因为这些通知器在ProxyFactoryBean的AOP配置下，是通过使用代理对象的拦截器机制起作用的。所以，这里依然沿用了拦截器这个名字，也算是旧瓶装新酒吧。
	3）定义target属性，作为target属性注入的Bean，是需要用AOP通知器中的切面应用来增强的对象，也就是前面提到的base对象。

### 3.3.3　ProxyFactoryBean生成AopProxy代理对象

![](https://pic.imgdb.cn/item/61d90efe2ab3f51d91c18f51.jpg)

### 3.3.4　JDK生成AopProxy代理对象

![](https://pic.imgdb.cn/item/61d913472ab3f51d91c47c5c.jpg)

![](https://pic.imgdb.cn/item/61d93bd62ab3f51d91e3ac03.jpg)

# 4. Spring MVC与Web环境

![](https://pic.imgdb.cn/item/61e92a4b2ab3f51d915a0991.jpg)

## 4.2 Web环境的Spring MVC

​	Spring IoC是一个独立的模块，它并不是直接在Web容器中发挥作用的，如果要在Web环境中使用IoC容器，需要Spring为IoC设计一个启动过程，把IoC容器导入，并在Web容器中建立起来。具体说来，这个启动过程是和Web容器的启动过程集成在一起的。在这个过程中，一方面处理Web容器的启动，另一方面通过设计特定的Web容器拦截器，将IoC容器载入到Web环境中来，并将其初始化。在这个过程建立完成以后，IoC容器才能正常工作，而Spring MVC是建立在IoC容器的基础上的，这样才能建立起MVC框架的运行机制，从而响应从Web容器传递的HTTP请求。

### 4.3 上下文在Web容器中的启动

![](https://pic.imgdb.cn/item/61ea65c92ab3f51d918b350e.jpg)

​	在web.xml中，已经配置了ContextLoaderListener，这个ContextLoaderListener是Spring提供的类，是为在Web容器中建立IoC容器服务的，它实现了ServletContextListener接口。

![](https://pic.imgdb.cn/item/61ea66892ab3f51d918c8514.jpg)

​	在ContextLoader中，完成了两个IoC容器建立的基本过程，一个是在Web容器中建立起双亲IoC容器，另一个是生成相应的WebApplicationContext并将其初始化。

### 4.3.2　Web容器中的上下文设计

![](https://pic.imgdb.cn/item/61ea68032ab3f51d918ef93d.jpg)

### 4.3.3　ContextLoader的设计与实现

​		对于Spring承载的Web应用而言，可以指定在Web应用程序启动时载入IoC容器（或者称为WebApplicationContext）。这个功能是由ContextLoaderListener这样的类来完成的，它是在Web容器中配置的监听器。

## 4.4 Spring MVC的设计与实现

​	面简要回顾了MVC模式，并且知道了Spring MVC是一个MVC模式的实现。在Spring MVC的使用中，看到了在web.xml中，除了需要配置ContextLoaderListener之外，还要对DispatcherServlet进行配置。作为一个Servlet，这个DispatcherServlet实现的是Sun的J2EE核心模式中的前端控制器模式（Front Controller），作为一个前端控制器，所有的Web请求都需要通过它来处理，进行转发、匹配、数据处理后，并转由页面进行展现，因此这个DispatcerServlet可以看成是Spring MVC实现中最为核心的部分，它的设计与分析也是下面分析Spring MVC的一条主线。

​	除了这条主线，在Spring MVC中，对于不同的Web请求的映射需求，Spring MVC提供了不同的HandlerMapping的实现，可以让应用开发选取不同的映射策略。

### 4.4.2 Spring MVC设计概览

​	在完成对ContextLoaderListener的初始化以后，Web容器开始初始化DispatcherServlet，这个初始化的启动与在web.xml中对载入次序的定义有关。DispatcherServlet会建立自己的上下文来持有Spring MVC的Bean对象，在建立这个自己持有的IoC容器时，会从ServletContext中得到根上下文作为DispatcherServlet持有上下文的双亲上下文。有了这个根上下文，再对自己持有的上下文进行初始化，最后把自己持有的这个上下文保存到ServletContext中，供以后检索和使用。

​	为了解这个过程，可以从DispatcherServlet的父类FrameworkServlet的代码入手，去探寻Dispatcher-Servlet的启动过程，它同时也是Spring MVC的启动过程。

![](https://pic.imgdb.cn/item/61ea68032ab3f51d918ef93d.jpg)



![](https://pic.imgdb.cn/item/61ea7a172ab3f51d91ab22ff.jpg)

​	DispatcherServlet的工作大致可以分为两个部分：一个是初始化部分，由initServletBean()启动，通过initWebApplicationContext()方法最终调用DispatcherServlet的initStrategies方法，在这个方法里，DispatcherServlet对MVC模块的其他部分进行了初始化，比如handlerMapping、ViewResolver等；另一个是对HTTP请求进行响应，作为一个Servlet，Web容器会调用Servlet的doGet()和doPost()方法，在经过FrameworkServlet的processRequest()简单处理后，会调用DispatcherServlet的doService()方法，在这个方法调用中封装了doDispatch()，这个doDispatch()是Dispatcher实现MVC模式的主要部分。

### 4.4.3　DispatcherServlet的启动和初始化

​	作为Servlet，DispatcherServlet的启动与Servlet的启动过程是相联系的。在Servlet的初始化过程中，Servlet的init方法会被调用，以进行初始化。DispatcherServlet的基类HttpServletBean中的这个初始化过程如代码清单4-8所示。在初始化开始时，需要读取配置在ServletContext中的Bean属性参数，这些属性参数设置在web.xml的Web容器初始化参数中。使用编程式的方式来设置这些Bean属性，在这里可以看到对PropertyValues和BeanWrapper的使用。对于这些和依赖注入相关的类的使用，在分析IoC容器的初始化时，尤其是在依赖注入实现分析时，有过“亲密接触”。只是这里的依赖注入是与Web容器初始化相关的，初始化过程由HttpServletBean来完成。

​	接着会执行DispatcherServlet持有的IoC容器的初始化过程，在这个初始化过程中，一个新的上下文被建立起来，这个DispatcherServlet持有的上下文被设置为根上下文的子上下文。可以认为，根上下文是和Web应用相对应的一个上下文，而DispatcherServlet持有的上下文是和Servlet对应的一个上下文。在一个Web应用中，往往可以容纳多个Servlet存在；与此相对应，对于应用在Web容器中的上下体系，一个根上下文可以作为许多Servlet上下文的双亲上下文。了解了这一点，对在Web环境中IoC容器中的Bean设置和检索会有更多的了解，因为了解IoC工作原理的读者知道，在向IoC容器getBean时，IoC容器会首先向其双亲上下文去getBean，也就是说，在根上下文中定义的Bean是可以被各个Servlet持有的上下文得到和共享的。DispatcherServlet持有的上下文被建立起来以后，也需要和其他IoC容器一样完成初始化，这个初始化也是通过refresh方法来完成的。最后，DispatcherServlet给这个自己持有的上下文命名，并把它设置到Web容器的上下文中，这个名称和在web.xml中设置的DispatcherServlet的Servlet名称有关，从而保证了这个上下文在Web环境上下文体系中的唯一性。

### 4.4.4 MVC处理HTTP分发请求

![](https://pic.imgdb.cn/item/61f3b2822ab3f51d91f39e39.jpg)



![](https://pic.imgdb.cn/item/61f3b8192ab3f51d91f94b34.jpg)



![](https://pic.imgdb.cn/item/61f4fe2e2ab3f51d913182af.jpg)


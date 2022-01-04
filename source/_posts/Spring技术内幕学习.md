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


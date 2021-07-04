# 架构图

![](https://pic.imgdb.cn/item/60dd67ed5132923bf88c418f.jpg)

![](https://pic.imgdb.cn/item/60e084ed5132923bf88eb35b.jpg)

![](https://pic.imgdb.cn/item/60e086675132923bf89cada0.jpg)

**服务的暴露过程**

​	首先，服务器端（服务提供者）在框架启动时，会初始化服务实例，通过Proxy组件调
用具体协议（Protocol ）,把服务端要暴露的接口封装成Invoker （真实类型是
AbstractProxylnvoker）,然后转换成Exporter,这个时候框架会打开服务端口等并记录服务实例
到内存中，最后通过Registry把服务元数据注册到注册中心。

* Proxy组件：我们知道，Dubbo中只需要引用一个接口就可以调用远程的服务，并且
  只需要像调用本地方法一样调用即可。其实是Dubbo框架为我们生成了代理类，调用
  的方法其实是Proxy组件生成的代理方法，会自动发起远程/本地调用，并返回结果,
  整个过程对用户完全透明。
* Protocol：顾名思义，协议就是对数据格式的一种约定。它可以把我们对接口的配置,根据不同的协议转换成不同的Invoker对象。例如：用DubboProtocol可以把XML文件中一个远程接口的配置转换成一个Dubbolnvokero
* Exporter：用于暴露到注册中心的对象，它的内部属性持有了Invoker对象，我们可以认为它在Invoker上包了一层。
* Registry：把Exporter注册到注册中心。

![](https://pic.imgdb.cn/item/60e087385132923bf8a480bb.jpg)

​	首先，调用过程也是从一个Proxy开始的，Proxy持有了一个Invoker对象。然后触发invoke调用。在invoke调用过程中，需要使用Cluster,Cluster负责容错，如调用失败的重试。

​	Cluster在调用之前会通过Directory获取所有可以调用的远程服务Invoker列表（一个接口可能有多个节点提供服务）。由于可以调用的远程服务有很多，此时如果用户配置了路由规则（如指定某些方法只能调用某个节点），那么还会根据路由规则将Invoker列表过滤一遍。

​	然后，存活下来的Invoker可能还会有很多，此时要调用哪一个呢？于是会继续通过LoadBalance方法做负载均衡，最终选出一个可以调用的Invokero这个Invoker在调用之前又会经过一个过滤器链，这个过滤器链通常是处理上下文、限流、计数等。

​	接着，会使用Client做数据传输，如我们常见的NettyClient等。传输之前肯定要做一些私有协议的构造，此时就会用到Codec接口。构造完成后，就对数据包做序列化（Serialization）,然后传输到服务提供者端。服务提供者收到数据包，也会使用Codec处理协议头及一些半包、粘包等。处理完成后再对完整的数据报文做反序列化处理。

​	随后，这个Request会被分配到线程池（ThreadPool）中进行处理oServer会处理这些Request,根据请求查找对应的Exporter（它内部持有了Invoker）0Invoker是被用装饰器模式一层一层套了非常多Filter的，因此在调用最终的实现类之前，又会经过一个服务提供者端的过滤器链。

​	随后，这个Request会被分配到线程池（ThreadPool）中进行处理oServer会处理这些Request,根据请求查找对应的Exporter（它内部持有了Invoker）0Invoker是被用装饰器模式一层一层套了非常多Filter的，因此在调用最终的实现类之前，又会经过一个服务提供者端的过滤器链。

​	最终，我们得到了具体接口的真实实现并调用，再原路把结果返回。

![](https://pic.imgdb.cn/item/60e088655132923bf8b044e6.jpg)



# 语雀

https://item.jd.com/12545055.html

https://www.yuque.com/apache-dubbo/dubbo3/pyrcyr

https://www.yuque.com/apache-dubbo


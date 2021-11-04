# dubbo-demo-api-provider

问题

## zookeeper not connected

修改timeout

```java
private static void startWithBootstrap() {
    ServiceConfig<DemoServiceImpl> service = new ServiceConfig<>();
    service.setInterface(DemoService.class);
    service.setRef(new DemoServiceImpl());
    RegistryConfig registryConfig =
        new RegistryConfig("zookeeper://192.168.1.200:2181,192.168.1.200:2182,192.168.1.200:2183");
    registryConfig.setTimeout(100000);
    DubboBootstrap bootstrap = DubboBootstrap.getInstance();
    bootstrap.application(new ApplicationConfig("dubbo-demo-api-provider"))
        .registry(registryConfig)
        .service(service)
        .start()
        .await();
}
```

## Failed to bind NettyServer 

Failed to bind NettyServer  on /192.168.1.4:20880, cause: Address already in use: bind

设置ProtocolConfig

```java
ProtocolConfig protocolConfig = new ProtocolConfig();
protocolConfig.setPort(-1);
DubboBootstrap bootstrap = DubboBootstrap.getInstance();
bootstrap.application(new ApplicationConfig("dubbo-demo-api-consumer"))
        .registry(registryConfig)
        .reference(reference)
        .protocol(protocolConfig)
        .start();
```

## 注册ShutDownHook

```
Runtime.getRuntime().addShutdownHook(this);
```

# todo

* CompletableFuture
* 
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

```java
Runtime.getRuntime().addShutdownHook(this);
Runtime.getRuntime().removeShutdownHook(this);
```

## 判断线程active

```java
if (this.isAlive()) {
    // DubboShutdownHook thread is running
    return;
}
```

## Class::isAssignableFrom

isAssignableFrom()方法与instanceof关键字的区别总结为以下两个点：

- isAssignableFrom()方法是从类继承的角度去判断，instanceof关键字是从实例继承的角度去判断。
- isAssignableFrom()方法是判断是否为某个类的父类，instanceof关键字是判断是否某个类的子类。

```
父类.class.isAssignableFrom(子类.class)

子类实例 instanceof 父类类型
```

# ParameterizedType

https://www.cnblogs.com/baiqiantao/p/7460580.html



# 源码

# Bootstrap启动流程

```
Future future = applicationDeployer.start();
	
```

![](https://pic.imgdb.cn/item/6183dab12ab3f51d91128b46.jpg)

## DefaultApplicationDepolyer::initialize

关键代码

```java
// register shutdown hook
registerShutdownHook();

startConfigCenter();

loadApplicationConfigs();

initModuleDeployers();

// @since 2.7.8
startMetadataCenter();

initMetadataService();

initialized.set(true);
```

### startConfigCenter()

```java
private void startConfigCenter() {

    // load application config
    configManager.loadConfigsOfTypeFromProps(ApplicationConfig.class);

    // try set model name
    if (StringUtils.isBlank(applicationModel.getModelName())) {
        applicationModel.setModelName(applicationModel.tryGetApplicationName());
    }

    // load config centers
    configManager.loadConfigsOfTypeFromProps(ConfigCenterConfig.class);

    useRegistryAsConfigCenterIfNecessary();

    // check Config Center
    Collection<ConfigCenterConfig> configCenters = configManager.getConfigCenters();
    if (CollectionUtils.isEmpty(configCenters)) {
        ConfigCenterConfig configCenterConfig = new ConfigCenterConfig();
        configCenterConfig.setScopeModel(applicationModel);
        configCenterConfig.refresh();
        ConfigValidationUtils.validateConfigCenterConfig(configCenterConfig);
        if (configCenterConfig.isValid()) {
            configManager.addConfigCenter(configCenterConfig);
            configCenters = configManager.getConfigCenters();
        }
    } else {
        for (ConfigCenterConfig configCenterConfig : configCenters) {
            configCenterConfig.refresh();
            ConfigValidationUtils.validateConfigCenterConfig(configCenterConfig);
        }
    }

    if (CollectionUtils.isNotEmpty(configCenters)) {
        CompositeDynamicConfiguration compositeDynamicConfiguration = new CompositeDynamicConfiguration();
        for (ConfigCenterConfig configCenter : configCenters) {
            // Pass config from ConfigCenterBean to environment
            environment.updateExternalConfigMap(configCenter.getExternalConfiguration());
            environment.updateAppExternalConfigMap(configCenter.getAppExternalConfiguration());

            // Fetch config from remote config center
            compositeDynamicConfiguration.addConfiguration(prepareEnvironment(configCenter));
        }
        environment.setDynamicConfiguration(compositeDynamicConfiguration);
    }
}
```



# todo

* CompletableFuture
* 


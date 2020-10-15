---
title: spring cloud stream with rocketmq
date: 2020-07-21 17:11:53
tags: [spring cloud straem, rocketmq, spring cloud, spring cloud alibaba]
---

 Spring Cloud Alibaba RocketMQ Example

https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/rocketmq-example/readme-zh.md

guide:

https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_spring_cloud_alibaba_rocketmq_binder

# 项目说明

本项目演示如何使用 RocketMQ Binder 完成 Spring Cloud 应用消息的订阅和发布。

[RocketMQ](https://rocketmq.apache.org/) 是一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。

在说明 RocketMQ 的示例之前，我们先了解一下 Spring Cloud Stream。

这是官方对 Spring Cloud Stream 的一段介绍：

Spring Cloud Stream 是一个用于构建基于消息的微服务应用框架。它基于 SpringBoot 来创建具有生产级别的单机 Spring 应用，并且使用 `Spring Integration` 与 Broker 进行连接。

Spring Cloud Stream 提供了消息中间件配置的统一抽象，推出了 publish-subscribe、consumer groups、partition 这些统一的概念。

Spring Cloud Stream 内部有两个概念：Binder 和 Binding。

* Binder: 跟外部消息中间件集成的组件，用来创建 Binding，各消息中间件都有自己的 Binder 实现。

比如 `Kafka` 的实现 `KafkaMessageChannelBinder`，`RabbitMQ` 的实现 `RabbitMessageChannelBinder` 以及 `RocketMQ` 的实现 `RocketMQMessageChannelBinder`。

* Binding: 包括 Input Binding 和 Output Binding。

Binding 在消息中间件与应用程序提供的 Provider 和 Consumer 之间提供了一个桥梁，实现了开发者只需使用应用程序的 Provider 或 Consumer 生产或消费数据即可，屏蔽了开发者与底层消息中间件的接触。

# 下载并启动 RocketMQ

**在接入 RocketMQ Binder 之前，首先需要启动 RocketMQ 的 Name Server 和 Broker。**

1. 下载[RocketMQ最新的二进制文件](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.3.2/rocketmq-all-4.3.2-bin-release.zip)，并解压
2. 启动 Name Server

```sh
sh bin/mqnamesrv
#windows是
start mqnamesrv.cmd
```

1. 启动 Broker

```sh
sh bin/mqbroker -n localhost:9876
#windows是
start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable=true
```

1. 创建 Topic: test-topic

```sh
sh bin/mqadmin updateTopic -n localhost:9876 -c DefaultCluster -t test-topic
#windows：
start mqadmin updateTopic -n localhost:9876 -c DefaultCluster -t test-topic
```

# 如何使用Spring Cloud Alibaba RocketMQ Binder

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-stream-binder-rocketmq</artifactId>
</dependency>
```

或者

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>
```

我这里选择的是

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```



最新的依赖2.2.1和文档配置有一些不一样，所以还是用的2.0.0版本。

# Spring Cloud Alibaba Rocketmq Binder工作原理

![](https://pic.imgdb.cn/item/5f18de9d14195aa594d613bf.jpg)



RocketMQ Binder实现依赖于Rocket-Spring Framework

Rocket-Spring Framework整合了RocketMQ和Spring Boot，它提供了以下主要的特性：

1. `RocketMQTemplate`：发送消息，包括同步消息、异步消息和事务消息；
2. `@RocketMQTransactionListener` : 监听检查事务消息；
3. `@RocketMQMessageListener`：消费消息。



	`RocketMQMessageChannelBinder` 是Binder的标准实现，会在内部构建`RocketMQInboundChannelAdapter`和`RocketMQMessageHandler`
	
	`RocketMQMessageHandler`会基于Binding configuration去构造`RocketMQTemplate`。`RocketMQTemplate`会在内部将 `spring-messaging(org.springframework.messaging.Message)` 转换成 RocketMQ message(`org.apache.rocketmq.common .message.Message`)，然后发送出去。
	
	`RocketMQInboundChannelAdapter`也会基于配置构建`RocketMQListenerBindingContainer`，然后`RocketMQListenerBindingContainer`会启动消费者去接收消息。


​	

	目前Binder支持在Header中设置一些RocketMQ Message相关的属性。
	
	比如 `TAGS`, `DELAY`, `TRANSACTIONAL_ARG`, `KEYS`, `WAIT_STORE_MSG_OK`, `FLAG`

```java
MessageBuilder builder = MessageBuilder.withPayload(msg)
    .setHeader(RocketMQHeaders.TAGS, "binder")
    .setHeader(RocketMQHeaders.KEYS, "my-key")
    .setHeader(MessageConst.PROPERTY_DELAY_TIME_LEVEL, "1");
Message message = builder.build();
output().send(message);
```

# 支持MessageSource

支持`MessageSource`,它支持用pull模式接收消息。



```java
SpringBootApplication
@EnableBinding(MQApplication.PolledProcessor.class)
public class MQApplication {

  private final Logger logger =
  	  LoggerFactory.getLogger(MQApplication.class);

  public static void main(String[] args) {
    SpringApplication.run(MQApplication.class, args);
  }

  @Bean
  public ApplicationRunner runner(PollableMessageSource source,
  	    MessageChannel dest) {
    return args -> {
      while (true) {
        boolean result = source.poll(m -> {
          String payload = (String) m.getPayload();
          logger.info("Received: " + payload);
          dest.send(MessageBuilder.withPayload(payload.toUpperCase())
              .copyHeaders(m.getHeaders())
              .build());
        }, new ParameterizedTypeReference<String>() { });
        if (result) {
          logger.info("Processed a message");
        }
        else {
          logger.info("Nothing to do");
        }
        Thread.sleep(5_000);
      }
    };
  }

  public static interface PolledProcessor {

    @Input
    PollableMessageSource source();

    @Output
    MessageChannel dest();

  }

}
```



# 配置项

## RocketMQ Binder属性

**spring.cloud.stream.rocketmq.binder.name-server**

	RocketMQ服务端的名称/地址，(老版本是namesrv-addr ).
	
	默认: `127.0.0.1:9876`.



**spring.cloud.stream.rocketmq.binder.access-key**

	The AccessKey of Alibaba Cloud Account.
	
	Default: null.

**spring.cloud.stream.rocketmq.binder.secret-key**

	The SecretKey of Alibaba Cloud Account.
	
	Default: null.

**spring.cloud.stream.rocketmq.binder.enable-msg-trace**

	对于所有的生产者和消费者启动消息追踪特性
	
	Default: `true`.

**spring.cloud.stream.rocketmq.binder.customized-trace-topic**

	The trace topic for message trace.
	
	Default: `RMQ_SYS_TRACE_TOPIC`.



## RocketMQ Consumer属性

	以下属性都以`spring.cloud.stream.rocketmq.bindings.<channelName>.consumer.`开头。

**enable**

	启用消费者Binding
	
	Default: `true`.

**tags**

	消费者订阅tag的表达式，用`||`切分.
	
	Default: empty.

**sql**

	消息订阅的sql表达式
	
	Default: empty.

**broadcasting**

	广播模式。
	
	Default: `false`.

**orderly**

	顺序消费
	
	Default: `false`.

**delayLevelWhenNextConsume**

	消费重试策略

- -1,不重试，直接放入死信队列；

- 0, broker控制重试的频率

- \>0,client 控制重试的策略

  Default: `0`.

**suspendCurrentQueueTimeMillis**

	花费在当前队列的时间
	
	Suspending pulling time for cases requiring slow pulling like flow-control scenario
	
	Default: `1000`.

## RocketMQ Provider属性

	以下属性都以`spring.cloud.stream.rocketmq.bindings.<channelName>.producer.`开头

**enable**

	启用生产者binding.
	
	Default: `true`.

**group**

	生产者组名
	
	Default: empty.

**maxMessageSize**

	允许最大消息长度byte
	
	Default: `8249344`.

**transactional**

	发送事务消息

Default: `false`.

**sync**

	发送同步消息
	
	Default: `false`.

**vipChannelEnabled**

	Send message with vip channel.
	
	Default: `true`.

**sendMessageTimeout**

	发送消息的过期时间
	
	Default: `3000`.

**compressMessageBodyThreshold**

	压缩消息阈值单位KB，默认超过4k就压缩
	
	Default: `4096`.

**retryTimesWhenSendFailed**

	同步模式下的最大重试次数
	
	Default: `2`.

**retryTimesWhenSendAsyncFailed**

	异步模式下最大重试次数
	
	Default: `2`.

**retryNextServer**

	重试是否用下一个broker发送失败消息。
	
	Default: `false`.

# RocketMQ Example

暂略 
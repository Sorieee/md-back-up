---
title: Kafka学习笔记
date: 2020-04-03 20:45:04
tags: [kafka,消息队列]
---

# 1.学习资源



kafka：

​	官方文档:

​		中文版：http://kafka.apachecn.org/

​		英文版:http://kafka.apache.org/

​	安装下载:

​			https://blog.csdn.net/woshixiazaizhe/article/details/80610432

​	Spring boot应用

​			https://blog.csdn.net/Black1499/article/details/90474929



​	spring kafka文档：

​			https://docs.spring.io/spring-kafka/docs/2.4.4.RELEASE/reference/html/



# 2.安装配置

## 系统环境

centos8

## 2.1JDK安装配置

可以参照:https://blog.csdn.net/pang_ping/article/details/80570011

从官网下载jdk13，然后在centos上安装lrzsz(rz还需要下载xshell)，上传jdk。

配置环境变量(vim /etc/profile)

因为是将jdk解压在了/opt/java文件夹中，所以配置环境变量在profile尾部添加

```
export JAVA_HOME=/opt/java/jdk-13
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

然后输入命令生效并验证

```
# source profile
# java -version
```

## 2.2Zookeeper安装

和jdk类似，下载Zookeeper3.5.7, 使用lrzsz上传到/opt,并解压

* 修改Zookeeper的配置文件，进入安装路径的conf目录，并将zoo_sample.cfg修改成zoo.cfg

文件内容如下:

```
# The number of milliseconds of each tick
# zk服务器心跳时间
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
# 投票选举新Leader的初始化时间
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# 数据目录
dataDir=/tmp/zookeeper
# 日志目录
dataLogDir=/tmp/zookeeper/log
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

修改数据目录到zookeeper安装目录的data文件夹，log文件夹指定到安装目录的newlog文件夹

然后**启动并验证**

```
sh bin/zkServer.sh start
ps -ef |grep zookeeper
jps -l
```

## 2.3Kafka安装

从官网下载，用lrzsz上传到/opt文件夹，并解压

启动命令 

```
bin/kafka-server-start.sh config/server.properties
```

server.properties需要关注以下几个参数

```
broker.id=0,表示broker的编号，如果集群有多个broker，则每个broker的编号需要设置的不同
listeners=PLAINTEXT://:9092 broker对外提供服务的入口地址
log.dirs=/tmp/kafka/log 设置存放消息日志文件的地址
zookeeper.connect=localhost:2181 Kafka所需的Zookeeper集群地址,教学中Zookeeper和Kafka都安装本机
```

验证启动成功

```
jps -l
```

## 2.4Kafka测试消息生成与消费

//todo

## 2.5.Java程序测试



在idea创建一个springboot项目

pom.xml如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.sorie</groupId>
    <artifactId>kafka-learn</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>kafka-learn</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <scala.version>2.11</scala.version>
        <slf4j.version>1.7.21</slf4j.version>
        <kafka.version>2.0.0</kafka.version>
        <lombok.version>1.18.8</lombok.version>
        <junit.version>4.11</junit.version>
        <gson.version>2.2.4</gson.version>
        <protobuff.version>1.5.4</protobuff.version>
        <spark.version>2.3.1</spark.version>
    </properties>

    <dependencies>
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
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_${scala.version}</artifactId>
            <version>${kafka.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>${gson.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <dependency>
            <groupId>io.protostuff</groupId>
            <artifactId>protostuff-runtime</artifactId>
            <version>${protobuff.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>${kafka.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.9.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_${scala.version}</artifactId>
            <version>${spark.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql-kafka-0-10_${scala.version}</artifactId>
            <version>${spark.version}</version>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>com.alibaba</groupId>-->
<!--            <artifactId>fastjson</artifactId>-->
<!--            <version>1.2.58</version>-->
<!--        </dependency>-->
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

### 创建生产者

```java
package com.sorie.kafkalearn.ch1;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class ProducerFastStart {
    private static final String brokerList = "192.168.1.6:9092";
    private static final String topic = "sorie";
    public static void main(String[] args) {
        Properties properties = new Properties();
        //设置key序列化器
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
        // 设置重试次数
        properties.put(ProducerConfig.RETRIES_CONFIG, 10);
        // 设置值序列化器
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringSerializer");
        // 设置集群地址
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);

        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        ProducerRecord<String, String> record = new ProducerRecord<>(topic, "kafka-demo", "hello");
        try {
            producer.send(record);
        } catch (Exception e) {
            e.printStackTrace();
        }
        producer.close();
    }
}
```

启动该程序，发现发送不成功

首先关闭kafka的防火墙(如果不设置关闭防火墙如何做？//todo)

```
# 查看防火墙是否开启
firewall-cmd --state
systemctl stop firewalld.service
```

关闭了防火墙依然报错，修改server.properties配置

```
listeners=PLAINTEXT://192.168.1.6:9092
```

然后执行程序 发送成功

### 创建消费者

```java
package com.sorie.kafkalearn.ch1;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.producer.ProducerConfig;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class ConsumerFastStart {
    private static final String brokerList = "192.168.1.6:9092";
    private static final String topic = "sorie";
    private static final String grpId = "grp.demo";

    public static void main(String[] args) {
        Properties properties = new Properties();
        //设置key反序列化器
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");
        //设置value序列化器
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                "org.apache.kafka.common.serialization.StringDeserializer");

        //设置集群地址
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
                brokerList);

        properties.put(ConsumerConfig.GROUP_ID_CONFIG,
                grpId);

        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(properties);
        consumer.subscribe(Collections.singletonList(topic));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record.value());
            }
        }
    }
}
```

创建后运行程序先运行消费者，再运行生产者，可以看到hello的输出

# 3.生产者详解

## 3.1消息发送

### 3.1.1 Kafka Java客户端数据生产流程解析

![](https://pic.imgdb.cn/item/5ea1a0a6c2a9a83be5338f3e.jpg)

### 3.1.2 必要参数配置

略

### 3.1.3 发送类型

**发送即忘记**

producer.send(record);

**同步发送**

```java
//通过send()发弄完一个消息后返回一个future对象，然后调用Future对象的get()等待kafka响应
//如果kafka正常响应，返回一个recordMetadata对象，改对象存储消息的偏移量
//如果kafka发送错误，无法正常响应，就会抛出异常，我们便可以进行异常处理
producer.send(record).get()
```

**异步发送**

```java
producer.send(record, new Callback() {
	public void onCompletion(RecordMetadata metadata, Exception exception) {
		if(exception == null) {
			//xxxxx
		}
	}
})
```

### 3.1.4 序列化器

消息要到网络上传输，必须进行序列化，而序列化器的作用就是如此。

Kafka提供了默认的字符串序列化器（org.apche.kafka.common.serialization.StringSerializer)，还有整型(IntegerSerializer)和字节数组(BytesSerializer),这些序列化器都实现了接口(org.apache.kafka.common.serialization.Serializer)基本上能够满足大部分场景的需求

### 3.1.5 分区器

本身kafka有自己的分区策略，如果未指定，就会使用默认的分区策略。

Kafka根据传递消息的key来进行分区的分配，即hash(key) % numPartitions。如果Key相同的话,那么就会分配到统一分区。

org.apache.kafka.clients.producer.internals.DefaultPartitioner

### 3.1.6拦截器

Producer拦截器(Interceptor)是个相当新的功能，它和consumer端interceptor是在Kafka 0.10版本被引入的，主要用于实现clients端的定制化控制逻辑。

生产者拦截器可以用在消息发送前做一些准备工作。

**使用场景**

1. 按照某个规则过滤掉不符合要求的信息
2. 修改消息的内容
3. 统计类需求

自定义拦截器需要实现org.apache.kafka.clients.producer.ProducerInterceptor
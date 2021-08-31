# Activiti7概述

https://www.bilibili.com/video/BV1H54y167gf?p=4&spm_id_from=pageDriver

设计器:https://camunda.com/download/modeler/

camunda有个坑点

xml需要对xmlns:camunda做如下替换

```
xmlns:camunda="http://activiti.org/bpmn"
```

网页版设计器:https://github.com/Yiuman/bpmn-vue-activiti



## BPM

​	BPM(Business Process Management), 即业务流程管理，是一种规范化的构造端到端的业务流程，以持续的提高组织业务效率。

## BPMN

​	建模的业务流程符号。

![](https://pic.imgdb.cn/item/6124a3a844eaada73913c92f.jpg)

![](https://pic.imgdb.cn/item/6124a3c844eaada73914165d.jpg)

## 使用步骤

### 部署activiti

​	下载jar包，可以访问activiti的接口。

### 流程定义

​	使用流程建模工具定义业务流程(.bpmn文件)。

### 流程定义部署

​	把xml的内容存储起来，可以查询数据。

### 启动一个流程实例

​	启动一个流程表示开始一次业务流程的运行。

### 用户查询待办任务(Task)

​	业务流程已经交给activiti管理，通过activi就可以查询当前流程执行到哪儿了，当前用户需要办理什么任务了。

### 用户办理任务

​	用户查询待办任务后，就可以办理某个任务，如果这个任务办理完成还需要其他yoghurt办理, 比如采购单创建后由部门经理审核，这个过程由activiti帮我们完成了。

### 流程结束

​	当任务办理完成没有下一个任务节点了，这个流程实例就完成了。

# Activiti环境

## 开发环境

​	jdk1.8或以上版本

​	Mysql5及以上版本

​	Tomcat8.5

​	IDEA

​	activiti的流程定义工具可以安装在IDEA下，也可以安装在Eclipse下。

## Activiti依赖

```xml
比如:在出差申请流程流转时如果出差天数大于3天则由总经理审核，香则由人事直接审核，出差天<properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
        <slf4j.version>1.7.32</slf4j.version>
        <log4j.version>2.14.1</log4j.version>
        <activiti.version>7.1.0.M6</activiti.version>
        <mysql.connector.version>8.0.26</mysql.connector.version>
        <mybatis.plus.version>3.4.3.1</mybatis.plus.version>
    </properties>	
<dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-engine</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-bpmn-model</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-bpmn-converter</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-json-model</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-bpmn-layout</artifactId>
            <version>${activiti.version}</version>
        </dependency>

        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-cloud-services-api</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-cloud-services-api</artifactId>
            <version>${activiti.version}</version>
        </dependency>

        <!-- Mysql -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.connector.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis.plus.version}</version>
        </dependency>

        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
```

### Log4j日志配置

resources下创建log4j.properties

```properties
log4j.rootCategory=debug, CONSOLE, LOGFILE
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r[%15.15t] %-5p %30.30c %x -%m\n
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=./logs/activiti.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r[%15.15t] %-5p %30.30c %x -%m\n
```

### activiti配置文件

​	使用activiti提供的默认方式来创建mysql的表, 在resources下创建activiti.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:conext="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">
</beans>
```

### 在activiti.cfg.xml中进行配置

​	默认方式要在activiti.cfg.xml中bean的名字叫processEnginConfiguration, 名字不可修改。

​	两种配置方式: 单独配置，不配置。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:conext="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">
<!--    默认方式下 不可变-->
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
<!--        配置数据库相关信息-->
        <property name="jdbcDriver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://192.168.170.19:3306/inteli_activiti?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=GMT%2B8&amp;nullCatalogMeansCurrent=true"></property>
        <property name="jdbcUsername" value="root"/>
        <property name="jdbcPassword" value="hzjy123"/>
<!--        activiti表生成策略 true 存在就使用 不存在创建-->
        <property name="databaseSchemaUpdate" value="true"/>
    </bean>
</beans>
```

### 通过测试类创建

```java
class ActivitiDemoApplicationTests {
    /***
     * 默认创建的方式来创建表
     */
    @Test
    public void testCreateDbTable() {
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    }

}
```

### 数据表介绍

https://www.codenong.com/cs109679091/

| 表分类       | 表名                  | 解释                                                     |
| ------------ | --------------------- | -------------------------------------------------------- |
| 一般数据     |                       |                                                          |
|              | act_ge_bytearray      | 二进制表，存储通用的流程资源                             |
|              | act_ge_property       | 系统存储表，存储整个流程引擎数据，默认存储三条数据       |
| 流程历史数据 |                       |                                                          |
|              | act_hi_actinst        | 历史节点表                                               |
|              | act_hi_attachment     | 历史附件表                                               |
|              | act_hi_comment        | 历史意见表                                               |
|              | act_hi_detail         | 历史的流程运行中的细节信息                               |
|              | act_hi_identitylink   | 历史的流程运行中用户关系                                 |
|              | act_hi_procinst       | 历史的流程实例                                           |
|              | act_hi_taskinst       | 历史的任务实例                                           |
|              | act_hi_varinst        | 历史变量表                                               |
|              | act_evt_log           | 流程引擎通用日志表                                       |
| 流程定义表   |                       |                                                          |
|              | act_re_deployment     | 部署信息表                                               |
|              | act_re_model          | 流程设计实体表                                           |
|              | act_re_procdef        | 流程定义数据表                                           |
|              | act_procdef_info      | 流程定义的动态变更信息(在Activiti5.20版本之前没有这张表) |
| 运行实例表   |                       |                                                          |
|              | act_ru_deadletter_job | 作业死信表-作业失败超过重试次数                          |
|              | act_ru_event_subscr   | 运行时事件                                               |
|              | act_ru_execution      | 运行时流程执行实例                                       |
|              | act_ru_identitylink   | 运行时用户关系信息，存储任务节点与参与者的相关信息       |
|              | act_ru_integration    | 运行时综合表                                             |
|              | act_ru_job            | 运行时作业                                               |
|              | act_ru_suspended_job  | 运行时作业暂停表 6.0新增                                 |
|              | act_ru_task           | 运行时任务                                               |
|              | act_ru_timer_job      | 运行时定时器作业表                                       |
|              | act_ru_variable       | 运行时变量表                                             |

# Activiti类关系图

## 类关系图

![](https://pic.imgdb.cn/item/6124c17b44eaada73960b1df.jpg)

## activiti.cfg.xml

​	activi的引擎配置文件，包括: ProcessEngineConfiguration的定义，数据源定义，事务管理器等。*<u>其实就是一个Spring配置文件。</u>*

## 流程引擎类配置

### StandaloneProcessEngineConfiguration

​	使用StandaloneProcessEngineConfiguration， Activiti可以单独运行，来创建ProcessEngin，Activiti会自己处理事务。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:conext="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">
<!--    默认方式下 不可变-->
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
<!--        配置数据库相关信息-->
        <property name="jdbcDriver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://192.168.170.19:3306/inteli_activiti?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=GMT%2B8&amp;nullCatalogMeansCurrent=true"></property>
        <property name="jdbcUsername" value="root"/>
        <property name="jdbcPassword" value="hzjy123"/>
<!--        activiti表生成策略 true 存在就使用 不存在创建-->
        <property name="databaseSchemaUpdate" value="true"/>
    </bean>
</beans>
```

还可以加入连接池

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:conext="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

<!--    方式2 连接池-->
    <bean id = "dataSource" class = "com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://192.168.170.19:3306/inteli_activiti?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=GMT%2B8&amp;nullCatalogMeansCurrent=true"/>
        <property name="username" value="root"/>
        <property name="password" value="hzjy123"/>
        <property name="maxActive" value="3"/>
        <property name="maxIdle" value="1"/>
    </bean>
    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
        <property name="dataSource" ref="dataSource"/>
        <property name="databaseSchemaUpdate" value="true"/>
    </bean>
</beans>
```

### SpringProcessEngineConfiguration

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:conext="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-3.1.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-3.1.xsd">
    <bean id = "processEnginConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
<!--       数据源-->
        <property name="dataSource" ref = "dataSource"/>
<!--        使用Spring事务管理器-->
        <property name="transactionManager" ref = "transactionManager"/>
<!--        数据库策略-->
        <property name="databaseSchemaUpdate" value="true"/>
<!--        Activiti定时任务关闭-->
        <property name="jobExecutorActivate" value="false"/>
    </bean>
<!--    流程引擎-->
    <bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
        <property name="processEngineConfiguration" ref="processEnginConfiguration"/>
    </bean>
<!--    资源服务service-->
    <bean id = "repositoryService" factory-bean="processEngine"
          factory-method="getRepositoryService"/>
<!--    流程运行Service-->
    <bean id = "runtimeService" factory-bean="processEngine"
          factory-method="getRuntimeService"/>
    <!--    任务管理Service-->
    <bean id = "taskService" factory-bean="processEngine"
          factory-method="getTaskService"/>
    <!--    历史管理Service-->
    <bean id = "historyService" factory-bean="processEngine"
          factory-method="getHistoryService"/>
<!--    引擎管理service-->
    <bean id = "managementService" factory-bean="processEngine"
          factory-method="getManagementService"/>

    <bean id = "dynamicBpmnService" factory-bean="processEngine"
          factory-method="getDynamicBpmnService"/>
<!-- 数据源-->
    <bean id = "dataSource" class = "com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://192.168.170.19:3306/inteli_activiti?useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false&amp;serverTimezone=GMT%2B8&amp;nullCatalogMeansCurrent=true"/>
        <property name="username" value="root"/>
        <property name="password" value="hzjy123"/>
        <property name="maxActive" value="3"/>
        <property name="maxIdle" value="1"/>
    </bean>
    <bean id = "transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
<!--    通知-->
    <tx:advice id = "txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
<!--            传播行为-->
            <tx:method name="save" propagation="REQUIRED"/>
            <tx:method name="insert" propagation="REQUIRED"/>
            <tx:method name="delete" propagation="REQUIRED"/>
            <tx:method name="update" propagation="REQUIRED"/>
            <tx:method name="find" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="ge t" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
<!--    切面，根据具体项目配置-->
    <aop:config proxy-target-class="true">
        <aop:advisor advice-ref="txAdvice" pointcut="execution(*
        top.sorie.activitidemo.service.impl.*.*(..))"/>
    </aop:config>
</beans>
```

## 工作流引擎创建

### 默认创建方式

```java
ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
```

### 一般创建方式

```java
ProcessEngineConfiguration configuration = ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(
                "activiti.cfg.xml",
                "processEngeneConfiguration"
        );
ProcessEngine engine = configuration.buildProcessEngine();
```

### Service创建方式

```java
RuntimeService runtimeService = engine.getRuntimeService();
RepositoryService repositoryService = engine.getRepositoryService();
TaskService taskService = engine.getTaskService();
```

### Service总览

| 名称              | 作用                     |
| ----------------- | ------------------------ |
| RepositoryService | activiti的资源管理类     |
| RuntimeService    | activiti的流程运行管理类 |
| TaskService       | activiti的任务管理类     |
| HistoryService    | activiti的历史管理类     |
| ManagerService    | activiti的引擎管理类     |

![](https://pic.imgdb.cn/item/6125977544eaada739bfef57.jpg)

![](https://pic.imgdb.cn/item/6125978844eaada739c004d8.jpg)

# Activiti入门操作

​	创建Activiti工作流主要包含以下几步

* 定义流程: 按照BPMN的规范，使用流程定义工具，使用流程符号把整个流程描述出来。
* 部署流程: 把画好的流程定义文件，加载到数据库中，生成表的数据。
* 启动流程: 使用Java代码操作数据库表的内容

## 流程符号

​	BPMN 2.0是业务流程建模符号2.0的缩写。

​	它由Business Process Management Initiative这个非营利协会创建并不断发展。作为一种标识，BPMN 2.0是使用一些符号来明确业务流程设计流程图的一整天符号规范，它能增进业务建模时的沟通效率。

​	目前BPMN2.0是最新的版本，它用于在BPM上下文中进行布局和可视化的沟通。

​	接下来我们先来了解在流程设计中常见的符号。

​	BPMN2.0的基本符号主要包含:

### 事件 Event

![](https://pic.imgdb.cn/item/6125988244eaada739c11445.jpg)

### 活动Activiti

​	活动是工作或任务的一个通用术语。-一个活动可以是一个任务， 还可以是一个当前流程的子处理流程;其次，你还可以为活动指定不同的类型。常见活动如下:

![](https://pic.imgdb.cn/item/612598a844eaada739c14275.jpg)

### 网关 GateWay

​	网关用来处理决策，有几种常用网关需要链接

![](https://pic.imgdb.cn/item/61259b6444eaada739c47df1.jpg)

#### 排他网关(x)

​	只有一条路径会被选择。流程执行到该网关时，按照输出流的顺序逐个计算，当条件的计算结果为true时, 继续执行当前网关的输出流;

​	如果多条线路计算结果都是true,则会执行第一个值为 true的线路。如果所有网关计算结果没有true,则引擎会抛出异常。

​	排他网关需要和条件顺序流结合使用，default 属性指定默认顺序流，当所有的条件不满足时会执行默认顺序流。

#### 并行网关(+)

​	所有路径会 被同时选择。

​	拆分--并行执行所有输出顺序流， 为每一条 顺序流创建一个并行执行线路。

​	合并--所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。

#### 包容网关(+)

​	可以同时执行多条线路，也可以在网关上设置条件。

​	拆分--计算每条线路上的表达式，当表达式计算结果为true时，创建一个并行线路并继续执行。

​	合并--所有从并行网关拆分并执行完成的线路均在此等候，直到所有的线路都执行完成才继续向下执行。

#### 事件网关(+)

​	专门为中间捕获事件设置的，允许设置多个输出流指向多个不同的中间捕获事件。当流程执行到事件网关后，流程处于等待状态，需要等待抛出事件才能将等待状态转换为活动状态。

### 流向 Flow

​	流是连接两个流程节，点的连线。常见的流向包含以下几种:

![](https://pic.imgdb.cn/item/61259c9e44eaada739c622c3.jpg)

## 流程设计器使用

### Activiti-Designer使用

### Palette(画板)

​	在idea中安装插件即可使用，画板中包括以下结点:

* Connection-连接
* Event--事件
* Task--任务
* Gateway--网关
* Container- - 容器
* Boundary event-边界事件
* Intermediate event- -中间事件
* 流程图设计完毕保存生成bpmn文件



idea 安装插件 Activiti BPMN visualizer

resource创建bpmn目录

右键 new 

![](https://pic.imgdb.cn/item/6125a27244eaada739d1529f.jpg)

然后再右键创建的文件

![](https://pic.imgdb.cn/item/6125a29e44eaada739d1b28e.jpg)

然后编辑图，右键图创建元素

![](https://pic.imgdb.cn/item/6125a42b44eaada739d54977.jpg)

最后通过点击上方的连接箭头连接元素

![](https://pic.imgdb.cn/item/6125a45844eaada739d5b4ec.jpg)

# 流程定义

## 流程定义

### 概述

​	流程定义是线下按照bpmn2.0标准去描述业务流程,通常使用ldea中的插件对业务流程进行建模。

​	使用ldea 下的designer设计器绘制流程，并会生成两个文件: .bpmn和.png

### .bpmn文件

​	使用actlt-desinger设计业务流程，会生成bpmn文件，上面我们已经创建好了bpmn文件。

​	BPMN 2.0根节点是definitions节点。这个元素中， 可以定义多个流程定义(不过我们建议每个文件只包含- -个流程定义，可以简化开发过程中的维护难度) 。注意, definitions元素 最少也要包含xmIns和targetNamespace的声明。targetNamespace可以是任意值， 它用来对流程实例进行分类。

​	流程定义部分:定义了流程每个结点的描述及结点之间的流程流转。

​	流程布局定义:定义流程每个结点在流程图上的位置坐标等信息。

### 生成.png图片文件

​	不知道为啥插件的导出无效。

​	IDEA工具中的操作方式

​	![](https://pic.imgdb.cn/item/6125a6cc44eaada739db33ea.jpg)



![](https://pic.imgdb.cn/item/6125a6e544eaada739db8ff3.jpg)

**如果有中文乱码**

![](https://pic.imgdb.cn/item/6125a70544eaada739dbedf2.jpg)

![](https://pic.imgdb.cn/item/6125a72444eaada739dc45dd.jpg)

![](https://pic.imgdb.cn/item/6125a73944eaada739dc78f1.jpg)

![](https://pic.imgdb.cn/item/6125a75f44eaada739dcd3dc.jpg)

### 部署

```java
@Test
public void testDeploy() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RepositoryService
    RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
    // 使用service进行流程部署, 定义一个流程名字, 把bpmn文件和png部署到数据库中。
    Deployment deployment = repositoryService.createDeployment()
            .name("出差申请流程")
            .addClasspathResource("bpmn/test1.bpmn20.xml")
            .addClasspathResource("bpmn/test1.bpmn20.png")
            .deploy();
    // 输出部署信息
    System.out.println("流程部署id=" + deployment.getId());
    System.out.println("流程部署名字=" + deployment.getName());
}
```

操作的数据库

* ACT_GE_PROPERTY
* ACT_RE_PROCDEF
* ACT_RE_DEPLOYMENT
* ACT_GE_BYTEARRAY

### 流程实例启动

```java
public void testStartProcess() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RuntimeService
    RuntimeService runtimeService = defaultProcessEngine.getRuntimeService();
    // 根据id启动流程
    ProcessInstance instance = runtimeService.startProcessInstanceByKey("test1");
    // 输出内容
    System.out.println("流程定义id" + instance.getProcessDefinitionId());
    System.out.println("流程实例id" + instance.getId());
    System.out.println("当前活动的id" + instance.getActivityId());
}
```

操作的表

```mysql
update ACT_GE_PROPERTY SET REV_ = ?, VALUE_ = ? where NAME_ = ? and REV_ = ?;
insert into ACT_HI_TASKINST;
insert into ACT_HI_PROCINST;
insert into ACT_HI_ACTINST;
insert into ACT_HI_IDENTITYLINK;
insert into ACT_RU_EXECUTION;
insert into ACT_RU_TASK;
insert into ACT_RU_IDENTITYLINK;
```

### 查询个人待执行任务的表

```java
@Test
public void testFindPersonalTaskList() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取TaskService
    TaskService taskService = defaultProcessEngine.getTaskService();
    // 根据流程key和任务负责人查询任务
    List<Task> list = taskService.createTaskQuery()
            .processDefinitionKey("test1")
            .taskAssignee("zhangsan")
            .list();
    // 输出
    list.forEach((task) -> {
        System.out.println("流程实例id" + task.getProcessInstanceId());
        System.out.println("任务id" + task.getId());
        System.out.println("任务负责人=" + task.getAssignee());
        System.out.println("任务名称" + task.getName());
    });
}
```

```mysql
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'schema.version';
 
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'cfg.execution-related-entities-count';
 
select distinct RES.*
 FROM ACT_RU_TASK RES
 INNER JOIN ACT_RE_PROCDEF D
 ON RES.PROC_DEF_ID_ = D.ID_
 WHERE RES.ASSIGNEE_ = 'zhangsan' and D.KEY_ = 'test1' order by RES.ID_ asc
 LIMIT 2147483647 OFFSET 0;
```

### 完成任务

```java
@Test
public void completeTask() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取TaskService
    TaskService taskService = defaultProcessEngine.getTaskService();
    // 根据id完成任务
    taskService.complete("2505");
}
```

```mysql
--  9  11:47:21.070 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'schema.version';
-- ---------------------------------------------------------------------------------------------------------------------
--  10  11:47:21.089 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'cfg.execution-related-entities-count';
-- ---------------------------------------------------------------------------------------------------------------------
--  11  11:47:21.093 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTask - ==>
select *
 FROM ACT_RU_TASK
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  12  11:47:21.096 [main] DEBUG org.activiti.engine.impl.persistence.entity.VariableInstanceEntityImpl.selectVariablesByTaskId - ==>
select *
 FROM ACT_RU_VARIABLE
 WHERE TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  13  11:47:21.098 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinition - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  14  11:47:21.100 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.selectDeployment - ==>
select *
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  15  11:47:21.102 [main] DEBUG org.activiti.engine.impl.persistence.entity.ResourceEntityImpl.selectResourcesByDeploymentId - ==>
select *
 FROM ACT_GE_BYTEARRAY
 WHERE DEPLOYMENT_ID_ = '1' order by NAME_ asc;
-- ---------------------------------------------------------------------------------------------------------------------
--  16  11:47:21.149 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionByDeploymentAndKey - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE DEPLOYMENT_ID_ = '1' and KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null);
-- ---------------------------------------------------------------------------------------------------------------------
--  17  11:47:21.153 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionInfoEntityImpl.selectProcessDefinitionInfoByProcessDefinitionId - ==>
select *
 FROM ACT_PROCDEF_INFO
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  18  11:47:21.155 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTasksByParentTaskId - ==>
select *
 FROM ACT_RU_TASK
 WHERE PARENT_TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  19  11:47:21.156 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.selectIdentityLinksByTask - ==>
select *
 FROM ACT_RU_IDENTITYLINK
 WHERE TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  20  11:47:21.158 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.selectHistoricTaskInstance - ==>
select *
 FROM ACT_HI_TASKINST
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  21  11:47:21.160 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.selectExecution - ==>
select E.*, S.PROC_INST_ID_ AS PARENT_PROC_INST_ID_
 FROM ACT_RU_EXECUTION E LEFT OUTER JOIN ACT_RU_EXECUTION S
 ON E.SUPER_EXEC_ = S.ID_
 WHERE E.ID_ = '2502';
-- ---------------------------------------------------------------------------------------------------------------------
--  22  11:47:21.164 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTasksByExecutionId - ==>
select distinct T.*
 FROM ACT_RU_TASK T
 WHERE T.EXECUTION_ID_ = '2502';
-- ---------------------------------------------------------------------------------------------------------------------
--  23  11:47:21.167 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.selectExecution - ==>
select E.*, S.PROC_INST_ID_ AS PARENT_PROC_INST_ID_
 FROM ACT_RU_EXECUTION E LEFT OUTER JOIN ACT_RU_EXECUTION S
 ON E.SUPER_EXEC_ = S.ID_
 WHERE E.ID_ = '2501';
-- ---------------------------------------------------------------------------------------------------------------------
--  24  11:47:21.181 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.selectUnfinishedHistoricActivityInstanceExecutionIdAndActivityId - ==>
select *
 FROM ACT_HI_ACTINST RES
 WHERE EXECUTION_ID_ = '2502' and ACT_ID_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018' and END_TIME_ is null;
-- ---------------------------------------------------------------------------------------------------------------------
--  25  11:47:21.193 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'next.dbid';
-- ---------------------------------------------------------------------------------------------------------------------
--  26  11:47:21.195 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.updateProperty - ==>
update ACT_GE_PROPERTY SET REV_ = 4, VALUE_ = '7501'
 WHERE NAME_ = 'next.dbid' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  27  11:47:21.247 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.selectIdentityLinksByProcessInstance - ==>
select *
 FROM ACT_RU_IDENTITYLINK
 WHERE PROC_INST_ID_ = '2501';
-- ---------------------------------------------------------------------------------------------------------------------
--  28  11:47:21.258 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.insertHistoricTaskInstance - ==>
insert into ACT_HI_TASKINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, OWNER_, ASSIGNEE_, START_TIME_, CLAIM_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TASK_DEF_KEY_, FORM_KEY_, PRIORITY_, DUE_DATE_, CATEGORY_, TENANT_ID_ ) values ( '5002', 'test1:1:4', '2501', '2502', '经理审批', null, null, null, 'jerry', '2021-08-25 11:47:21.247', null, null, null, null, 'sid-06afb39c-50d1-4760-ba33-66103b859802', null, 50, null, null, '' );
-- ---------------------------------------------------------------------------------------------------------------------
--  29  11:47:21.260 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.insertHistoricActivityInstance - ==>
insert into ACT_HI_ACTINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, ACT_ID_, TASK_ID_, CALL_PROC_INST_ID_, ACT_NAME_, ACT_TYPE_, ASSIGNEE_, START_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TENANT_ID_ ) values ( '5001', 'test1:1:4', '2501', '2502', 'sid-06afb39c-50d1-4760-ba33-66103b859802', '5002', null, '经理审批', 'userTask', 'jerry', '2021-08-25 11:47:21.235', null, null, null, '' );
-- ---------------------------------------------------------------------------------------------------------------------
--  30  11:47:21.262 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricIdentityLinkEntityImpl.insertHistoricIdentityLink - ==>
insert into ACT_HI_IDENTITYLINK (ID_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_) values ('5003', 'participant', 'jerry', null, null, '2501');
-- ---------------------------------------------------------------------------------------------------------------------
--  31  11:47:21.264 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.insertTask - ==>
insert into ACT_RU_TASK (ID_, REV_, NAME_, BUSINESS_KEY_, PARENT_TASK_ID_, DESCRIPTION_, PRIORITY_, CREATE_TIME_, OWNER_, ASSIGNEE_, DELEGATION_, EXECUTION_ID_, PROC_INST_ID_, PROC_DEF_ID_, TASK_DEF_KEY_, DUE_DATE_, CATEGORY_, SUSPENSION_STATE_, TENANT_ID_, FORM_KEY_, CLAIM_TIME_, APP_VERSION_) values ('5002', 1, '经理审批', null, null, null, 50, '2021-08-25 11:47:21.236', null, 'jerry', null, '2502', '2501', 'test1:1:4', 'sid-06afb39c-50d1-4760-ba33-66103b859802', null, null, 1, '', null, null, null );
-- ---------------------------------------------------------------------------------------------------------------------
--  32  11:47:21.309 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.insertIdentityLink - ==>
insert into ACT_RU_IDENTITYLINK (ID_, REV_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_, PROC_DEF_ID_) values ('5003', 1, 'participant', 'jerry', null, null, '2501', null);
-- ---------------------------------------------------------------------------------------------------------------------
--  33  11:47:21.357 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 2, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:1:4', ACT_ID_ = 'sid-06afb39c-50d1-4760-ba33-66103b859802', IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = false, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = '2501', SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '2501', SUSPENSION_STATE_ = 1, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '2502' and REV_ = 1;
-- ---------------------------------------------------------------------------------------------------------------------
--  34  11:47:21.410 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.updateHistoricActivityInstance - ==>
update ACT_HI_ACTINST set EXECUTION_ID_ = '2502', ASSIGNEE_ = 'zhangsan', END_TIME_ = '2021-08-25 11:47:21.184', DURATION_ = 1558184, DELETE_REASON_ = null
 WHERE ID_ = '2504';
-- ---------------------------------------------------------------------------------------------------------------------
--  35  11:47:21.411 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.updateHistoricTaskInstance - ==>
update ACT_HI_TASKINST set PROC_DEF_ID_ = 'test1:1:4', EXECUTION_ID_ = '2502', NAME_ = '创建出差申请', PARENT_TASK_ID_ = null, DESCRIPTION_ = null, OWNER_ = null, ASSIGNEE_ = 'zhangsan', CLAIM_TIME_ = null, END_TIME_ = '2021-08-25 11:47:21.16', DURATION_ = 1558160, DELETE_REASON_ = null, TASK_DEF_KEY_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018', FORM_KEY_ = null, PRIORITY_ = 50, DUE_DATE_ = null, CATEGORY_ = null
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  36  11:47:21.413 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.deleteTask - ==>
delete
 FROM ACT_RU_TASK
 WHERE ID_ = '2505' and REV_ = 1;
-- ---------------------------------------------------------------------------------------------------------------------
--  37  11:47:21.070 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'schema.version';
-- ---------------------------------------------------------------------------------------------------------------------
--  38  11:47:21.089 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'cfg.execution-related-entities-count';
-- ---------------------------------------------------------------------------------------------------------------------
--  39  11:47:21.093 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTask - ==>
select *
 FROM ACT_RU_TASK
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  40  11:47:21.096 [main] DEBUG org.activiti.engine.impl.persistence.entity.VariableInstanceEntityImpl.selectVariablesByTaskId - ==>
select *
 FROM ACT_RU_VARIABLE
 WHERE TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  41  11:47:21.098 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinition - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  42  11:47:21.100 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.selectDeployment - ==>
select *
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  43  11:47:21.102 [main] DEBUG org.activiti.engine.impl.persistence.entity.ResourceEntityImpl.selectResourcesByDeploymentId - ==>
select *
 FROM ACT_GE_BYTEARRAY
 WHERE DEPLOYMENT_ID_ = '1' order by NAME_ asc;
-- ---------------------------------------------------------------------------------------------------------------------
--  44  11:47:21.149 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionByDeploymentAndKey - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE DEPLOYMENT_ID_ = '1' and KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null);
-- ---------------------------------------------------------------------------------------------------------------------
--  45  11:47:21.153 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionInfoEntityImpl.selectProcessDefinitionInfoByProcessDefinitionId - ==>
select *
 FROM ACT_PROCDEF_INFO
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  46  11:47:21.155 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTasksByParentTaskId - ==>
select *
 FROM ACT_RU_TASK
 WHERE PARENT_TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  47  11:47:21.156 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.selectIdentityLinksByTask - ==>
select *
 FROM ACT_RU_IDENTITYLINK
 WHERE TASK_ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  48  11:47:21.158 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.selectHistoricTaskInstance - ==>
select *
 FROM ACT_HI_TASKINST
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  49  11:47:21.160 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.selectExecution - ==>
select E.*, S.PROC_INST_ID_ AS PARENT_PROC_INST_ID_
 FROM ACT_RU_EXECUTION E LEFT OUTER JOIN ACT_RU_EXECUTION S
 ON E.SUPER_EXEC_ = S.ID_
 WHERE E.ID_ = '2502';
-- ---------------------------------------------------------------------------------------------------------------------
--  50  11:47:21.164 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.selectTasksByExecutionId - ==>
select distinct T.*
 FROM ACT_RU_TASK T
 WHERE T.EXECUTION_ID_ = '2502';
-- ---------------------------------------------------------------------------------------------------------------------
--  51  11:47:21.167 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.selectExecution - ==>
select E.*, S.PROC_INST_ID_ AS PARENT_PROC_INST_ID_
 FROM ACT_RU_EXECUTION E LEFT OUTER JOIN ACT_RU_EXECUTION S
 ON E.SUPER_EXEC_ = S.ID_
 WHERE E.ID_ = '2501';
-- ---------------------------------------------------------------------------------------------------------------------
--  52  11:47:21.181 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.selectUnfinishedHistoricActivityInstanceExecutionIdAndActivityId - ==>
select *
 FROM ACT_HI_ACTINST RES
 WHERE EXECUTION_ID_ = '2502' and ACT_ID_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018' and END_TIME_ is null;
-- ---------------------------------------------------------------------------------------------------------------------
--  53  11:47:21.193 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'next.dbid';
-- ---------------------------------------------------------------------------------------------------------------------
--  54  11:47:21.195 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.updateProperty - ==>
update ACT_GE_PROPERTY SET REV_ = 4, VALUE_ = '7501'
 WHERE NAME_ = 'next.dbid' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  55  11:47:21.247 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.selectIdentityLinksByProcessInstance - ==>
select *
 FROM ACT_RU_IDENTITYLINK
 WHERE PROC_INST_ID_ = '2501';
-- ---------------------------------------------------------------------------------------------------------------------
--  56  11:47:21.258 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.insertHistoricTaskInstance - ==>
insert into ACT_HI_TASKINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, NAME_, PARENT_TASK_ID_, DESCRIPTION_, OWNER_, ASSIGNEE_, START_TIME_, CLAIM_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TASK_DEF_KEY_, FORM_KEY_, PRIORITY_, DUE_DATE_, CATEGORY_, TENANT_ID_ ) values ( '5002', 'test1:1:4', '2501', '2502', '经理审批', null, null, null, 'jerry', '2021-08-25 11:47:21.247', null, null, null, null, 'sid-06afb39c-50d1-4760-ba33-66103b859802', null, 50, null, null, '' );
-- ---------------------------------------------------------------------------------------------------------------------
--  57  11:47:21.260 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.insertHistoricActivityInstance - ==>
insert into ACT_HI_ACTINST ( ID_, PROC_DEF_ID_, PROC_INST_ID_, EXECUTION_ID_, ACT_ID_, TASK_ID_, CALL_PROC_INST_ID_, ACT_NAME_, ACT_TYPE_, ASSIGNEE_, START_TIME_, END_TIME_, DURATION_, DELETE_REASON_, TENANT_ID_ ) values ( '5001', 'test1:1:4', '2501', '2502', 'sid-06afb39c-50d1-4760-ba33-66103b859802', '5002', null, '经理审批', 'userTask', 'jerry', '2021-08-25 11:47:21.235', null, null, null, '' );
-- ---------------------------------------------------------------------------------------------------------------------
--  58  11:47:21.262 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricIdentityLinkEntityImpl.insertHistoricIdentityLink - ==>
insert into ACT_HI_IDENTITYLINK (ID_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_) values ('5003', 'participant', 'jerry', null, null, '2501');
-- ---------------------------------------------------------------------------------------------------------------------
--  59  11:47:21.264 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.insertTask - ==>
insert into ACT_RU_TASK (ID_, REV_, NAME_, BUSINESS_KEY_, PARENT_TASK_ID_, DESCRIPTION_, PRIORITY_, CREATE_TIME_, OWNER_, ASSIGNEE_, DELEGATION_, EXECUTION_ID_, PROC_INST_ID_, PROC_DEF_ID_, TASK_DEF_KEY_, DUE_DATE_, CATEGORY_, SUSPENSION_STATE_, TENANT_ID_, FORM_KEY_, CLAIM_TIME_, APP_VERSION_) values ('5002', 1, '经理审批', null, null, null, 50, '2021-08-25 11:47:21.236', null, 'jerry', null, '2502', '2501', 'test1:1:4', 'sid-06afb39c-50d1-4760-ba33-66103b859802', null, null, 1, '', null, null, null );
-- ---------------------------------------------------------------------------------------------------------------------
--  60  11:47:21.309 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.insertIdentityLink - ==>
insert into ACT_RU_IDENTITYLINK (ID_, REV_, TYPE_, USER_ID_, GROUP_ID_, TASK_ID_, PROC_INST_ID_, PROC_DEF_ID_) values ('5003', 1, 'participant', 'jerry', null, null, '2501', null);
-- ---------------------------------------------------------------------------------------------------------------------
--  61  11:47:21.357 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 2, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:1:4', ACT_ID_ = 'sid-06afb39c-50d1-4760-ba33-66103b859802', IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = false, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = '2501', SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '2501', SUSPENSION_STATE_ = 1, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '2502' and REV_ = 1;
-- ---------------------------------------------------------------------------------------------------------------------
--  62  11:47:21.410 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricActivityInstanceEntityImpl.updateHistoricActivityInstance - ==>
update ACT_HI_ACTINST set EXECUTION_ID_ = '2502', ASSIGNEE_ = 'zhangsan', END_TIME_ = '2021-08-25 11:47:21.184', DURATION_ = 1558184, DELETE_REASON_ = null
 WHERE ID_ = '2504';
-- ---------------------------------------------------------------------------------------------------------------------
--  63  11:47:21.411 [main] DEBUG org.activiti.engine.impl.persistence.entity.HistoricTaskInstanceEntityImpl.updateHistoricTaskInstance - ==>
update ACT_HI_TASKINST set PROC_DEF_ID_ = 'test1:1:4', EXECUTION_ID_ = '2502', NAME_ = '创建出差申请', PARENT_TASK_ID_ = null, DESCRIPTION_ = null, OWNER_ = null, ASSIGNEE_ = 'zhangsan', CLAIM_TIME_ = null, END_TIME_ = '2021-08-25 11:47:21.16', DURATION_ = 1558160, DELETE_REASON_ = null, TASK_DEF_KEY_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018', FORM_KEY_ = null, PRIORITY_ = 50, DUE_DATE_ = null, CATEGORY_ = null
 WHERE ID_ = '2505';
-- ---------------------------------------------------------------------------------------------------------------------
--  64  11:47:21.413 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.deleteTask - ==>
delete
 FROM ACT_RU_TASK
 WHERE ID_ = '2505' and REV_ = 1;
-- ---------------------------------------------------------------------------------------------------------------------
```

完成最后的流程后，task表会删除对应数据。

## 用Zip方法进行部署

```java
public void testDeployByZip() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RepositoryService
    RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
    // 使用service进行流程部署, 定义一个流程名字, 把bpmn文件和png部署到数据库中。
    InputStream inputStream = this.getClass().getClassLoader()
            .getResourceAsStream("bpmn/evection.zip");
    ZipInputStream zipInputStream = new ZipInputStream(inputStream);
    Deployment deployment = repositoryService.createDeployment()
            .addZipInputStream(zipInputStream)
            .deploy();
    System.out.println("流程部署id=" + deployment.getId());
    System.out.println("流程部署名字=" + deployment.getName());
}
```

## 流程定义查询

```java
public void queryProcessDefinition() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // RepositoryService
    RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
    // 获取ProcessDefinitionQuery对象 查询当前所有流程定义
    List<ProcessDefinition> definitions = repositoryService.createProcessDefinitionQuery()
            .processDefinitionKey("test1")
            .orderByProcessDefinitionVersion()
            .desc()
            .list();
    // 输出信息
    definitions.forEach((p) -> {
        System.out.println("流程定义id：" + p.getId());
        System.out.println("流程定义名称：" + p.getName());
        System.out.println("流程定义key：" + p.getKey());
        System.out.println("流程定义版本：" + p.getVersion());
    });
}
```

## 流程部署删除

```java
public void deleteDeployment() {
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // RepositoryService
        RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
        // 通过部署id来删除流程信息
        String deploymentId = "1";
        repositoryService.deleteDeployment(deploymentId);
    }
```

```mysql
--  364  14:12:58.107 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'schema.version';
-- ---------------------------------------------------------------------------------------------------------------------
--  365  14:12:58.125 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'cfg.execution-related-entities-count';
-- ---------------------------------------------------------------------------------------------------------------------
--  366  14:12:58.128 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.selectDeployment - ==>
select *
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  367  14:12:58.150 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_PROCDEF RES
 WHERE RES.DEPLOYMENT_ID_ = '1' order by RES.ID_ asc
 LIMIT 2147483647 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  368  14:12:58.154 [main] DEBUG org.activiti.engine.impl.persistence.entity.ModelEntityImpl.selectModelsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_MODEL RES
 WHERE RES.DEPLOYMENT_ID_ = '1' order by RES.ID_ asc
 LIMIT 2147483647 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  369  14:12:58.155 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionInfoEntityImpl.selectProcessDefinitionInfoByProcessDefinitionId - ==>
select *
 FROM ACT_PROCDEF_INFO
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  370  14:12:58.159 [main] DEBUG org.activiti.engine.impl.persistence.entity.TimerJobEntityImpl.selectTimerJobByTypeAndProcessDefinitionId - ==>
select J.*
 FROM ACT_RU_TIMER_JOB J
 WHERE J.HANDLER_TYPE_ = 'timer-start-event' and J.PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  371  14:12:58.160 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectLatestProcessDefinitionByKey - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null) and VERSION_ = (select max(VERSION_)
 FROM ACT_RE_PROCDEF
 WHERE KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null));
-- ---------------------------------------------------------------------------------------------------------------------
--  372  14:12:58.162 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_PROCDEF RES
 WHERE RES.KEY_ = 'test1' and RES.VERSION_ < 1 and (RES.TENANT_ID_ = '' or RES.TENANT_ID_ is null) order by RES.VERSION_ desc
 LIMIT 1 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  373  14:12:58.165 [main] DEBUG org.activiti.engine.impl.persistence.entity.ResourceEntityImpl.deleteResourcesByDeploymentId - ==>
delete
 FROM ACT_GE_BYTEARRAY
 WHERE DEPLOYMENT_ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  374  14:12:58.167 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.deleteDeployment - ==>
delete
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  375  14:12:58.232 [main] DEBUG org.activiti.engine.impl.persistence.entity.EventSubscriptionEntityImpl.deleteEventSubscriptionsForProcessDefinition - ==>
delete
 FROM ACT_RU_EVENT_SUBSCR
 WHERE PROC_DEF_ID_ = 'test1:1:4' and EXECUTION_ID_ is null and PROC_INST_ID_ is null;
-- ---------------------------------------------------------------------------------------------------------------------
--  376  14:12:58.233 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.deleteIdentityLinkByProcDef - ==>
delete
 FROM ACT_RU_IDENTITYLINK
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  377  14:12:58.234 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.deleteProcessDefinitionsByDeploymentId - ==>
delete
 FROM ACT_RE_PROCDEF
 WHERE DEPLOYMENT_ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  378  14:12:58.107 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'schema.version';
-- ---------------------------------------------------------------------------------------------------------------------
--  379  14:12:58.125 [main] DEBUG org.activiti.engine.impl.persistence.entity.PropertyEntityImpl.selectProperty - ==>
select *
 FROM ACT_GE_PROPERTY
 WHERE NAME_ = 'cfg.execution-related-entities-count';
-- ---------------------------------------------------------------------------------------------------------------------
--  380  14:12:58.128 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.selectDeployment - ==>
select *
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  381  14:12:58.150 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_PROCDEF RES
 WHERE RES.DEPLOYMENT_ID_ = '1' order by RES.ID_ asc
 LIMIT 2147483647 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  382  14:12:58.154 [main] DEBUG org.activiti.engine.impl.persistence.entity.ModelEntityImpl.selectModelsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_MODEL RES
 WHERE RES.DEPLOYMENT_ID_ = '1' order by RES.ID_ asc
 LIMIT 2147483647 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  383  14:12:58.155 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionInfoEntityImpl.selectProcessDefinitionInfoByProcessDefinitionId - ==>
select *
 FROM ACT_PROCDEF_INFO
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  384  14:12:58.159 [main] DEBUG org.activiti.engine.impl.persistence.entity.TimerJobEntityImpl.selectTimerJobByTypeAndProcessDefinitionId - ==>
select J.*
 FROM ACT_RU_TIMER_JOB J
 WHERE J.HANDLER_TYPE_ = 'timer-start-event' and J.PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  385  14:12:58.160 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectLatestProcessDefinitionByKey - ==>
select *
 FROM ACT_RE_PROCDEF
 WHERE KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null) and VERSION_ = (select max(VERSION_)
 FROM ACT_RE_PROCDEF
 WHERE KEY_ = 'test1' and (TENANT_ID_ = '' or TENANT_ID_ is null));
-- ---------------------------------------------------------------------------------------------------------------------
--  386  14:12:58.162 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.selectProcessDefinitionsByQueryCriteria - ==>
select distinct RES.*
 FROM ACT_RE_PROCDEF RES
 WHERE RES.KEY_ = 'test1' and RES.VERSION_ < 1 and (RES.TENANT_ID_ = '' or RES.TENANT_ID_ is null) order by RES.VERSION_ desc
 LIMIT 1 OFFSET 0;
-- ---------------------------------------------------------------------------------------------------------------------
--  387  14:12:58.165 [main] DEBUG org.activiti.engine.impl.persistence.entity.ResourceEntityImpl.deleteResourcesByDeploymentId - ==>
delete
 FROM ACT_GE_BYTEARRAY
 WHERE DEPLOYMENT_ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  388  14:12:58.167 [main] DEBUG org.activiti.engine.impl.persistence.entity.DeploymentEntityImpl.deleteDeployment - ==>
delete
 FROM ACT_RE_DEPLOYMENT
 WHERE ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
--  389  14:12:58.232 [main] DEBUG org.activiti.engine.impl.persistence.entity.EventSubscriptionEntityImpl.deleteEventSubscriptionsForProcessDefinition - ==>
delete
 FROM ACT_RU_EVENT_SUBSCR
 WHERE PROC_DEF_ID_ = 'test1:1:4' and EXECUTION_ID_ is null and PROC_INST_ID_ is null;
-- ---------------------------------------------------------------------------------------------------------------------
--  390  14:12:58.233 [main] DEBUG org.activiti.engine.impl.persistence.entity.IdentityLinkEntityImpl.deleteIdentityLinkByProcDef - ==>
delete
 FROM ACT_RU_IDENTITYLINK
 WHERE PROC_DEF_ID_ = 'test1:1:4';
-- ---------------------------------------------------------------------------------------------------------------------
--  391  14:12:58.234 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.deleteProcessDefinitionsByDeploymentId - ==>
delete
 FROM ACT_RE_PROCDEF
 WHERE DEPLOYMENT_ID_ = '1';
-- ---------------------------------------------------------------------------------------------------------------------
```

如果需要强制删除，需要通过级联删除

```java
repositoryService.deleteDeployment(deploymentId, true);
```

## 流程资源下载

​	现在我们的流程资源文件已经上传到数据库了，如果其他用户想要查看这些资源文件，可以从数据库中把资源文件下载到本地。

​	解决方案有:

* jdbcx对blob类型， clob类型数据读取出来,保存到文件目录。
* 2.使用activitl的apl来实现

​	使用commons-io.jar解决io的操作。

​	引入commons-io依赖包

```java
public void testDownloadResource() throws IOException {
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // RepositoryService
        RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
        // 获取查询对象 ProcessDefinitionQuery，查询定义
        ProcessDefinitionQuery query = repositoryService.createProcessDefinitionQuery();
        // 通过流程定义信息, 获取部署ID
        ProcessDefinition definition = repositoryService.createProcessDefinitionQuery()
                .processDefinitionKey("test1")
                .singleResult();
        // 通过RepositoryService，传递部署id参数，获取资源信息
        // --获取png图片的流 为啥是null 注意png文件不能有多个点
        // https://github.com/Activiti/Activiti/issues/3714
        System.out.println(definition.getDiagramResourceName());
        InputStream pngResourceAsStream = repositoryService.getResourceAsStream(
                definition.getDeploymentId(), definition.getDiagramResourceName());
        // --获取bpmn的流
        System.out.println(definition.getResourceName());
        InputStream bpmnResourceAsStream = repositoryService.getResourceAsStream(
                definition.getDeploymentId(), definition.getResourceName());
        // 构造Outputstream流
        File pngFile = new File("D:/temp/evectionFlow01.png");
        File bpmn = new File("D:/temp/evectionFlow01.bpmn20.xml");
        FileOutputStream pngFileOutputSteam = new FileOutputStream(pngFile);
        FileOutputStream bpmnFileOutputSteam = new FileOutputStream(bpmn);

        // 输入流，输出流的转换
        IOUtils.copy(pngResourceAsStream, pngFileOutputSteam);
        IOUtils.copy(bpmnResourceAsStream, bpmnFileOutputSteam);
        // 关闭流
        IOUtils.close(pngResourceAsStream);
        IOUtils.close(bpmnResourceAsStream);
        IOUtils.close(pngFileOutputSteam);
        IOUtils.close(bpmnFileOutputSteam);
}
```

## 历史查询

```java
public void queryHistoryInfo() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // HistoryService
    HistoryService historyService = defaultProcessEngine.getHistoryService();
    HistoricActivityInstanceQuery historicActivityInstanceQuery =
            historyService.createHistoricActivityInstanceQuery();
    // 查询
    List<HistoricActivityInstance> list = historicActivityInstanceQuery.processInstanceId("15001").list();
    list.forEach((p) -> {
        System.out.println(p.getActivityId());
        System.out.println(p.getActivityName());
        System.out.println(p.getProcessDefinitionId());
        System.out.println(p.getProcessInstanceId());
        System.out.println("============================");
    });
}
```

输出

```
myEvection
null
test1:2:7504
15001
============================
sid-7248fded-0e06-4920-9ffd-92c3d7351018
创建出差申请
test1:2:7504
15001
============================
```

# Activiti进阶

## 流程实例

### 什么是流程实例

​	**流程实例(ProcessInstance)** 代表流程定义的执行实例。

​	一个流程实例包括了所有的运行节点。我们可以利用这个对象来了解当前流程实例的进度等信息。

​	例如:用户或程序按照流程定义内容发起一个流程，这就是一个流程实例。

​	流程定义和流程实例的图解:

![](https://pic.imgdb.cn/item/6125f1f244eaada739a91483.jpg)

### 启动流程实例 并添加Businesskey(业务标识)

![](https://pic.imgdb.cn/item/6125f26944eaada739aa8cb1.jpg)

![](https://pic.imgdb.cn/item/6125f31c44eaada739acc183.jpg)

```java
public void addBusinessKey() {
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // 获取RuntimeService
        RuntimeService runtimeService = defaultProcessEngine.getRuntimeService();
        // 根据id启动流程
        ProcessInstance instance = runtimeService.startProcessInstanceByKey("test1", "2");
        // 输出内容
        System.out.println("流程定义id" + instance.getProcessDefinitionId());
        System.out.println("流程实例id" + instance.getId());
        System.out.println("当前活动的id" + instance.getActivityId());
        System.out.println("business key=" + instance.getBusinessKey());
}
```

Activiti的act_ ru_ execution中存储业务标识:

![](https://pic.imgdb.cn/item/6125f47344eaada739b11998.jpg)

### 挂起、激活流程实例

​	某些情况可能由于流程变更需要将当前运行的流程暂停而不是直接删除，流程暂停后将不会继续执行。

#### 所有流程实例挂起和激活

​	每月最后一天不处理出差申请。

```java
public void suspendAllProcessInstance() {
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // 获取RepositoryService
        RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
        // 查询流程定义
        ProcessDefinition test1 = repositoryService.createProcessDefinitionQuery()
                .processDefinitionKey("test1")
                .singleResult();
        // 获取当前流程实例是否都是挂起状态
        boolean suspended = test1.isSuspended();
        String id = test1.getId();
        // 如果是挂起，就激活
        // 如果是正常，改为挂起
        if (suspended) {
            // 参数1: 流程定义id 参数2: 是否激活 参数3 激活时间
            repositoryService.activateProcessDefinitionById(id, true, null);
            System.out.println("激活");
        } else {
            // 参数1: 流程定义id 参数2: 是否暂停 参数3 暂停时间
            repositoryService.suspendProcessDefinitionById(id, true, null);
            System.out.println("挂起");
        }
    }
```

```mysql
-- ---------------------------------------------------------------------------------------------------------------------
--  1230  16:21:37.307 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.updateTask - ==>
update ACT_RU_TASK SET REV_ = 4, NAME_ = '经理审批', BUSINESS_KEY_ = '1001', PARENT_TASK_ID_ = null, PRIORITY_ = 50, CREATE_TIME_ = '2021-08-25 16:18:13.0', OWNER_ = null, ASSIGNEE_ = 'jerry', DELEGATION_ = null, EXECUTION_ID_ = '17502', PROC_DEF_ID_ = 'test1:2:7504', DESCRIPTION_ = null, DUE_DATE_ = null, CATEGORY_ = null, SUSPENSION_STATE_ = 2, FORM_KEY_ = null, CLAIM_TIME_ = null
 WHERE ID_= '20002' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  1231  16:21:37.310 [main] DEBUG org.activiti.engine.impl.persistence.entity.TaskEntityImpl.updateTask - ==>
update ACT_RU_TASK SET REV_ = 4, NAME_ = '创建出差申请', BUSINESS_KEY_ = null, PARENT_TASK_ID_ = null, PRIORITY_ = 50, CREATE_TIME_ = '2021-08-25 15:12:05.0', OWNER_ = null, ASSIGNEE_ = 'zhangsan', DELEGATION_ = null, EXECUTION_ID_ = '15002', PROC_DEF_ID_ = 'test1:2:7504', DESCRIPTION_ = null, DUE_DATE_ = null, CATEGORY_ = null, SUSPENSION_STATE_ = 2, FORM_KEY_ = null, CLAIM_TIME_ = null
 WHERE ID_= '15005' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  1232  16:21:37.312 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 4, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018', IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = false, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = '15001', SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '15001', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '15002' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  1233  16:21:37.314 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 4, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = null, IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = true, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = null, SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '15001', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '15001' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  1234  16:21:37.316 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 12, BUSINESS_KEY_ = '1001', PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = null, IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = true, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = null, SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '17501', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '17501' and REV_ = 11;
-- ---------------------------------------------------------------------------------------------------------------------
--  1235  16:21:37.317 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 13, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = 'sid-06afb39c-50d1-4760-ba33-66103b859802', IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = false, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = '17501', SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '17501', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '17502' and REV_ = 12;
-- ---------------------------------------------------------------------------------------------------------------------
--  1236  16:21:37.319 [main] DEBUG org.activiti.engine.impl.persistence.entity.ProcessDefinitionEntityImpl.updateProcessDefinition - ==>
update ACT_RE_PROCDEF set REV_ = 8, SUSPENSION_STATE_ = 2, CATEGORY_ = 'http://www.activiti.org/processdef'
 WHERE ID_ = 'test1:2:7504' and REV_ = 7;
-- ---------------------------------------------------------------------------------------------------------------------
```

#### 单个流程实例挂起和激活

```java
public void suspendProcessInstance() {
    // 创建engine
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    // 获取RuntimeService
    RuntimeService runtimeService = defaultProcessEngine.getRuntimeService();
    // 获取实例对象
    ProcessInstance instance = runtimeService.createProcessInstanceQuery()
            .processInstanceId("17501")
            .singleResult();
    // 切换状态
    if (instance.isSuspended()) {
        runtimeService.activateProcessInstanceById(instance.getId());
        System.out.println("激活");
    } else {
        runtimeService.suspendProcessInstanceById(instance.getId());
        System.out.println("挂起");
    }
}
```

```mysql
update ACT_RU_TASK SET REV_ = 4, NAME_ = '创建出差申请', BUSINESS_KEY_ = '1001', PARENT_TASK_ID_ = null, PRIORITY_ = 50, CREATE_TIME_ = '2021-08-25 15:40:20.0', OWNER_ = null, ASSIGNEE_ = 'zhangsan', DELEGATION_ = null, EXECUTION_ID_ = '17502', PROC_DEF_ID_ = 'test1:2:7504', DESCRIPTION_ = null, DUE_DATE_ = null, CATEGORY_ = null, SUSPENSION_STATE_ = 2, FORM_KEY_ = null, CLAIM_TIME_ = null
 WHERE ID_= '17505' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  852  16:11:47.013 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 4, BUSINESS_KEY_ = '1001', PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = null, IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = true, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = null, SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '17501', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '17501' and REV_ = 3;
-- ---------------------------------------------------------------------------------------------------------------------
--  853  16:11:47.015 [main] DEBUG org.activiti.engine.impl.persistence.entity.ExecutionEntityImpl.updateExecution - ==>
update ACT_RU_EXECUTION set REV_ = 4, BUSINESS_KEY_ = null, PROC_DEF_ID_ = 'test1:2:7504', ACT_ID_ = 'sid-7248fded-0e06-4920-9ffd-92c3d7351018', IS_ACTIVE_ = true, IS_CONCURRENT_ = false, IS_SCOPE_ = false, IS_EVENT_SCOPE_ = false, IS_MI_ROOT_ = false, PARENT_ID_ = '17501', SUPER_EXEC_ = null, ROOT_PROC_INST_ID_ = '17501', SUSPENSION_STATE_ = 2, NAME_ = null, IS_COUNT_ENABLED_ = false, EVT_SUBSCR_COUNT_ = 0, TASK_COUNT_ = 0, JOB_COUNT_ = 0, TIMER_JOB_COUNT_ = 0, SUSP_JOB_COUNT_ = 0, DEADLETTER_JOB_COUNT_ = 0, VAR_COUNT_ = 0, ID_LINK_COUNT_ = 0, APP_VERSION_ = null
 WHERE ID_ = '17502' and REV_ = 3;
```

#### 挂起后完成任务

```java
public void completeTask() {
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // 获取TaskService
        TaskService taskService = defaultProcessEngine.getTaskService();
        // 根据id完成任务
//        taskService.complete("2505");
        Task task = taskService.createTaskQuery()
                .processDefinitionKey("test1")
                .taskAssignee("zhangsan")
                .processInstanceBusinessKey("1001")
                .singleResult();

        System.out.println("流程实例id" + task.getProcessInstanceId());
        System.out.println("任务id" + task.getId());
        System.out.println("任务负责人=" + task.getAssignee());
        System.out.println("任务名称" + task.getName());
        taskService.complete(task.getId());
    }
```



```
org.activiti.engine.ActivitiException: Cannot complete a suspended task
	at org.activiti.engine.impl.cmd.NeedsActiveTaskCmd.execute(NeedsActiveTaskCmd.java:53)
	at org.activiti.engine.impl.interceptor.CommandInvoker$1.run(CommandInvoker.java:37)
	at org.activiti.engine.impl.interceptor.CommandInvoker.executeOperation(CommandInvoker.java:78)

```

## 个人任务

### 分配任务负责人

#### 固定分配

![](https://pic.imgdb.cn/item/6125fec844eaada739d1836f.jpg)

#### 表达式分配

​	由于固定分配方式，任务只管-步-步执行任务,执行到每一个任务将按照bpmn的配置去分配任务负责人。

##### UEL 表达式

​	Activiti使用UEL表达式，UEL是java EE6规范的一部分，UEL (Unified Expression Language) 即统一表达式语言，activiti 支持两个UEL表达式: UEL-value 和UEL-method。

**1. UEL-value定义**

![](https://pic.imgdb.cn/item/6125ff8044eaada739d3ca8d.jpg)

**2.UEL-method定义**

![](https://pic.imgdb.cn/item/6125ffb044eaada739d4637a.jpg)

**3. UEL-method与UEL-value结合**

​	再比如: 

​	${ldapSeric.findManagerForEmployee(emp)}

​	IdapService是spring容器的一个bean, findManagerForEmployee 是该bean的一个方法，emp是activiti流程变量，emp 作为参数传到IdapServcefindManagerForEmployee方法中。

**4. 其它**

​	表达式支持解析基础类型、bean、 list array 和map,也可作为条件判断。

​	如下:

​	${order .price > 100 && order.price < 250}

##### 编写代码配置负责人

**1. 定义任务分配流程变量**

设置流程负责人为变量

![](https://pic.imgdb.cn/item/6126e0b044eaada739a98bc3.jpg)

```java
@Test
public void testDeploy() {
    ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
    RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
    Deployment deployment = repositoryService.createDeployment()
            .name("出差申请-uel")
            .addClasspathResource("bpmn/test2.bpmn20.xml")
            .addClasspathResource("bpmn/test2.png")
            .deploy();
    // 输出部署信息
    System.out.println("deploymentId=" + deployment.getId());
    System.out.println("deploymentName=" + deployment.getName());
}

@Test
public void testAssigneeUel() {
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 启动流程实例
    Map<String, Object> uelParam = new HashMap<>();
    uelParam.put("assignee0", "张三");
    uelParam.put("assignee1", "李经理");
    uelParam.put("assignee2", "王总经理");
    uelParam.put("assignee3", "赵财务");
    ProcessInstance instance = runtimeService.startProcessInstanceByKey("test2", uelParam);
}
```

执行成功后，可以在act_ru_variable项表中看到刚才map中的数据:

![](https://pic.imgdb.cn/item/6126e10044eaada739aa0900.jpg)

### 使用监听器分配

注意:

https://blog.csdn.net/weixin_43844343/article/details/84777768

之前的不支持, 切换编辑器comuda:

https://blog.csdn.net/qq_25701293/article/details/98846694

坑点xml需要对xmlns:camunda做如下替换

```
xmlns:camunda="http://activiti.org/bpmn"
```

​	

可以使用监听器来完成很多Activit流程的业务。

​	在本章我们使用监听器的方式来指定负责人。那么在流程设计时就不需要指定assignee。

​	任务监听器是发生对应的任务相关事件时执行自定义java逻辑或表达式。

​	任务相当事件包括:

![](https://pic.imgdb.cn/item/6126e1c644eaada739aaf5e9.jpg)

TaskEvent包含

* Create: 任务创建后触发。
* Assignment: 任务分配后触发。
* Delete: 任务删除后触发。
* All: 所有事件发生都触发(暂时没看到这个。
* Complete: 完成后触发。
* timeout: 超时触发

![](https://pic.imgdb.cn/item/612709b844eaada739ed87dc.jpg)





```java
public class MyTaskListener implements TaskListener {
    @Override
    public void notify(DelegateTask delegateTask) {
        if ("创建出差申请".equals(delegateTask.getName())) {
            delegateTask.setAssignee("张三");
        }
    }
}
```

```java
public class ActivitiTestTaskListener {

    @Test
    public void testDeploy() {
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        RepositoryService repositoryService = defaultProcessEngine.getRepositoryService();
        Deployment deployment = repositoryService.createDeployment()
                .name("出差申请-task")
                .addClasspathResource("bpmn/test3.bpmn20.xml")
                .deploy();
        // 输出部署信息
        System.out.println("deploymentId=" + deployment.getId());
        System.out.println("deploymentName=" + deployment.getName());
    }


    @Test
    public void testAssigneeListener() {
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        // 启动流程实例
        ProcessInstance instance = runtimeService.startProcessInstanceByKey("test3");
    }
}
```

## 流程变量

### 什么是流程变量

​	流程变量在activiti 中是一个非常重 要的角色，流程运转有时需要靠流程变量,业务系统和activiti结合时少不了流程变量，流程变量就是activiti 在管理工作流时根据管理需要而设置的变量。

​	比如:在出差申请流程流转时如果出差天数大于3天则由总经理审核，香则由人事直接审核，出差天数就可以设置为流程变量，在流程流转时使用。

​	注意:虽然流程变量中可以存储业务数据可以通过activiti的api查询流程变量从而实现查询业务数据，但是不建议这样使用，因为业务数据查询由业务系统负责, activiti设置流程变显是为 了流程执行需要而创建。

### 流程变量类型

​	如果将polo存储到流程变量中，必须实现序列化接口serlalzable, 为了防止由于新增字段无去反序列化，需要生成serialVersionUID。

![](https://pic.imgdb.cn/item/61270f2144eaada739f74a1b.jpg)

### 流程变量作用域

​	流程变量的作用域可以是一个流程实例(processInstance), 或一个任务(task)， 或-个执行实例(execution)

#### global变量

​	流程变量的默认作用域是流程实例。当-个流程变量的作用域为流程实例时，可以称为global变量

​	注意:

​	如: Global变量: userld (变量名)、zhangsan (变量值)

​	global变量中变量名不允许重复,设置相同名称的变量，后设置的值会覆盖前设置的变量值。

#### local变量

​	任务和执行或例仅仅是针对一个任务和一 个执行实例范围，范围没有流程实例大，称为local变量。

​	Local变量由于在不同的任务或不同的执行实例中，作用域互不影响，变量名可以相同没有影响。Local 变最名也可以和global变量名相同，没有影响。

### 流程变量使用方法

#### 在属性上使用UEL表达式

​	可以在assignee处设置UEL表达式,表达式的值为任务的负责人，比如: ${assignee}， assignee 就是一-个流程变量名称。

​	Activiti获取UEL表达式的值，即流程变显assignee的值，将assignee的值作为任务的负责人进行任务分配。

#### 在连线上使用UEL表达式

​	可以在连线上设置UEL表达式，决定流程走向。

​	比如: ${price<10000} 。price就是一个流程变量名称，uel表达式结果类型为布尔类型。

​	如果UEL表达式是true，要决定流程执行走向。

### 使用Global变量控制流程

#### 需求

​	员工创建出差申请单，由部门经理审核，部门经理审核通过后出差3天及一下由财务直接审批，3天以上先由总经理审核，总经理审核通过再由财务审批。

![](https://pic.imgdb.cn/item/6127118544eaada739fb77e2.jpg)

#### 流程定义

1. 出差天数大于等于3连线条件

![](https://pic.imgdb.cn/item/6127278044eaada73929a622.jpg)

#### 启动流程时设置变量

```java
public void testStartProcessSetVariables() {
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    RuntimeService runtimeService = processEngine.getRuntimeService();
    // 启动流程实例
    Map<String, Object> uelParam = new HashMap<>();
    uelParam.put("days", 3);
    ProcessInstance instance = runtimeService.startProcessInstanceByKey("test4", uelParam);
}
```

![](https://pic.imgdb.cn/item/612729b444eaada7392e598c.jpg)

完成任务

![](https://pic.imgdb.cn/item/612731e744eaada73941abc3.jpg)

#### 任务办理时设置变量

```java
public void testCompleteTask() {
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 创建engine
        ProcessEngine defaultProcessEngine = ProcessEngines.getDefaultProcessEngine();
        // 获取TaskService
        TaskService taskService = defaultProcessEngine.getTaskService();
        // 根据id完成任务
//        taskService.complete("2505");
        Task task = taskService.createTaskQuery()
                .processDefinitionKey("test4")
                .taskAssignee("王五")
                .singleResult();
        if (task != null) {
            System.out.println("流程实例id" + task.getProcessInstanceId());
            System.out.println("任务id" + task.getId());
            System.out.println("任务负责人=" + task.getAssignee());
            System.out.println("任务名称" + task.getName());
            Map<String, Object> uelParam = new HashMap<>();
            uelParam.put("days", 3);
            taskService.complete(task.getId(), uelParam);
        }
    }
```

#### 通过当前流程实例设置

通过流程实例id设置全局变量，该流程实例必须未执行完成。

```java
public void setGlobalVariableByExecutionId() {
        String executionId = "40002";
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        runtimeService.setVariable(executionId, "days", 3);
        runtimeService.setVariables(executionId, new HashMap<>());
}
```

​	executionld必须当前未结束流程实例的执行id,通常此id设置流程实例的id。也可以通runtimeService.getVariable()获取流程变量。

#### 通过当前任务设置

```java
@Test
public void setGlobalVariableByTaskId() {
    String taskId = "40005";
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    taskService.setVariable(taskId, "days", 3);
    taskService.setVariables(taskId, new HashMap<>());
}
```

​	任务id必须是当前待办任务id, act. ru. _task中存在。如果该任务已结束，会报错。

​	也可以通过taskService.getVariable)获取流程变星。

#### 注意事项

* 如果UEL表达式中流程变量名不存在则报错。
* 如果UEL表达式中流程变量值为空NULL，流程不按UEL表达式去执行，而流程结束。
* 如果UEL表达式都不符合条件,流程结束。
* 如果连线不设置条件，会走flow序号小的那条线。

#### 操作数据库表

​	设置流程变星会在当前执行流程变量表插入记录，同时也会在历史流程变量表也插入记录。

```mysql
# 当前流程变量表
SELECT * FROM act.ru_variable
```

​	记录当前运行流程实例可使用的流程变量，包括global和local变量

* ld_:主键
* Type_:变量类型
* Name_ :变量名称
* Execution_id_:所属流程实例执行id, global和local变量都存储
* Task_id_: 所属任务id, local变量存储
* Bytearray_ : serializable类型变 量存储对应act_ge_bytearray表的id。

### 设置local流程变量

#### 任务办理时设置

​	任务办理时设置local|流程变量，当前运行的流程实例只能在该任务结束前使用，任务结束该变量无法在当前流程实例使用，可以通过查询历史任务查询。

```java
@Test
public void setLocalVariableByTaskId() {
    String taskId = "40005";
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    taskService.setVariableLocal(taskId, "days", 3);
    taskService.setVariablesLocal(taskId, new HashMap<>());
}
```

​	任务id必须是当前待办任务id, act_ru_task中存在。

#### 通过实例执行

​	对于runtimeService来说 setVariableLocal和没有setVariable没有区别。

```
runtimeService.setVariableLocal();
runtimeService.setVariablesLocal();
```

#### 注意事项

* Local变量在任务结束后无法在当前流程实例执行中使用，如果后续的流程执行需要用到此变星则会报错。

* 在部门经理审核、总经理审核、财务审核时设置local变量，可通过historyService查询每 个历史任务时将流程变量的值也查询出来。

![](https://pic.imgdb.cn/item/612739ad44eaada73955d706.jpg)

## 组任务

### 需求

​	在流程定义中在任务结点的assignee固定设置任务负责人，在流程定义时将参与者固定设置在.bpmn文件中，如果临时任务负责人变更则需要修改流程定义，系统可扩展性差。

​	针对这种情况可以给任务设置多个候选人，可以从候选人中选择参与者来完成任务。

### 设置任务候选人

​	在流程图中任务节点的配置中设置candidate-users(候选人),多个候选人之间用逗号分开。

![](https://pic.imgdb.cn/item/61273ae044eaada739588560.jpg)

### 组任务

**查询组任务**

​	指定候选人，查询该候选人当前的待办任务。

​	候选人不能立即办理任务。

**拾取(claim)任务**

​	该组任务的所有候选人都能拾取。

​	将候选人的组任务，变成个人任务。原来候选人就变成了该任务的负责人。

​	如果拾取后不想办理该任务?

​	需要将已经拾取的个人任务归还到组里边，将个人任务变成了组任务。

**查询个人任务**

​	查询方式同个人任务部分，根据assignee查询用户负责的个人任务。

**办理个人任务**

#### 查询组任务

```java
public void testFindGroupTaskList() {
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    String candicateUser = "王五";
    // 启动流程实例

    List<Task> list = taskService.createTaskQuery()
            .processDefinitionKey(testKey)
            .taskCandidateUser(candicateUser)
            .list();
    for (Task task : list) {
        System.out.println("id:" + task.getId());
        System.out.println("负责人" + task.getAssignee());
    }
}
```

#### 拾取任务

```java
public void testClaimTask() {
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    TaskService taskService = processEngine.getTaskService();
    String candicateUser = "王五";
    String taskId = "40005";
    // 启动流程实例
    Task task = taskService.createTaskQuery()
            .taskId(taskId)
            .taskCandidateUser(candicateUser)
            .singleResult();
    if (task != null) {
        taskService.claim(taskId, candicateUser);
        System.out.println("拾取");
    }
}
```

#### 归还任务

```java
@Test
    public void tesReturnTask() {
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        TaskService taskService = processEngine.getTaskService();
        String assigne = "王五";
        String taskId = "40005";
        // 启动流程实例
        Task task = taskService.createTaskQuery()
                .taskId(taskId)
                .taskAssignee(assigne)
                .singleResult();
        if (task != null) {
//            taskService.setAssignee(taskId, null);
            taskService.unclaim(taskId);
            System.out.println("归还");
        }
    }
```

#### 交接任务

修改负责人即可。

## 网关

​	网关用来控制流程的流向

### 排他网关ExclusiveGateway

#### 什么是排他网关:

​	排他网关，用来在流程中实现决策。当流程执行到这个网关， 所有分支都会判断条件是否为true,如果为true则执行该分支。	

​	注意:排他网关只会选择一个 为true的分支执行。如果有两个分支条件都为true,排他网关会选择ld值较小的一条分支去执行。

​	为什么要用排他网关?

​	不用排他网关也可以实现分支，如:在连线的condition条件上设置分支条件。	

​	在连线设置condition条件的缺点:如果条件都不满足，流程就结束了(是异常结束)。

​	如果使用排他网关决定分支的走向，如下:

![](https://pic.imgdb.cn/item/612740aa44eaada73966d468.jpg)

​	如果从网关出去的线所有条件都不满足则系统抛出异常。

![](https://pic.imgdb.cn/item/6127412d44eaada739681f3f.jpg)

#### 流程定义

![](https://pic.imgdb.cn/item/6127414c44eaada73968712a.jpg)

### 并行网关 Parallel Gateway

​	并行网关允许将流程分成多条分支，也可以把多条分支汇聚到-起， 并行网关的功能是基于进入和外出顺序流的:

fork分支:

​	并行后的所有外出顺序流，为每个顺序流都创建一个并发分支。

join汇聚:

​	所有到达并行网关，在此等待的进入分支，直到所有进 入顺序流的分支都到达以后，流程就会通过汇聚网关。

​	注意,如果同一个并行网关有多个进入和多个外出顺序流，它就同时具有分支和汇聚功能。 这时，网关会先汇聚所有进入的顺序流，然后再切分成多个并行分支。



​	**与其他网关的主要区别是，并行网关不会解析条件。即使顺序流中定义了条件, 也会被忽略。**

例子:

![](https://pic.imgdb.cn/item/6127423244eaada7396abf84.jpg)

#### 流程定义



![](https://pic.imgdb.cn/item/6127427f44eaada7396b76da.jpg)

### 包含网关 Inclusive Gateway

​	包含网关可以看做是排他网关和并行网关的结合体。和排他网关一样, 你可以在外出顺序流上定义条件,包含网关会解析它们。但是主要的区别是包含网关可以选择多于一条顺序流，这和并行网关一样。

​	包含网关的功能是基于进入和外出顺序流的:

分支:

​	所有外出顺序流的条件都会被解析，结果为true的顺序流会以并行方式继续执行，会为每个顺序流创建一个分支。

汇聚:

​	所有并行分支到达包含网关，会进入等待状态，直到每个包含流程token的进入顺序流的分支都到达。这是与并行网关的最大不同。换句话说，包含网关只会等待被选中执行了的进入顺序流。在汇 聚之后，流程会穿过包含网关继续执行。

#### 流程定义

​	出差申请大于等于3天需要由项目经理审批，小于3天由技术经理审批，其中任意-人审批完成后流程向下流转。

​	包含网关图标，红框内:

![](https://pic.imgdb.cn/item/6127435544eaada7396dceb0.jpg)

![](https://pic.imgdb.cn/item/6127436844eaada7396e3423.jpg)

### 事件网关 Event Based Gateway

复制于: https://docs.awspaas.com/reference-guide/aws-paas-process-gateway-reference-guide/event-based_gateway/README.html

可以参考: https://www.cnblogs.com/dengjiahai/p/9147404.html

​	通常网关根据连线条件来决定后继路径，这就要求条件信息必须存在于流程自身之中。但是，当需要选择的后继路径的条件不能来自该流程时，就可以使用事件网关。事件网关只有分支行为，允许从多个候选分支中选择事件最先到达的分支（如时间事件、消息事件），并取消其他分支。

​	当候选分支的某个事件到达时，若取消其他分支的事件任务（标记为Cancel）发生异常或事件自身的处理发生异常，该事件的任务实例将被处理成错误（标记为Error），路径将中断于此。可以在AWS控制台的“运行”模块检查和管理这些出错任务。

![](https://pic.imgdb.cn/item/6127446744eaada73970cdeb.jpg)

​	事件网关之后的连线（Sequence Flow）目标必须是一个“中间捕获事件”，事件网关支持以下类型的“中间捕获事件”，而关于如何使用这些捕获类事件请参见相关文档。

![](https://pic.imgdb.cn/item/6127448d44eaada739712bfb.jpg)

### 复杂网关 Complex Gateway

https://docs.awspaas.com/reference-guide/aws-paas-process-gateway-reference-guide/complex_gateway/README.html

## 子流程

https://blog.csdn.net/zhuchunyan_aijia/article/details/101310253

# Activiti整合Spring开发

## 整合Spring

略

## 整合Spring boot

```xml
<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    <activiti.version>7.1.0.M6</activiti.version>
    <mysql.connector.version>8.0.26</mysql.connector.version>
    <mybatis.plus.version>3.4.3.1</mybatis.plus.version>
    <druid.version>1.2.5</druid.version>
    <lombok.version>1.18.20</lombok.version>
    <knife4j.version>3.0.3</knife4j.version>
    <gson.version>2.8.7</gson.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.activiti/activiti-spring-boot-starter -->
    <dependency>
        <groupId>org.activiti</groupId>
        <artifactId>activiti-spring-boot-starter</artifactId>
        <version>${activiti.version}</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.connector.version}</version>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>${mybatis.plus.version}</version>
    </dependency>
    <!-- Druid数据库连接池 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>${druid.version}</version>
    </dependency>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>${gson.version}</version>
    </dependency>
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>knife4j-spring-boot-starter</artifactId>
        <version>${knife4j.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <scope>provided</scope>
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
```

application.yml

```yml
# 应用名称
spring:
  application:
    name: activiti-boot-demo
  datasource:
    url: jdbc:mysql://192.168.170.19:3306/inteli-activiti?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT%2B8
    username: root
    password: hzjy123
    type: com.alibaba.druid.pool.DruidDataSource
    driverClassName: com.mysql.cj.jdbc.Driver
    # Druid StatViewServlet配置
    druid:
      stat-view-servlet:
        # 默认true 内置监控页面首页/druid/index.html
        enabled: true
        url-pattern: /druid/*
        # 允许清空统计数据
        reset-enable: true
        login-username: root
        login-password: hzjy123
        # IP白名单 多个逗号分隔
        allow:
        # IP黑名单
        deny:
      filter:
        stat:
          # 开启监控sql
          enabled: true
          # 显示并标注慢sql 默认当超过3秒显示
          log-slow-sql: true
          slow-sql-millis: 3000
          merge-sql: true
        # 防SQL注入过滤
        wall:
          config:
            # 允许多条sql同时执行
            multi-statement-allow: true
    jackson:
      time-zone: GMT+8
      serialization:
        fail-on-empty-beans: false
  activiti:
    #1. flase:默认值。activiti在启动时，对比数据库表中保存的版本，
    #   如果没有表或者版本不匹配，将抛出异常
    #2. true: agtiviti会对数据库中所有表进行更新操作。如果表不存在，则自动创建
    #3. create_drop: #3.create_ _drop: 在activit1启动时创建表， 在关闭时删除表(必须手动关闭引擎，才能删除表)
    #4. drop-create:在activit启动时删除原 来的旧表，然后在创建新表(不需要手动关闭引擎)
    database-schema-update: true
    # 生成历史表
    db-history-used: true
    # 历史记录存储登记
    # 记录历史等级可配置的历史级别有none, activity, audit, fu1l
    #none:不保存任何的历史数据，因此，在流程执行过程中，这是最高效的。
    #activity: 级别高于none， 保存流程实例与流程行为，其他数据不保存。
    #audit:除activity级 别会保存的数据外，还会你存全部的流程任务及其属性。audi t为history的默认值。
    #fu1l:保存历史数据的最高级别，除了会保存audit级别的数据外，还会保存其他全部流程相关的细节数据，包括一些流程参 数等。
    history-level: full
    #校验流程文件，默认校验resources下的processes文件夹里的流程文件
    check-process-definitions: false
```

config

```java
@Slf4j
@Configuration
public class DemoConfig {

    @Bean
    public UserDetailsService myUserDetailsService() {
        InMemoryUserDetailsManager inMemoryUserDetailsManager = new InMemoryUserDetailsManager();
        String[][] usersGroupAndRoles = {
                { "jack", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                { "rose", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                { "tom", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                { "jerry", "password", "ROLE_ACTIVITI_USER", "GROUP_activitiTeam"},
                { "other", "password", "ROLE_ACTIVITI_USER", "GROUP_otherTeam"},
                { "system", "password", "ROLE_ACTIVITI_USER"},
                { "admin", "password", "ROLE_ACTIVITI_ADMIN"},
                { "zhangsan", "password", "ROLE_ACTIVITI_ADMIN"},
        };
        Arrays.stream(usersGroupAndRoles).forEach(p -> {
            List<String> authorites = Arrays.asList(Arrays.copyOfRange(p, 2, p.length));
            inMemoryUserDetailsManager.createUser(new User(p[0], passwordEncoder().encode(p[1]),
                    authorites.stream().map(str -> new SimpleGrantedAuthority(str)).collect(Collectors.toList())
            ));
        });

        return inMemoryUserDetailsManager;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

#### 创建Bpmn文件

​	在resources目录下，创建-个新的目录processes, 用来放置bpmn文件。

​	创建一个简单的Bpmn流程文件，并设置任务的用户组Candidate Groups。

​	Candidate Groups中的内容与上面DemoApplicationConfiguration类中出现的用户组名称要保持一致，可以填写: activitiTeam 或者otherTeam。

​	这样填写的好处:当不确定到底由谁来负责当前任务的时候，只是Groups内的用户都可以拾取这个任务。

不知道为啥 camunda画的图starter用不了

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:activiti="http://activiti.org/bpmn" xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC" xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" typeLanguage="http://www.w3.org/2001/XMLSchema" expressionLanguage="http://www.w3.org/1999/XPath" targetNamespace="http://www.activiti.org/processdef">
  <process id="test1" name="test1" isExecutable="true">
    <startEvent id="myEvection"/>
    <userTask id="sid-7248fded-0e06-4920-9ffd-92c3d7351018" name="创建出差申请" activiti:assignee="zhangsan"/>
    <userTask id="sid-06afb39c-50d1-4760-ba33-66103b859802" name="经理审批" activiti:assignee="jerry"/>
    <userTask id="sid-f8b200e8-d9c8-4288-8f56-217aa37a5f4d" name="总经理审批" activiti:assignee="jack"/>
    <userTask id="sid-d2650341-5692-41ba-b983-432d2bdac191" name="财务审批" activiti:assignee="rose"/>
    <endEvent id="sid-14d50490-007f-4844-a75f-33428775d005"/>
    <sequenceFlow id="sid-b8c6ddb6-12d1-452f-b0b6-3e8acc5dcc83" sourceRef="myEvection" targetRef="sid-7248fded-0e06-4920-9ffd-92c3d7351018"/>
    <sequenceFlow id="sid-cd068a55-d624-42f2-a68a-6421993a83dc" sourceRef="sid-06afb39c-50d1-4760-ba33-66103b859802" targetRef="sid-f8b200e8-d9c8-4288-8f56-217aa37a5f4d"/>
    <sequenceFlow id="sid-0c877962-5937-4eee-b2f5-e58272a16d87" sourceRef="sid-7248fded-0e06-4920-9ffd-92c3d7351018" targetRef="sid-06afb39c-50d1-4760-ba33-66103b859802"/>
    <sequenceFlow id="sid-b987e773-be74-4cdd-83ce-8cf94bc500f6" sourceRef="sid-f8b200e8-d9c8-4288-8f56-217aa37a5f4d" targetRef="sid-d2650341-5692-41ba-b983-432d2bdac191"/>
    <sequenceFlow id="sid-57a2ae14-0ef6-4f2b-8d38-fd4d44023228" sourceRef="sid-d2650341-5692-41ba-b983-432d2bdac191" targetRef="sid-14d50490-007f-4844-a75f-33428775d005"/>
  </process>
  <bpmndi:BPMNDiagram id="BPMNDiagram_test1">
    <bpmndi:BPMNPlane bpmnElement="test1" id="BPMNPlane_test1">
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="shape-bcb7480e-bd23-4777-bd9c-5c8098a384c0" bpmnElement="myEvection">
        <omgdc:Bounds x="-108.75" y="-130.908" width="30.0" height="30.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="shape-d2ced46a-514d-487b-a0ab-fb07164e0123" bpmnElement="sid-7248fded-0e06-4920-9ffd-92c3d7351018">
        <omgdc:Bounds x="-143.75" y="-76.0" width="100.0" height="80.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="shape-61266b48-bf1c-48ca-aa1e-c887b97c9a89" bpmnElement="sid-06afb39c-50d1-4760-ba33-66103b859802">
        <omgdc:Bounds x="-143.75" y="35.0" width="100.0" height="80.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="shape-d0f9a23a-3bb2-4d5d-8fa4-df8881dfced5" bpmnElement="sid-f8b200e8-d9c8-4288-8f56-217aa37a5f4d">
        <omgdc:Bounds x="-143.75" y="148.99599" width="100.0" height="80.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="sid-f13f8827-ba71-41f3-95d4-c4bea29bf219" bpmnElement="sid-d2650341-5692-41ba-b983-432d2bdac191">
        <omgdc:Bounds x="-143.75" y="264.10403" width="100.0" height="80.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNShape xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="shape-0dd4a17f-2c20-401d-a233-75b5b2a630f1" bpmnElement="sid-14d50490-007f-4844-a75f-33428775d005">
        <omgdc:Bounds x="-108.75" y="385.10007" width="30.0" height="30.0"/>
      </bpmdi:BPMNShape>
      <bpmdi:BPMNEdge xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="edge-aa74f563-d72b-4129-ad60-8c79ba29ea61" bpmnElement="sid-b8c6ddb6-12d1-452f-b0b6-3e8acc5dcc83">
        <omgdi:waypoint x="-93.75" y="-100.908005"/>
        <omgdi:waypoint x="-93.75" y="-76.0"/>
      </bpmdi:BPMNEdge>
      <bpmdi:BPMNEdge xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="edge-333a3d4e-8d02-4477-ada2-b20087dd354b" bpmnElement="sid-cd068a55-d624-42f2-a68a-6421993a83dc">
        <omgdi:waypoint x="-93.75" y="115.0"/>
        <omgdi:waypoint x="-93.75" y="148.99599"/>
      </bpmdi:BPMNEdge>
      <bpmdi:BPMNEdge xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="edge-22ee7e96-9b2d-4406-83d7-229074e85415" bpmnElement="sid-0c877962-5937-4eee-b2f5-e58272a16d87">
        <omgdi:waypoint x="-93.75" y="4.0"/>
        <omgdi:waypoint x="-93.75" y="35.0"/>
      </bpmdi:BPMNEdge>
      <bpmdi:BPMNEdge xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="edge-7d62aa23-8b2a-45d8-9df0-96f3bb883a79" bpmnElement="sid-b987e773-be74-4cdd-83ce-8cf94bc500f6">
        <omgdi:waypoint x="-93.75" y="228.99599"/>
        <omgdi:waypoint x="-93.75" y="264.10403"/>
      </bpmdi:BPMNEdge>
      <bpmdi:BPMNEdge xmlns:bpmdi="http://www.omg.org/spec/BPMN/20100524/DI" id="edge-76fbba53-8db0-4da6-97a8-5b156c3f3ef1" bpmnElement="sid-57a2ae14-0ef6-4f2b-8d38-fd4d44023228">
        <omgdi:waypoint x="-93.75" y="344.10403"/>
        <omgdi:waypoint x="-93.75" y="385.10007"/>
      </bpmdi:BPMNEdge>
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>

```

```java
@Component
public class SecurityUtil {
    private Logger logger = LoggerFactory.getLogger(SecurityUtil.class);

     @Autowired
     @Qualifier("myUserDetailsService")
     private UserDetailsService userDetailsService;
 
    public void logInAs(String username) {
     UserDetails user = userDetailsService.loadUserByUsername(username);

     if (user == null) {
         throw new IllegalStateException("User " + username + " doesn't exist, please provide a valid user");
     }
     logger.info("> Logged in as: " + username);

     SecurityContextHolder.setContext(
             new SecurityContextImpl(
                     new Authentication() {
                         @Override
                         public Collection<? extends GrantedAuthority> getAuthorities() {
                             return user.getAuthorities();
                         }
                         @Override
                         public Object getCredentials() {
                             return user.getPassword();
                         }
                         @Override
                         public Object getDetails() {
                             return user;
                         }
                         @Override
                         public Object getPrincipal() {
                             return user;
                         }
                         @Override
                         public boolean isAuthenticated() {
                             return true;
                         }
                         @Override
                         public void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException { }
                         @Override
                         public String getName() {
                             return user.getUsername();
                         }
     }));
     org.activiti.engine.impl.identity.Authentication.setAuthenticatedUserId(username);
 }
  }
```

#### 查询定义

```java
/**
 * 前提activiti7可以自动部署流程
 */
@Test
public void findProcess() {
    securityUtil.logInAs("jack");
    Page<ProcessDefinition> processDefinitions =
            processRuntime.
                    processDefinitions(Pageable.of(0, 100));
    System.out.println(processDefinitions.getTotalItems());
    System.out.println(processDefinitions.getContent());
}
```

输出

```
1
[ProcessDefinition{id='test1:1:691c90d2-0651-11ec-a922-70cf496007a2', name='test1', key='test1', description='null', formKey='null', version=1}]
```



#### 启动流程

会报错，在数据库设置定义的appVersion就可以了

```java
public void startProcessRuntime() {
    securityUtil.logInAs("system");
    ProcessInstance test1 = processRuntime
            .start(ProcessPayloadBuilder.start()
                    .withProcessDefinitionKey("test1").build());
}
```

#### 完成任务

```java
@Test 
public void doTask() {
    securityUtil.logInAs("system");
    // 查询
    Page<Task> taskPage = taskRuntime.tasks(Pageable.of(0, 10));
    // 拾取
    for (Task task : taskPage.getContent()) {
        taskRuntime.claim(TaskPayloadBuilder
            .claim()
            .withTaskId(task.getId())
            .build());
        // 完成
        taskRuntime.complete(TaskPayloadBuilder
                .complete()
                .withTaskId(task.getId())
                .build());
    }
}
```

# 实践

* 用户撤回

https://blog.csdn.net/qq_35167373/article/details/90598547


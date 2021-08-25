# Activiti7概述

https://www.bilibili.com/video/BV1H54y167gf?p=4&spm_id_from=pageDriver

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
<properties>
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

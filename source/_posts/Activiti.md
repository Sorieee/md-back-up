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

​	activi的引擎配置文件，包括: ProcessEngineConfiguration的定义，数据源定义，事务管理器等。其实就是一个Spring配置文件。

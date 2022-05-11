from https://www.yiibai.com/maven/

# Maven教程

## Maven是什么？

​	Maven是一个项目管理和综合工具。[Maven](http://www.yiibai.com/maven)提供了开发人员构建一个完整的生命周期框架。开发团队可以自动完成项目的基础工具建设，Maven使用标准的目录结构和默认构建生命周期。

## 安装

略

https://www.yiibai.com/maven/maven_environment_setup.html

# Maven启用代理访问

## 1. Maven配置文件

找到文件` {M2_HOME}/conf/settings.xml`, 并把你的代理服务器信息配置写入。

```xml
<!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>
```

取消注释代理选项，填写您的代理服务器的详细信息。

```sh
<!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
      <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>yiibai</username>
      <password>password</password>
      <host>proxy.yiibai.com</host>
      <port>8888</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
  </proxies>
```

## 2. 保存文件

​	完成后，Apache Maven 应该是能够通过代理服务器立即连接到Internet。

​	**注意：**重新启动不是必需的。Maven 只是一个命令，当你调用它，它会再次读取该文件。

# Maven本地资源库

默认情况下，Maven的本地资源库默认为 .m2 目录文件夹：

Unix/Mac OS X – `~/.m2 `

Windows – `C:\Documents and Settings\{your-username}\.m2`

## 1. 更新Maven的本地库

​	通常情况下，可改变默认的 .m2 目录下的默认本地存储库文件夹到其他更有意义的名称，例如， maven-repo

​	找到 `{M2_HOME}\conf\setting.xml`, 更新 localRepository 到其它名称。

​	`{M2_HOME}\conf\setting.xml`

```xml
<settings><!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ~/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  --><localRepository>D:\software\yiibai.com\apache-maven\repository</localRepository>
```

## 2. 保存文件

​	执行之后，新的 Maven 本地存储库现在改为 D:\software\yiibai.com\apache-maven\repository.

​	执行命令：

```
C:\worksp> mvn archetype:generate -DgroupId=com.yiibai -DartifactId=NumberGenerator -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

​	详见如下图：

![img](https://www.yiibai.com/uploads/tutorial/20151026/1-151026220036208.png)

​	当你建立一个 Maven 的项目，Maven 会检查你的 pom.xml 文件，以确定哪些依赖下载。首先，Maven 将从本地资源库获得 Maven 的本地资源库依赖资源，如果没有找到，然后把它会从默认的 Maven 中央存储库 – [http://repo1.maven.org/maven2/](http://repo1.maven.org/maven/) 查找下载。

​	Maven中心储存库网站已经改版本，目录浏览可能不再使用。这将直接被重定向到 http://search.maven.org/。这就好多了，现在有一个搜索功能：

# 如何从Maven远程存储库下载？

​	在Maven中，当你声明的库不存在于本地存储库中，也没有不存在于Maven中心储存库，该过程将停止并将错误消息输出到 Maven 控制台。

## 1. 示例

​	 org.jvnet.localizer 只适用于 [Java.net资源库](https://maven.java.net/content/repositories/public/) 

​	pom.xml

​	

```xml
<dependency>
        <groupId>org.jvnet.localizer</groupId>
        <artifactId>localizer</artifactId>
        <version>1.8</version>
</dependency>
```

​	2. 声明Java.net储存库

​	告诉 Maven 来获得 Java.net 的依赖，你需要声明远程仓库在 pom.xml 文件这样：

​	pom.xml

```xml
 <repositories>
	<repository>
	    <id>java.net</id>
	    <url>https://maven.java.net/content/repositories/public/</url>
	</repository>
    </repositories>
```

​	现在，Maven的依赖库查询顺序更改为：

1. 在 Maven 本地资源库中搜索，如果没有找到，进入第 2 步，否则退出。 
2. 在 Maven 中央存储库搜索，如果没有找到，进入第 3 步，否则退出。 
3. 在java.net Maven的远程存储库搜索，如果没有找到，提示错误信息，否则退出。

# Maven添加远程仓库

默认情况下，Maven从Maven中央仓库下载所有依赖关系。但是，有些库丢失在中央存储库，只有在Java.net或JBoss的储存库远程仓库中能找到。 

## 1. Java.net资源库 

添加Java.net远程仓库的详细信息在“pom.xml”文件。 

pom.xml 

```xml
<project ...>
<repositories>
    <repository>
      <id>java.net</id>
      <url>https://maven.java.net/content/repositories/public/</url>
    </repository>
 </repositories>
</project>
```

**注**
 旧的 “*http://download.java.net/maven/2*” 仍然可用, 但建议升级到最新储存库。 

## 2. JBoss Maven存储库 

​		1. 添加JBoss远程仓库的详细信息在 “pom.xml” 文件中。 

​		pom.xml 

```xml
<project ...>
    <repositories>
      <repository>
	<id>JBoss repository</id>
	<url>http://repository.jboss.org/nexus/content/groups/public/</url>
      </repository>
    </repositories>
</project>
```

​		**注意：**旧的 *http://repository.jboss.com/maven2/* 已过时，不再使用。

# Maven依赖机制

在 Maven 依赖机制的帮助下自动下载所有必需的依赖库，并保持版本升级。

## 	案例分析

​	让我们看一个案例研究，以了解它是如何工作的。假设你想使用 Log4j 作为项目的日志。这里你要做什么？

## 	1.在传统方式

1. ​		访问 http://logging.apache.org/log4j/  
2. ​		下载 Log4 j的 jar 库 
3. ​		复制 jar 到项目类路径 
4. ​		手动将其包含到项目的依赖 
5. ​		所有的管理需要一切由自己做 

​	如果有 Log4j 版本升级，则需要重复上述步骤一次。

## 	2. 在Maven的方式

1. 你需要知道 log4j 的 Maven 坐标，例如：

   ```xml
   <groupId>log4j</groupId>
   <artifactId>log4j</artifactId>
   <version>1.2.14</version>
   ```

2. 它会自动下载 log4j 的1.2.14 版本库。如果“version”标签被忽略，它会自动升级库时当有新的版本时。 	

3. 声明 Maven 的坐标转换成 pom.xml 文件。

   ```xml
   <dependencies>
       <dependency>
   	<groupId>log4j</groupId>
   	<artifactId>log4j</artifactId>
   	<version>1.2.14</version>
       </dependency>
   </dependencies>
   ```

4. 当 Maven 编译或构建，log4j 的 jar 会自动下载，并把它放到 Maven 本地存储库 

5. 所有由 Maven 管理

## 解释说明

​	看看有什么不同？那么到底在Maven发生了什么？当建立一个Maven的项目，pom.xml文件将被解析，如果看到 log4j 的 Maven 坐标，然后 Maven 按此顺序搜索 log4j 库：

1. 在 Maven 的本地仓库搜索 log4j 	
2. 在 Maven 中央存储库搜索 log4j 
3. 在 Maven 远程仓库搜索 log4j(如果在 pom.xml 中定义) 

​	Maven 依赖库管理是一个非常好的工具，为您节省了大量的工作。

**如何找到 Maven 坐标？**
	 访问 Maven 中心储存库，搜索下载您想要的jar。

# 定制库到Maven本地资源库

这里有2个案例，需要手动发出Maven命令包括一个 jar 到 Maven 的本地资源库。

1. ​		要使用的 jar 不存在于 Maven 的中心储存库中。 
2. ​		您创建了一个自定义的 jar ，而另一个 Maven 项目需要使用。 

​	PS，还是有很多 jar 不支持 Maven 的。

## 案例学习

​	例如，kaptcha，它是一个流行的第三方Java库，它被用来生成 “验证码” 的图片，以阻止垃圾邮件，但它不在 Maven 的中央仓库中。

​	在本教程中，我们将告诉你如何安装 “kaptcha” jar 到Maven 的本地资源库。

## 1. mvn 安装

​	下载 “kaptcha”，将其解压缩并将 kaptcha-version.jar 复制到其他地方，比如：C盘。发出下面的命令：

```
mvn install:install-file -Dfile=c:\kaptcha-{version}.jar -DgroupId=com.google.code -DartifactId=kaptcha -Dversion={version} -Dpackaging=jar
```

​	示例：

```sh
D:\>mvn install:install-file -Dfile=c:\kaptcha-2.3.jar -DgroupId=com.google.code 
-DartifactId=kaptcha -Dversion=2.3 -Dpackaging=jar
[INFO] Scanning for projects...
[INFO] Searching repository for plugin with prefix: 'install'.
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Default Project
[INFO]    task-segment: [install:install-file] (aggregator-style)
[INFO] ------------------------------------------------------------------------
[INFO] [install:install-file]
[INFO] Installing c:\kaptcha-2.3.jar to 
D:\maven_repo\com\google\code\kaptcha\2.3\kaptcha-2.3.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: < 1 second
[INFO] Finished at: Tue May 12 13:41:42 SGT 2014
[INFO] Final Memory: 3M/6M
[INFO] ------------------------------------------------------------------------
```

​	现在，“kaptcha” jar被复制到 Maven 本地存储库。

## 2. pom.xml

​	安装完毕后，就在 pom.xml 中声明 kaptcha 的坐标。

```xml
<dependency>
      <groupId>com.google.code</groupId>
      <artifactId>kaptcha</artifactId>
      <version>2.3</version>
 </dependency>
```

## 	3. 完成

​	构建它，现在 “kaptcha” jar 能够从你的 Maven 本地存储库检索了。

# 使用Maven创建Java项目

## 1. 从 Maven 模板创建一个项目	 				 				 			

​	在本教程中，我们将向你展示如何使用 Maven 来创建一个 Java 项目，导入其到Eclipse IDE，并打包 Java 项目到一个 JAR 文件。 

​	所需要的工具：

1. Maven 3.3.3 
2. Eclipse 4.2 
3. JDK 8 

​	注意：请确保 Maven 是正确安装和配置（在Windows，*nix，Mac OSX系统中），然后再开始本教程，避免 mvn 命令未找到错误。

## 	1. 从 Maven 模板创建一个项目

​	在终端（* UNIX或Mac）或命令提示符（Windows）中，浏览到要创建 Java 项目的文件夹。键入以下命令：

```
mvn archetype:generate -DgroupId={project-packaging} -DartifactId={project-name}-DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

这告诉 Maven 来从 maven-archetype-quickstart 模板创建 Java 项目。如果忽视 archetypeArtifactId 选项，一个巨大的 Maven 模板列表将列出。

​	例如，这里的工作目录是：C:\worksp，执行命令过程时间可能比较久，看个人的网络状况。

```
C:\worksp>mvn archetype:generate -DgroupId=com.yiibai -DartifactId=NumberGenerat
or -DarchetypeArtifactId=maven -archetype-quickstart -DinteractiveMode=false
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources
@ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources
@ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom --
-
[INFO] Generating project in Batch mode
[INFO] -------------------------------------------------------------------------
---
[INFO] Using following parameters for creating project from Old (1.x) Archetype:
 maven-archetype-quickstart:1.0
[INFO] -------------------------------------------------------------------------
---
[INFO] Parameter: basedir, Value: C:\worksp
[INFO] Parameter: package, Value: com.yiibai
[INFO] Parameter: groupId, Value: com.yiibai
[INFO] Parameter: artifactId, Value: NumberGenerator
[INFO] Parameter: packageName, Value: com.yiibai
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] project created from Old (1.x) Archetype in dir: C:\worksp\NumberGenerato
r
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 23.166 s
[INFO] Finished at: 2015-10-27T11:03:48+08:00
[INFO] Final Memory: 17M/114M
[INFO] ------------------------------------------------------------------------
```

​	在上述情况下，一个新的Java项目命名 “NumberGenerator”, 而整个项目的目录结构会自动创建。

​	**注意**
 有少数用户说 mvn archetype:generate 命令未能生成项目结构。 如果您有任何类似的问题，不用担心，只需跳过此步骤，手动创建文件夹，请参阅步骤2的项目结构。

作者： 					mgmonster 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

​	在本教程中，我们将向你展示如何使用 Maven 来创建一个 Java 项目，导入其到Eclipse IDE，并打包 Java 项目到一个 JAR 文件。 

​	所需要的工具：

1. ​		Maven 3.3.3 
2. ​		Eclipse 4.2 
3. ​		JDK 8 

​	注意：请确保 Maven 是正确安装和配置（在Windows，*nix，Mac OSX系统中），然后再开始本教程，避免 mvn 命令未找到错误。

## 	1. 从 Maven 模板创建一个项目

​	在终端（* UNIX或Mac）或命令提示符（Windows）中，浏览到要创建 Java 项目的文件夹。键入以下命令：

```
mvn archetype:generate -DgroupId={project-packaging} -DartifactId={project-name}-DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

​	这告诉 Maven 来从 maven-archetype-quickstart 模板创建 Java 项目。如果忽视 archetypeArtifactId 选项，一个巨大的 Maven 模板列表将列出。

​	例如，这里的工作目录是：C:\worksp，执行命令过程时间可能比较久，看个人的网络状况。

```
C:\worksp>mvn archetype:generate -DgroupId=com.yiibai -DartifactId=NumberGenerat
or -DarchetypeArtifactId=maven -archetype-quickstart -DinteractiveMode=false
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources
@ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources
@ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom --
-
[INFO] Generating project in Batch mode
[INFO] -------------------------------------------------------------------------
---
[INFO] Using following parameters for creating project from Old (1.x) Archetype:
 maven-archetype-quickstart:1.0
[INFO] -------------------------------------------------------------------------
---
[INFO] Parameter: basedir, Value: C:\worksp
[INFO] Parameter: package, Value: com.yiibai
[INFO] Parameter: groupId, Value: com.yiibai
[INFO] Parameter: artifactId, Value: NumberGenerator
[INFO] Parameter: packageName, Value: com.yiibai
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] project created from Old (1.x) Archetype in dir: C:\worksp\NumberGenerato
r
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 23.166 s
[INFO] Finished at: 2015-10-27T11:03:48+08:00
[INFO] Final Memory: 17M/114M
[INFO] ------------------------------------------------------------------------
```

​	在上述情况下，一个新的Java项目命名 “NumberGenerator”, 而整个项目的目录结构会自动创建。

​	**注意**
 有少数用户说 mvn archetype:generate 命令未能生成项目结构。 如果您有任何类似的问题，不用担心，只需跳过此步骤，手动创建文件夹，请参阅步骤2的项目结构。

## 2.Maven目录布局

​	使用 mvn archetype:generate + maven-archetype-quickstart 模板, 以下项目的目录结构被创建。

```
NumberGenerator
   |-src
   |---main
   |-----java
   |-------com
   |---------yiibai   
   |-----------App.java
   |---test|-----java
   |-------com
   |---------yiibai
   |-----------AppTest.java
   |-pom.xml
```

​	很简单的，所有的源代码放在文件夹 /src/main/java/, 所有的单元测试代码放入 /src/test/java/.

​	**注意，请阅读** [Maven标准目录布局](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html) 

​	附加的一个标准的 pom.xml 被生成。这个POM文件类似于 Ant build.xml 文件，它描述了整个项目的信息，一切从目录结构，项目的插件，项目依赖，如何构建这个项目等，请阅读[POM官方指南](http://maven.apache.org/guides/introduction/introduction-to-the-pom.html) 

# 使用Maven创建Web应用程序项目

## 1. 从Maven模板创建Web项目

​	您可以通过使用Maven的maven-archetype-webapp模板来创建一个快速启动Java Web应用程序的项目。在终端(* UNIX或Mac)或命令提示符(Windows)中，导航至您想要创建项目的文件夹。

​	键入以下命令：

```
$ mvn archetype:generate -DgroupId=com.yiibai -DartifactId=CounterWebApp -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

# Maven POM

POM代表项目对象模型。它是 Maven 中工作的基本单位，这是一个 XML 文件。它始终保存在该项目基本目录中的 `pom.xml` 文件。
POM 包含的项目是使用 Maven 来构建的，它用来包含各种配置信息。
POM 也包含了目标和插件。在执行任务或目标时，Maven 会使用当前目录中的 POM。它读取POM得到所需要的配置信息，然后执行目标。部分的配置可以在 POM 使用如下：

* project dependencies
* plugins
* goals
* build profiles
* project version
* developers
* mailing list

创建一个POM之前，应该要先决定项目组(groupId)，它的名字(artifactId)和版本，因为这些属性在项目仓库是唯一标识的。

## POM的例子

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.yiibai.project-group</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
<project>
```

要注意的是，每个项目只有一个POM文件。

* 所有的 POM 文件要项目元素必须有三个必填字段: `groupId`，`artifactId`，`version`
* 在库中的项目符号是：`groupId:artifactId:version` 
* pom.xml 的根元素是 `project`，它有三个主要的子节点。

| 节点       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| groupId    | 这是项目组的编号，这在组织或项目中通常是独一无二的。 例如，一家银行集团`com.company.bank`拥有所有银行相关项目。 |
| artifactId | 这是项目的ID。这通常是项目的名称。 例如，`consumer-banking`。 除了`groupId`之外，`artifactId`还定义了`artifact`在存储库中的位置。 |
| `version`  | 这是项目的版本。与`groupId`一起使用，`artifact`在存储库中用于将版本彼此分离。 例如：`com.company.bank:consumer-banking:1.0`，`com.company.bank:consumer-banking:1.1` |

## SUPER POM

所有的POM继承自父类(尽管明确界定)。这个基础的 POM 被称为SUPER POM，并包含继承默认值。 Maven使用有效的POM(SUPER POM加项目配置的配置)执行有关目标。它可以帮助开发人员指定最低配置的详细信息写在 `pom.xml` 中。虽然配置可以很容易被覆盖。 一个简单的方法来看看超级POM的默认配置，通过运行下面的命令：`mvn help:effective-pom` 创建一个 `pom.xml` 。 在下面的例子中，已经创建了一个 `pom.xml` 在`C:\MVN\` 项目文件夹中。 现在，打开命令控制台，进入包含 `pom.xml` 文件夹并执行以下  `mvn` 命令。

# Maven 构建生命周期

作者： 					易百 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

## 	构建生命周期是什么？

​	构建生命周期阶段的目标是执行顺序是一个良好定义的序列。
 这里使用一个例子，一个典型的 [Maven](http://www.yiibai.com/maven) 构建生命周期是由下列顺序的阶段：

| 阶段     | 处理     | 描述                                     |
| -------- | -------- | ---------------------------------------- |
| 准备资源 | 资源复制 | 资源复制可以进行定制                     |
| 编译     | 执行编译 | 源代码编译在此阶段完成                   |
| 包装     | 打包     | 创建JAR/WAR包如在 pom.xml 中定义提及的包 |
| 安装     | 安装     | 这一阶段在本地/远程Maven仓库安装程序包   |

作者： 					易百 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

## 	构建生命周期是什么？

​	构建生命周期阶段的目标是执行顺序是一个良好定义的序列。
 这里使用一个例子，一个典型的 [Maven](http://www.yiibai.com/maven) 构建生命周期是由下列顺序的阶段：

| 阶段     | 处理     | 描述                                     |
| -------- | -------- | ---------------------------------------- |
| 准备资源 | 资源复制 | 资源复制可以进行定制                     |
| 编译     | 执行编译 | 源代码编译在此阶段完成                   |
| 包装     | 打包     | 创建JAR/WAR包如在 pom.xml 中定义提及的包 |
| 安装     | 安装     | 这一阶段在本地/远程Maven仓库安装程序包   |

​	可用于注册必须执行一个特定的阶段之前或之后的目标，有之前处理和之后阶段。
 当 Maven 开始建立一个项目，它通过定义序列阶段步骤和执行注册的每个阶段的目标。 Maven有以下三种标准的生命周期：

* clean 	
* default(或 build) 	
* site

目标代表一个特定的任务，它有助于项目的建设和管理。可以被绑定到零个或多个生成阶段。一个没有绑定到任何构建阶段的目标，它的构建生命周期可以直接调用执行。
 执行的顺序取决于目标和构建阶段折调用顺序。例如，考虑下面的命令。清理和打包（mvn clean）参数的构建阶段，而 dependency:copy-dependencies package 是一个目标。

```
mvn clean dependency:copy-dependencies package
```

​	在这里，清洁的阶段，将首先执行，然后是依赖关系：复制依赖性的目标将被执行，并终于将执行包阶段。

作者： 					易百 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

## 	构建生命周期是什么？

​	构建生命周期阶段的目标是执行顺序是一个良好定义的序列。
 这里使用一个例子，一个典型的 [Maven](http://www.yiibai.com/maven) 构建生命周期是由下列顺序的阶段：

| 阶段     | 处理     | 描述                                     |
| -------- | -------- | ---------------------------------------- |
| 准备资源 | 资源复制 | 资源复制可以进行定制                     |
| 编译     | 执行编译 | 源代码编译在此阶段完成                   |
| 包装     | 打包     | 创建JAR/WAR包如在 pom.xml 中定义提及的包 |
| 安装     | 安装     | 这一阶段在本地/远程Maven仓库安装程序包   |

​	可用于注册必须执行一个特定的阶段之前或之后的目标，有之前处理和之后阶段。
 当 Maven 开始建立一个项目，它通过定义序列阶段步骤和执行注册的每个阶段的目标。 Maven有以下三种标准的生命周期：

* ​			clean 	
* ​			default(或 build) 	
* ​			site 	

​	目标代表一个特定的任务，它有助于项目的建设和管理。可以被绑定到零个或多个生成阶段。一个没有绑定到任何构建阶段的目标，它的构建生命周期可以直接调用执行。
 执行的顺序取决于目标和构建阶段折调用顺序。例如，考虑下面的命令。清理和打包（mvn clean）参数的构建阶段，而 dependency:copy-dependencies package 是一个目标。

```
mvn clean dependency:copy-dependencies package
```

​	在这里，清洁的阶段，将首先执行，然后是依赖关系：复制依赖性的目标将被执行，并终于将执行包阶段。

## 清洁生命周期

​	当我们执行命令 mvn clean 命令后，Maven 调用清洁的生命周期由以下几个阶段组成：

* pre-clean 	
* clean 	
* post-clean 	

​	Maven 清洁目标（clean:clean）被绑定清洁干净的生命周期阶段。clean:clean 目标删除 build 目录下的构建输出。因此，当 mvn clean 命令执行时，Maven会删除编译目录。

目标清洁生命周期在上述阶段，我们可以自定义此行为。
 在下面的示例中，我们将附加 maven-antrun-plugin:run 对目标进行预清洁，清洁和清洁后这三个阶段。这将使我们能够调用的信息显示清理生命周期的各个阶段。
 现在来创建了一个 pom.xml 文件在 ***C:\MVN\*** **项目文件夹中，具体内容如下：**

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.companyname.projectgroup</groupId>
<artifactId>project</artifactId>
<version>1.0</version>
<build>
<plugins>
   <plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-antrun-plugin</artifactId>
   <version>1.1</version>
   <executions>
      <execution>
         <id>id.pre-clean</id>
         <phase>pre-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>pre-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.clean</id>
         <phase>clean</phase>
         <goals>
          <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.post-clean</id>
         <phase>post-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>post-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
   </executions>
   </plugin>
</plugins>
</build>
</project>
```

作者： 					易百 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

## 	构建生命周期是什么？

​	构建生命周期阶段的目标是执行顺序是一个良好定义的序列。
 这里使用一个例子，一个典型的 [Maven](http://www.yiibai.com/maven) 构建生命周期是由下列顺序的阶段：

| 阶段     | 处理     | 描述                                     |
| -------- | -------- | ---------------------------------------- |
| 准备资源 | 资源复制 | 资源复制可以进行定制                     |
| 编译     | 执行编译 | 源代码编译在此阶段完成                   |
| 包装     | 打包     | 创建JAR/WAR包如在 pom.xml 中定义提及的包 |
| 安装     | 安装     | 这一阶段在本地/远程Maven仓库安装程序包   |

​	可用于注册必须执行一个特定的阶段之前或之后的目标，有之前处理和之后阶段。
 当 Maven 开始建立一个项目，它通过定义序列阶段步骤和执行注册的每个阶段的目标。 Maven有以下三种标准的生命周期：

* ​			clean 	
* ​			default(或 build) 	
* ​			site 	

​	目标代表一个特定的任务，它有助于项目的建设和管理。可以被绑定到零个或多个生成阶段。一个没有绑定到任何构建阶段的目标，它的构建生命周期可以直接调用执行。
 执行的顺序取决于目标和构建阶段折调用顺序。例如，考虑下面的命令。清理和打包（mvn clean）参数的构建阶段，而 dependency:copy-dependencies package 是一个目标。

```
mvn clean dependency:copy-dependencies package
```

​	在这里，清洁的阶段，将首先执行，然后是依赖关系：复制依赖性的目标将被执行，并终于将执行包阶段。

## 	清洁生命周期

​	当我们执行命令 mvn clean 命令后，Maven 调用清洁的生命周期由以下几个阶段组成：

* ​			pre-clean 	
* ​			clean 	
* ​			post-clean 	

​	Maven 清洁目标（clean:clean）被绑定清洁干净的生命周期阶段。clean:clean 目标删除 build 目录下的构建输出。因此，当 mvn clean 命令执行时，Maven会删除编译目录。

​	目标清洁生命周期在上述阶段，我们可以自定义此行为。
 在下面的示例中，我们将附加 maven-antrun-plugin:run 对目标进行预清洁，清洁和清洁后这三个阶段。这将使我们能够调用的信息显示清理生命周期的各个阶段。
 现在来创建了一个 pom.xml 文件在 ***C:\MVN\*** **项目文件夹中，具体内容如下：**

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.companyname.projectgroup</groupId>
<artifactId>project</artifactId>
<version>1.0</version>
<build>
<plugins>
   <plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-antrun-plugin</artifactId>
   <version>1.1</version>
   <executions>
      <execution>
         <id>id.pre-clean</id>
         <phase>pre-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>pre-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.clean</id>
         <phase>clean</phase>
         <goals>
          <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>clean phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.post-clean</id>
         <phase>post-clean</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>post-clean phase</echo>
            </tasks>
         </configuration>
      </execution>
   </executions>
   </plugin>
</plugins>
</build>
</project>
```

​	现在，打开命令控制台，到该文件夹包含 pom.xml 并执行以下 mvn 命令。

```
C:\MVN\project>mvn post-clean
```

​	Maven将开始处理并显示清理生命周期的所有阶段。

```
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------
[INFO] Building Unnamed - com.companyname.projectgroup:project:jar:1.0
[INFO]    task-segment: [post-clean]
[INFO] ------------------------------------------------------------------
[INFO] [antrun:run {execution: id.pre-clean}]
[INFO] Executing tasks
     [echo] pre-clean phase
[INFO] Executed tasks
[INFO] [clean:clean {execution: default-clean}]
[INFO] [antrun:run {execution: id.clean}]
[INFO] Executing tasks
     [echo] clean phase
[INFO] Executed tasks
[INFO] [antrun:run {execution: id.post-clean}]
[INFO] Executing tasks
     [echo] post-clean phase
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
[INFO] Total time: < 1 second
[INFO] Finished at: Sat Jul 07 13:38:59 IST 2012
[INFO] Final Memory: 4M/44M
[INFO] ------------------------------------------------------------------
```

​	你可以尝试调整 mvn 清洁命令，该命令将显示清洁前什么都不会被执行。

## 	默认（或生成）生命周期

​	这是 Maven 主要的生命周期，用于构建应用程序。它有以下 23 个阶段。

| 生命周期阶段          | 描述                                                      |
| --------------------- | --------------------------------------------------------- |
| validate              | 验证项目是否正确，并且所有必要的信息可用于完成构建过程    |
| initialize            | 建立初始化状态，例如设置属性                              |
| generate-sources      | 产生任何的源代码包含在编译阶段                            |
| process-sources       | 处理源代码，例如，过滤器值                                |
| generate-resources    | 包含在包中产生的资源                                      |
| process-resources     | 复制和处理资源到目标目录，准备打包阶段                    |
| compile               | 编译该项目的源代码                                        |
| process-classes       | 从编译生成的文件提交处理，例如：Java类的字节码增强/优化   |
| generate-test-sources | 生成任何测试的源代码包含在编译阶段                        |
| process-test-sources  | 处理测试源代码，例如，过滤器任何值                        |
| test-compile          | 编译测试源代码到测试目标目录                              |
| process-test-classes  | 处理测试代码文件编译生成的文件                            |
| test                  | 运行测试使用合适的单元测试框架（JUnit）                   |
| prepare-package       | 执行必要的任何操作的实际打包之前准备一个包                |
| package               | 提取编译后的代码，并在其分发格式打包，如JAR，WAR或EAR文件 |
| pre-integration-test  | 完成执行集成测试之前所需操作。例如，设置所需的环境        |
| integration-test      | 处理并在必要时部署软件包到集成测试可以运行的环境          |
| pre-integration-test  | 完成集成测试已全部执行后所需操作。例如，清理环境          |
| verify                | 运行任何检查，验证包是有效的，符合质量审核规定            |
| install               | 将包安装到本地存储库，它可以用作当地其他项目的依赖        |
| deploy                | 复制最终的包到远程仓库与其他开发者和项目共享              |

​	有涉及到Maven 生命周期值得一提几个重要概念：

* ​			当一个阶段是通过 Maven命令调用，例如：mvn compile，只有阶段到达并包括这个阶段才会被执行。 	
* ​			不同的 Maven 目标绑定到 Maven生命周期的不同阶段这是这取决于包类型(JAR/WAR/EAR)。 		

​	在下面的示例中，将附加 Maven 的 antrun 插件：运行目标构建生命周期的几个阶段。这将使我们能够回显的信息显示生命周期的各个阶段。
 我们已经更新了在 C:\MVN\ 项目文件夹中的 pom.xml 文件。

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.companyname.projectgroup</groupId>
<artifactId>project</artifactId>
<version>1.0</version>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-antrun-plugin</artifactId>
<version>1.1</version>
<executions>
   <execution>
      <id>id.validate</id>
      <phase>validate</phase>
      <goals>
         <goal>run</goal>       </goals>
      <configuration>
         <tasks>
            <echo>validate phase</echo>
         </tasks>
      </configuration>
   </execution>
   <execution>
      <id>id.compile</id>
      <phase>compile</phase>
      <goals>
         <goal>run</goal>
      </goals>
      <configuration>
         <tasks>
            <echo>compile phase</echo>
         </tasks>
      </configuration>
   </execution>
   <execution>
      <id>id.test</id>
      <phase>test</phase>
      <goals>
         <goal>run</goal>
      </goals>
      <configuration>
         <tasks>
            <echo>test phase</echo>
         </tasks>
      </configuration>
   </execution>
   <execution>
         <id>id.package</id>
         <phase>package</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
         <tasks>
            <echo>package phase</echo>
         </tasks>
      </configuration>
   </execution>
   <execution>
      <id>id.deploy</id>
      <phase>deploy</phase>
      <goals>
         <goal>run</goal>
      </goals>
      <configuration>
      <tasks>
         <echo>deploy phase</echo>
      </tasks>
      </configuration>
   </execution>
</executions>
</plugin>
</plugins>
</build>
</project>
```

​	现在，打开命令控制台，进入包含 pom.xml 并执行以下 mvn 命令。

```
C:\MVN\project>mvn compile
```

​	编译阶段，Maven 将开始构建生命周期的阶段处理并显示。

```
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------
[INFO] Building Unnamed - com.companyname.projectgroup:project:jar:1.0
[INFO]    task-segment: [compile]
[INFO] ------------------------------------------------------------------
[INFO] [antrun:run {execution: id.validate}]
[INFO] Executing tasks
     [echo] validate phase
[INFO] Executed tasks
[INFO] [resources:resources {execution: default-resources}]
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources,
i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory C:\MVN\project\src\main\resources
[INFO] [compiler:compile {execution: default-compile}]
[INFO] Nothing to compile - all classes are up to date
[INFO] [antrun:run {execution: id.compile}]
[INFO] Executing tasks
     [echo] compile phase
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
[INFO] Total time: 2 seconds
[INFO] Finished at: Sat Jul 07 20:18:25 IST 2012
[INFO] Final Memory: 7M/64M
[INFO] ------------------------------------------------------------------
```

## 	网站的生命周期

​	Maven的网站插件通常用于创建新的文档，创建报告，部署网站等。
 阶段

* ​			pre-site 	
* ​			site 	
* ​			post-site 	
* ​			site-deploy 	

​	在下面的示例中，我们将附加 maven-antrun-plugin:run 目标网站的生命周期的所有阶段。这将使我们能够调用短信显示的生命周期的各个阶段。
 现在更新 pom.xml 文件在 C:\MVN\ 项目文件夹中。

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
  http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.companyname.projectgroup</groupId>
<artifactId>project</artifactId>
<version>1.0</version>
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-antrun-plugin</artifactId>
<version>1.1</version>
   <executions>
      <execution>
         <id>id.pre-site</id>
         <phase>pre-site</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>pre-site phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.site</id>
         <phase>site</phase>
         <goals>
         <goal>run</goal>
         </goals>
         <configuration><tasks>
               <echo>site phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.post-site</id>
         <phase>post-site</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>post-site phase</echo>
            </tasks>
         </configuration>
      </execution>
      <execution>
         <id>id.site-deploy</id>
         <phase>site-deploy</phase>
         <goals>
            <goal>run</goal>
         </goals>
         <configuration>
            <tasks>
               <echo>site-deploy phase</echo>
            </tasks>
         </configuration>
      </execution>
   </executions>
</plugin>
</plugins>
</build>
</project>
```

​	打开命令控制台，进入该文件夹包含 pom.xml 并执行以下 mvn 命令。

```
C:\MVN\project>mvn site
```

​	Maven将开始处理并显示网站的生命周期阶段的各个阶段。

```
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------
[INFO] Building Unnamed - com.companyname.projectgroup:project:jar:1.0
[INFO]    task-segment: [site]
[INFO] ------------------------------------------------------------------
[INFO] [antrun:run {execution: id.pre-site}]
[INFO] Executing tasks
     [echo] pre-site phase
[INFO] Executed tasks
[INFO] [site:site {execution: default-site}]
[INFO] Generating "About" report.
[INFO] Generating "Issue Tracking" report.
[INFO] Generating "Project Team" report.
[INFO] Generating "Dependencies" report.
[INFO] Generating "Project Plugins" report.
[INFO] Generating "Continuous Integration" report.
[INFO] Generating "Source Repository" report.
[INFO] Generating "Project License" report.
[INFO] Generating "Mailing Lists" report.
[INFO] Generating "Plugin Management" report.
[INFO] Generating "Project Summary" report.
[INFO] [antrun:run {execution: id.site}]
[INFO] Executing tasks
     [echo] site phase
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
[INFO] Total time: 3 seconds
[INFO] Finished at: Sat Jul 07 15:25:10 IST 2012
[INFO] Final Memory: 24M/149M
[INFO] -
```

# Maven 构建配置文件

## 什么是构建配置文件?

​	生成配置文件是一组可以用来设置或覆盖 Maven 构建配置值的默认值。使用生成配置文件，你可以针对不同的环境，如：生产VS开发环境自定义构建。

​	配置文件中指定 pom.xml 文件使用其配置文件/配置文件元素和多种方式来触发。配置文件修改 POM 后，在编译的时候是用来给不同的目标环境参数（例如，开发，测试和生产环境的数据库服务器的路径）。

## 生成配置文件的类型

​	创建配置文件的文件主要有三种类型：

| 类型        | 定义位置                                                     |
| ----------- | ------------------------------------------------------------ |
| Per Project | 在项目中定义的POM文件, pom.xml                               |
| Per User    | 定义在 Maven 中的设置 XML 文件(%USER_HOME%/.m2/settings.xml) |
| Global      | 定义在 Maven 中的全局设置 xml 文件 (%M2_HOME%/conf/settings.xml) |

## 配置文件激活

​	Maven 构建配置文件的文件，可以使用以下几种方式来激活。

* 明确使用命令从控制台输入。 	
* 通过 Maven 设置。 	
* 基于环境变量（用户/系统变量）。 	
* OS设置（例如，Windows系列）。 	
* 呈现/丢失的文件。

现在下*src/main/resources* 有三个特定的文件：

| 文件名称            | 描述                               |
| ------------------- | ---------------------------------- |
| env.properties      | 如果没有配置文件关联则使用默认配置 |
| env.test.properties | 当测试配置文件用于测试配置         |
| env.prod.properties | 生产配置时，prod信息被使用         |

作者： 					易百 				**Java技术QQ群：227270512 / Linux QQ群：479429477** 			



​					 				 				 				 			

## 	什么是构建配置文件?

​	生成配置文件是一组可以用来设置或覆盖 Maven 构建配置值的默认值。使用生成配置文件，你可以针对不同的环境，如：生产V/S开发环境自定义构建。

​	配置文件中指定 pom.xml 文件使用其配置文件/配置文件元素和多种方式来触发。配置文件修改 POM 后，在编译的时候是用来给不同的目标环境参数（例如，开发，测试和生产环境的数据库服务器的路径）。

## 	生成配置文件的类型

​	创建配置文件的文件主要有三种类型：

| 类型        | 定义位置                                                     |
| ----------- | ------------------------------------------------------------ |
| Per Project | 在项目中定义的POM文件, pom.xml                               |
| Per User    | 定义在 Maven 中的设置 XML 文件(%USER_HOME%/.m2/settings.xml) |
| Global      | 定义在 Maven 中的全局设置 xml 文件 (%M2_HOME%/conf/settings.xml) |

## 	配置文件激活

​	Maven 构建配置文件的文件，可以使用以下几种方式来激活。

* ​			明确使用命令从控制台输入。 	
* ​			通过 Maven 设置。 	
* ​			基于环境变量（用户/系统变量）。 	
* ​			OS设置（例如，Windows系列）。 	
* ​			呈现/丢失的文件。 	

## 	配置文件的文件激活的例子

​	我们假设你的项目如下的目录结构：

![Maven Build Profile](/uploads/allimg/131116/21121332H-0.png)

​	现在下*src/main/resources* 有三个特定的文件：

| 文件名称            | 描述                               |
| ------------------- | ---------------------------------- |
| env.properties      | 如果没有配置文件关联则使用默认配置 |
| env.test.properties | 当测试配置文件用于测试配置         |
| env.prod.properties | 生产配置时，prod信息被使用         |

## 	显式配置文件激活

​	在下面的例子中，我们会附加上maven-antrun-plugin:run 插件：运行测试阶段目标。这将使我们能够为不同的配置文件调用文本消息。将在 pom.xml 中定义不同的配置文件，并在命令控制台使用 maven 命令将配置文件启动。

​	假设，我们建立如下的 pom.xml 在 C:\MVN\project 文件夹。 

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.companyname.projectgroup</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
   <profiles>
      <profile>
      <id>test</id>
      <build>
      <plugins>
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-antrun-plugin</artifactId>
            <version>1.1</version>
            <executions>
               <execution>
                  <phase>test</phase>
                  <goals>
                     <goal>run</goal>
                  </goals>
                  <configuration>
                  <tasks>
                     <echo>Using env.test.properties</echo>
            <copy file="src/main/resources/env.test.properties" tofile
		    ="${project.build.outputDirectory}/env.properties"/>
                  </tasks>
                  </configuration>
               </execution>
            </executions>
         </plugin>
      </plugins>
      </build>
      </profile>
   </profiles>
   <dependencies>
      <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>3.8.1</version>
         <scope>test</scope>
      </dependency>
   </dependencies>
</project>
```

​	*env.properties* 

```
environment=debug
```

​	*env.test.properties* 

```
environment=test
```

​	*env.prod.properties* 

```
environment=prod
```

​	现在，打开命令控制台，进入到包含 pom.xml 的文件夹并执行以下 mvn 命令。通过配置文件名作为参数可使用 -P 选项。 

```
C:\MVN\project>mvn test -P test
```

Maven会开始处理并显示构建配置文件的测试结果。

```
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------
[INFO] Building Unnamed - com.companyname.projectgroup:project:jar:1.0
[INFO]    task-segment: [test]
[INFO] ------------------------------------------------------------------
[INFO] [resources:resources {execution: default-resources}]
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources,
i.e. build is platform dependent!
[INFO] Copying 3 resources
[INFO] [compiler:compile {execution: default-compile}]
[INFO] Nothing to compile - all classes are up to date
[INFO] [resources:testResources {execution: default-testResources}]
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources,
i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory C:MVNprojectsrc	est
esources
[INFO] [compiler:testCompile {execution: default-testCompile}]
[INFO] Nothing to compile - all classes are up to date
[INFO] [surefire:test {execution: default-test}]
[INFO] Surefire report directory: C:MVNproject	argetsurefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
There are no tests to run.

Results :

Tests run: 0, Failures: 0, Errors: 0, Skipped: 0

[INFO] [antrun:run {execution: default}]
[INFO] Executing tasks
     [echo] Using env.test.properties
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
[INFO] Total time: 1 second
[INFO] Finished at: Sun Jul 08 14:55:41 IST 2012
[INFO] Final Memory: 8M/64M
[INFO] ------------------------------------------------------------------
```

​	**现在，作为一个练习，可以做以下步骤：** 

* ​			在另一个配置文件pom.xml 添加元素（复制现有的配置文件元素 profiles，并将其粘贴轮廓元素结束）。 	
* ​			更新在此档案中元素的 ID 并测试正常。 	
* ​			更新任务部分使用 env.properties 并将 env.properties 复制到目标目录 	
* ​			再次重复上述三个步骤，更新 ID 和 env.prod.properties 	
* ​			现在准备建立配置文件文件（正常/测试/产品）。 	

​	现在，打开命令控制台，进入到该文件夹包含的 pom.xml 并执行以下 mvn 命令。通过配置文件名作为参数使用 -P 选项。

```
C:\MVN\project>mvn test -Pnormal
C:\MVN\project>mvn test -Pprod
```

​	建立检查输出看到它们的差别了

## 配置文件通过Maven设置激活

​	打开 Maven 的 settings.xml 文件，可在 %USER_HOME%/.m2 目录，％USER_HOME％ 表示用户的主目录。 settings.xml 文件如果不存在，那么就创建一个新的。

​	添加测试配置文件作为使用activeProfiles节点主动配置文件，示例如下所示：

```xml
<settings xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
   <mirrors>
      <mirror>
         <id>maven.dev.snaponglobal.com</id>
         <name>Internal Artifactory Maven repository</name>
         <url>http://repo1.maven.org/maven2/</url>
         <mirrorOf>*</mirrorOf>
      </mirror>
   </mirrors>
   <activeProfiles>
      <activeProfile>test</activeProfile>
   </activeProfiles>
</settings>
```

​	现在，打开命令控制台，进入到该文件夹包含 pom.xml 并执行以下 mvn 命令。不要通过使用 -P 选项。Maven会显示结果，测试配置文件是一个活跃的配置文件。

```
C:\MVN\project>mvn test
```

## 通过系统环境变量配置文件

​	现在从 Maven 的 settings.xml 配置文件删除和更新 pom.xml 中提到的测试配置文件。 profile 元素添加 activation 元素，如下所示。 

​	当系统属性 “env” 的值设置为 “test” 时指定的测试配置文件将被触发。
 创建一个环境变量 “env”，并设置其值为 “test” 。

```xml
<profile>
   <id>test</id>
   <activation>
      <property>
         <name>env</name>
         <value>test</value>
      </property>
   </activation>
</profile>
```

​	打开命令控制台，进入到包含 pom.xml 文件的文件夹下并执行以下 mvn 命令。

```
C:\MVN\project>mvn test
```

## 通过操作系统激活配置文件

​	activation 元素包括 os 细节，如下所示。测试配置文件文件时，将触发的系统是 windows XP。

```xml
<profile>
   <id>test</id>
   <activation>
      <os>
         <name>Windows XP</name>
         <family>Windows</family>
         <arch>x86</arch>
         <version>5.1.2600</version>
      </os>
   </activation>
</profile>
```

​	现在，打开命令控制台，进入到文件夹包含 pom.xml 并执行以下 mvn 命令。不要通过使用选项 -P 配置文件名称。 Maven 会显示结果：

```
C:\MVN\project>mvn test
```

## 配置文件激活通过现状/丢失文件

​	载入 activation 元件包括 os 的细节，如下所示。test配置文件将因 target/generated-sources/axistools/wsdl2java/com/companyname/group 缺少触发。

```xml
<profile>
   <id>test</id>
   <activation>
      <file>
         <missing>target/generated-sources/axistools/wsdl2java/
		 com/companyname/group</missing>
      </file>
   </activation>
</profile>
```

​	现在，打开命令控制台，进入到包含 pom.xml 的文件夹并执行以下 mvn 命令。不要使用选项 -P传递配置文件名称 。 Maven 会显示结果，test 配置文件是一个 活动配置文件。

```
C:\MVN\project>mvn test
```

# Maven存储库

## 什么是Maven资源库?

​	在 Maven 术语里存储库是一个目录，即目录中保存所有项目的 jar 库，插件或任何其他项目特定文件，并可以容易由 Maven 使用。

​	Maven库中有三种类型

* local - 本地库 	
* central - 中央库 	
* remote - 远程库

## 本地库

​	Maven 本地存储库是一个在本地计算机上的一个文件夹位置。当你第一次运行 maven 命令的时候它就被创建了。

​	Maven 的本地资源库让您的项目可依赖这些项目（插件库 jar 文件，jar文件等）。当运行 Maven 构建，那么 Maven 会自动下载所有依赖的jar到本地存储库中。它有助于避免依赖存储在远程机器上的项目建立参考。

​	Maven 本地存储库，默认情况下创建在 ％USER_HOME％ 目录。要覆盖默认位置，可在 Maven 的 settings.xml 文件中修改 ％M2_HOME％conf 目录指向另一个路径。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository>C:/MyLocalRepository</localRepository>
</settings>
```

​	当运行 Maven 命令，Maven 会下载依赖到您的自定义路径。

## 中央储存库

​	Maven中央存储库是由Maven社区提供的资源库。它包含了大量的常用程序库。 

​	当Maven没有在本地存储库找到任何依赖，就会开始搜索在中央存储库，它会使用下列网址: http://repo1.maven.org/maven2/ 

​	中央仓库的关键概念：

* 此系统信息库由Maven社区管理 	
* 它不要求配置 	
* 搜索时需要互联网接入 	

​	要浏览中央 Maven 仓库的内容，Maven 社区提供了一个网址：http://search.maven.org/#browse 。使用这个库，开发人员可以在中央存储库中搜索所有可用的库。

## 远程仓库

​	有时，Maven不能从依赖中央存储库找到上述库，那么它停下构建过程并输出错误消息到控制台。为了防止这种情况，Maven提供远程仓库概念，这是开发商的自定义库包含所需的库文件或其他项目 jar 文件。

​	例如，使用以下提到的 pom.xml，Maven 会从远程仓库下载依赖项（不在中央存储库中提供）。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.companyname.projectgroup</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
   <dependencies>
      <dependency>
         <groupId>com.companyname.common-lib</groupId>
         <artifactId>common-lib</artifactId>
         <version>1.0.0</version>
      </dependency>
   <dependencies>
   <repositories>
      <repository>
         <id>companyname.lib1</id>
         <url>http://download.companyname.org/maven2/lib1</url>
      </repository>
      <repository>
         <id>companyname.lib2</id>
         <url>http://download.companyname.org/maven2/lib2</url>
      </repository>
   </repositories>
</project>
```

## Maven 依赖搜索序列

​	当我们执行 Maven 构建命令，Maven 依赖库按以下顺序进行搜索：

* 第1步 - 搜索依赖本地资源库，如果没有找到，跳到第2步，否则，如果找到那么会做进一步处理。 	
* 第2步 - 搜索依赖中央存储库，如果没有找到，则从远程资源库/存储库中，然后移动到步骤4，否则如果找到，那么它下载到本地存储库中，以备将来参考使用。 	
* 第3步 - 如果没有提到远程仓库，Maven 则会停止处理并抛出错误（找不到依赖库）。 	
* 第4步 - 远程仓库或储存库中的搜索依赖，如果找到它会下载到本地资源库以供将来参考使用，否则 Maven 停止处理并抛出错误（找不到依赖库）。

# Maven插件

## 什么是 Maven 的插件？

​	Maven 是一个执行插件的框架，每一个任务实际上是由插件完成的。Maven 插件通常用于：

* 创建 jar 文件 	
* 创建 war 文件 		
* 编译代码文件 	
* 进行代码单元测试 	
* 创建项目文档 	
* 创建项目报告

一个插件通常提供了一组目标，可使用以下语法来执行：

```
mvn [plugin-name]:[goal-name]
```

​	例如，一个 Java 项目可以使用 Maven 编译器插件来编译目标，通过运行以下命令编译

```
mvn compiler:compile
```

## 插件类型

​	Maven 提供以下两种类型插件：

| 类型     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 构建插件 | 在生成过程中执行，并在 pom.xml 中的`<build/> `元素进行配置   |
| 报告插件 | 在网站生成期间执行，在 pom.xml 中的 `<reporting/>` 元素进行配置 |

以下是一些常见的插件列表：

| 插件     | 描述                                  |
| -------- | ------------------------------------- |
| clean    | 编译后的清理目标，删除目标目录        |
| compiler | 编译 Java 源文件                      |
| surefile | 运行JUnit单元测试，创建测试报告       |
| jar      | 从当前项目构建 JAR 文件               |
| war      | 从当前项目构建 WAR 文件               |
| javadoc  | 产生用于该项目的 Javadoc              |
| antrun   | 从构建所述的任何阶段运行一组 Ant 任务 |

# Maven外部依赖

要处理这种情况，需要添加外部依赖项，如使用下列方式在 Maven 的 pom.xml 。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
   http://maven.apache.org/maven-v4_0_0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.companyname.bank</groupId>
   <artifactId>consumerBanking</artifactId>
   <packaging>jar</packaging>
   <version>1.0-SNAPSHOT</version>
   <name>consumerBanking</name>
   <url>http://maven.apache.org</url>

   <dependencies>
      <dependency>
         <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>3.8.1</version>
         <scope>test</scope>
      </dependency>

      <dependency>
         <groupId>ldapjdk</groupId>
         <artifactId>ldapjdk</artifactId>
         <scope>system</scope>
         <version>1.0</version>
         <systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>
      </dependency>
   </dependencies>

</project>
```

# Maven项目模板

maven 使用 Archetype 概念为用户提供不同类型的项目模板，它是一个非常大的列表（614个数字）。 maven 使用下面的命令来帮助用户快速开始构建一个新的 Java 项目。

```
mvn archetype:generate
```

## 	什么是Archetype？

​	Archetype 是一个 Maven 插件，其任务是按照其模板来创建一个项目结构。在这里我们将使用 *quickstart* 原型插件来创建一个简单的 Java应用程序

## 使用项目模板

​	让我们打开命令控制台，进入到 **C:\>MVN** 目录，然后执行以下命令 mvn 命令，如下代码所示：

```
C:\MVN>mvn archetype:generate 
```

​	Maven 开始处理，并按要求选择所需的原型，执行结果如下图中所示：

```
INFO] Scanning for projects...
[INFO] Searching repository for plugin with prefix: 'archetype'.
[INFO] -------------------------------------------------------------------
[INFO] Building Maven Default Project
[INFO]    task-segment: [archetype:generate] (aggregator-style)
[INFO] -------------------------------------------------------------------
[INFO] Preparing archetype:generate
...
600: remote -> org.trailsframework:trails-archetype (-)
601: remote -> org.trailsframework:trails-secure-archetype (-)
602: remote -> org.tynamo:tynamo-archetype (-)
603: remote -> org.wicketstuff.scala:wicket-scala-archetype (-)
604: remote -> org.wicketstuff.scala:wicketstuff-scala-archetype 
Basic setup for a project that combines Scala and Wicket,
depending on the Wicket-Scala project. 
Includes an example Specs test.)
605: remote -> org.wikbook:wikbook.archetype (-)
606: remote -> org.xaloon.archetype:xaloon-archetype-wicket-jpa-glassfish (-)
607: remote -> org.xaloon.archetype:xaloon-archetype-wicket-jpa-spring (-)
608: remote -> org.xwiki.commons:xwiki-commons-component-archetype 
(Make it easy to create a maven project for creating XWiki Components.)
609: remote -> org.xwiki.rendering:xwiki-rendering-archetype-macro 
(Make it easy to create a maven project for creating XWiki Rendering Macros.)
610: remote -> org.zkoss:zk-archetype-component (The ZK Component archetype)
611: remote -> org.zkoss:zk-archetype-webapp (The ZK wepapp archetype)
612: remote -> ru.circumflex:circumflex-archetype (-)
613: remote -> se.vgregion.javg.maven.archetypes:javg-minimal-archetype (-)
614: remote -> sk.seges.sesam:sesam-annotation-archetype (-)
Choose a number or apply filter 
(format: [groupId:]artifactId, case sensitive contains): 203:
```

​	按 Enter 键选择默认选项（203：maven-archetype-quickstart）

​	Maven会要求原型的特定版本

```
Choose org.apache.maven.archetypes:maven-archetype-quickstart version:
1: 1.0-alpha-1
2: 1.0-alpha-2
3: 1.0-alpha-3
4: 1.0-alpha-4
5: 1.0
6: 1.1
Choose a number: 6:
```

​	按 Enter 键选择默认选项（6：maven-archetype-quickstart：1.1）

​	Maven会要求填写项目细节信息。如果要使用默认值可直接按回车。也可以通过输入自己的值覆盖它们。

```
Define value for property 'groupId': : com.companyname.insurance
Define value for property 'artifactId': : health
Define value for property 'version': 1.0-SNAPSHOT:
Define value for property 'package': com.companyname.insurance:
```

​	Maven会要求确认项目的细节信息，可按回车键或按 Y 来确认。

```
Confirm properties configuration:
groupId: com.companyname.insurance
artifactId: health
version: 1.0-SNAPSHOT
package: com.companyname.insurance
Y:
```

​	现在，Maven将开始创建项目结构，并会显示如下内容：

```
[INFO] -----------------------------------------------------------------------
[INFO] Using following parameters for creating project 
from Old (1.x) Archetype: maven-archetype-quickstart:1.1
[INFO] -----------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.companyname.insurance
[INFO] Parameter: packageName, Value: com.companyname.insurance
[INFO] Parameter: package, Value: com.companyname.insurance
[INFO] Parameter: artifactId, Value: health
[INFO] Parameter: basedir, Value: C:MVN
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] project created from Old (1.x) Archetype in dir: C:MVNhealth
[INFO] -----------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] -----------------------------------------------------------------------
[INFO] Total time: 4 minutes 12 seconds
[INFO] Finished at: Fri Jul 13 11:10:12 IST 2012
[INFO] Final Memory: 20M/90M
[INFO] -----------------------------------------------------------------------
```

## 	创建项目

​	现在进入到 C:\mvn 目录。会看到有一个 java 应用程序项目已创建了，它是在创建项目时给出 artifactId 命名：health 。 Maven 将创建一个标准的目录结构布局，如下图所示：

![project structure](/uploads/allimg/131227/0GK03P7-0.jpg)

## 	创建pom.xml

​	Maven 项目中的生成如下所列出 pom.xml 文件，其内容如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.companyname.insurance</groupId>
   <artifactId>health</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>jar</packaging>
   <name>health</name>
   <url>http://maven.apache.org</url>
   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
   <dependencies>
      <dependency>
      <groupId>junit</groupId>
         <artifactId>junit</artifactId>
         <version>3.8.1</version>
         <scope>test</scope>
      </dependency>
   </dependencies>
</project>
```

​	

## 	创建App.java

​	Maven 示例生成 Java 源文件，App.java下面列出项目：

​	位置：C:\>MVN\health\src\main\java\com\companyname\insurance> App.java**
**

```java
package com.companyname.insurance;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}
```





## 	创建 AppTest.java

​	Maven 实例生成 Java 源测试文件，项目中的 AppTest.java 测试文件如下面列出：

​	位置: C:\ > MVN > health > src > test > java > com > companyname > insurance > AppTest.java

```java
package com.companyname.insurance;

import junit.framework.Test;
import junit.framework.TestCase;
import junit.framework.TestSuite;

/**
 * Unit test for simple App.
 */
public class AppTest 
    extends TestCase
{
    /**
     * Create the test case
     *
     * @param testName name of the test case
     */
    public AppTest( String testName )
    {
        super( testName );
    }

    /**
     * @return the suite of tests being tested
     */
    public static Test suite()
    {
        return new TestSuite( AppTest.class );
    }

    /**
     * Rigourous Test :-)
     */
    public void testApp()
    {
        assertTrue( true );
    }
}
```

​	就是这样。现在就可以看到 Maven 的功能了。可以使用 maven 单一命令来创建任何类型的项目并开始开发。

# Maven快照

大型应用软件一般由多个模块组成，一般它是多个团队开发同一个应用程序的不同模块，这是比较常见的场景。例如，一个团队正在对应用程序的应用程序，用户界面项目(app-ui.jar:1.0) 的前端进行开发，他们使用的是数据服务工程 (data-service.jar:1.0)。

​	现在，它可能会有这样的情况发生，工作在数据服务团队开发人员快速地开发 bug 修复或增强功能，他们几乎每隔一天就要释放出库到远程仓库。

​	现在，如果数据服务团队上传新版本后，会出现下面的问题：

* 数据服务团队应该发布更新时每次都告诉应用程序UI团队，他们已经发布更新了代码。 	
* UI团队需要经常更新自己 pom.xml 以获得更新应用程序的版本。

为了处理这类情况，引入快照的概念，并发挥作用。

## 什么是快照？

​	快照（SNAPSHOT ）是一个特殊版本，指出目前开发拷贝。不同于常规版本，Maven 每生成一个远程存储库都会检查新的快照版本。

​	现在，数据服务团队将在每次发布代码后更新快照存储库为：data-service:1.0-SNAPSHOT 替换旧的 SNAPSHOT jar。

## 快照与版本

​	在使用版本时，如果 Maven 下载所提到的版本为 data-service:1.0，那么它永远不会尝试在库中下载已经更新的版本1.0。要下载更新的代码，data-service的版本必须要升级到1.1。

​	在使用快照（SNAPSHOT）时，Maven会在每次应用程序UI团队建立自己的项目时自动获取最新的快照（data-service:1.0-SNAPSHOT）。

## app-ui pom.xml

​	app-ui 项目使用数据服务（data-service）的 1.0-SNAPSHOT 

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>app-ui</groupId>
   <artifactId>app-ui</artifactId>
   <version>1.0</version>
   <packaging>jar</packaging>
   <name>health</name>
   <url>http://maven.apache.org</url>
   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
   <dependencies>
      <dependency>
      <groupId>data-service</groupId>
         <artifactId>data-service</artifactId>
         <version>1.0-SNAPSHOT</version>
         <scope>test</scope>
      </dependency>
   </dependencies>
</project>
```

## data-service pom.xml

​	数据服务（data-service）项目对于每一个微小的变化释放 1.0 快照：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>data-service</groupId>
   <artifactId>data-service</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>jar</packaging>
   <name>health</name>
   <url>http://maven.apache.org</url>
   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
   </project>
```

​	虽然，在使用快照（SNAPSHOT）时，Maven 自动获取最新的快照版本。不过我们也可以强制使用 -U 切换到任何 maven 命令来下载最新的快照版本。

```
mvn clean package -U
```

​	打开命令控制台，进入到 C: \MVN\app-ui 目录，然后执行以下命令mvn命令。

```
C:\MVN\app-ui>mvn clean package -U
```

​	Maven会下载数据服务的最新快照后并开始构建该项目，如下所输出：

```
[INFO] Scanning for projects...
[INFO] -------------------------------------------------------------------
[INFO] Building consumerBanking
[INFO]    task-segment: [clean, package]
[INFO] -------------------------------------------------------------------
[INFO] Downloading data-service:1.0-SNAPSHOT
[INFO] 290K downloaded.
[INFO] [clean:clean {execution: default-clean}]
[INFO] Deleting directory C:MVNapp-ui	arget
[INFO] [resources:resources {execution: default-resources}]
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources,
i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory C:MVNapp-uisrcmain
resources
[INFO] [compiler:compile {execution: default-compile}]
[INFO] Compiling 1 source file to C:MVNapp-ui	argetclasses
[INFO] [resources:testResources {execution: default-testResources}]
[WARNING] Using platform encoding (Cp1252 actually) to copy filtered resources,
i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory C:MVNapp-uisrc	est
resources
[INFO] [compiler:testCompile {execution: default-testCompile}]
[INFO] Compiling 1 source file to C:MVNapp-ui	arget	est-classes
[INFO] [surefire:test {execution: default-test}]
[INFO] Surefire report directory: C:MVNapp-ui	arget
surefire-reports
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.companyname.bank.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.027 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] [jar:jar {execution: default-jar}]
[INFO] Building jar: C:MVNapp-ui	arget
app-ui-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2 seconds
[INFO] Finished at: Tue Jul 10 16:52:18 IST 2015
[INFO] Final Memory: 16M/89M
[INFO] ------------------------------------------------------------------------
```

# Maven构建自动化


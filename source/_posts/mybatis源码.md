# 2. JDBC

# 2.1　JDBC API简介

使用JDBC操作数据源大致需要以下几个步骤：

（1）与数据源建立连接。

（2）执行SQL语句。

（3）检索SQL执行结果。

（4）关闭连接

### 2.1.1　建立数据源连接

​	（1）DriverManager：这是一个在JDBC 1.0规范中就已经存在、完全由JDBCAPI实现的驱动管理类。当应用程序第一次尝试通过URL连接数据源时，DriverManager会自动加载CLASSPATH下所有的JDBC驱动。DriverManager类提供了一系列重载的getConnection()方法，用来获取Connection对象，例如：

![](https://pic.imgdb.cn/item/61669a712ab3f51d917a1bce.jpg)

​	（2）DataSource：这个接口是在JDBC 2.0规范可选包中引入的API。它比DriverManager更受欢迎，因为它提供了更多底层数据源相关的细节，而且对应用来说，不需要关注JDBC驱动的实现。一个DataSource对象的属性被设置后，它就代表一个特定的数据源。当DataSource实例的getConnection()方法被调用后，DataSource实例就会返回一个与数据源建立连接的Connection对象。在应用程序中修改DataSource对象的属性后，就可以通过DataSource对象获取指向不同数据源的Connection对象。同样，数据源的具体实现修改后，不需要修改应用程序代码。

​	需要注意的是，JDBC API中只提供了DataSource接口，没有提供DataSource的具体实现，DataSource具体的实现由JDBC驱动程序提供。另外，目前一些主流的数据库连接池（例如DBCP、C3P0、Druid等）也提供了DataSource接口的具体实现。

### 2.1.2　执行SQL语句

​		通过2.1.1节的学习，我们了解到Connection是JDBC对数据源连接的抽象，一旦建立了连接，使用JDBC API的应用程序就可以对目标数据源执行查询和更新操作。JDBC API提供了访问SQL:2003规范中常用的实现特性，因为不同的JDBC厂商对这些特性的支持程度各不相同，所以JDBC API中提供了一个DatabaseMetadata接口，应用程序可以使用DatabaseMetadata的实例来确定目前使用的数据源是否支持某一特性。JDBC API中还定义了转意语法，让我们使用JDBC应用程序能够访问JDBC厂商提供的非标准的特性。

​	获取到JDBC中的Connection对象之后，我们可以通过Connection对象设置事务属性，并且可以通过Connection接口中提供的方法创建Statement、PreparedStatement或者CallableStatement对象。

​	Statement接口可以理解为JDBC API中提供的SQL语句的执行器，我们可以调用Statement接口中定义的executeQuery()方法执行查询操作，调用executeUpdate()方法执行更新操作，另外还可以调用executeBatch()方法执行批量处理。当我们不知道SQL语句的类型时，例如编写一个通用的方法，既可以执行查询语句，又可以执行更新语句，此时可以调用execute()方法进行统一的操作，然后通过execute()方法的返回值来判断SQL语句类型。最后可以通过Statement接口提供的getResultSet()方法来获取查询结果集，或者通过getUpdateCount()方法来获取更新操作影响的行数。

### 2.1.3　处理SQL执行结果

​	SQL语句执行完毕后，通常我们需要获取执行的结果，例如执行一条SELECT语句后，我们需要获取查询的结果，执行UPDATE或者INSERT语句后，我们需要通过影响的记录数来确定是否更新成功。JDBC API中提供了ResultSet接口，该接口的实现类封装SQL查询的结果，我们可以对ResultSet对象进行遍历，然后通过ResultSet提供的一系列getXXX()方法（例如getString）获取查询结果集。

### 2.1.4　使用JDBC操作数据库

​	略。

## 2.2　JDBC API中的类与接口

### 2.2.1　java.sql包详解

​	java.sql包中涵盖JDBC最核心的API，下面是java.sql包中的所有接口、枚举和类：

![](https://pic.imgdb.cn/item/61669b752ab3f51d917b71b0.jpg)

![](https://pic.imgdb.cn/item/61669b822ab3f51d917b82b7.jpg)

​	如上面的列表所示，java.sql包中的内容不多，大致可以分为数据类型接口、枚举类、驱动相关类和接口、异常类。

​	除这几部分外，剩下的就是作为Java开发人员需要掌握的API，主要包括下面几个接口：

![](https://pic.imgdb.cn/item/61669b972ab3f51d917b9e78.jpg)

​	这些接口都继承了java.sql.Wrapper接口。许多JDBC驱动程序提供超越传统JDBC的扩展，为了符合JDBC API规范，驱动厂商可能会在原始类型的基础上进行包装，Wrapper接口为使用JDBC的应用程序提供访问原始类型的功能，从而使用JDBC驱动中一些非标准的特性。

​	java.sql.Wrapper接口提供了两个方法，具体如下：

![](https://pic.imgdb.cn/item/61669be72ab3f51d917c0b5d.jpg)

​	如上面的代码所示，Oracle数据库驱动中提供了一些非JDBC标准的方法，如果需要使用这些非标准的方法，则可以调用Wrapper接口提供的unwrap()方法获取Oracle驱动的原始类型，然后调用原始类型提供的非标准方法就可以访问Oracle数据库特有的一些特性了。

​	JDBC API中的Connection、Statement、ResultSet等接口都继承自Wrapper接口，这些接口都提供了对JDBC驱动原始类型的访问能力。

​	Connection、Statement、ResultSet之间的关系如图2-1所示。

![](https://pic.imgdb.cn/item/61669c0f2ab3f51d917c44c9.jpg)

### 2.2.2　javax.sql包详解

​	javax.sql包中的类和接口最早是由JDBC 2.0版本的可选包提供的，这个可选包最早出现在J2SE 1.2版本中，这个包中的内容不多，主要包括下面几个类和接口：

![](https://pic.imgdb.cn/item/61669c7b2ab3f51d917cdc2d.jpg)

​	JDBC 1.0中使用DriverManager类来产生一个与数据源连接的Connection对象。相对于DriverManager，JDBC 2.0提供的DataSource接口是一个更好的连接数据源的方式。

​	首先，应用程序不需要像使用DriverManager一样对加载的数据库驱动程序信息进行硬编码。开发人员可以选择通过JNDI注册这个数据源对象，然后在程序中使用一个逻辑名称来引用它，JNDI会自动根据我们给出的名称找到与这个名称绑定的DataSource对象。然后我们就可以使用这个DataSource对象来建立和具体数据库的连接了。

​	其次，使用DataSource接口的第二个优势体现在连接池和分布式事务上。连接池通过对连接的复用，而不是每次需要操作数据源时都新建一个物理连接来显著地提高程序的效率，适用于任务繁忙、负担繁重的企业级应用。

​	javax.sql.DataSource与java.sql.Connection之间的关系如图2-2所示。

![	](https://pic.imgdb.cn/item/61669cbc2ab3f51d917d32e8.jpg)

​	javax.sql包下还提供了一个PooledConnection接口。PooledConnection和Connection的不同之处在于，它提供了连接池管理的句柄。一个PooledConnection表示与数据源建立的物理连接，该连接在应用程序使用完后可以回收而不用关闭它，从而减少了与数据源建立连接的次数。

​	应用程序开发人员一般不会直接使用PooledConnection接口，而是通过一个管理连接池的中间层基础设施使用。当应用程序调用DataSource对象的getConnection()方法时，它返回一个Connection对象。但是当我们使用数据库连接池时（例如Druid），该Connection对象实际上是到PooledConnection对象的句柄，这是一个物理连接。连接池管理器（通常为应用程序服务器）维护所有的PooledConnection 对象资源。如果在池中存在可用的PooledConnection对象，则连接池管理器返回作为到该物理连接的句柄的Connection对象。如果不存在可用的PooledConnection对象，则连接池管理器调用ConnectionPoolDataSource对象的getConnection()方法创建新的物理连接。

​	连接池实现模块可以调用PooledConnection对象的addConnectionEventListener()将自己注册成为一个PooledConnection对象的监听者，当数据库连接需要重用或关闭的时候会产生一个ConnectionEvent对象，它表示一个连接事件，连接池实现模块将会得到通知。

​	javax.sql.PooledConnection与java.sql.Connection之间的关系如图2-3所示。

![](https://pic.imgdb.cn/item/61669cec2ab3f51d917d6f7b.jpg)

​	另外，javax.sql包中还包含XADataSource、XAResource和XAConnection接口，这些接口提供了分布式事务的支持，具体由JDBC驱动来实现。更多分布式事务相关细节可参考JTA（Java Transaction API）规范文档。

​	XAConnection接口继承了PooledConnection接口，因此它具有所有PooledConnection的特性，我们可以调用XAConnection实例的getConnection()方法获取java.sql.Connection对象，它们与java.sql.Connection之间的关系如图2-4所示。

![](https://pic.imgdb.cn/item/61669d1d2ab3f51d917dacfb.jpg)

​	JTA规范文档：http://download.oracle.com/otndocs/jcp/jta-1.1-spec-oth-JSpec/?submit=Download。

​	javax.sql包中还提供了一个RowSet接口，该接口继承自java.sql包下的ResultSet接口。RowSet用于为数据源和应用程序在内容中建立一个映射。RowSet对象可以建立一个与数据源的连接并在其整个生命周期中维持该连接，在这种情况下，该对象被称为连接的RowSet。RowSet对象还可以建立一个与数据源的连接，从其获取数据，然后关闭它，这种RowSet被称为非连接RowSet。非连接Rowset可以在断开时更改其数据，然后将这些更改写回底层数据源，不过它必须重新建立连接才能完成此操作。

​	相较于java.sql.ResultSet而言，RowSet的离线操作能够有效地利用计算机越来越充足的内存减轻数据库服务器的负担。由于数据操作都是在内存中进行的，然后批量提交到数据源，因此灵活性和性能都有了很大的提高。

​	RowSet默认是一个可滚动、可更新、可序列化的结果集，而且它作为一个JavaBean组件，可以方便地在网络间传输，用于两端的数据同步。通俗来讲，RowSet就相当于数据库表数据在应用程序内存中的映射，我们所有的操作都可以直接与RowSet对象交互。RowSet与数据库之间的数据同步，作为开发人员不需要关注。

![](https://pic.imgdb.cn/item/61669d492ab3f51d917de93f.jpg)

### 2.3.2　java.sql.Driver接口

​	所有的JDBC驱动都必须实现Driver接口，而且实现类必须包含一个静态初始化代码块。我们知道，类的静态初始化代码块会在类初始化时调用，驱动实现类需要在静态初始化代码块中向DriverManager注册自己的一个实例，例如：

![](https://pic.imgdb.cn/item/6166a1c52ab3f51d91838b62.jpg)

​	当我们加载驱动实现类时，上面的静态初始化代码块就会被调用，向DriverManager中注册一个驱动类的实例。这就是为什么我们使用JDBC操作数据库时一般会先加载驱动。

​	为了确保驱动程序可以使用这种机制加载，Driver实现类需要提供一个无参数的构造方法。		

​	DriverManager类与注册的驱动程序进行交互时会调用Driver接口中提供的方法。Driver接口中提供了一个acceptsURL()方法，DriverManager类可以通过Driver实现类的acceptsURL()来判断一个给定的URL是否能与数据库成功建立连接。当我们试图使用DriverManager与数据库建立连接时，会调用Driver接口中提供的connect()方法

> **注意**
>
> ​	在DriverManager类初始化时，会试图加载所有jdbc.drivers属性指定的驱动类，因此我们可以通过jdbc.drivers属性来加载驱动。

​	DriverManager类中定义了静态初始化代码块，代码如下：

![](https://pic.imgdb.cn/item/6166a9f32ab3f51d918cfcf6.jpg)

​	如上面的代码所示，DriverManager类的静态代码块会在我们调用DriverManager的getConnection()方法之前调用。静态代码块中调用loadInitialDrivers()方法加载驱动实现类，该方法的关键代码如下：

![](https://pic.imgdb.cn/item/6166aa002ab3f51d918d0928.jpg)

​	如上面的代码所示，在loadInitialDrivers()方法中，通过JDK内置的ServiceLoader机制加载java.sql.Driver接口的实现类，然后对所有实现类进行遍历，这样就完成了驱动类的加载。驱动实现类会在自己的静态代码块中将驱动实现类的实例注册到DriverManager中，这样就取代了通过调用Class.forName()方法加载驱动的过程。

### 2.3.4 java.sql.DriverAction接口

​	前面我们了解到，Driver实现类在被加载时会调用DriverManager类的registerDriver()方法注册驱动。我们也可以在应用程序中显式地调用DriverManager类的deregisterDriver()方法来解除注册。JDBC驱动可以通过实现DriverAction接口来监听DriverManager类的deregisterDriver()方法的调用。

​	JDBC规范中不建议DriverAction接口的实现类在应用程序中被使用，因此DriverAction实现类通常会作为私有的内部类，从而避免被其他程序使用。

​	JDBC驱动的静态初始化代码块可以调用DriverManager.registerDriver(java.sql.Driver,java.sql.DriverAction)方法来确保DriverManager类的deregisterDriver()方法调用被监听，例如：

![](https://pic.imgdb.cn/item/6166ac0f2ab3f51d918f3865.jpg)

### 2.3.5　java.sql.DriverManager类

​	DriverManager类通过Driver接口为JDBC客户端管理一组可用的驱动实现，当客户端通过DriverManager类和数据库建立连接时，DriverManager类会根据getConnection()方法参数中的URL找到对应的驱动实现类，然后使用具体的驱动实现连接到对应的数据库。

**DriverManager类提供了两个关键的静态方法：**

​	registerDriver()：该方法用于将驱动的实现类注册到DriverManager类中，这个方法会在驱动加载时隐式地调用，而且通常在每个驱动实现类的静态初始化代码块中调用。

​	getConnection()：这个方法是提供给JDBC客户端调用的，可以接收一个JDBC URL作为参数，DriverManager类会对所有注册驱动进行遍历，调用Driver实现的connect()方法找到能够识别JDBC URL的驱动实现后，会与数据库建立连接，然后返回Connection对象。

​	JDBC URL的格式如下：jdbc:`<subprotocol>:<subname>subprotocol`用于指定数据库连接机制由一个或者多个驱动程序提供支持，subname的内容取决于subprotocol。

### 2.3.6　javax.sql.DataSource接口

​	javax.sql.DataSource接口最早是由JDBC 2.0版本扩展包提供的，它是比较推荐的获取数据源连接的一种方式，JDBC驱动程序都会实现DataSource接口，通过DataSource实现类的实例，返回一个Connection接口的实现类的实例。

​	使用DataSource对象可以提高应用程序的可移植性。在应用程序中，可以通过逻辑名称来获取DataSource对象，而不用为特定的驱动指定特定的信息。我们可以使用JNDI（Java Naming and Directory Interface）把一个逻辑名称和数据源对象建立映射关系。

​	DataSource对象用于表示能够提供数据库连接的数据源对象。如果数据库相关的信息发生了变化，则可以简单地修改DataSource对象的属性来反映这种变化，而不用修改应用程序的任何代码。

​	DataSource接口可以被实现，提供如下两种功能：

​		通过连接池提高系统性能和伸缩性。

​		通过XADataSource接口支持分布式事务。

> 注意
>
> DataSource接口的实现必须包含一个无参构造方法。



​	JDBC API中定义了一组属性来表示和描述数据源实现。具体有哪些属性，取决于DataSource对象的类型，包括DataSource、ConnectionPoolDataSource和XADataSource。表2-1是DataSource所有标准属性及其描述。

![](https://pic.imgdb.cn/item/6166adff2ab3f51d91917c77.jpg)

​	DataSource属性遵循JavaBeans 1.01规范中对JavaBean组件属性的约定，可以在这些属性的基础上增加一些特定的属性扩展（这些扩展的属性不能与标准属性冲突）。DataSource实现类必须为支持的每个属性提供对应的Getter和Setter方法，而且这些属性需要在创建DataSource对象时初始化。

​	DataSource对象的属性不建议被JDBC客户端直接访问，可以通过增强DataSource实现类的属性访问方法来实现，而不是在应用程序中使用DataSource接口时控制。此外，客户端所操作的对象可以是实现了DataSource接口的包装类，它的属性对应的Setter和Getter方法不需要暴露给客户端。一些管理工具如果需要访问DataSource实现类的属性，则可以使用Java的内省机制。


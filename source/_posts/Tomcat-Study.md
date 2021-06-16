---
title: Tomcat-Study
date: 2021-05-10 20:01:12
tags: [Tomcat, Web容器]
---

# How Tomcat Works

## 简单Web Server

* java.net.Socket
* java.net.ServerSocket



### HTTP

* 请求
  * URI和版本
  * 请求头
  * 实体
* 响应
  * 协议状态，返回码，描述。
  * 响应头
  * 实体

### Socket类

* an endpoint of a network connection
* `public Socket (java.lang.String host, int port)`
* 发送
  * 通过getOutputStream获取java.io.OutputStream对象。
  * 通过流构造PrintWriter，然后发送消息。
* 接收消息
  * 通过getInputStream获取java.io.InputStream对象。



实例

```java
public class test1 {
    public static void main(String[] args) throws IOException, InterruptedException {
        Socket socket = new Socket("127.0.0.1", 8080);
        OutputStream os = socket.getOutputStream();
        boolean autoflush = true;
        PrintWriter out = new PrintWriter(socket.getOutputStream(), autoflush);
        BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        // send an HTTP request to the web server
        out.println("GET /index.jsp HTTP/1.1");
        out.println("Host: localhost:8080");
        out.println("Connection: Close");
        out.println();
        // read the response
        boolean loop = true;
        StringBuffer sb = new StringBuffer(8096);
        while (loop) {
            if (in.ready()) {
                int i = 0;
                while (i != -1) {
                    i = in.read();
                    sb.append((char) i);
                }
                loop = false;
            }
            Thread.currentThread().sleep(50);
        }
        System.out.println(sb.toString());
        socket.close();
    }
} // display the response to the out console
```

### ServerSocket类

* Socket用于客户端，ServerSocket用于服务端。

* java.net.ServerSocket
* 等待客户端连接请求。
* 接收到一个请求后，会创建一个Socket实例去和客户端交互。

```java
public ServerSocket(int port, int backLog, InetAddress bindingAddress);
```

​	InetAddress可以通过byName快速创建。

```java
InetAddress.getByName("127.0.0.1");
new ServerSocket(8080, 1, InetAddress.getByName("127.0.0.1"));
```

### 应用

* HttpServer
  * 入口(main方法)
* Request
* Response

**HttpServer**

```java
public class HttpServer {

  /** WEB_ROOT is the directory where our HTML and other files reside.
   *  For this package, WEB_ROOT is the "webroot" directory under the working
   *  directory.
   *  The working directory is the location in the file system
   *  from where the java command was invoked.
   */
  public static final String WEB_ROOT =
    System.getProperty("user.dir") + File.separator  + "webroot";

  // shutdown command
  private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";

  // the shutdown command received
  private boolean shutdown = false;

  public static void main(String[] args) {
    HttpServer server = new HttpServer();
    server.await();
  }

  public void await() {
    ServerSocket serverSocket = null;
    int port = 8080;
    try {
      serverSocket =  new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
    }
    catch (IOException e) {
      e.printStackTrace();
      System.exit(1);
    }

    // Loop waiting for a request
    while (!shutdown) {
      Socket socket = null;
      InputStream input = null;
      OutputStream output = null;
      try {
        socket = serverSocket.accept();
        input = socket.getInputStream();
        output = socket.getOutputStream();

        // create Request object and parse
        Request request = new Request(input);
        request.parse();

        // create Response object
        Response response = new Response(output);
        response.setRequest(request);
        response.sendStaticResource();

        // Close the socket
        socket.close();

        //check if the previous URI is a shutdown command
        shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
      }
      catch (Exception e) {
        e.printStackTrace();
        continue;
      }
    }
  }
}
```

**Request类**

```java
public class Request {

  private InputStream input;
  private String uri;

  public Request(InputStream input) {
    this.input = input;
  }

  public void parse() {
    // Read a set of characters from the socket
    StringBuffer request = new StringBuffer(2048);
    int i;
    byte[] buffer = new byte[2048];
    try {
      i = input.read(buffer);
    }
    catch (IOException e) {
      e.printStackTrace();
      i = -1;
    }
    for (int j=0; j<i; j++) {
      request.append((char) buffer[j]);
    }
    System.out.print(request.toString());
    uri = parseUri(request.toString());
  }

  private String parseUri(String requestString) {
    int index1, index2;
    index1 = requestString.indexOf(' ');
    if (index1 != -1) {
      index2 = requestString.indexOf(' ', index1 + 1);
      if (index2 > index1)
        return requestString.substring(index1 + 1, index2);
    }
    return null;
  }

  public String getUri() {
    return uri;
  }

}
```

**Response类**

```java
public class Response {

  private static final int BUFFER_SIZE = 1024;
  Request request;
  OutputStream output;

  public Response(OutputStream output) {
    this.output = output;
  }

  public void setRequest(Request request) {
    this.request = request;
  }

  public void sendStaticResource() throws IOException {
    byte[] bytes = new byte[BUFFER_SIZE];
    FileInputStream fis = null;
    try {
      File file = new File(HttpServer.WEB_ROOT, request.getUri());
      if (file.exists()) {
        fis = new FileInputStream(file);
        int ch = fis.read(bytes, 0, BUFFER_SIZE);
        while (ch!=-1) {
          output.write(bytes, 0, ch);
          ch = fis.read(bytes, 0, BUFFER_SIZE);
        }
      }
      else {
        // file not found
        String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
          "Content-Type: text/html\r\n" +
          "Content-Length: 23\r\n" +
          "\r\n" +
          "<h1>File Not Found</h1>";
        output.write(errorMessage.getBytes());
      }
    }
    catch (Exception e) {
      // thrown if cannot instantiate a File object
      System.out.println(e.toString() );
    }
    finally {
      if (fis!=null)
        fis.close();
    }
  }
}
```

## A Simple Servlet Container

​	本章通过两个程序来说明你如何开发自己的servlet 容器。第一个程序被设计得足够简单使
得你能理解一个servlet 容器是如何工作的。然后它演变为第二个稍微复杂的servlet 容器。

​	我们可以使用PrimitiveServlet来测试container。

```java
public class PrimitiveServlet implements Servlet {
    public PrimitiveServlet() {
    }

    public void init(ServletConfig var1) throws ServletException {
        System.out.println("init");
    }

    public void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException {
        System.out.println("from service");
        PrintWriter var3 = var2.getWriter();
        var3.println("Hello. Roses are red.");
        var3.print("Violets are blue.");
    }

    public void destroy() {
        System.out.println("destroy");
    }

    public String getServletInfo() {
        return null;
    }

    public ServletConfig getServletConfig() {
        return null;
    }
}
```

### The javax.servlet.Servlet Interface

* javax.servlet 
* javax.servlet.http.



​	javax.servlet.Servlet尤为重要

> All servlets must implement this interface or extend a class that does.All servlets must implement this interface or extend a class that does.

​	init, service, and destroy are life cycle methods.

* init 方法由servlet container在servlet类实例化时调用。
* 接收到任何请求前，必须初始化完毕。



service

* 有请求时，container调用service方法。
* container传递javax.servlet.ServletRequest object and a javax.servlet.ServletResponse object给service方法。



destroy

* servlet实例从service移除时，调用销毁方法。
* 一般发生在关闭容器或者需要一些空闲空间时。
* 提供了清理资源的能力。

### 应用1

![](https://pic.imgdb.cn/item/609a2ff5d1a9ae528f98d4c6.jpg)

​	servlet容器对每个HTTP

* 当第一次调用servlet 的时候，加载该servlet 类并调用servlet 的init 方法(仅仅一
  次)。
* 对每次请求，构造一个javax.servlet.ServletRequest 实例和一个
  javax.servlet.ServletResponse 实例。
* 调用servlet 的service 方法，同时传递ServletRequest 和ServletResponse 对象。
* 当servlet 类被关闭的时候，调用servlet 的destroy 方法并卸载servlet 类。



​	本章的第一个servlet 容器不是全功能的。因此，她不能运行什么除了非常简单的servlet，
而且也不调用servlet 的init 方法和destroy 方法。相反它做了下面的事情：

* 等待HTTP 请求。
* 构造一个ServletRequest 对象和一个ServletResponse 对象。
* 假如该请求需要一个静态资源的话，调用StaticResourceProcessor 实例的process 方
  法，同时传递ServletRequest 和ServletResponse 对象。
* 假如该请求需要一个servlet 的话，加载servlet 类并调用servlet 的service 方法，
  同时传递ServletRequest 和ServletResponse 对象。



​	每一次servlet 被请求的时候，servlet 类都会被加载。第一个应用程序由6 个类组成：

* HttpServer1
* Request
* Response
* StaticResourceProcessor
* ServletProcessor1
* Constants

**HttpServer1 类**

```java
package ex02.pyrmont;

import java.net.Socket;
import java.net.ServerSocket;
import java.net.InetAddress;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.IOException;

public class HttpServer1 {

  /** WEB_ROOT is the directory where our HTML and other files reside.
   *  For this package, WEB_ROOT is the "webroot" directory under the working
   *  directory.
   *  The working directory is the location in the file system
   *  from where the java command was invoked.
   */
  // shutdown command
  private static final String SHUTDOWN_COMMAND = "/SHUTDOWN";

  // the shutdown command received
  private boolean shutdown = false;

  public static void main(String[] args) {
    HttpServer1 server = new HttpServer1();
    server.await();
  }

  public void await() {
    ServerSocket serverSocket = null;
    int port = 8080;
    try {
      serverSocket =  new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
    }
    catch (IOException e) {
      e.printStackTrace();
      System.exit(1);
    }

    // Loop waiting for a request
    while (!shutdown) {
      Socket socket = null;
      InputStream input = null;
      OutputStream output = null;
      try {
        socket = serverSocket.accept();
        input = socket.getInputStream();
        output = socket.getOutputStream();

        // create Request object and parse
        Request request = new Request(input);
        request.parse();

        // create Response object
        Response response = new Response(output);
        response.setRequest(request);

        // check if this is a request for a servlet or a static resource
        // a request for a servlet begins with "/servlet/"
        if (request.getUri().startsWith("/servlet/")) {
          ServletProcessor1 processor = new ServletProcessor1();
          processor.process(request, response);
        }
        else {
          StaticResourceProcessor processor = new StaticResourceProcessor();
          processor.process(request, response);
        }

        // Close the socket
        socket.close();
        //check if the previous URI is a shutdown command
        shutdown = request.getUri().equals(SHUTDOWN_COMMAND);
      }
      catch (Exception e) {
        e.printStackTrace();
        System.exit(1);
      }
    }
  }
}
```

**Request 类**

```java
public class Request implements ServletRequest {

  private InputStream input;
  private String uri;

  public Request(InputStream input) {
    this.input = input;
  }

  public String getUri() {
    return uri;
  }

  private String parseUri(String requestString) {
    int index1, index2;
    index1 = requestString.indexOf(' ');
    if (index1 != -1) {
      index2 = requestString.indexOf(' ', index1 + 1);
      if (index2 > index1)
        return requestString.substring(index1 + 1, index2);
    }
    return null;
  }

  public void parse() {
    // Read a set of characters from the socket
    StringBuffer request = new StringBuffer(2048);
    int i;
    byte[] buffer = new byte[2048];
    try {
      i = input.read(buffer);
    }
    catch (IOException e) {
      e.printStackTrace();
      i = -1;
    }
    for (int j=0; j<i; j++) {
      request.append((char) buffer[j]);
    }
    System.out.print(request.toString());
    uri = parseUri(request.toString());
  }
  ...
 }
```

Response 类****

```java
package ex02.pyrmont;

import java.io.OutputStream;
import java.io.IOException;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.File;
import java.io.PrintWriter;
import java.util.Locale;
import javax.servlet.ServletResponse;
import javax.servlet.ServletOutputStream;

public class Response implements ServletResponse {

  private static final int BUFFER_SIZE = 1024;
  Request request;
  OutputStream output;
  PrintWriter writer;

  public Response(OutputStream output) {
    this.output = output;
  }

  public void setRequest(Request request) {
    this.request = request;
  }

  /* This method is used to serve a static page */
  public void sendStaticResource() throws IOException {
    byte[] bytes = new byte[BUFFER_SIZE];
    FileInputStream fis = null;
    try {
      /* request.getUri has been replaced by request.getRequestURI */
      File file = new File(Constants.WEB_ROOT, request.getUri());
      fis = new FileInputStream(file);
      /*
         HTTP Response = Status-Line
           *(( general-header | response-header | entity-header ) CRLF)
           CRLF
           [ message-body ]
         Status-Line = HTTP-Version SP Status-Code SP Reason-Phrase CRLF
      */
      int ch = fis.read(bytes, 0, BUFFER_SIZE);
      while (ch!=-1) {
        output.write(bytes, 0, ch);
        ch = fis.read(bytes, 0, BUFFER_SIZE);
      }
    }
    catch (FileNotFoundException e) {
      String errorMessage = "HTTP/1.1 404 File Not Found\r\n" +
        "Content-Type: text/html\r\n" +
        "Content-Length: 23\r\n" +
        "\r\n" +
        "<h1>File Not Found</h1>";
      output.write(errorMessage.getBytes());
    }
    finally {
      if (fis!=null)
        fis.close();
    }
  }


  /** implementation of ServletResponse  */
  public void flushBuffer() throws IOException {
  }

  public int getBufferSize() {
    return 0;
  }

  public String getCharacterEncoding() {
    return null;
  }

  public Locale getLocale() {
    return null;
  }

  public ServletOutputStream getOutputStream() throws IOException {
    return null;
  }

  public PrintWriter getWriter() throws IOException {
    // autoflush is true, println() will flush,
    // but print() will not.
    writer = new PrintWriter(output, true);
    return writer;
  }

  public boolean isCommitted() {
    return false;
  }

  public void reset() {
  }

  public void resetBuffer() {
  }

  public void setBufferSize(int size) {
  }

  public void setContentLength(int length) {
  }

  public void setContentType(String type) {
  }

  public void setLocale(Locale locale) {
  }
}
```

**StaticResourceProcessor 类**

```java
package ex02.pyrmont;

import java.io.IOException;

public class StaticResourceProcessor {

  public void process(Request request, Response response) {
    try {
      response.sendStaticResource();
    }
    catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

**ServletProcessor1 类**

用于处理servlet 的HTTP 请求。

```java
package ex02.pyrmont;

import java.net.URL;
import java.net.URLClassLoader;
import java.net.URLStreamHandler;
import java.io.File;
import java.io.IOException;
import javax.servlet.Servlet;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public class ServletProcessor1 {

  public void process(Request request, Response response) {

    String uri = request.getUri();
    String servletName = uri.substring(uri.lastIndexOf("/") + 1);
    URLClassLoader loader = null;

    try {
      // create a URLClassLoader
      URL[] urls = new URL[1];
      URLStreamHandler streamHandler = null;
      File classPath = new File(Constants.WEB_ROOT);
      // the forming of repository is taken from the createClassLoader method in
      // org.apache.catalina.startup.ClassLoaderFactory
      String repository = (new URL("file", null, classPath.getCanonicalPath() + File.separator)).toString() ;
      // the code for forming the URL is taken from the addRepository method in
      // org.apache.catalina.loader.StandardClassLoader class.
      urls[0] = new URL(null, repository, streamHandler);
      loader = new URLClassLoader(urls);
    }
    catch (IOException e) {
      System.out.println(e.toString() );
    }
    Class myClass = null;
    try {
      myClass = loader.loadClass(servletName);
    }
    catch (ClassNotFoundException e) {
      System.out.println(e.toString());
    }

    Servlet servlet = null;

    try {
      servlet = (Servlet) myClass.newInstance();
      servlet.service((ServletRequest) request, (ServletResponse) response);
    }
    catch (Exception e) {
      System.out.println(e.toString());
    }
    catch (Throwable e) {
      System.out.println(e.toString());
    }
  }
}
```

​	process 方法加载servlet。要完成这个，你需要创建一个类加载器并告诉这个类加载器要加载的类的位置。对于这个servlet 容器，类加载器直接在Constants 指向的目录里边查找。WEB_ROOT 就是指向工作目录下面的webroot 目录。

​	要加载servlet，你可以使用java.net.URLClassLoader 类，它是java.lang.ClassLoader类的一个直接子类。一旦你拥有一个URLClassLoader 实例，你使用它的loadClass 方法去加载一个servlet 类

```java
public URLClassLoader(URL[] urls);
```

​	urls 是一个java.net.URL 的对象数组，这些对象指向了加载类时候查找的位置。

​	对于这个应用程序来说，我们使用Tomcat 中的另一个类的相同的构造方法。

```java
public URL(URL context, java.lang.String spec, URLStreamHandler hander)
	throws MalformedURLException;
public URL(java.lang.String protocol, java.lang.String host,
	java.lang.String file) throws MalformedURLException;
```

​	当有了一个类加载器，你可以使用loadClass 方法加载一个servlet：

```java
Class myClass = null;
try {
	myClass = loader.loadClass(servletName);
} catch (ClassNotFoundException e) {
	System.out.println(e.toString());
}
```

要在Windows 上运行该应用程序，在工作目录下面敲入以下命令：j

java -classpath ./lib/servlet.jar;./ ex02.pyrmont.HttpServer1
在Linux 下，你使用一个冒号来分隔两个库：
java -classpath ./lib/servlet.jar:./ ex02.pyrmont.HttpServer1
要测试该应用程序，在浏览器的地址栏或者网址框中敲入：

### 应用2

​	第一个应用程序有一个严重的问题。在ServletProcessor1 类的process 方法，你向上转换ex02.pyrmont.Request 实例为javax.servlet.ServletRequest，并作为第一个参数传递给servlet 的service 方法。你也向下转换ex02.pyrmont.Response 实例为javax.servlet.ServletResponse，并作为第二个参数传递给servlet 的service 方法。

```java
try {
	servlet = (Servlet) myClass.newInstance();
	servlet.service((ServletRequest) request,(ServletResponse) response);
}
```

​	这会危害安全性。知道这个servlet 容器的内部运作的Servlet 程序员可以分别把ServletRequest 和ServletResponse 实例向下转换为ex02.pyrmont.Request 和ex02.pyrmont.Response，并调用他们的公共方法。拥有一个Request 实例，它们就可以调用parse方法。拥有一个Response 实例，就可以调用sendStaticResource 方法。

​	你不可以把parse 和sendStaticResource 方法设置为私有的，因为它们将会被其他的类调用。不过，这两个方法是在个servlet 内部是不可见的。其中一个解决办法就是让Request 和Response 类拥有默认访问修饰，所以它们不能在ex02.pyrmont 包的外部使用。不过，这里有一个更优雅的解决办法：通过使用facade 类。请看Figure 2.2 中的UML 图。

![](https://pic.imgdb.cn/item/609a332dd1a9ae528fbb666f.jpg)

​	在这第二个应用程序中，我们增加了两个façade 类: RequestFacade 和ResponseFacade。RequestFacade 实现了ServletRequest 接口并通过在构造方法中传递一个引用了ServletRequest 对象的Request 实例作为参数来实例化。ServletRequest 接口中每个方法的实现都调用了Request 对象的相应方法。然而ServletRequest 对象􁴀身是私有的，并不能在类的外部访问。我们构造了一个RequestFacade 对象并把它传递给service 方法，而不是向下转换Request 对象为ServletRequest 对象并传递给service 方法。Servlet 程序员仍然可以向下转换ServletRequest 实例为RequestFacade，不过它们只可以访问ServletRequest 接口里边的公共方法。现在parseUri 方法就是安全的了。

```java
public class RequestFacade implements ServletRequest {

  private ServletRequest request = null;

  public RequestFacade(Request request) {
    this.request = request;
  }

  /* implementation of the ServletRequest*/
  public Object getAttribute(String attribute) {
    return request.getAttribute(attribute);
  }

  public Enumeration getAttributeNames() {
    return request.getAttributeNames();
  }
  ...
}
```

## 连接器

​	Catalina 中有两个主要的模块：连接器和容器。

​	一个符合Servlet 2.3 和2.4规范的连接器必须创建javax.servlet.http.HttpServletRequest 和javax.servlet.http.HttpServletResponse，并传递给被调用的servlet 的service 方法。

​	在本章的应用程序中，连接器解析HTTP 请求头部并让servlet 可以获得头部, cookies, 参数名/值等等。你将会完善第2 章中Response 类的getWriter 方法，让它能够正确运行。由于这些改进，你将会从 PrimitiveServlet 中获取一个完整的响应，并能够运行更加复杂的ModernServlet。

**StringManager 类**

​	Tomcat 所采用的方法是在一个属性文件里边存储错误信息，这样，可以容易的修改这些信息。不过，Tomcat 中有数以百计的类。把所有类使用的错误信 息存储到一个大的属性文件里边将会容易产生维护的噩梦。为了避免这一情况，Tomcat 为每个包都分配一个属性文件。例如，在包 org.apache.catalina.connector 里边的属性文件包􀧿了该包所有的类抛出的所有错误信息。每个属性文件都会被一个 org.apache.catalina.util.StringManager 类的实例所处理。

```java
private static Hashtable managers = new Hashtable();
public synchronized static StringManager
	getManager(String packageName) {
	StringManager mgr = (StringManager)managers.get(packageName);
	if (mgr == null) {
		mgr = new StringManager(packageName);
		managers.put(packageName, mgr);
	}
	return mgr;
}
```

### 应用

​	从本章开始，每章附带的应用程序都会分成模块。这章的应用程序由三个模块组成：connector,startup 和core。

​	startup 模块只有一个类，Bootstrap，用来启动应用的。connector 模块的类可以分为五组：

* 连接器和它的支撑类(HttpConnector 和HttpProcessor)。
* 指代HTTP 请求的类(HttpRequest)和它的辅助类。
* 指代HTTP 响应的类(HttpResponse)和它的辅助类。
* Facade 类(HttpRequestFacade 和HttpResponseFacade)。
* Constant 类。



​	core 模块由两个类组成：ServletProcessor 和StaticResourceProcessor。

<img src="https://pic.imgdb.cn/item/609a3ce5d1a9ae528f1d8458.jpg" style="zoom:200%;" />

​	HttpServer 类被分离为两个类：HttpConnector和HttpProcessor，Request 被 HttpRequest 所取代，而Response 被HttpResponse 所取代。同样，本章的应用使用了更多的类。

* HTTP 请求对象由实现了javax.servlet.http.HttpServletRequest 的HttpRequest
  类来代表。
* Tomcat的默认连接器和我们的连接器使用SocketInputStream 类来从套接字的InputStream中读取字节流。

### 启动应用程序

```java
package ex03.pyrmont.startup;
import ex03.pyrmont.connector.http.HttpConnector;
public final class Bootstrap {
	public static void main(String[] args) {
		HttpConnector connector = new HttpConnector();
		connector.start();
	}
}
```



```java
public class HttpConnector implements Runnable {

  boolean stopped;
  private String scheme = "http";

  public String getScheme() {
    return scheme;
  }

  public void run() {
    ServerSocket serverSocket = null;
    int port = 8080;
    try {
      serverSocket =  new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
    }
    catch (IOException e) {
      e.printStackTrace();
      System.exit(1);
    }
    while (!stopped) {
      // Accept the next incoming connection from the server socket
      Socket socket = null;
      try {
        socket = serverSocket.accept();
      }
      catch (Exception e) {
        continue;
      }
      // Hand this socket off to an HttpProcessor
      HttpProcessor processor = new HttpProcessor(this);
      processor.process(socket);
    }
  }

  public void start() {
    Thread thread = new Thread(this);
    thread.start();
  }
}
```

### 连接器

* 等待http请求
* 为每个请求创建HttpProccessor实例。
* 调用HttpProccessor.process

HttpProcessor 类的process 方法接受前来的HTTP 请求的套接字，会做下面的事情

* 创建一个HttpRequest 对象。
* 创建一个HttpResponse 对象。
* 解析HTTP 请求的第一行和头部，并放到HttpRequest 对象。
* 解析HttpRequest 和HttpResponse 对象到一个ServletProcessor 或者
  StaticResourceProcessor。

```java
public void process(Socket socket) {
    SocketInputStream input = null;
    OutputStream output = null;
    try {
      input = new SocketInputStream(socket.getInputStream(), 2048);
      output = socket.getOutputStream();

      // create HttpRequest object and parse
      request = new HttpRequest(input);

      // create HttpResponse object
      response = new HttpResponse(output);
      response.setRequest(request);

      response.setHeader("Server", "Pyrmont Servlet Container");

      parseRequest(input, output);
      parseHeaders(input);

      //check if this is a request for a servlet or a static resource
      //a request for a servlet begins with "/servlet/"
      if (request.getRequestURI().startsWith("/servlet/")) {
        ServletProcessor processor = new ServletProcessor();
        processor.process(request, response);
      }
      else {
        StaticResourceProcessor processor = new StaticResourceProcessor();
        processor.process(request, response);
      }

      // Close the socket
      socket.close();
      // no shutdown for this application
    }
    catch (Exception e) {
      e.printStackTrace();
    }
  }
```

**创建一个HttpRequest 对象**

![](https://pic.imgdb.cn/item/609a4a4ad1a9ae528f843e99.jpg)

​	servlet 程序员已经可以从到来的HTTP 请求中获得头部，cookies 和参数。这三种类型的值被存储在下面几个引用变量中：

```java
protected HashMap headers = new HashMap();
protected ArrayList cookies = new ArrayList();
protected ParameterMap parameters = null;
```

​	因此，一个servlet 程序员可以从javax.servlet.http.HttpServletRequest 中的下列方法中取得正确的返回 值：getCookies,getDateHeader,getHeader, getHeaderNames, getHeaders,getParameter, getPrameterMap,getParameterNames 和getParameterValues 。就像你在HttpRequest 类中看到的一样，一旦你取得了头部，cookies 和填充了正确的值的参数，相关的方法的实现是很简单的。

​	因为HTTP 请求的解析是一项相当复杂的任务，所以本节会分为以下几个小节：

* 读取套接字的输入流
* 解析请求行
* 解析头部
* 解析cookies
* 获取参数

**读取套接字的输入流**

​	在本章的应用程序中，你拥有ex03.pyrmont.connector.http.SocketInputStream 类， 这是
org.apache.catalina.connector.http.SocketInputStream 的一个拷贝。这个类提供了方法不
仅用来获取请求行，还有请求头部。

**解析请求行**

```
GET /myApp/ModernServlet?userName=tarzan&password=pwd HTTP/1.1
```

```java
private void parseRequest(SocketInputStream input, OutputStream output)
    throws IOException, ServletException {

    // Parse the incoming request line
    input.readRequestLine(requestLine);
    // 解析方法
    String method =
      new String(requestLine.method, 0, requestLine.methodEnd);
    String uri = null;
    // 获取protocol和版本
    String protocol = new String(requestLine.protocol, 0, requestLine.protocolEnd);

    // Validate the incoming request line
    if (method.length() < 1) {
      throw new ServletException("Missing HTTP request method");
    }
    else if (requestLine.uriEnd < 1) {
      throw new ServletException("Missing HTTP request URI");
    }
    // Parse any query parameters out of the request URI
    int question = requestLine.indexOf("?");
    // 获取uri和参数
    if (question >= 0) {
      request.setQueryString(new String(requestLine.uri, question + 1,
        requestLine.uriEnd - question - 1));
      uri = new String(requestLine.uri, 0, question);
    }
    else {
      request.setQueryString(null);
      uri = new String(requestLine.uri, 0, requestLine.uriEnd);
    }


    // Checking for an absolute URI (with the HTTP protocol)
    // 如果uri不是以/开始的，说明是绝对路径
    if (!uri.startsWith("/")) {
      int pos = uri.indexOf("://");
      // Parsing out protocol and host name
      if (pos != -1) {
        pos = uri.indexOf('/', pos + 3);
        if (pos == -1) {
          uri = "";
        }
        else {
          uri = uri.substring(pos);
        }
      }
    }

    // Parse any requested session ID out of the request URI
    // sessionId
    String match = ";jsessionid=";
    int semicolon = uri.indexOf(match);
    if (semicolon >= 0) {
      String rest = uri.substring(semicolon + match.length());
      int semicolon2 = rest.indexOf(';');
      if (semicolon2 >= 0) {
        request.setRequestedSessionId(rest.substring(0, semicolon2));
        rest = rest.substring(semicolon2);
      }
      else {
        request.setRequestedSessionId(rest);
        rest = "";
      }
      request.setRequestedSessionURL(true);
      uri = uri.substring(0, semicolon) + rest;
    }
    else {
      request.setRequestedSessionId(null);
      request.setRequestedSessionURL(false);
    }

    // Normalize URI (using String operations at the moment)
    String normalizedUri = normalize(uri);

    // Set the corresponding request properties
    ((HttpRequest) request).setMethod(method);
    request.setProtocol(protocol);
    if (normalizedUri != null) {
      ((HttpRequest) request).setRequestURI(normalizedUri);
    }
    else {
      ((HttpRequest) request).setRequestURI(uri);
    }

    if (normalizedUri == null) {
      throw new ServletException("Invalid URI: " + uri + "'");
    }
  }
```

**解析头部**

​	一个HTTP 头部是用类HttpHeader 来代表的。

* 你可以通过使用类的无参数构造方法构造一个HttpHeader 实例。
* 一旦你拥有一个HttpHeader 实例，你可以把它传递给SocketInputStream 的readHeader
  方法。假如这里有头部需要读取，readHeader 方法将会相应的填充HttpHeader 对象。
  假如再也没有头部需要读取了，HttpHeader 实例的nameEnd 和valueEnd 字段将会置零。
* 为了获取头部的名称和值，使用下面的方法：
  * `String name = new String(header.name, 0, header.nameEnd);`
  * `String value = new String(header.value, 0, header.valueEnd);`

**解析Cookies**

```java
Cookie: userName=budi; password=pwd;
```

```java
public static Cookie[] parseCookieHeader(String header) {

    if ((header == null) || (header.length() < 1))
        return (new Cookie[0]);

    ArrayList cookies = new ArrayList();
    while (header.length() > 0) {
        int semicolon = header.indexOf(';');
        if (semicolon < 0)
            semicolon = header.length();
        if (semicolon == 0)
            break;
        String token = header.substring(0, semicolon);
        if (semicolon < header.length())
            header = header.substring(semicolon + 1);
        else
            header = "";
        try {
            int equals = token.indexOf('=');
            if (equals > 0) {
                String name = token.substring(0, equals).trim();
                String value = token.substring(equals+1).trim();
                cookies.add(new Cookie(name, value));
            }
        } catch (Throwable e) {
            ;
        }
    }

    return ((Cookie[]) cookies.toArray(new Cookie[cookies.size()]));

}
```

**获取参数**

​	你不需要马上解析查询字符串或者HTTP 请求内容，直到servlet 需要通过调用javax.servlet.http.HttpServletRequest 的getParameter,getParameterMap, getParameterNames 或者getParameterValues 方法来读取参数。因此，HttpRequest 的这四个方法开头调用了parseParameter 方法。

​	类HttpRequest 使用一个布尔变量parsed 来指示是否已经解析过了。

​	类ParameterMap 继承java.util.HashMap，并使用了一个布尔变量locked。当locked 是false 的时候，名/值对仅仅可以添加，更新或者移除。否则，异常IllegalStateException 会抛出。而随时都可以读取参数值。

​	代码略

**创建一个HTTPResponse对象**

![](https://pic.imgdb.cn/item/609a56e2d1a9ae528fd4e27e.jpg)

```java
public PrintWriter getWriter() throws IOException {
	ResponseStream newStream = new ResponseStream(this);
	newStream.setCommit(false);
	OutputStreamWriter osr =
	new OutputStreamWriter(newStream, getCharacterEncoding());
	writer = new ResponseWriter(osr);
	return writer;
}
```

**静态资源处理器和Servlet 处理器**

​	只有一个方法：process。然而ex03.pyrmont.connector.ServletProcessor 中的process 方法接受
一个HttpRequest 和HttpResponse，代替了Requese 和Response 实例。
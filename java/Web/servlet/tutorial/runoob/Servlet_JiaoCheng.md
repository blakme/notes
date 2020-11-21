# [Servlet 教程](https://www.runoob.com/servlet/servlet-tutorial.html)

- [Servlet 教程](#servlet-教程)
  - [Servlet 简介](#servlet-简介)
    - [Servlet 是什么？](#servlet-是什么)
    - [Servlet 架构](#servlet-架构)
    - [Servlet 任务](#servlet-任务)
    - [Servlet 包](#servlet-包)
  - [Servlet 环境设置](#servlet-环境设置)
    - [设置 Java 开发工具包（Java Development Kit）](#设置-java-开发工具包java-development-kit)
    - [设置 Web 服务器：Tomcat](#设置-web-服务器tomcat)
    - [设置 CLASSPATH](#设置-classpath)
  - [Servlet 生命周期](#servlet-生命周期)
    - [init() 方法](#init-方法)
    - [service() 方法](#service-方法)
    - [doGet() 方法](#doget-方法)
    - [doPost() 方法](#dopost-方法)
    - [destroy() 方法](#destroy-方法)
    - [架构图](#架构图)
  - [Servlet 实例](#servlet-实例)
    - [Hello World 示例代码](#hello-world-示例代码)
    - [编译 Servlet](#编译-servlet)

Servlet 为创建基于 web 的应用程序提供了基于组件、独立于平台的方法，可以不受 CGI 程序的性能限制。Servlet 有权限访问所有的 Java API，包括访问企业级数据库的 JDBC API。

## [Servlet 简介](https://www.runoob.com/servlet/servlet-intro.html)

### Servlet 是什么？

Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。

Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：

- 性能明显更好。
- Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
- Servlet 是独立于平台的，因为它们是用 Java 编写的。
- 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
- Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

### Servlet 架构

下图显示了 Servlet 在 Web 应用程序中的位置。

![fig1](./fig/servlet-arch.jpg)

### Servlet 任务

Servlet 执行以下主要任务：

- 读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
- 读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等。
- 处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
- 发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
- 发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

### Servlet 包

Java Servlet 是运行在带有支持 Java Servlet 规范的解释器的 web 服务器上的 Java 类。

Servlet 可以使用 `javax.servlet` 和 `javax.servlet.http` 包创建，它是 Java 企业版的标准组成部分，Java 企业版是支持大型开发项目的 Java 类库的扩展版本。

这些类实现 Java Servlet 和 JSP 规范。在写本教程的时候，二者相应的版本分别是 Java Servlet 2.5 和 JSP 2.1。

Java Servlet 就像任何其他的 Java 类一样已经被创建和编译。在您安装 Servlet 包并把它们添加到您的计算机上的 Classpath 类路径中之后，您就可以通过 JDK 的 Java 编译器或任何其他编译器来编译 Servlet。

## [Servlet 环境设置](https://www.runoob.com/servlet/servlet-environment-setup.html)

开发环境是您可以开发、测试、运行 Servlet 的地方。

就像任何其他的 Java 程序，您需要通过使用 Java 编译器 javac 编译 Servlet，在编译 Servlet 应用程序后，将它部署在配置的环境中以便测试和运行。

如果你使用的是 Eclipse 环境，可以直接参阅：[Eclipse JSP/Servlet 环境搭建](https://www.runoob.com/jsp/eclipse-jsp.html)。

这个开发环境设置包括以下步骤：

### 设置 Java 开发工具包（Java Development Kit）

### 设置 Web 服务器：Tomcat

在市场上有许多 Web 服务器支持 Servlet。有些 Web 服务器是免费下载的，Tomcat 就是其中的一个。

Apache Tomcat 是一款 Java Servlet 和 JavaServer Pages 技术的开源软件实现，可以作为测试 Servlet 的独立服务器，而且可以集成到 Apache Web 服务器。下面是在电脑上安装 Tomcat 的步骤：

从 <http://tomcat.apache.org/> 上下载最新版本的 Tomcat。
一旦您下载了 Tomcat，解压缩到一个方便的位置。例如，如果您使用的是 Windows，则解压缩到 C:\apache-tomcat-5.5.29 中，如果您使用的是 Linux/Unix，则解压缩到 /usr/local/apache-tomcat-5.5.29 中，并创建 `CATALINA_HOME` 环境变量指向这些位置。
在 Windows 上，可以通过执行下面的命令来启动 Tomcat：

    %CATALINA_HOME%\bin\startup.bat

 或者

    C:\apache-tomcat-5.5.29\bin\startup.bat

在 Unix（Solaris、Linux 等） 上，可以通过执行下面的命令来启动 Tomcat：

    $CATALINA_HOME/bin/startup.sh

 或者

    /usr/local/apache-tomcat-5.5.29/bin/startup.sh

有关配置和运行 Tomcat 的进一步信息可以查阅应用程序安装的文档，或者可以访问 Tomcat 网站：http://tomcat.apache.org。

在 Windows 上，可以通过执行下面的命令来停止 Tomcat：

    C:\apache-tomcat-5.5.29\bin\shutdown

在 Unix（Solaris、Linux 等） 上，可以通过执行下面的命令来停止 Tomcat：

    /usr/local/apache-tomcat-5.5.29/bin/shutdown.sh

### 设置 CLASSPATH

由于 Servlet 不是 Java 平台标准版的组成部分，所以您必须为编译器指定 Servlet 类的路径。

## [Servlet 生命周期](https://www.runoob.com/servlet/servlet-life-cycle.html)

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

- Servlet 初始化后调用 `init ()` 方法。
- Servlet 调用 `service()` 方法来处理客户端的请求。
- Servlet 销毁前调用 `destroy()` 方法。
- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

现在让我们详细讨论生命周期的方法。

### init() 方法

init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。因此，它是用于一次性初始化，就像 Applet 的 init 方法一样。

Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

init 方法的定义如下：

    public void init() throws ServletException {
    // 初始化代码...
    }

### service() 方法

service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

下面是该方法的特征：

    public void service(ServletRequest request, 
                        ServletResponse response) 
        throws ServletException, IOException{
    }

service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以，您不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

doGet() 和 doPost() 方法是每次服务请求中最常用的方法。下面是这两种方法的特征。

### doGet() 方法

GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理。

    public void doGet(HttpServletRequest request,
                    HttpServletResponse response)
        throws ServletException, IOException {
        // Servlet 代码
    }

### doPost() 方法

POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理。

    public void doPost(HttpServletRequest request,
                    HttpServletResponse response)
        throws ServletException, IOException {
        // Servlet 代码
    }

### destroy() 方法

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：

    public void destroy() {
        // 终止化代码...
    }

### 架构图

下图显示了一个典型的 Servlet 生命周期方案。

- 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
- Servlet 容器在调用 service() 方法之前加载 Servlet。
- 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。

![fig2](./fig/Servlet-LifeCycle.jpg)

## [Servlet 实例](https://www.runoob.com/servlet/servlet-first-example.html)

Servlet 是服务 HTTP 请求并实现 javax.servlet.Servlet 接口的 Java 类。Web 应用程序开发人员通常**编写 Servlet 来扩展 javax.servlet.http.HttpServlet**，并实现 Servlet 接口的抽象类专门用来处理 HTTP 请求。

### Hello World 示例代码

下面是 Servlet 输出 Hello World 的示例源代码：

    // 导入必需的 java 库
    import java.io.*;
    import javax.servlet.*;
    import javax.servlet.http.*;

    // 扩展 HttpServlet 类
    public class HelloWorld extends HttpServlet {
    
        private String message;

        public void init() throws ServletException
        {
            // 执行必需的初始化
            message = "Hello World";
        }

        public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
                throws ServletException, IOException
        {
            // 设置响应内容类型
            response.setContentType("text/html");

            // 实际的逻辑是在这里
            PrintWriter out = response.getWriter();
            out.println("<h1>" + message + "</h1>");
        }
    
        public void destroy()
        {
            // 什么也不做
        }
    }

### 编译 Servlet





















TODO java servlet ssssssssssssssssssssssssssssssssssss
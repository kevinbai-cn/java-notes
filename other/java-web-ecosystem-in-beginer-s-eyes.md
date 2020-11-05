这几天看了下廖雪峰的《Java 教程》Web 开发的几个章节，对 Java Web 的技术体系算是有了一点点认识，在此摘录下。

# 几个 E

![-w241](media/16045871492438.jpg)

- JavaSE，简单的理解为标准版 JDK
- JavaME，裁剪后的「微型版」JDK，现在使用很少，不用管它
- JavaEE，并不是一个软件产品，它更多的是一种软件架构和设计思想。我们可以把 JavaEE 看作是在 JavaSE 的基础上，开发的一系列基于服务器的组件、API 标准和通用架构

JavaEE 最核心的组件就是基于 Servlet 标准的 Web 服务器，开发者编写的应用程序是基于 Servlet API 并运行在 Web 服务器内部的。如下图

![-w201](media/16045875663900.jpg)

此外，JavaEE 有一系列的技术标准

- EJB：Enterprise JavaBean，企业级 JavaBean，早期经常用于实现应用程序的业务逻辑，现在基本被轻量级框架如 Spring 所取代
- JMS：Java Message Service，用于消息服务
- JTA：Java Transaction API，用于分布式事务
- ...

目前流行的基于 Spring 的轻量级 JavaEE 开发架构，使用最广泛的是 Servlet 和 JMS，以及一系列开源组件。

# 基于 Servlet 的 Web 开发

## Servlet

在 JavaEE 平台上，处理 TCP 连接，解析 HTTP 协议这些底层工作统统扔给现成的 Web 服务器去做，我们只需要把自己的应用程序跑在 Web 服务器上。为了实现这一目的，JavaEE 提供了 Servlet API，我们使用 Servlet API 编写自己的 Servlet 来处理 HTTP 请求，Web 服务器实现 Servlet API 接口，实现底层功能

![-w357](media/16045880967039.jpg)

写一个最简单的 Servlet

```
// WebServlet 注解表示这是一个 Servlet，并映射到地址 /
@WebServlet(urlPatterns = "/")
public class HelloServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // 设置响应类型:
        resp.setContentType("text/html");
        // 获取输出流:
        PrintWriter pw = resp.getWriter();
        // 写入响应:
        pw.write("<h1>Hello, world!</h1>");
        // 最后不要忘记flush强制输出:
        pw.flush();
    }
}
```

Servlet 应用一般会打成 war 包。

普通的 Java 程序是通过启动 JVM，然后执行 main() 方法开始运行。但是 Web 应用程序有所不同，我们无法直接运行 war 文件，必须先启动 Web 服务器，再由 Web 服务器加载我们编写的 HelloServlet，这样就可以让 HelloServlet 处理浏览器发送的请求。

常见的支持 Servlet API 的 Web 服务器有

- Tomcat：由 Apache 开发的开源免费服务器
- Jetty：由 Eclipse 开发的开源免费服务器
- GlassFish：一个开源的全功能 JavaEE 服务器

到这里，一个 Servlet 应用从编写到运行就能串起来了。

## JSP

不过，从上面可以看出，如果要输出 HTML 的话，需要自己格式化进行输出，太麻烦，这样也就有了 JSP

```
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <%-- JSP Comment --%>
    <h1>Hello World!</h1>
    <p>
    <%
         out.println("Your IP address is ");
    %>
    <span style="color:red">
        <%= request.getRemoteAddr() %>
    </span>
    </p>
</body>
</html>
```

整个 JSP 的内容实际上是一个 HTML，但是稍有不同

- 包含在 `<%--` 和 `--%>` 之间的是 JSP 的注释，它们会被完全忽略
- 包含在 `<%` 和 `%>` 之间的是 Java 代码，可以编写任意 Java 代码
- 如果使用 `<%= xxx %>` 则可以快捷输出一个变量的值

当然，还有其它的一些语法，这里了解下 JSP 文件长什么样就行。

## Filter

为了把一些公用逻辑从各个 Servlet 中抽离出来，JavaEE 的 Servlet 规范还提供了一种 Filter 组件，即过滤器。它的作用是，在 HTTP 请求到达 Servlet 之前，可以被一个或多个 Filter 预处理，类似打印日志、登录检查等逻辑，完全可以放到 Filter 中

例如，我们编写一个最简单的 EncodingFilter，它强制把输入和输出的编码设置为 `UTF-8`

```
@WebFilter(urlPatterns = "/*")
public class EncodingFilter implements Filter {
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("EncodingFilter:doFilter");
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        chain.doFilter(request, response);
    }
}
```

添加了Filter之后，整个请求的处理架构如下

![-w765](media/16045901074271.jpg)

上面 Filter 的过滤路径都是 `/*`，它会对所有请求进行过滤。也可以编写只对特定路径进行过滤的 Filter，比如

```
@WebFilter(urlPatterns = "/user/*")
```

这样就只过滤以 `/user/` 开头的路径。

## Listener

除此之外，JavaEE 的 Servlet 规范还提供了第三种组件 Listener，顾名思义就是监听器。有好几种 Listener，其中最常用的是 ServletContextListener，我们编写一个实现了 ServletContextListener 接口的类如下

```
@WebListener
public class AppListener implements ServletContextListener {
    // 在此初始化 WebApp，例如打开数据库连接池等
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("WebApp initialized.");
    }

    // 在此清理 WebApp，例如关闭数据库连接池等
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("WebApp destroyed.");
    }
}
```

任何标注为 @WebListener，且实现了特定接口的类会被 Web 服务器自动初始化。上述 AppListener 实现了 ServletContextListener 接口，它会在整个 Web 应用程序初始化完成后，以及 Web 应用程序关闭后获得回调通知。我们可以把初始化数据库连接池等工作放到 contextInitialized() 回调方法中，把清理资源的工作放到 contextDestroyed() 回调方法中，因为 Web 服务器保证在 contextInitialized() 执行后，才会接受用户的 HTTP 请求。

除了 ServletContextListener 外，还有几种 Listener

- HttpSessionListener：监听 HttpSession 的创建和销毁事件
- ServletRequestListener：监听 ServletRequest 请求的创建和销毁事件
- ServletRequestAttributeListener：监听 ServletRequest 请求的属性变化事件（即调用 ServletRequest.setAttribute() 方法）
- ServletContextAttributeListener：监听 ServletContext 的属性变化事件（即调用 ServletContext.setAttribute() 方法）

## ServletContext

一个 Web 服务器可以运行一个或多个 WebApp，对于每个 WebApp，Web 服务器都会为其创建一个全局唯一的 ServletContext 实例，我们在 AppListener 里面编写的两个回调方法实际上对应的就是 ServletContext 实例的创建和销毁。

ServletRequest、HttpSession 等很多对象也提供 getServletContext() 方法获取到同一个 ServletContext 实例。ServletContext 实例最大的作用就是设置和共享全局信息。

此外，ServletContext 还提供了动态添加 Servlet、Filter、Listener 等功能，它允许应用程序在运行期间动态添加一个组件，虽然这个功能不是很常用。

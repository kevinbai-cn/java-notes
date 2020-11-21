这几天看了下廖雪峰的《Java 教程》Web 开发的几个章节，对 Java Web 的技术体系算是有了一点点认识，在此摘录下。

# 1 几个 E

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

# 2 基于 Servlet 的 Web 开发

## 2.1 Servlet

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

## 2.2 JSP

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

## 2.3 Filter

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

## 2.4 Listener

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

## 2.5 ServletContext

一个 Web 服务器可以运行一个或多个 WebApp，对于每个 WebApp，Web 服务器都会为其创建一个全局唯一的 ServletContext 实例，我们在 AppListener 里面编写的两个回调方法实际上对应的就是 ServletContext 实例的创建和销毁。

ServletRequest、HttpSession 等很多对象也提供 getServletContext() 方法获取到同一个 ServletContext 实例。ServletContext 实例最大的作用就是设置和共享全局信息。

此外，ServletContext 还提供了动态添加 Servlet、Filter、Listener 等功能，它允许应用程序在运行期间动态添加一个组件，虽然这个功能不是很常用。

# 3 基于 Servlet 编写简单的 MVC 框架

通过前面可以大概了解到

- Servlet 适合编写 Java 代码，实现各种复杂的业务逻辑，但不适合输出复杂的 HTML
- JSP 适合编写 HTML，并在其中插入动态内容，但不适合编写复杂的 Java 代码

可以将两者结合起来，发挥各自的优点，避免各自的缺点。

假设我们已经编写了几个 JavaBean

```
public class User {
    public long id;
    public String name;
    public School school;
}

public class School {
    public String name;
    public String address;
}
```

在 UserServlet 中，我们可以从数据库读取 User、School 等信息，然后，把读取到的 JavaBean 先放到 HttpServletRequest 中，再通过 forward() 传给 user.jsp 处理

```
@WebServlet(urlPatterns = "/user")
public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 假装从数据库读取
        School school = new School("No.1 Middle School", "101 South Street");
        User user = new User(123, "Bob", school);
        // 放入 Request 中
        req.setAttribute("user", user);
        // forward 给 user.jsp
        req.getRequestDispatcher("/WEB-INF/user.jsp").forward(req, resp);
    }
}
```

在 user.jsp 中，我们只负责展示相关 JavaBean 的信息，不需要编写访问数据库等复杂逻辑

```
<%@ page import="com.itranswarp.learnjava.bean.*"%>
<%
    User user = (User) request.getAttribute("user");
%>
<html>
<head>
    <title>Hello World - JSP</title>
</head>
<body>
    <h1>Hello <%= user.name %>!</h1>
    <p>School Name:
    <span style="color:red">
        <%= user.school.name %>
    </span>
    </p>
    <p>School Address:
    <span style="color:red">
        <%= user.school.address %>
    </span>
    </p>
</body>
</html>
```

需要注意的是

- 需要展示的 User 被放入 HttpServletRequest 中以便传递给 JSP，因为一个请求对应一个 HttpServletRequest，我们也无需清理它，处理完该请求后 HttpServletRequest 实例将被丢弃
- 把 user.jsp 放到 /WEB-INF/ 目录下，是因为 WEB-INF 是一个特殊目录，Web Server 会阻止浏览器对 WEB-INF 目录下任何资源的访问，这样就防止用户通过 /user.jsp 路径直接访问到 JSP 页面
- JSP 页面首先从 request 变量获取 User 实例，然后在页面中直接输出，此处未考虑 HTML 的转义问题，有潜在安全风险

我们在浏览器访问的时候，请求首先由 UserServlet 处理，然后交给 user.jsp 渲染

![-w365](media/16049341856940.jpg)

我们把 UserServlet 看作业务逻辑处理，把 User 看作模型，把 user.jsp 看作渲染，这种设计模式通常被称为 MVC：Model-View-Controller，即 UserServlet 作为控制器（Controller），User 作为模型（Model），user.jsp 作为视图（View）。

通过结合 Servle t和 JSP 的 MVC 模式，我们可以发挥二者各自的优点

- Servlet 实现业务逻辑
- JSP 实现展示逻辑

但是，直接把 MVC 搭在 Servlet 和 JSP 之上还是不太好，原因如下

- Servlet 提供的接口仍然偏底层，需要实现 Servlet 调用相关接口
- JSP 对页面开发不友好，更好的替代品是模板引擎
- 业务逻辑最好由纯粹的 Java 类实现，而不是强迫继承自 Servlet

能不能通过普通的 Java 类实现 MVC 的 Controller？类似下面的代码

```
public class UserController {
    @GetMapping("/signin")
    public ModelAndView signin() {
        ...
    }

    @PostMapping("/signin")
    public ModelAndView doSignin(SignInBean bean) {
        ...
    }

    @GetMapping("/signout")
    public ModelAndView signout(HttpSession session) {
        ...
    }
}
```

上面的这个 Java 类每个方法都对应一个 GET 或 POST 请求，方法返回值是 ModelAndView，它包含一个 View 的路径以及一个 Model，这样，再由 MVC 框架处理后返回给浏览器。

如果是 GET 请求，我们希望 MVC 框架能直接把 URL 参数按方法参数对应起来然后传入

```
@GetMapping("/hello")
public ModelAndView hello(String name) {
    ...
}
```

如果是 POST 请求，我们希望 MVC 框架能直接把 Post 参数变成一个 JavaBean 后通过方法参数传入

```
@PostMapping("/signin")
public ModelAndView doSignin(SignInBean bean) {
    ...
}
```

为了增加灵活性，如果 Controller 的方法在处理请求时需要访问 HttpServletRequest、HttpServletResponse、HttpSession 这些实例时，只要方法参数有定义，就可以自动传入

```
@GetMapping("/signout")
public ModelAndView signout(HttpSession session) {
    ...
}
```

以上就是我们在设计 MVC 框架时，上层代码所需要的一切信息。

如何设计一个 MVC 框架？在上文中，我们已经定义了上层代码编写 Controller 的一切接口信息，并且并不要求实现特定接口，只需返回 ModelAndView 对象，该对象包含一个 View 和一个 Model。实际上 View 就是模板的路径，而 Model 可以用一个 Map<String, Object> 表示，因此，ModelAndView 定义非常简单

```
public class ModelAndView {
    Map<String, Object> model;
    String view;
}
```

比较复杂的是我们需要在 MVC 框架中创建一个接收所有请求的 Servlet，通常我们把它命名为 DispatcherServlet，它总是映射到 /，然后，根据不同的 Controller 的方法定义的 @Get 或 @Post 的 Path 决定调用哪个方法，最后，获得方法返回的 ModelAndView 后，渲染模板，写入HttpServletResponse，即完成了整个 MVC 的处理。以下是大致的架构图

![-w372](media/16049347057737.jpg)

DispatcherServlet 细节这里不做说明。

最后渲染只需要实现一个简单的 render () 方法

```
public class ViewEngine {
    public void render(ModelAndView mv, Writer writer) throws IOException {
        String view = mv.view;
        Map<String, Object> model = mv.model;
        // 根据 view 找到模板文件
        Template template = getTemplateByPath(view);
        // 渲染并写入 Writer
        template.write(writer, model);
    }
}
```

Java 有很多开源的模板引擎，常用的有

- Thymeleaf
- FreeMarker
- Velocity

他们的用法都大同小异。这里我们推荐一个使用 Jinja 语法的模板引擎 Pebble，它的特点是语法简单，支持模板继承，编写出来的模板类似

```
<html>
<body>
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
</body>
</html>
```

即变量用 `{{ xxx }}` 表示，控制语句用 `{% xxx %}` 表示。

至此，设计一个比较高级的 MVC 需要实现的东西也都被串起来了。

本节内容没有涉及过多的细节，只是想让大家了解到，为了开发的便捷，Servlet 是怎么样一步一步被扩展的。不管是低层级的 MVC，或是比较高级的 MVC，或是以后我们要学习的新技术，都要先了解下它究竟解决了什么问题。


# 4 Spring Framework

什么是 Spring？

Spring 是一个支持快速开发 Java EE 应用程序的框架。它提供了一系列底层容器和基础设施，并可以和大量常用的开源框架无缝集成，可以说是开发 JavaEE 应用程序的必备。

Spring 最早是由 Rod Johnson 在他的《Expert One-on-One J2EE Development without EJB》一书中提出的用来取代 EJB 的轻量级框架。随后他又开始专心开发这个基础框架，并起名为 Spring Framework。

随着 Spring 越来越受欢迎，在 Spring Framework 基础上，又诞生了 Spring Boot、Spring Cloud、Spring Data、Spring Security 等一系列基于 Spring Framework 的项目。本节我们只介绍 Spring Framework，即最核心的 Spring 框架。

Spring Framework 主要包括几个模块

- 支持 IoC 和 AOP 的容器
- 支持 JDBC、ORM 的数据访问模块
- 支持基于 Servlet 的 MVC 开发
- 支持集成 JMS、JavaMail、JMX、缓存等其它模块

## 4.1 IoC

在学习 Spring 框架时，我们遇到的第一个也是最核心的概念就是容器。

什么是容器？容器是一种为某种特定组件的运行提供必要支持的一个软件环境。例如，Tomcat 就是一个 Servlet 容器，它可以为 Servlet 的运行提供运行环境。类似 Docker 这样的软件也是一个容器，它提供了必要的 Linux 环境以便运行一个特定的 Linux 进程。

Spring 的核心就是提供了一个 IoC 容器，它可以管理所有轻量级的 JavaBean 组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP 支持，以及建立在 AOP 基础上的声明式事务服务等。

什么是 IoC？

IoC 全称 Inversion of Control，直译为控制反转。那么何谓 IoC？在理解 IoC 之前，我们先看看通常的 Java 组件是如何协作的。

### 4.1.1 使用 XML 配置

我们假定一个在线书店，通过 BookService 获取书籍

```
public class BookService {
    private HikariConfig config = new HikariConfig();
    private DataSource dataSource = new HikariDataSource(config);

    public Book getBook(long bookId) {
        try (Connection conn = dataSource.getConnection()) {
            ...
            return book;
        }
    }
}
```

为了从数据库查询书籍，BookService 持有一个 DataSource。为了实例化一个 HikariDataSource，又不得不实例化一个 HikariConfig。

现在，我们继续编写 UserService 获取用户

```
public class UserService {
    private HikariConfig config = new HikariConfig();
    private DataSource dataSource = new HikariDataSource(config);

    public User getUser(long userId) {
        try (Connection conn = dataSource.getConnection()) {
            ...
            return user;
        }
    }
}
```

因为 UserService 也需要访问数据库，因此，我们不得不也实例化一个 HikariDataSource。

在处理用户购买的 CartServlet 中，我们需要实例化 UserService 和 BookService

```
public class CartServlet extends HttpServlet {
    private BookService bookService = new BookService();
    private UserService userService = new UserService();

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        long currentUserId = getFromCookie(req);
        User currentUser = userService.getUser(currentUserId);
        Book book = bookService.getBook(req.getParameter("bookId"));
        cartService.addToCart(currentUser, book);
        ...
    }
}
```

类似的，在购买历史 HistoryServlet 中，也需要实例化 UserService 和 BookService

```
public class HistoryServlet extends HttpServlet {
    private BookService bookService = new BookService();
    private UserService userService = new UserService();
}
```

上述每个组件都采用了一种简单的通过 new 创建实例并持有的方式。仔细观察，会发现以下缺点

- 实例化一个组件其实很难，例如，BookService 和 UserService 要创建 HikariDataSource，实际上需要读取配置，才能先实例化 HikariConfig，再实例化 HikariDataSource
- 没有必要让 BookService 和 UserService 分别创建 DataSource 实例，完全可以共享同一个 DataSource，但谁负责创建 DataSource，谁负责获取其他组件已经创建的 DataSource，不好处理。类似的，CartServlet 和 HistoryServlet 也应当共享 BookService 实例和 UserService 实例，但也不好处理
- 很多组件需要销毁以便释放资源，例如 DataSource，但如果该组件被多个组件共享，如何确保它的使用方都已经全部被销毁？
- 随着更多的组件被引入，例如，书籍评论，需要共享的组件写起来会更困难，这些组件的依赖关系会越来越复杂
- 测试某个组件，例如 BookService，是复杂的，因为必须要在真实的数据库环境下执行

解决这一问题的核心方案就是 IoC。

传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，例如：CartServlet 创建了 BookService，在创建 BookService 的过程中，又创建了 DataSource 组件。这种模式的缺点是，一个组件如果要使用另一个组件，必须先知道如何正确地创建它。

在 IoC 模式下，控制权发生了反转，即从应用程序转移到了 IoC 容器，所有组件不再由应用程序自己创建和配置，而是由 IoC 容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在 IoC 容器中被「装配」出来，需要某种「注入」机制，例如，BookService 自己并不会创建 DataSource，而是等待外部通过 setDataSource() 方法来注入一个 DataSource

```
public class BookService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

不直接 new 一个 DataSource，而是注入一个 DataSource，这个小小的改动虽然简单，却带来了一系列好处

- BookService 不再关心如何创建 DataSource，因此，不必编写读取数据库配置之类的代码
- DataSource 实例被注入到 BookService，同样也可以注入到 UserService，因此，共享一个组件非常简单
- 测试 BookService 更容易，因为注入的是 DataSource，可以使用内存数据库，而不是真实的 MySQL 配置

因此，IoC 又称为依赖注入（DI：Dependency Injection），它解决了一个最主要的问题：将组件的创建 + 配置与组件的使用相分离，并且，由 IoC 容器负责管理组件的生命周期。

因为 IoC 容器要负责实例化所有的组件，因此，有必要告诉容器如何创建组件，以及各组件的依赖关系。一种最简单的配置是通过 XML 文件来实现，例如

```
<beans>
    <bean id="dataSource" class="HikariDataSource" />
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userService" class="UserService">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```

上述 XML 配置文件指示 IoC 容器创建 3 个 JavaBean 组件，并把 id 为 dataSource 的组件通过属性 dataSource（即调用 setDataSource() 方法）注入到另外两个组件中。

在 Spring 的 IoC 容器中，我们把所有组件统称为 JavaBean，即配置一个组件就是配置一个 Bean。

我们从上面的代码可以看到，依赖注入可以通过 set() 方法实现。但依赖注入也可以通过构造方法实现。

很多 Java 类都具有带参数的构造方法，如果我们把 BookService 改造为通过构造方法注入，那么实现代码如下：

```
public class BookService {
    private DataSource dataSource;

    public BookService(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

Spring 的 IoC 容器同时支持属性注入和构造方法注入，并允许混合使用。

### 4.1.2 使用注解配置

使用 XML 配置的优点是所有的 Bean 都能一目了然地列出来，并通过配置注入能直观地看到每个 Bean 的依赖。它的缺点是写起来非常繁琐，每增加一个组件，就必须把新的 Bean 配置到 XML 中

有没有其他更简单的配置方式呢？

有！我们可以使用 Annotation 配置，可以完全不需要 XML，让 Spring 自动扫描 Bean 并组装它们。

首先，我们给 MailService 添加一个 @Component 注解

```
@Component
public class MailService {
    ...
}
```

这个 @Component 注解就相当于定义了一个 Bean，它有一个可选的名称，默认是 mailService，即小写开头的类名。

然后，我们给 UserService 添加一个 @Component 注解和一个 @Autowired 注解

```
@Component
public class UserService {
    @Autowired
    MailService mailService;

    ...
}
```

使用 @Autowired 就相当于把指定类型的 Bean 注入到指定的字段中。和 XML 配置相比，@Autowired 大幅简化了注入，因为它不但可以写在 set() 方法上，还可以直接写在字段上，甚至可以写在构造方法中

```
@Component
public class UserService {
    MailService mailService;

    public UserService(@Autowired MailService mailService) {
        this.mailService = mailService;
    }
    ...
}
```

我们一般把 @Autowired 写在字段上，通常使用 package 权限的字段，便于测试。

最后，编写一个 AppConfig 类启动容器

```
@Configuration
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}
```

除了 main() 方法外，AppConfig 标注了 @Configuration，表示它是一个配置类，因为我们创建 ApplicationContext 时

```
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

使用的实现类是 AnnotationConfigApplicationContext，必须传入一个标注了 @Configuration 的类名。

此外，AppConfig 还标注了 @ComponentScan，它告诉容器，自动搜索当前类所在的包以及子包，把所有标注为 @Component 的 Bean 自动创建出来，并根据 @Autowired 进行装配。


## 4.2 AOP

什么是 AOP？

AOP 是 Aspect Oriented Programming，即面向切面编程。

我们先回顾一下 OOP：Object Oriented Programming，OOP 作为面向对象编程的模式，获得了巨大的成功，OOP 的主要功能是数据封装、继承和多态。

而 AOP 是一种新的编程方式，它和 OOP 不同，OOP 把系统看作多个对象的交互，AOP 把系统分解为不同的关注点，或者称之为切面（Aspect）。

要理解 AOP 的概念，我们先用 OOP 举例，比如一个业务组件 BookService，它有几个业务方法

- createBook：添加新的Book
- updateBook：修改Book
- deleteBook：删除Book

对每个业务方法，例如，createBook()，除了业务逻辑，还需要安全检查、日志记录和事务处理，它的代码像这样

```
public class BookService {
    public void createBook(Book book) {
        securityCheck();
        Transaction tx = startTransaction();
        try {
            // 核心业务逻辑
            tx.commit();
        } catch (RuntimeException e) {
            tx.rollback();
            throw e;
        }
        log("created book: " + book);
    }
}
```

继续编写 updateBook()，代码如下

```
public class BookService {
    public void updateBook(Book book) {
        securityCheck();
        Transaction tx = startTransaction();
        try {
            // 核心业务逻辑
            tx.commit();
        } catch (RuntimeException e) {
            tx.rollback();
            throw e;
        }
        log("updated book: " + book);
    }
}
```

对于安全检查、日志、事务等代码，它们会重复出现在每个业务方法中。使用 OOP，我们很难将这些四处分散的代码模块化。

考察业务模型可以发现，BookService 关系的是自身的核心逻辑，但整个系统还要求关注安全检查、日志、事务等功能，这些功能实际上「横跨」多个业务方法，为了实现这些功能，不得不在每个业务方法上重复编写代码。

一种可行的方式是使用 Proxy 模式，将某个功能，例如，权限检查，放入 Proxy 中

```
public class SecurityCheckBookService implements BookService {
    private final BookService target;

    public SecurityCheckBookService(BookService target) {
        this.target = target;
    }

    public void createBook(Book book) {
        securityCheck();
        target.createBook(book);
    }

    public void updateBook(Book book) {
        securityCheck();
        target.updateBook(book);
    }

    public void deleteBook(Book book) {
        securityCheck();
        target.deleteBook(book);
    }

    private void securityCheck() {
        ...
    }
}
```

这种方式的缺点是比较麻烦，必须先抽取接口，然后，针对每个方法实现 Proxy。

另一种方法是，既然 SecurityCheckBookService 的代码都是标准的 Proxy 样板代码，不如把权限检查视作一种切面（Aspect），把日志、事务也视为切面，然后，以某种自动化的方式，把切面织入到核心逻辑中，实现 Proxy 模式。

如果我们以 AOP 的视角来编写上述业务，可以依次实现

- 核心逻辑，即 BookService
- 切面逻辑，即权限检查的 Aspect、日志的 Aspect、事务的 Aspect

然后，以某种方式，让框架来把上述 3 个 Aspect 以 Proxy 的方式「织入」到 BookService 中，这样一来，就不必编写复杂而冗长的 Proxy 模式。

在 Java 平台上，对于 AOP 的织入，有 3 种方式 

1. 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ 就扩展了 Java 编译器，使用关键字 aspect 来实现织入
2. 类加载器：在目标类被装载到 JVM 时，通过一个特殊的类加载器，对目标类的字节码重新「增强」
3. 运行期：目标对象和切面都是普通 Java 类，通过 JVM 的动态代理功能或者第三方库实现运行期动态织入

在 Spring 中，我们可以引入 AspectJ 依赖实现 AOP，这里细节不进行说明，大家了解下 AOP 的基本作用和大致原理就行。


## 4.3 访问数据库

数据库基本上是现代应用程序的标准存储，绝大多数程序都把自己的业务数据存储在关系数据库中，可见，访问数据库几乎是所有应用程序必备能力。

### 4.3.1 JDBC

什么是 JDBC？JDBC 是 Java DataBase Connectivity 的缩写，它是 Java 程序访问数据库的标准接口。各数据库厂商以「驱动」的形式实现接口。应用程序要使用哪个数据库，就把该数据库厂商的驱动以 jar 包形式引入进来，同时自身仅使用 JDBC 接口，编译期并不需要特定厂商的驱动。

使用 JDBC 查询的代码类似这样

```
public Ingredient findById(String id) {
    Connection connection = null;
    PreparedStatement statement = null;
    ResultSet resultSet = null;
    try {
        connection = dataSource.getConnection();
        statement = connection.prepareStatement(
                "select id, name, type from Ingredient");
        statement.setString(1, id);
        resultSet = statement.executeQuery();
        Ingredient ingredient = null;
        if(resultSet.next()) {
            ingredient = new Ingredient(
                    resultSet.getString("id"),
                    resultSet.getString("name"),
                    Ingredient.Type.valueOf(resultSet.getString("type")));
        }
        return ingredient;
    } catch (SQLException e) {
        // ??? What should be done here ???
    } finally {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {}
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {}
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {}
        }
    }
    return null;
}
```

在创建连接、创建语句或执行查询的时候，可能会出现很多错误。这就要求我们捕获 SQLException，它对于找出哪里出现了问题或如何解决问题可能有所帮助，也可能毫无用处。

SQLException 是一个检查型异常，它需要在 catch 代码块中进行处理。但是，对于常见的问题，如创建到数据库的连接失败或者输入的查询有错误，在 catch 代码块中是无法解决的，并且有可能要继续抛出以便于上游进行处理。

### 4.3.2 JdbcTemplate

Spring 提供的 JdbcTemplate 采用 Template 模式，提供了一系列以回调为特点的工具方法，目的是避免繁琐的 `try...catch` 语句。

```
private JdbcTemplate jdbc;
    
public Ingredient findById(String id) {
    return jdbc.queryForObject(
            "select id, name, type from Ingredient where id=?",
            this::mapRowToIngredient, id);
}
    
private Ingredient mapRowToIngredient(ResultSet rs, int rowNum)
        throws SQLException {
    return new Ingredient(
            rs.getString("id"),
            rs.getString("name"),
            Ingredient.Type.valueOf(rs.getString("type")));
}
```

不管是 JDBC 还是 JdbcTemplate，除了查询外还提供了更新等操作，这里我们不一一介绍。

### 4.3.3 JPA

JPA 是 JavaEE 的一个 ORM 标准，用户如果使用 JPA，那么引用的就是 javax.persistence 这个「标准」包。

使用 JPA 可以选择 Hibernate 作为底层实现，但也可以选择其它的 JPA 提供方，比如 EclipseLink。Spring 内置了 JPA 的集成，并支持选择Hibernate 或 EclipseLink 作为实现。这里我们以 Hibernate 作为 JPA 实现为例子，演示 JPA 的基本用法。

假设有如下的数据库表

```
CREATE TABLE user
    id BIGINT NOT NULL AUTO_INCREMENT,
    email VARCHAR(100) NOT NULL,
    password VARCHAR(100) NOT NULL,
    name VARCHAR(100) NOT NULL,
    createdAt BIGINT NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `email` (`email`)
);
```

其中，id 是自增主键，email、password、name 是 VARCHAR 类型，email 带唯一索引以确保唯一性，createdAt 存储整型类型的时间戳。用 JavaBean 表示如下

```
public class User {
    private Long id;
    private String email;
    private String password;
    private String name;
    private Long createdAt;

    // getters and setters
    ...
}
```

这种映射关系十分易懂，但我们需要添加一些注解来告诉 Hibernate 如何把 User 类映射到表记录

```
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    public Long getId() { ... }

    @Column(nullable = false, unique = true, length = 100)
    public String getEmail() { ... }

    @Column(nullable = false, length = 100)
    public String getPassword() { ... }

    @Column(nullable = false, length = 100)
    public String getName() { ... }

    @Column(nullable = false, updatable = false)
    public Long getCreatedAt() { ... }
}
```

如果一个 JavaBean 被用于映射，我们就标记一个 @Entity。默认情况下，映射的表名是 user，如果实际的表名不同，例如实际表名是 users，可以追加一个 @Table(name="users") 表示

```
@Entity
@Table(name="users)
public class User {
    ...
}
```

每个属性到数据库列的映射用 @Column() 标识，nullable 指示列是否允许为 NULL，updatable 指示该列是否允许被用在 UPDATE 语句，length 指示 String 类型的列的长度（如果没有指定，默认是 255）。

对于主键，还需要用 @Id 标识，自增主键再追加一个 @GeneratedValue，以便 Hibernate 能读取到自增主键的值。

查询的时候类似这样

```
public User getUserById(long id) {
    User user = this.em.find(User.class, id);
    if (user == null) {
        throw new RuntimeException("User not found by id: " + id);
    }
    return user;
}
```

JPA 同样支持 Criteria 查询，比如我们需要的查询如下

```
SELECT * FROM user WHERE email = ?
```

使用 Criteria 查询的代码如下

```
public User fetchUserByEmail(String email) {
    // CriteriaBuilder
    var cb = em.getCriteriaBuilder();
    CriteriaQuery<User> q = cb.createQuery(User.class);
    Root<User> r = q.from(User.class);
    q.where(cb.equal(r.get("email"), cb.parameter(String.class, "e")));
    TypedQuery<User> query = em.createQuery(q);
    // 绑定参数
    query.setParameter("e", email);
    // 执行查询
    List<User> list = query.getResultList();
    return list.isEmpty() ? null : list.get(0);
}
```

一个简单的查询用 Criteria 写出来就像上面那样复杂，太恐怖了，如果条件多加几个，这种写法谁读得懂？

所以，正常人还是建议写 JPQL 查询，它的语法类似 SQL

```
public User getUserByEmail(String email) {
    // JPQL查询
    TypedQuery<User> query = em.createQuery("SELECT u FROM User u WHERE u.email = :e", User.class);
    query.setParameter("e", email);
    List<User> list = query.getResultList();
    if (list.isEmpty()) {
        throw new RuntimeException("User not found by email.");
    }
    return list.get(0);
}
```

插入操作的话类似这样

```
public User register(String email, String password, String name) {
    // 创建一个User对象
    User user = new User();
    // 设置好各个属性
    user.setEmail(email);
    user.setPassword(password);
    user.setName(name);
    // 不要设置id，因为使用了自增主键
    // 保存到数据库
    hibernateTemplate.save(user);
    // 现在已经自动获得了id
    System.out.println(user.getId());
    return user;
}
```

删除类似这样

```
public boolean deleteUser(Long id) {
    User user = hibernateTemplate.get(User.class, id);
    if (user != null) {
        hibernateTemplate.delete(user);
        return true;
    }
    return false;
}
```

更新类似这样

```
public void updateUser(Long id, String name) {
    User user = hibernateTemplate.load(User.class, id);
    user.setName(name);
    hibernateTemplate.update(user);
}
```

可以看出使用 ORM 的方式可以比较方便的对数据库中的数据进行操作。

### 4.3.4 MyBatis

使用 JPA 操作数据库时，这类 ORM 干的主要工作就是把 ResultSet 的每一行变成 JavaBean，或者把 JavaBean 自动转换到 INSERT 或 UPDATE 语句的参数中，从而实现 ORM。

而 ORM 框架之所以知道如何把行数据映射到 JavaBean，是因为我们在 JavaBean 的属性上给了足够的注解作为元数据，ORM 框架获取 JavaBean 的注解后，就知道如何进行双向映射。

那么，ORM 框架是如何跟踪 JavaBean 的修改，以便在 update() 操作中更新必要的属性？

答案是使用 Proxy 模式，从 ORM 框架读取的 User 实例实际上并不是 User 类，而是代理类，代理类继承自 User 类，但针对每个 setter 方法做了覆写

```
public class UserProxy extends User {
    boolean _isNameChanged;

    public void setName(String name) {
        super.setName(name);
        _isNameChanged = true;
    }
}
```

这样，代理类可以跟踪到每个属性的变化。

```
public class UserProxy extends User {
    Session _session;
    boolean _isNameChanged;

    public void setName(String name) {
        super.setName(name);
        _isNameChanged = true;
    }

    /**
     * 获取User对象关联的Address对象:
     */
    public Address getAddress() {
        Query q = _session.createQuery("from Address where userId = :userId");
        q.setParameter("userId", this.getId());
        List<Address> list = query.list();
        return list.isEmpty() ? null : list(0);
    }
}
```

为了实现这样的查询，UserProxy 必须保存 Hibernate 的当前 Session。但是，当事务提交后，Session 自动关闭，此时再获取 getAddress() 将无法访问数据库，或者获取的不是事务一致的数据。因此，ORM 框架总是引入了 Attached/Detached 状态，表示当前此 JavaBean 到底是在 Session 的范围内，还是脱离了 Session 变成了一个「游离」对象。很多初学者无法正确理解状态变化和事务边界，就会造成大量的 PersistentObjectException 异常。这种隐式状态使得普通 JavaBean 的生命周期变得复杂。

此外，Hibernate 和 JPA 为了实现兼容多种数据库，它使用 HQL 或 JPQL查询，经过一道转换，变成特定数据库的 SQL，理论上这样可以做到无缝切换数据库，但这一层自动转换除了少许的性能开销外，给 SQL 级别的优化带来了麻烦。

最后，ORM 框架通常提供了缓存，并且还分为一级缓存和二级缓存。

一级缓存是指在一个 Session 范围内的缓存，常见的情景是根据主键查询时，两次查询可以返回同一实例

```
User user1 = session.load(User.class, 123);
User user2 = session.load(User.class, 123);
```

二级缓存是指跨 Session 的缓存，一般默认关闭，需要手动配置。二级缓存极大的增加了数据的不一致性，原因在于 SQL 非常灵活，常常会导致意外的更新。例如

```
// 线程1读取
User user1 = session1.load(User.class, 123);
...
// 一段时间后，线程2读取
User user2 = session2.load(User.class, 123);
```

当二级缓存生效的时候，两个线程读取的 User 实例是一样的，但是，数据库对应的行记录完全可能被修改，例如

```
UPDATE users SET bonus = bonus + 100 WHERE createdAt <= ?
```

ORM 无法判断 id=123 的用户是否受该 UPDATE 语句影响。考虑到数据库通常会支持多个应用程序，此 UPDATE 语句可能由其他进程执行，ORM 框架就更不知道了。

我们把这种 ORM 框架称之为全自动 ORM 框架。

对比 Spring 提供的 JdbcTemplate，它和 ORM 框架相比，主要有几点差别

- 查询后需要手动提供 Mapper 实例以便把 ResultSet 的每一行变为 Java 对象
- 增删改操作所需的参数列表，需要手动传入，即把 User 实例变为 `[user.id, user.name, user.email]` 这样的列表，比较麻烦

但是 JdbcTemplate 的优势在于它的确定性：即每次读取操作一定是数据库操作而不是缓存，所执行的 SQL 是完全确定的，缺点就是代码比较繁琐，构造 `INSERT INTO users VALUES (?,?,?)` 更是复杂。

所以，介于全自动 ORM 如 Hibernate 和手写全部如 JdbcTemplate 之间，还有一种半自动的 ORM，它只负责把 ResultSet 自动映射到 Java Bean，或者自动填充 Java Bean 参数，但仍需自己写出 SQL。MyBatis 就是这样一种半自动化 ORM 框架。

MyBatis 使用 Mapper 来实现映射，而且 Mapper 必须是接口。我们以 User 类为例，在 User 类和 users 表之间映射的 UserMapper 编写如下

```
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User getById(@Param("id") long id);
}
```

注意：这里的 Mapper 不是 JdbcTemplate 的 RowMapper 的概念，它是定义访问 users 表的接口方法。比如我们定义了一个 User getById(long) 的主键查询方法，不仅要定义接口方法本身，还要明确写出查询的 SQL，这里用注解 @Select 标记。SQL 语句的任何参数，都与方法参数按名称对应。例如，方法参数 id 的名字通过注解 @Param() 标记为 id，则 SQL 语句里将来替换的占位符就是 `#{id}`。

如果有多个参数，那么每个参数命名后直接在 SQL 中写出对应的占位符即可

```
@Select("SELECT * FROM users LIMIT #{offset}, #{maxResults}")
List<User> getAll(@Param("offset") int offset, @Param("maxResults") int maxResults);
```

注意：MyBatis 执行查询后，将根据方法的返回类型自动把 ResultSet 的每一行转换为 User 实例，转换规则当然是按列名和属性名对应。如果列名和属性名不同，最简单的方式是编写 SELECT 语句的别名：

```
-- 列名是created_time，属性名是createdAt
SELECT id, name, email, created_time AS createdAt FROM users
```

执行 INSERT、UPDATE、DELETE 也大体是按这种模式来，这里不一一说明。

使用 MyBatis 最大的问题是所有 SQL 都需要全部手写，优点是执行的 SQL 就是我们自己写的 SQL，对 SQL 进行优化非常简单，也可以编写任意复杂的 SQL，或者使用数据库的特定语法，但切换数据库可能就不太容易。好消息是大部分项目并没有切换数据库的需求，完全可以针对某个数据库编写尽可能优化的 SQL。

## 4.4 Spring MVC

从之前的内容我们大致了解了 JavaEE 中 Web 开发的基础：Servlet。具体地说，有以下几点

- Servlet 规范定义了几种标准组件：Servlet、JSP、Filter 和 Listener
- Servlet 的标准组件总是运行在 Servlet 容器中，如 Tomcat、Jetty、WebLogic 等

直接使用 Servlet 进行 Web 开发好比直接在 JDBC 上操作数据库，比较繁琐，更好的方法是在 Servlet 基础上封装 MVC 框架，基于 MVC 开发 Web 应用，大部分时候，不需要接触 Servlet API，开发省时省力。

我们在介绍 Servlet 相关知识时大概说了下如何编写 MVC 框架。当然，自己写的 MVC 主要是理解原理，要实现一个功能全面的 MVC 需要大量的工作以及广泛的测试。

因此，开发 Web 应用，首先要选择一个优秀的 MVC 框架。常用的 MVC 框架有

- Struts：最古老的一个 MVC 框架，目前版本是 2，和 1.x 有很大的区别
- WebWork：一个比 Struts 设计更优秀的 MVC 框架，但不知道出于什么原因，从 2.0 开始把自己的代码全部塞给 Struts 2 了
- Turbine：一个重度使用 Velocity，强调布局的 MVC 框架
- 其他 100+ MVC 框架...（略）

Spring 虽然都可以集成任何 Web 框架，但是，Spring 本身也开发了一个 MVC 框架，就叫 Spring MVC。这个 MVC 框架设计得足够优秀以至于我们已经不想再费劲去集成类似 Struts 这样的框架了。

### 4.4.1 配置 Spring MVC

和普通 Spring 配置一样，我们编写正常的 AppConfig 后，只需加上 @EnableWebMvc 注解，就「激活」了 Spring MVC

```
@Configuration
@ComponentScan
@EnableWebMvc // 启用 Spring MVC
@EnableTransactionManagement
@PropertySource("classpath:/jdbc.properties")
public class AppConfig {
    ...
}
```

另外，需要创建几个用于 Spring MVC 的 Bean

```
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            registry.addResourceHandler("/static/**").addResourceLocations("/static/");
        }
    };
}
```

WebMvcConfigurer 并不是必须的，但我们在这里创建一个默认的 WebMvcConfigurer，只覆写 addResourceHandlers()，目的是让 Spring MVC 自动处理静态文件，并且映射路径为 `/static/**`。

另一个必须要创建的 Bean 是 ViewResolver，因为 Spring MVC 允许集成任何模板引擎，使用哪个模板引擎，就实例化一个对应的 ViewResolver

```
@Bean
ViewResolver createViewResolver(@Autowired ServletContext servletContext) {
    PebbleEngine engine = new PebbleEngine.Builder().autoEscaping(true)
            .cacheActive(false)
            .loader(new ServletLoader(servletContext))
            .extension(new SpringExtension())
            .build();
    PebbleViewResolver viewResolver = new PebbleViewResolver();
    viewResolver.setPrefix("/WEB-INF/templates/");
    viewResolver.setSuffix("");
    viewResolver.setPebbleEngine(engine);
    return viewResolver;
}
```

ViewResolver 通过指定 prefix 和 suffix 来确定如何查找 View。上述配置使用 Pebble 引擎，指定模板文件存放在 `/WEB-INF/tempaltes/` 目录下。

剩下的 Bean 都是普通的 @Component，但 Controller 必须标记为 @Controller，例如

```
// Controller 使用 @Controller 标记而不是 @Component
@Controller
public class UserController {
    // 正常使用 @Autowired 注入
    @Autowired
    UserService userService;

    // 处理一个 URL 映射
    @GetMapping("/")
    public ModelAndView index() {
        ...
    }
    ...
}
```

如果是普通的 Java 应用程序，我们通过 main() 方法可以很简单地创建一个 Spring 容器的实例

```
public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
}
```

但是问题来了，现在是 Web 应用程序，而 Web 应用程序总是由 Servlet 容器创建，那么，Spring 容器应该由谁创建？在什么时候创建？Spring 容器中的 Controller 又是如何通过 Servlet 调用的？

在 Web 应用中启动 Spring 容器有很多种方法，可以通过 Listener 启动，也可以通过 Servlet 启动，可以使用 XML 配置，也可以使用注解配置。这里，我们只介绍一种最简单的启动 Spring 容器的方式。

第一步，我们在 web.xml 中配置 Spring MVC 提供的 DispatcherServlet

```
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

初始化参数 contextClass 指定使用注解配置的 AnnotationConfigWebApplicationContext，配置文件的位置参数 contextConfigLocation 指向 AppConfig 的完整类名，最后，把这个 Servlet 映射到 `/*`，即处理所有 URL。

上述配置可以看作一个样板配置，有了这个配置，Servlet 容器会首先初始化 Spring MVC 的DispatcherServlet，在 DispatcherServlet 启动时，它根据配置 AppConfig 创建了一个类型是 WebApplicationContext 的 IoC 容器，完成所有 Bean 的初始化，并将容器绑到 ServletContext 上。

因为 DispatcherServlet 持有 IoC 容器，能从 IoC 容器中获取所有 @Controller 的 Bean，因此，DispatcherServlet 接收到所有 HTTP 请求后，根据 Controller 方法配置的路径，就可以正确地把请求转发到指定方法，并根据返回的 ModelAndView 决定如何渲染页面。

最后，我们在 AppConfig 中通过 main() 方法启动嵌入式 Tomcat

```
public static void main(String[] args) throws Exception {
    Tomcat tomcat = new Tomcat();
    tomcat.setPort(Integer.getInteger("port", 8080));
    tomcat.getConnector();
    Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
    WebResourceRoot resources = new StandardRoot(ctx);
    resources.addPreResources(
            new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
    ctx.setResources(resources);
    tomcat.start();
    tomcat.getServer().await();
}
```

上述 Web 应用程序就是我们使用 Spring MVC 时的一个最小启动功能集。


### 4.4.2 编写 Controller

有了 Web 应用程序的最基本的结构，我们的重点就可以放在如何编写 Controller 上。Spring MVC 对 Controller 没有固定的要求，也不需要实现特定的接口。以 UserController 为例，编写 Controller 只需要遵循以下要点

总是标记 @Controller 而不是 @Component

```
@Controller
public class UserController {
    ...
}
```

一个方法对应一个 HTTP 请求路径，用 @GetMapping 或 @PostMapping 表示 GET 或 POST 请求

```
@PostMapping("/signin")
public ModelAndView doSignin(
        @RequestParam("email") String email,
        @RequestParam("password") String password,
        HttpSession session) {
    ...
}
```

需要接收的 HTTP 参数以 @RequestParam() 标注，可以设置默认值。如果方法参数需要传入 HttpServletRequest、HttpServletResponse 或者 HttpSession，直接添加这个类型的参数即可，Spring MVC 会自动按类型传入。

返回的 ModelAndView 通常包含 View 的路径和一个 Map 作为 Model，但也可以没有 Model，例如

```
return new ModelAndView("signin.html"); // 仅 View，没有 Model
```

返回重定向时既可以写 `new ModelAndView("redirect:/signin")`，也可以直接返回 String

```
public String index() {
    if (...) {
        return "redirect:/signin";
    } else {
        return "redirect:/profile";
    }
}
```

如果在方法内部直接操作 HttpServletResponse 发送响应，返回 null 表示无需进一步处理

```
public ModelAndView download(HttpServletResponse response) {
    byte[] data = ...
    response.setContentType("application/octet-stream");
    OutputStream output = response.getOutputStream();
    output.write(data);
    output.flush();
    return null;
}
```

对 URL 进行分组，每组对应一个 Controller 是一种很好的组织形式，并可以在 Controller 的 class 定义出添加 URL 前缀，例如

```
@Controller
@RequestMapping("/user")
public class UserController {
    // 注意实际 URL 映射是 /user/profile
    @GetMapping("/profile")
    public ModelAndView profile() {
        ...
    }

    // 注意实际 URL 映射是 /user/changePassword
    @GetMapping("/changePassword")
    public ModelAndView changePassword() {
        ...
    }
}
```

实际方法的 URL 映射总是前缀 + 路径，这种形式还可以有效避免不小心导致的重复的 URL 映射。

可见，Spring MVC 允许我们编写既简单又灵活的 Controller 实现。

### 4.4.3 其它

使用 Spring MVC 开发 Web 应用程序的主要工作就是编写 Controller 逻辑。在 Web 应用中，除了需要使用 MVC 给用户显示页面外，还有一类 API 接口，我们称之为 REST，通常输入输出都是 JSON，便于第三方调用或者使用页面 JavaScript 与之交互。直接在 Controller 中处理 JSON 是可以的，不过要配合一大堆注解写 REST 太麻烦了，因此，Spring 额外提供了一个 @RestController 注解，使用 @RestController 替代 @Controller 后，每个方法自动变成 API 接口方法。

在 Servlet 中我们可以集成 Filter，Spring 也是支持的，另外还支持 Interceptor 只对 Controller 进行拦截。

另外，对于 CORS（Cross-Origin Resource Sharing：跨域资源共享）的处理、国际化、异步处理、使用 WebSocket 等等，这些 Spring 也有支持。

上面的内容的细节就不一一介绍了，大家了解下 Spring MVC 能做些什么就行。


## 4.5 集成第三方组件

Spring 框架不仅提供了标准的 IoC 容器、AOP 支持、数据库访问以及 Web MVC 等标准功能，还可以非常方便地集成许多常用的第三方组件。

### 4.5.1 JavaMail

在 Spring 中，可以集成 JavaMail 来收发电子邮件。

### 4.5.2 JMS

JMS，即 Java Message Service，是 JavaEE 的消息服务接口。JMS 主要有两个版本：1.1 和 2.0。2.0 和 1.1 相比，主要是简化了收发消息的代码。

### 4.5.3 Scheduler

在很多应用程序中，经常需要执行定时任务。

Java 标准库本身就提供了定时执行任务的功能。在 Spring 中，使用定时任务更简单，不需要手写线程池相关代码，只需要加几个注解就行

- 在 AppConfig 中标注 @EnableScheduling
- 在定时任务的方法上标注 @Scheduled（@Scheduled 需传入适当的参数）
- 在定时任务的方法所在的 class 标注 @Component

上面任务的调度都是在每个 JVM 进程中的。如果在本机启动两个进程，或者在多台机器上启动应用，这些进程的定时任务都是独立运行的，互不影响。

如果一些定时任务要以集群的方式运行，例如每天 23:00 执行检查任务，只需要集群中的一台运行即可，这个时候，可以考虑使用 Quartz。

### 4.5.4 JMX

JMX，即 Java Management Extensions，它是一个 Java 平台的管理和监控接口。为什么要搞 JMX 呢？因为在所有的应用程序中，对运行中的程序进行监控都是非常重要的，Java 应用程序也不例外。我们肯定希望知道 Java 应用程序当前的状态，例如，占用了多少内存，分配了多少内存，当前有多少活动线程，有多少休眠线程等等。如何获取这些信息呢？

为了标准化管理和监控，Java 平台使用 JMX 作为管理和监控的标准接口，任何程序，只要按 JMX 规范访问这个接口，就可以获取所有管理与监控信息。

实际上，常用的运维监控如 Zabbix、Nagios 等工具对 JVM 本身的监控都是通过 JMX 获取的信息。

因为 JMX 是一个标准接口，所以它不但可以用于管理 JVM，还可以管理应用程序自身。

# 5 Spring Boot

Spring Framework 主要功能包括 IoC 容器、AOP 支持、MVC 开发以及强大的第三方集成功能等。

那么，Spring Boot 又是什么？它和 Spring 是什么关系？

Spring Boot 是一个基于 Spring 的套件，它帮我们预组装了 Spring 的一系列组件，以便以尽可能少的代码和配置来开发基于 Spring 的 Java 应用程序。

以汽车为例，如果我们想组装一辆汽车，我们需要发动机、传动、轮胎、底盘、外壳、座椅、内饰等各种部件，然后把它们装配起来。Spring 就相当于提供了一系列这样的部件，但是要装好汽车上路，还需要我们自己动手。而 Spring Boot 则相当于已经帮我们预装好了一辆可以上路的汽车，如果有特殊的要求，例如把发动机从普通款换成涡轮增压款，可以通过修改配置或编写少量代码完成。

因此，Spring Boot 和 Spring 的关系就是整车和零部件的关系，它们不是取代关系，试图跳过 Spring 直接学习 Spring Boot 是不可能的。

Spring Boot 的目标就是提供一个开箱即用的应用程序架构，我们基于 Spring Boot 的预置结构继续开发，省时省力。

## 5.1 目录结构

Spring Boot 的目录结构类似这样

```
springboot-hello
├── pom.xml
├── src
│   └── main
│       ├── java
│       └── resources
│           ├── application.yml
│           ├── logback-spring.xml
│           ├── static
│           └── templates
└── target
```

其中，在 `src/main/resources` 目录下，注意到几个文件

- `application.yml`

这是 Spring Boot 默认的配置文件，它采用 YAML 格式而不是 `.properties` 格式，文件名必须是 `application.yml` 而不是其他名称。

YAML 格式比 `key=value` 格式的 `.properties` 文件更易读。比较一下两者的写法

使用 `.properties` 格式

```
# application.properties

spring.application.name=${APP_NAME:unnamed}

spring.datasource.url=jdbc:hsqldb:file:testdb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.dirver-class-name=org.hsqldb.jdbc.JDBCDriver

spring.datasource.hikari.auto-commit=false
spring.datasource.hikari.connection-timeout=3000
spring.datasource.hikari.validation-timeout=3000
spring.datasource.hikari.max-lifetime=60000
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=1
```

使用 YAML 格式

```
# application.yml

spring:
  application:
    name: ${APP_NAME:unnamed}
  datasource:
    url: jdbc:hsqldb:file:testdb
    username: sa
    password:
    dirver-class-name: org.hsqldb.jdbc.JDBCDriver
    hikari:
      auto-commit: false
      connection-timeout: 3000
      validation-timeout: 3000
      max-lifetime: 60000
      maximum-pool-size: 20
      minimum-idle: 1
```

可见，YAML 是一种层级格式，它和 `.properties` 很容易互相转换，它的优点是去掉了大量重复的前缀，并且更加易读。

在配置文件中，我们可以使用环境变量对 key 进行配置

```
app:
  db:
    host: ${DB_HOST:localhost}
    user: ${DB_USER:root}
    password: ${DB_PASSWORD:password}
```

这种 `${DB_HOST:localhost}` 意思是，首先从环境变量查找 `DB_HOST`，如果环境变量定义了，那么使用环境变量的值，否则，使用默认值 localhost。

这使得我们在开发和部署时更加方便，因为开发时无需设定任何环境变量，直接使用默认值即本地数据库，而实际线上运行的时候，只需要传入环境变量即可

```
$ DB_HOST=10.0.1.123 DB_USER=prod DB_PASSWORD=xxxx java -jar xxx.jar
```

- `logback-spring.xml`

这是 Spring Boot 的 logback 配置文件名称（也可以使用 `logback.xml`）。

另外，static 是静态文件目录，templates 是模板文件目录。


## 5.2 源码目录结构

我们来看源码目录结构

```
src/main/java
└── com
    └── itranswarp
        └── learnjava
            ├── Application.java
            ├── entity
            │   └── User.java
            ├── service
            │   └── UserService.java
            └── web
                └── UserController.java
```

在存放源码的 `src/main/java` 目录中，Spring Boot 对 Java 包的层级结构有一个要求。注意到我们的根 package 是 `com.itranswarp.learnjava`，下面还有 entity、service、web 等子 package。Spring Boot 要求 main() 方法所在的启动类必须放到根 package 下，命名不做要求，这里我们以 Application.java 命名，它的内容如下

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }
}
```

启动 Spring Boot 应用程序只需要一行代码加上一个注解 @SpringBootApplication，该注解实际上又包含了

- @SpringBootConfiguration
    - @Configuration
- @EnableAutoConfiguration
    - @AutoConfigurationPackage
- @ComponentScan

这样一个注解就相当于启动了自动配置和自动扫描。

我们再观察 `pom.xml`，它的内容如下

```
<project ...>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
    </parent>

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>springboot-hello</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
        <pebble.version>3.1.2</pebble.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <!-- 集成Pebble View -->
        <dependency>
            <groupId>io.pebbletemplates</groupId>
            <artifactId>pebble-spring-boot-starter</artifactId>
            <version>${pebble.version}</version>
        </dependency>

        <!-- JDBC驱动 -->
        <dependency>
            <groupId>org.hsqldb</groupId>
            <artifactId>hsqldb</artifactId>
        </dependency>
    </dependencies>
</project>
```

使用 Spring Boot 时，强烈推荐从 `spring-boot-starter-parent` 继承，因为这样就可以引入 Spring Boot 的预置配置。

紧接着，我们引入了依赖 `spring-boot-starter-web` 和 `spring-boot-starter-jdbc`，它们分别引入了 Spring MVC 相关依赖和 Spring JDBC 相关依赖，无需指定版本号，因为引入的 `<parent>` 内已经指定了，只有我们自己引入的某些第三方 jar 包需要指定版本号。这里我们引入 `pebble-spring-boot-starter` 作为 View，以及 hsqldb 作为嵌入式数据库。hsqldb 已在 `spring-boot-starter-jdbc` 中预置了版本号 `2.5.0`，因此此处无需指定版本号。

根据 `pebble-spring-boot-starter` 的文档，加入如下配置到 `application.yml`

```
pebble:
  # 默认为".pebble"，改为"":
  suffix:
  # 开发阶段禁用模板缓存:
  cache: false
```

对 Application 稍作改动，添加 WebMvcConfigurer 这个 Bean

```
@SpringBootApplication
public class Application {
    ...

    @Bean
    WebMvcConfigurer createWebMvcConfigurer(@Autowired HandlerInterceptor[] interceptors) {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                // 映射路径`/static/`到classpath路径:
                registry.addResourceHandler("/static/**")
                        .addResourceLocations("classpath:/static/");
            }
        };
    }
}
```

现在就可以直接运行 Application。Spring Boot 自动启动了嵌入式 Tomcat，当看到 Started Application in xxx seconds 时，Spring Boot 应用启动成功。

现在有个问题，数据源、声明式事务、JdbcTemplate 等在哪创建的？怎么就可以直接注入到自己编写的 UserService 中呢？

这些自动创建的 Bean 就是 Spring Boot 的特色：AutoConfiguration。

当我们引入 `spring-boot-starter-jdbc` 时，启动时会自动扫描所有的 `XxxAutoConfiguration`

- DataSourceAutoConfiguration：自动创建一个 DataSource，其中配置项从 `application.yml` 的 `spring.datasource` 读取
- DataSourceTransactionManagerAutoConfiguration：自动创建了一个基于 JDBC 的事务管理器
- JdbcTemplateAutoConfiguration：自动创建了一个 JdbcTemplate

因此，我们自动得到了一个 DataSource、一个 DataSourceTransactionManager 和一个 JdbcTemplate。

类似的，当我们引入 `spring-boot-starter-web` 时，自动创建了

- ServletWebServerFactoryAutoConfiguration：自动创建一个嵌入式 Web 服务器，默认是 Tomcat
- DispatcherServletAutoConfiguration：自动创建一个 DispatcherServlet
- HttpEncodingAutoConfiguration：自动创建一个 CharacterEncodingFilter
- WebMvcAutoConfiguration：自动创建若干与 MVC 相关的 Bean
- ...

引入第三方 `pebble-spring-boot-starter` 时，自动创建了

- PebbleAutoConfiguration：自动创建了一个 PebbleViewResolver

Spring Boot 大量使用 XxxAutoConfiguration 来使得许多组件被自动化配置并创建，而这些创建过程又大量使用了 Spring 的 Conditional 功能。例如，我们观察 JdbcTemplateAutoConfiguration，它的代码如下

```
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ DataSource.class, JdbcTemplate.class })
@ConditionalOnSingleCandidate(DataSource.class)
@AutoConfigureAfter(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties(JdbcProperties.class)
@Import({ JdbcTemplateConfiguration.class, NamedParameterJdbcTemplateConfiguration.class })
public class JdbcTemplateAutoConfiguration {
}
```

当满足条件

- @ConditionalOnClass：在 classpath 中能找到 DataSource 和 JdbcTemplate
- `@ConditionalOnSingleCandidate(DataSource.class)`：在当前 Bean 的定义中能找到唯一的 DataSource

该 JdbcTemplateAutoConfiguration 就会起作用。实际创建由导入的 JdbcTemplateConfiguration 完成

```
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(JdbcOperations.class)
class JdbcTemplateConfiguration {
    @Bean
    @Primary
    JdbcTemplate jdbcTemplate(DataSource dataSource, JdbcProperties properties) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
        JdbcProperties.Template template = properties.getTemplate();
        jdbcTemplate.setFetchSize(template.getFetchSize());
        jdbcTemplate.setMaxRows(template.getMaxRows());
        if (template.getQueryTimeout() != null) {
            jdbcTemplate.setQueryTimeout((int) template.getQueryTimeout().getSeconds());
        }
        return jdbcTemplate;
    }
}
```

创建 JdbcTemplate 之前，要满足 `@ConditionalOnMissingBean(JdbcOperations.class)`，即不存在 JdbcOperations 的 Bean。

如果我们自己创建了一个 JdbcTemplate，例如，在 Application 中自己写个方法

```
@SpringBootApplication
public class Application {
    ...
    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

那么根据条件 `@ConditionalOnMissingBean(JdbcOperations.class)`，Spring Boot 就不会再创建一个重复的 JdbcTemplate（因为 JdbcOperations 是 JdbcTemplate 的父类）。

可见，Spring Boot 自动装配功能是通过自动扫描 + 条件装配实现的，这一套机制在默认情况下工作得很好，但是，如果我们要手动控制某个 Bean 的创建，就需要详细地了解 Spring Boot 自动创建的原理，很多时候还要跟踪 XxxAutoConfiguration，以便设定条件使得某个 Bean 不会被自动创建。

小结下

- Spring Boot 是一个基于 Spring 提供了开箱即用的一组套件，它可以让我们基于很少的配置和代码快速搭建出一个完整的应用程序
- Spring Boot 有非常强大的 AutoConfiguration 功能，它是通过自动扫描 + 条件装配实现的


# 6 Spring Cloud

Spring Cloud，顾名思义是跟云相关的，云程序实际上就是指分布式应用程序，所以 Spring Cloud 就是为了让分布式应用程序编写更方便，更容易而提供的一组基础设施，它的核心是 Spring 框架，利用 Spring Boot 的自动配置，力图实现最简化的分布式应用程序开发。

# 7 小结

本文泛泛的介绍了目前 Java Web 的技术体系，主要是 Servlet 和 Spring。文章内容大部分是从廖雪峰的《Java 教程》中摘录过来的，忽略了很多细节，目的是让一个刚接触 Java Web 的同学对当前的技术体系有个大致的认识，同时知道哪些是重点知识。

有了这些了解后，你就可以结合自己的情况有选择性的进行学习了。比如，如果你工作中需要使用 Spring Boot 框架，那就了解下 Spring 的 IoC、AOP 等基础概念后直接进行 Spring Boot 的学习，在能完成基本的开发需求后，再慢慢熟悉底层的原理，如：怎么和 Spring Framework 结合的，后者是怎么设计的，Spring MVC 又是怎么和 Servlet 结合的，等等。也就是要知道自己学习的一个短期和稍微长期的目标。

至于选择什么资料进行学习，本文参考的《Java 教程》挺适合初学者的，入门之后你可以尝试看下相关的官方文档、书籍、源码啥的，这些就不在本文的内容范围内了，你可以根据自己的需要在搜索引擎上检索下。

# 8 参考

- https://www.liaoxuefeng.com/wiki/1252599548343744/1255945497738400
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266263217140032
- 《Spring 实战（第 5 版）》，第 1 部分（Spring 基础），第 3 章（使用数据）
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266265175882464
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266263401691296

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
- 支持 JDBC、声明式事务、ORM 的数据访问模块
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

在 IoC 模式下，控制权发生了反转，即从应用程序转移到了 IoC 容器，所有组件不再由应用程序自己创建和配置，而是由 IoC 容器负责，这样，应用程序只需要直接使用已经创建好并且配置好的组件。为了能让组件在 IoC 容器中被「装配」出来，需要某种「注入」机制，例如，BookService 自己并不会创建DataSource，而是等待外部通过 setDataSource() 方法来注入一个DataSource

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
- 测试 BookService 更容易，因为注入的是 DataSource，可以使用内存数据库，而不是真实的MySQL配置

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

很多Java类都具有带参数的构造方法，如果我们把 BookService 改造为通过构造方法注入，那么实现代码如下：

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
- 切面逻辑，即：权限检查的 Aspect、日志的 Aspect、事务的 Aspect

然后，以某种方式，让框架来把上述 3 个 Aspect 以 Proxy 的方式「织入」到 BookService 中，这样一来，就不必编写复杂而冗长的 Proxy 模式。

在 Java 平台上，对于 AOP 的织入，有 3 种方式 

1. 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ 就扩展了 Java 编译器，使用关键字 aspect 来实现织入
2. 类加载器：在目标类被装载到 JVM 时，通过一个特殊的类加载器，对目标类的字节码重新「增强」
3. 运行期：目标对象和切面都是普通 Java 类，通过 JVM 的动态代理功能或者第三方库实现运行期动态织入

在 Spring 中，我们可以引入 AspectJ 依赖实现 AOP，这里细节不进行说明，大家了解下 AOP 的基本作用和大致原理就行。

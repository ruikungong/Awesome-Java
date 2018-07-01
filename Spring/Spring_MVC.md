# Spring MVC

## 1、简介

### 1.1 Web MVC

在Web开发中，通常是浏览器发送请求到服务器，由服务器接收请求并将响应传递给客户端，并由客户端渲染之后展示给用户。因此，一般服务器是无法主动通知客户端更新内容的，虽然有些推送技术可以实现主动通知客户端。

在标准的MVC里，服务器是可以主动将数据推送给客户端的，但是实际的WebMVC是做不到的，如果用户想要视图更新，需要再发送一次请求。

Web端的开发经历了从**CGI->Servlet->JSP->Model1->Model2->Front Controller+PageController**的过程：

- **CGI**：(Common Gateway Interface)公共网关接口，用于接收web用户请求并处理，最后动态产生响应给用户，但每次请求将产生一个进程，重量级。
- **Servlet**：接收web用户请求并处理，最后动态产生响应给用户。但每次请求只产生一个线程（而且有线程池），轻量级。本质就是在java代码里面 输出 html流。
- **JSP**：运行在标准的HTML页面中嵌入脚本语言，本质就是在html代码中嵌入java代码。JSP最终还是会被编译为Servlet。
- **Model1**：JSP的增强版，可以认为是jsp+javabean，使用<jsp:useBean>标准动作简化javabean的获取/创建，及将请求参数封装到javabean。
- **Model2**：Web MVC模型，只是控制器采用Servlet、模型采用JavaBean、视图采用JSP。
- **Front Controller+PageController**：即前端控制器+应用控制器+页面控制器（也有称其为动作）+上下文，也是Web MVC，只是责任更加明确。

### 1.2 Spring MVC

Spring MVC与Spring搭配可以为我们提供强大的功能，更加简洁的配置方式。以下是Spring Web MVC处理请求的流程：

![Spring Web MVC处理请求的流程](res/spring_mvc_concept.JPG)

从上面的图中，我们总结Spring MVC的请求处理流程。当用户发送请求之后：

1. 用户请求达到**前端控制器**，前端控制器根据URL找到处理请求的**控制器**，并把请求委托给它；
2. 控制器收到请求之后会调用**业务处理对象**对其进行处理，并对得到的数据模型进行处理从而得到**ModelAndView**，并将其返回给前端控制器；
3. 前端控制器根据返回的视图名选择相应的视图进行渲染，并把最终的响应返回给用户。

结合Spring MVC框架中具体的类，我们又可以得到下面的这张更加详尽的图：

![Spring Web MVC架构](res/spring_mvc_framework.JPG)

在SpringMVC中的核心的类是`DispatcherServlet`，根据上面的分析，它可以算的上是中间调度的桥梁。我们引入Spring MVC的依赖之后看下相关的代码。
在`DispatcherServlet`中核心的方法是`doDispatch()`，下面是它的代码：

    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            try {
                ModelAndView mv = null;
                Object dispatchException = null;

                try {
                    // 1.首先查看是否是Multipart，即是否包含要上传的文件
                    processedRequest = this.checkMultipart(request);
                    multipartRequestParsed = processedRequest != request;
                    // 2.根据请求和handlerMappings(列表)，找到对应的HandlerExecutionChain
                    mappedHandler = this.getHandler(processedRequest);
                    if (mappedHandler == null || mappedHandler.getHandler() == null) {
                        this.noHandlerFound(processedRequest, response);
                        return;
                    }

                    // 3.同样的方式，从一个HandlerAdapter的列表中获取到对应的HandlerAdapter
                    HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
                    String method = request.getMethod();
                    boolean isGet = "GET".equals(method);
                    if (isGet || "HEAD".equals(method)) {
                        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                        if (this.logger.isDebugEnabled()) {
                            this.logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                        }

                        if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                            return;
                        }
                    }

                    // 4.执行处理器相关的拦截器的预处理
                    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                        return;
                    }

                    // 5.由适配器执行处理器（调用处理器相应功能处理方法），注意返回了ModelAndView
                    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
                    if (asyncManager.isConcurrentHandlingStarted()) {
                        return;
                    }

                    this.applyDefaultViewName(processedRequest, mv);
                    // 6.执行处理器相关的拦截器的后处理
                    mappedHandler.applyPostHandle(processedRequest, response, mv);
                } catch (Exception var20) {
                    dispatchException = var20;
                } catch (Throwable var21) {
                    dispatchException = new NestedServletException("Handler dispatch failed", var21);
                }
				
                // 7.处理分发结果，其中包含了一部分异常处理，也包括根据视图名称获取视图解析器并进行渲染等等
                this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
            } catch (Exception var22) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
            } catch (Throwable var23) {
                this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
            }
        } finally {
            if (asyncManager.isConcurrentHandlingStarted()) {
                if (mappedHandler != null) {
                    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
                }
            } else if (multipartRequestParsed) {
                // 清理multipart请求占用的资源
                this.cleanupMultipart(processedRequest);
            }
        }
    }

    // 该方法用来处理从处理器拿到的结果，不论是异常还是得到的ModelAndView都会被处理
    private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
            HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
        boolean errorView = false;
		
        // 处理异常信息
        if (exception != null) {
            if (exception instanceof ModelAndViewDefiningException) {
                this.logger.debug("ModelAndViewDefiningException encountered", exception);
                mv = ((ModelAndViewDefiningException)exception).getModelAndView();
            } else {
                Object handler = mappedHandler != null ? mappedHandler.getHandler() : null;
                mv = this.processHandlerException(request, response, handler, exception);
                errorView = mv != null;
            }
        }

        // 解析视图并进行视图的渲染 
        if (mv != null && !mv.wasCleared()) {
            // 实际的渲染方法，会根据ModelAndView的名称找到对应的视图解析器，并渲染得到一个视图View
            this.render(mv, request, response);
            if (errorView) {
                WebUtils.clearErrorRequestAttributes(request);
            }
        } else if (this.logger.isDebugEnabled()) {
            this.logger.debug("Null ModelAndView returned to DispatcherServlet with name '" + this.getServletName() + "': assuming HandlerAdapter completed request handling");
        }

        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
                mappedHandler.triggerAfterCompletion(request, response, (Exception)null);
            }
        }
    }

所以，我们可以总结Spriung MVC的工作原理如下：

1. 用户发送的请求首先会到达`DispatcherServlet`，并由它进行处理；
2. `DispatcherServlet`先通过`HandlerMapping`找到该请求对应的`HandlerExecutionChain`（包含一个`Handler`处理器、多个`HandlerInterceptor`拦截器），通过这种策略模式，很容易添加新的映射策略；
3. 然后`DispatcherServlet`继续将请求发送给`HandlerAdapter`，`HandlerAdapter`会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式，从而很容易支持很多类型的处理器；
4. `HandlerAdapter`将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理，并返回一个`ModelAndView`对象（包含模型数据、逻辑视图名）；
5. 再由`DispatcherServlet`根据`ModelAndView`找到对应的`ViewResolver`，并由它把逻辑视图名解析为具体的`View`，通过这种策略模式，很容易更换其他视图技术；
6. 接下来会对`View`进行渲染，`View`会根据传进来的`Model`模型数据进行渲染，Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. 返回控制权给`DispatcherServlet`，由`DispatcherServlet`返回响应给用户，到此一个流程结束。

## 2、使用Spring MVC

### 2.1 Spring MVC基于XML的配置方式

首先，要使用Spring MVC需要加入相关的依赖：

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>

我们上面分析过的`DispatcherServlet`是注册到web.xml里面的，我们需要加入如下的配置：

	<!DOCTYPE web-app PUBLIC
			"-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
			"http://java.sun.com/dtd/web-app_2_3.dtd" >
	<web-app>
		<display-name>Archetype Created Web Application</display-name>

		<servlet>
			<servlet-name>spring_is_coming</servlet-name>
			<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<load-on-startup>1</load-on-startup>
		</servlet>
		<servlet-mapping>
			<servlet-name>spring_is_coming</servlet-name>
			<url-pattern>*.mvc</url-pattern>
		</servlet-mapping>
	</web-app>

这里我们注册了一个`servlet`，即上面提到的`DispatcherServlet`，并为其指定一个名称为`spring_is_coming`。对应的，我们还要为其指定一个`servlet-mapping`。
如上所示，我们在`url-pattern`中指定了该servlet要处理的url的模式，即所有以`*.mvc`结尾的url。load-on-startup表示启动容器时初始化该Servlet。

默认情况下，`DispatcherServlet`会加载`WEB-INF/[DispatcherServlet的Servlet名字]-servlet.xml`下面的配置文件。根据上面的配置我们需要在当前项目的`WEB-INF`目录下面加入`spring_is_coming-servlet.xml`文件。并在该文件中加入如下的代码：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
       xmlns:p="http://www.springframework.org/schema/p">

    <!-- HandlerMapping -->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!-- HandlerAdapter -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!-- ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean name="/hello.mvc" class="me.shouheng.spring.mvc.HelloController"/>

</beans>
```

这里我们声明了几个Bean，即HandlerMapping和HandlerAdapter的默认实现。然后，我们还定义了一个视图解析器`InternalResourceViewResolver`。
它是一个默认的视图解析器，在它的属性配置中，我们指定了加载的文件的前缀和后缀路径。实际上，当我们指定某个视图名字为hello，那么该视图解析器就会加载文件
`/WEB-INF/jsp/hello.jsp`。

接下来我们定义了一个控制器，该控制器的名字对应于指定的url。因为之前我们使用了Bean的映射规则是`BeanNameUrlHandlerMapping`，也就是说Bean的名称和url对应。

然后就是上面定义的那个控制器的代码：

```
public class HelloController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView mv = new ModelAndView();
        // 添加模型数据 可以是任意的POJO对象
        mv.addObject("message", "Hello Spring MVC!");
        // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("hello");
        return mv;
    }
}
```

这里我们用了`ModelAndView.setViewName()`为该ModelAndView指定了对应的jsp文件的名称，会按照我们上面配置的视图解析的规则到指定目录下记载指定名称的文件。

启动服务器，在浏览器中输入地址：http://localhost:8080/hello.mvc，进行测试即可。

### 2.2 使用过滤器

我们还可以在分发器处理请求之前对其进行处理，我们通过过滤器来实现。我们可以用过滤器来做一些基础的工作，比如字符串编码之类的问题。下面我们通过一个自定义的简单的例子来演示一下过滤器的使用：

首先，我们自定义一个简单的过滤器，并在其中输出一些信息到控制台上：

```
public class TheFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init, filterConfig" + filterConfig);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("==================================== doFilter, servletRequest: "
                + servletRequest
                + "\nservletResponse: "
                + servletResponse
                + "\nfilterChain: " + filterChain);
        // 必须调用这个方法，否则请求就要在这里被拦截了
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    }
}
```

这里我们要实现Fileter接口的三个与声明周期相关的方法。然后，将其配置到web.xml中就可以了：


    <filter>
        <filter-name>the_filter</filter-name>
        <filter-class>me.shouheng.spring.mvc.TheFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>the_filter</filter-name>
        <url-pattern>*.mvc</url-pattern>
    </filter-mapping>

这样我们就可以对所有以`*.mvc`结尾的请求进行处理了。

### 2.3 DispatcherServlet详解

**实际上DispatcherServlet是Servlet的一种，也就是当我们使用Spring MVC的时候需要配置该类，因为它是Spring MVC的核心配置类。而当我们不打算使用Spring MVC而想要为其他客户端提供接口的时候就需要配置其他类型的Servlet，不过它们的配置和在Servlet种被使用的原理都是一样的。**。

上面我们已经提到过了`DispatcherServlet`的两个参数的意义，并且使用了配置文件`spring_is_coming-servlet.xml`。而实际上为`DispatcherServlet`指定配置文件的方式可以有多种：

第一种，默认会使用`[DispatcherServlet的Servlet名字]-servlet.xml`这种命名规则到`WEB-INF`目录下面加载该文件，这也是我们上面使用的方式。

第二种方式是在配置servlet的时候，在`<servlet>`标签中使用`contextClass`，并且指定一个配置类，该配置类需要实现`WebApplicationContext`接口的类。如果这个参数没有指定， 默认使用`XmlWebApplicationContext`。

第三种方式是与第二种类似，都是在配置servlet的时候指定，不过这里使用的是`contextConfigLocation`，并用字符串来指定上下文的位置。这个字符串可以被分成多个字符串（使用逗号作为分隔符） 来支持多个上下文（在多上下文的情况下，如果同一个bean被定义两次，后面一个优先）。如下所示：

   <servlet>
        <servlet-name>chapter2</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-servlet-config.xml</param-value>
        </init-param>
    </servlet>

通常在我们配置上下文的时候会指定多个上下文，各个上下文也有自己的职责范围。除了之前我们配置的Servlet的上下文，在使用Spring的时候，我们还需要配置整个应用的上下文：

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
         classpath:spring-common-config.xml,
         classpath:spring-budget-config.xml
    </param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

这里配置的就是整个应用的上下文，配置的`ContextLoaderListener`会在容器启动的时候自动初始化应用程序上下文。而该应用程序上下文的配置文件就由上面的`context-param`来指定。应用程序上下文通常用来加载整个程序的基础类，比如DAO层和Service层等。这样它们就可以与任何其他的Web层配合使用。而Servlet配置的上下文通常用来加载Web层需要的类，比如Controller、HandlerMapping、HandlerAdapter等等。














### 3.使用注解定义SpringMVC

#### 1.示例

首先定义一个控制器，这里使用@RequestMapping注解指定它映射到的路径，

	@Controller
	public class HelloController {
	
	    @RequestMapping(value = "/hello")
	    public String sayHello() throws Exception {
	        ModelAndView mv = new ModelAndView();
	        //添加模型数据 可以是任意的POJO对象
	        mv.addObject("message", "Hello World!");
	        //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
	        mv.setViewName("hello");
	        return "hello";
	    }
    }

在springmvc-servlet.xml中做如下定义：

    <context:component-scan base-package="my.shouheng.*"/>
    <context:annotation-config/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

注意这里使用了默认的HandlerMapping和Adapter。这里的包名会有影响。

浏览器输入http://localhost:8080/palm/web/hello可以达到相同的效果。

这里的@RequstMapping也可以被用在类上面，还可以指定多个映射路径，比如@RequestMapping({"/", "/homepage"})

#### 2.接受请求的输入


1.通过路径参数接受输入：

    @RequestMapping(value = "/hello")
    public String sayHello(@RequestParam(value = "num", defaultValue = "10") int num) throws Exception {
        System.out.println(num);
        return "hello";
    }

这里通过@RequestParam指定参数的名称，可以使用defaultValue指定默认参数。

使用的方式是：http://localhost:8080/palm/web/hello?num=22

另一种方式是:

    @RequestMapping(value = "/hi/{num}")
    public String sayHi(@PathVariable("num") int num) throws Exception {
        System.out.println(num);
        return "hello";
    }

它的使用方式是：http://localhost:8080/palm/web/hi/22



### 4.过滤器

首先定义Filter：

	public class TheFilter implements Filter{
	
	    public void init(FilterConfig filterConfig) throws ServletException {
	        System.out.println("init, filterConfig" + filterConfig);
	    }
	
	    public void doFilter(ServletRequest servletRequest,
	                         ServletResponse servletResponse,
	                         FilterChain filterChain) throws IOException, ServletException {
	        System.out.println("doFilter, servletRequest: " + servletRequest
	                + "\nservletResponse: " + servletResponse
	                + "\nfilterChain: " + filterChain);
	        // 必须调用这个方法，否则请求就要在这里被拦截了
	        filterChain.doFilter(servletRequest, servletResponse);
	    }
	
	    public void destroy() {
	        System.out.println("destroy");
	    }
	}

注意，如果想要请求继续执行就必须调用filterChain.doFilter(servletRequest, servletResponse)，它让其他的过滤器对请求继续进行处理。如果没有这句话的话，该请求就会在这里被拦截掉。

然后在XML中配置过滤器：

    <filter>
        <filter-name>TheFilter</filter-name>
        <filter-class>my.shouheng.mvc.TheFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>TheFilter</filter-name>
        <url-pattern>/web/*</url-pattern>
    </filter-mapping>






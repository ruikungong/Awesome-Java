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

### 2.4 拦截器

在`DispatcherServlet`种拦截器处理的逻辑是非常简单易懂的。
`DispatcherServlet`会在的核心方法`doDispatch()`的不同的处理阶段调用`HandlerExecutionChain`的三个方法（在新版本的Spring中相关的逻辑被抽取出来封装成了独立的方法）。

1. `HandlerExecutionChain.applyPreHandle()`会在`Contoller`的方法被执行之前调用；
2. `HandlerExecutionChain.applyPostHandle()`会在`Contoller`的方法被执行之后，并且视图被渲染之前调用，如果中间出现异常则不会被调用；`
3. `HandlerExecutionChain.triggerAfterCompletion()会在`Contoller`的视图被渲染之后调用，无论是否异常，总是被调用；`

那么，在这三个方法中又做了什么呢？下面就是相关的逻辑，实际上三个方法是类似的，即从一个数组中取出定义的拦截器进行遍历调用：

    boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = 0; i < interceptors.length; this.interceptorIndex = i++) {
                HandlerInterceptor interceptor = interceptors[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    this.triggerAfterCompletion(request, response, (Exception)null);
                    return false;
                }
            }
        }
        return true;
    }

所以，这部分的逻辑也不复杂。那么，那么我们看下如何在Spring MVC中使用拦截器：

    <bean name="interceptor" class="me.shouheng.spring.mvc.TestInterceptor"/>

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/hello2.mvc"/>
            <ref bean="interceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

我们还是在之前的`spring_is_coming-servlet.xml`中加入上面的几行代码，这里我们用到了一个自定义的拦截器，下面我们给出它的定义：

```
public class TestInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("=========preHandle");
        return super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("=========postHandle");
        super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("=========afterCompletion");
        super.afterCompletion(request, response, handler, ex);
    }

    @Override
    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("=========afterConcurrentHandlingStarted");
        super.afterConcurrentHandlingStarted(request, response, handler);
    }
}
```

然后，我们使用`<mvc:interceptor>`标签定义了对应的拦截器及其要匹配的路径。这样当访问指定的url时，在触发响应的Controller的时候就会调用到我们定义的拦截器。

这里我们还要说明一下`HandlerInterceptorAdapter`中的几个方法。
实际上所有的拦截器最终都是实现了接口`HandlerInterceptor`，而上面的类中的前三个方法实际上就是来自于该接口。
在`preHandle()`的返回值默认是true，表示当前拦截器处理完毕之后会继续让下一个拦截器来处理。
实际上参考上面的`HandlerExecutionChain.applyPreHandle()`方法也能看出这一点。

### 2.5 基于注解的配置方式

在基于注解的配置方式中，我们需要对上面的配置做一些修改。首先，我们使用基于注解的适配器和映射机制。在`spring_is_coming-servlet.xml`中，我们将之前的代码替换为：

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>

    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

注意上面的这两行代码是Spring 3.1之后的配置方式，而3.1之前的代码在最新的Spring版本中已经移除，不再赘诉。

然后，我们定义如下的Controller：

```
@Controller
public class HelloController3 {

    @RequestMapping(value = "/hello3")
    public ModelAndView handle() {
        ModelAndView mv = new ModelAndView();
        // 添加模型数据 可以是任意的POJO对象
        mv.addObject("message", "Hello Spring MVC!");
        // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("hello3");
        return mv;
    }
}
```

这里的配置方式和之前基于Bean名称映射的机制类似，只是这里使用都是基于注解的配置方式。在作为Controller使用的类上面，我们要使用`@Controller`注解，具体的业务层方法上面使用`@RequestMapping`注解并指定映射的路径。然后，我们将该Bean注册到上下文当中：

    <bean class="me.shouheng.spring.mvc.HelloController3"/>

这样基本的配置方式就已经完成了。然后，运行Web容器并输入url:http://localhost:8080/hello3即可。还要注意下，还要修改web.xml中的Servlet的匹配路径，如果是`*.mvc`的话要改成`/`。

其实这里的配置方式和之前的配置方式唯一的区别也就在于，将映射的规则从之前的**Bean名到url**转换成了**注解到url**。

除了上面的那种方式配置url路径，我们还可以添加各种子路径。比如：

```
@Controller
@RequestMapping("/user")
public class HelloController3 {

    @RequestMapping(value = "/hello3")
    public ModelAndView handle() {
        ModelAndView mv = new ModelAndView();
        // 添加模型数据 可以是任意的POJO对象
        mv.addObject("message", "Hello Spring MVC!");
        // 设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
        mv.setViewName("hello3");
        return mv;
    }
}
```

按照上面的方式，将会匹配到：/user/hello3。

从上面看出使用注解的配置方式中，核心的配置应该属于`@RequestMapping`注解。下面是该注解的定义：

```
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};

    String[] params() default {};

    String[] headers() default {};

    String[] consumes() default {};

    String[] produces() default {};
}
```

其中:

1. `name`用来为当前的控制器指定一个名称；
2. `value`和`path`是等价的，都是用来指定url的匹配规则的；
3. `method`用来指定匹配的方法，比如POST, GET等等；
4. `consumes`用来指定请求的提交内容类型（Content-Type），例如application/json, text/html；
5. `params`：指定request中必须包含某些参数值是，才让该方法处理；
6. `headers`：指定request中必须包含某些指定的header值，才能让该方法处理请求；
7. `produces`：指定request中必须包含某些指定的header值，才能让该方法处理请求；

下面我们对其中的几个方法进行简单说明。

#### 2.5.1 value和path

这两个参数的效果是等价的，因为它们相互之间只是一个别名的关系。这两个参数用来指定该控制器要映射的url，这里我们列举一下常见的url映射配置方式：

1. `@RequestMapping(value={"/test1", "/user/create"})`：多个URL路径映射到同一个处理器；
2. `@RequestMapping(value="/users/{userId}")`：使用url占位符，如"/users/123456"或"/users/abcd"，通过`@PathVariable`可以提取URI模板模式中的变量；
3. `@RequestMapping(value="/users/**")`：可以匹配“/users/abc/abc”，但“/users/123”将会被2中的模式优先映射到；
4. `@RequestMapping(value="/product?")`：可匹配“/product1”或“/producta”，但不匹配“/product”或“/productaa”；
5. `@RequestMapping(value="/product*")`：可匹配“/productabc”或“/product”，但不匹配“/productabc/abc”；
6. `@RequestMapping(value="/product/*")`：可匹配“/product/abc”，但不匹配“/productabc”
7. `@RequestMapping(value="/products/**/{productId}")`：可匹配“/products/abc/abc/123”或“/products/123”；
8. 基于正则表达式的方式。

配置方式2方式的特别说明，指定路径中的参数并在方法中获取参数的具体示例：

    @RequestMapping("/testPathVariable/{id}")
    public String testPathVariable(@PathVariable("id") Integer id2) {
        System.out.println("testPathVariable: " + id2);
        return SUCCESS;
    }

#### 2.5.2 params

该参数用来限制只有当请求中包含指定参数名的数据时才会被处理，比如:

```
@Controller
@RequestMapping("/parameter1")                                      //①处理器的通用映射前缀
public class RequestParameterController1 {
    @RequestMapping(params="create", method=RequestMethod.GET) 
    public String showForm() {
	    ....
    }
    @RequestMapping(params="create", method=RequestMethod.POST)  
    public String submit() {
	    ....       
    }
}
```

其中的第一个方法表示请求中有“create”的参数名且请求方法为“GET”即可匹配，如可匹配的请求URL“http://×××/parameter1?create”；
第二个方法表示请求中有“create”的参数名且请求方法为“POST”即可匹配。

当然你还可以进一步限制当请求中包含指定的参数并且为指定的值时才能被处理，比如`@RequestMapping(params="submitFlag=create", method=RequestMethod.GET)`：表示请求中有“submitFlag=create”且请求方法为“GET”才可匹配。

还要注意，从`@RequestMapping`中的`params`定义中可以看出，它是一个数组，当指定多个值的时候，这些值之间属于'且'的关系，即两个参数同时包含才行。

#### 2.5.3 consumes

consumes用来指定该控制器要处理的请求的数据类型，所谓媒体类型就是指`text/plain` `application/json`等等。
它们会被放在请求的请求头中，比如`Content-Type: application/x-www-form-urlencoded`表示请求的数据为key/value数据，
只有当请求数据与控制器在`@RequestMapping`中指定的数据相同的时候，指定的请求才会被该控制器处理。

    @RequestMapping(value = "/testMethod", method = RequestMethod.POST,consumes="application/json")
    public String testMethod() {
        System.out.println("testMethod");
        return SUCCESS;
    }

比如以上控制器只接受json类型的数据。当请求的数据非json的时候是不会被其处理的。

#### 2.5.4 produces

produces用来指定当前的请求希望得到什么类型的数据，这个参数在请求的时候会被放到请求头的Accept中。
只有当请求的Accept类型与控制器中使用`produces`指定的类型相同的时候才会被该控制器接受并处理。

#### 2.5.5 headers

如果说前面的consumes和produces用来指定请求的和希望得到的数据类型是一种特例的话，
那么这里的headers则是可以用来更加灵活地指定headers中需要包含那些信息才能被当前的控制器处理。
比如：

    @RequestMapping(value = "testParamsAndHeaders", params = { "username","age!=10" }, headers = { "Accept-Language=US,zh;q=0.8" })
    public String testParamsAndHeaders() {
        System.out.println("testParamsAndHeaders");
        return SUCCESS;
    }

用来设定请求头中第一语言必须为US。

## 3、其他

### 3.1 拦截器和过滤器的区别

1. 拦截器是基于java的反射机制的，过滤器是基于函数回调；
2. 拦截器不依赖于servlet容器，过滤器依赖于servlet容器，因为过滤器是Servlet规范规定的，只用于Web程序中，而拦截器可以用在Appliaction和Swing等中；
3. 在Action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次；
4. 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑;
5. 过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前。

实际上从上面的配置中也可以看出来，过滤器在请求达到Servlet之前被调用的，它属于Servlet而不属于Spring。
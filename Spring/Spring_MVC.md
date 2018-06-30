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
4. `HandlerAdapter`将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个`ModelAndView`对象（包含模型数据、逻辑视图名）；
5. 再由`DispatcherServlet`根据`ModelAndView`找到对应的ViewResolver，并由它把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；
6. 接下来会对`View`进行渲染，`View`会根据传进来的`Model`模型数据进行渲染，Model实际是一个Map数据结构，因此很容易支持其他视图技术；
7. 返回控制权给`DispatcherServlet`，由`DispatcherServlet`返回响应给用户，到此一个流程结束。









### 1.基本原理

#### 1.Spring MVC的处理请求的流程：

![](http://sishuok.com/forum/upload/2012/7/14/529024df9d2b0d1e62d8054a86d866c9__1.JPG)

#### 2.Spring Web MVC架构

![](http://sishuok.com/forum/upload/2012/7/14/57ea9e7edeebd5ee2ec0cf27313c5fb6__2.JPG)

### 2.Hello实例

在web.xml中加入以下内容来注册DispatcherServlet：

    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.mvc</url-pattern>
    </servlet-mapping>

注意：

1. 这里`url-pattern`标签定义了url的匹配模式，即所有以`.mvc`结尾的请求会被MVC处理；

在springmvc-servlet中加入以下内容。

    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!-- HandlerAdapter -->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!-- ViewResolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <bean name="/hello.mvc" class="my.shouheng.mvc.HelloController"/>

注意：

1. 这里的servlet配置文件的命名规则是，在web.xml中定义的servlet-name的后面加上`-servlet.xml`。如果采用其他的Servlet名称，也必须在后面加入`-servlet.xml`。
2. `InternalResourceViewResolver`定义了jsp文件的前缀和后缀路径，查找jsp文件的时候会到指定的前缀路径中查找指定后缀的文件；
3. 这里定义了bean，它的name就是url路径的匹配模式，也就是`hello.mvc`结尾的请求会被HelloController处理。
4. BeanNameUrlHandlerMapping的映射规则是，将Bean的name与url映射，比如这里的url为`/hello.mvc`，那么就必须有一个对应的name为`/hello.mvc`的Bean。
5. SimpleControllerHandlerAdapter表示所有实现了`Controller`接口的Bean可以作为Spring Web MVC中的处理器。如果需要其他类型的处理器可以通过实现HadlerAdapter来解决。
6. HandlerMapping是用来将url映射到指定的Handler的（策略模式），而HandlerAdapter是用来对指定的Handler进行适配，以支持多种Handler（适配器模式）。这样DispatcherServlet就可以将指定的url调用指定的Handler进行处理。

定义如下控制器：

	public class HelloController implements Controller {
	
	    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, javax.servlet.http.HttpServletResponse httpServletResponse) throws Exception {
	        ModelAndView mv = new ModelAndView();
	        //添加模型数据 可以是任意的POJO对象
	        mv.addObject("message", "Hello World!");
	        //设置逻辑视图名，视图解析器会根据该名字解析到具体的视图页面
	        mv.setViewName("hello");
	        return mv;
	    }
	}

注意：

1. 这里通过setViewName方法指定jsp的名称，这里会到/WEB-INF/jsp/中查找hello.jsp

**回顾一下以上程序的整个执行过程：**

![](http://sishuok.com/forum/upload/2012/7/14/8b42eeaa9b2423b154944c651ed23667__3.JPG)

首先用户请求会传递到DispatcherServlet（前段控制器），它接收所有以*.mvc结尾的请求。然后，它根据BeanNameUriHandlerMapping将指定的url，即/hello.mvc，映射到HelloWorldController。然后，DispatcherServlet再将HelloWorldController传递给SimpleControllerHandlerAdapter，它作为一个适配器，在这里真正调用HelloWorldController的方法执行业务逻辑。执行完毕业务逻辑之后，将结果ModelAndView传递到DispatcherServler。这时DispatcherServler再将ModelAndView传递给视图解析InternalResourceViewResolver，后者根据setViewName时赋值的名称，得到JstlView对象。然后，DispatcherServler对视图进行渲染，得到渲染结果之后返回给用户。

可见，其实这里的HandlerMapping和HandlerAdapter和ViewResolver是可以通过配置来指定不同的实现策略的。这里就像是模板、策略和适配器设计模式的组合，将一些可以由用户定制的处理逻辑交给我们来实现，而整个处理流程由SpringMVC进行控制。

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






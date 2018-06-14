# Spring AOP

## 1、基本

### 1.1 概念

1. **连接点(Jointpoint)**：表示需要在程序中插入横切关注点的扩展点，连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等；
2. **切入点(Pointcut)**：选择一组相关连接点的模式，即可以认为连接点的集合；
3. **通知(Advice)**：在连接点上执行的行为,包括前置通知、后置通知、环绕通知，在Spring中通过代理模式实现AOP;
4. **方面/切面(Aspect)**：可以认为是通知、引入和切入点的组合；
5. **引入(inter-type declaration)**：也称为内部类型声明，为已有的类添加额外新的字段或方法；
6. **目标对象(Target Object)**：需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被通知的对象；
7. **AOP代理(AOP Proxy)**：AOP框架使用代理模式创建的对象，从而实现在连接点处插入通知（即应用切面），就是通过代理来对目标对象应用切面。
在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。
8. **织入(Weaving)**：织入是一个过程，是将切面应用到目标对象从而创建出AOP代理对象的过程，织入可以在编译期、类装载期、运行期进行。

![AOP的概念关系图](res/aop_conceptions.JPG)

### 1.2 通知类型

1. **前置通知(Before Advice)**:在切入点选择的连接点处的方法之前执行的通知。
2. **后置通知(After Advice)**:在切入点选择的连接点处的方法之后执行的通知，包括如下类型的后置通知：
3. **后置返回通知(After returning Advice)**:在切入点选择的连接点处的方法正常执行完毕时执行的通知，必须是连接点处的方法没抛出任何异常正常返回时才调用后置通知。
4. **后置异常通知(After throwing Advice)**: 在切入点选择的连接点处的方法抛出异常返回时执行的通知，必须是连接点处的方法抛出任何异常返回时才调用异常通知。
5. **后置最终通知(After finally Advice)**: 在切入点选择的连接点处的方法返回时执行的通知，不管抛没抛出异常都执行，类似于Java中的finally块。
6. **环绕通知(Around Advices)**：环绕通知可以在方法调用之前和之后自定义任何行为，并且可以决定是否执行连接点处的方法、替换返回值、抛出异常等等。

### 1.3 其他

Spring使用JDK动态代理或CGLIB代理来实现

## 2、简单的例子

我们用一个简单的例子来测试一下Spring的AOP功能，这里我们定义了两个类，一个是Worker，一个是LogLog。LogLog有一个名为`say`的方法，该方法中输出一行文字到控制台。
我们希望Worker的每个方法在执行的时候都能够调用LogLog的`say`方法。我们可以做如下的配置：

首先，我们要在基本的命名空间基础上增加一行

```
xmlns:aop="http://www.springframework.org/schema/aop"
```

来启用`aop`命名空间。此外，我们还要增加`schemaLocation`中的配置：

```
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
```
然后，我们就可以使用AOP来完成我们想要的功能了：

    <bean id="logLog" class="me.shouheng.spring.aop.LogLog"/>
    <bean id="worker" class="me.shouheng.spring.aop.Worker"/>
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* me.shouheng.spring.aop.Worker.*(..)))"/>
        <aop:aspect ref="logLog">
            <aop:before method="say" pointcut-ref="pointcut"/>
            <aop:after method="say" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>

以上的配置没有问题，我们可以实现期望的功能。这里，我们可以看出，即使是AOP也是需要目标对象和切点定义成Bean。
`pointcut`是所有方法的一个合集，我们使用`expression`来指定这些方法的规则。
`aspect`整合了切入点和通知，它需要引用Bean，并且在子标签中将通知类型、要执行的方法和切入点结合起来。

另外，还需要注意的一点是，切入点和切面都可以被定义多个，但是是有顺序要求的。









### 使用XML

除了Spring包之外还要加入

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.2</version>
        </dependency>

切面类，实现了MethodBeforeAdvice和AfterReturningAdvice两个接口：

	public class LogAOP implements MethodBeforeAdvice, AfterReturningAdvice, ThrowsAdvice{
	
	    public void before(Method method, Object[] objects, Object o) throws Throwable {
	        System.out.println("before");
	    }
	
	    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
	        System.out.println("afterReturning");
	    }
	
        public void afterThrowing(Object[] args, ServletException e, Method method, Object target){
            System.out.println("afterThrowing" );
        }
	
	    public void after(JoinPoint joinPoint) {
	        System.out.println("after");
	    }
	
	    public void around(ProceedingJoinPoint pjp) throws Throwable{
	        long current = System.currentTimeMillis();
	        pjp.proceed();
	        System.out.println("around cost:" + (System.currentTimeMillis() - current));
	    }
	}

定义的类:

	public class MyBean {
	
	    public void say() {
	        System.out.println("say");
	    }
	
	    public void sayHello() {
	        System.out.println("Hllo");
	    }
	}

XML配置文件：

    <bean name="myBean" class="my.shouheng.aop.MyBean"/>

    <bean id="log" class="my.shouheng.aop.LogAOP"/>

    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(public * say*(..))"/>
        <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
        <aop:aspect ref="log">
            <aop:after method="after" pointcut-ref="pointcut"/>
            <aop:around method="around" pointcut-ref="pointcut"/>
            <!--<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut"/>-->
        </aop:aspect>
    </aop:config>

1. 这里pointcut标签中定义了切点的id和规则expression，即public的任何返回类型的以say开头的任何输入参数的方法会被当作切点；
2. 这里的advisor标签指定了切点要执行的逻辑，这里是LogAOP类，以及针对的切点。
3. 所以，简单来理解就是“pointcut定义了插入的规则，advice定义了切入处要执行的逻辑，advisor将pointcut和advice联系起来”

输出结果：

	before method:say
	say
	afterReturning method:say
	afterReturning cost:73
	before method:sayHello
	Hllo
	afterReturning method:sayHello
	afterReturning cost:0

### 切入点指示符

#### 1.类签名表达式`Whithin(<type name>)`

1. within(my.palm..*)：匹配my.palm及其子包中所有类的所有方法
2. within(my.palm.ClassName):匹配my.palm.ClassName类中所有方法
3. within(MyInterface+)：匹配MyInterface接口所有实现类的所有方法
4. within(my.palm.BaseClass)：匹配my.palm.BaseClass及其子类的所有方法

#### 2.方法签名`execution(<scope> <return type> <full-qulified-class-name>.<method>(params))`

1. execution(* my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有方法
2. execution(public * my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有公共方法
3. execution(public * my.shouhengn.Palm.*(long,..)):my.shouhengn.Palm中所有第一个参数为long型的公共方法
4. execution(public String my.shouhengn.Palm.*(..)):my.shouhengn.Palm中所有返回类型为String的公共方法

#### 3.其他

1. bean(*Service):所有后缀名为Service的Bean
2. @anotation(my.shouheng.Annotation):所有使用Annotaion注解的方法会被调用
3. @within(my.shouheng.Annotation):所有使用Annotaion注解的类的方法会被调用
4. this(my.shouheng.MyInterface):所有实现了my.shouheng.MyInterface接口的代理对象的方法会被调用

#### 4.通配符

1. `..`:任意参数列表，任何数量的包
2. `+`:给定类的任何子类
3. `*`:任何数量的字符

### 使用注解的方式

声明切面：

	@Aspect
	public class LogAOP {
	
	    @Before(value = "execution(public * say*(..))")
	    public void before() throws Throwable {
	        System.out.println("before");
	    }
	
	    @AfterReturning(value = "execution(public * say*(..))")
	    public void afterReturning() throws Throwable {
	        System.out.println("afterReturning");
	    }
	
	    @AfterThrowing("execution(public * say*(..))")
	    public void afterThrowing(){
	        System.out.println("afterThrowing" );
	    }
	
	    @After("execution(public * say*(..))")
	    public void after(JoinPoint joinPoint) {
	        System.out.println("after");
	    }
	
	    @Around("execution(public * say*(..))")
	    public void around(ProceedingJoinPoint pjp) throws Throwable{
	        long current = System.currentTimeMillis();
	        pjp.proceed();
	        System.out.println("around cost:" + (System.currentTimeMillis() - current));
	    }
	}

除了在每个方法上面定义切面的切入点规则，也可以显得定义一个方法在上面使用@PointCur注解指定切入规则。然后，在各个注解中调用该切入点即可。

使用切面的类：

	public class MyBean {
	
	    public void say() {
	        System.out.println("say");
	    }
	
	    public void sayHello() {
	        System.out.println("Hllo");
	    }
	
	    public void sayWithException() {
	        System.out.println("sayWithException");
	        throw new IllegalArgumentException("Exception");
	    }
	}

配置文件：

    <bean name="myBean" class="my.shouheng.aop.MyBean"/>
    <bean name="log" class="my.shouheng.aop.LogAOP"/>
    <aop:aspectj-autoproxy/>

测试类：

    public static void main(String ...args) {
        ClassPathXmlApplicationContext ctx =
                new ClassPathXmlApplicationContext("/XmlBeanContext.xml");
        MyBean myBean = ctx.getBean("myBean", MyBean.class);
        myBean.say();
        myBean.sayWithException();
    }

输出结果：

	before
	Exception in thread "main" java.lang.IllegalArgumentException: Exception
		......
	say
	around cost:32
	after
	afterReturning
	before
	sayWithException
	after
	afterThrowing

#### 当指定的方法中具有参数的时候

比如如下方法，这里具有一个int类型的参数count，那么我们怎么在切面的方法中获取到从count方法中传入的参数呢？

    public void count(int count) {  }
	
	@Aspect
	public class LogAOP {
	
	    @Before(value = "execution(public * count(int)) && args(count)")
	    public void before(int count) throws Throwable {
	        System.out.println("before " + count);
	    }
   }

这里是在切面中获取切点的一个示例——我们需要在切点的定义中指定args参数，然后在其中指定参数的名称。这里，当我们调用count方法的时候，就会调用方法之前先调用这里的方法输出count的值。





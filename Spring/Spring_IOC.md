# Spring IOC

## 1、Spring Ioc的基本原理

Spring Ioc容器的代表就是`BeanFactory`接口，它其中定义了一些获取Bean的方法`getBean()`。
`ApplicationContext`接口扩展了`BeanFactory`，还提供了与Spring AOP集成、国际化处理、事件传播及提供不同层次的context实现。
我们可以从`BeanFactory`和`ApplicationContext`任选一个来使用，但是`BeanFactory`过于低级，所以`ApplicationContext`用得更加广泛。

`ApplicationContext`有多个具体的实现，可以供我们从不同类型的配置文件中获取Bean。以下是比较常用的一些：

1. **AnnotationConfigApplicationContext**：从一个或多个基于Java的配置类中加载应用上下文
2. **AnnotationConfigWebContextLoader**：从一个或多个基于Java配置类中加载Web应用上下文
3. **ClassPathXmlApplicationContext**：从类路径下面一个或多个配置文件中加载上下文
4. **FileSystemXmlApplicationContext**：从文件系统中的一个或多个配置文件中加载上下文

当获取到了`ApplicationContext`之后，我们就可以使用它来获取Bean实例。`ApplicationContext`提供了一些方法来供我们获取Bean：

    Object getBean(String var1) throws BeansException;
    <T> T getBean(Class<T> var1) throws BeansException;
    <T> Map<String, T> getBeansOfType(Class<T> var1) throws BeansException;

这里我们给的是三种基本的方法，它们还有一些多态的方法。与其区别仅在于，增加了一些可选的参数。

## 2、Ioc的配置

### 2.1 Xml配置的结构

当我们使用XML来配置bean的时候，其基本的格式如下。这里我们省略了命名空间的声明。下面是一个基本的结构，在`beans`标签中主要用三种类型标签供我们使用：

    <beans>
        <import resource="../hello/HelloWorld.xml"/>
        <bean id="universal" class="me.shouheng.spring.universal.UniversalBean"/>
        <alias name="universal" alias="my_universal"/>
    </beans>

其中的`import`标签用于导入其他的XML配置文件，这样就可以在这一个文件中使用其他文件中的Bean。然后是bean标签，它用来声明一个bean的定义。
最后是`alias`标签，它用来为一个Bean起一个别名。
	
Bean在容器中由`BeanDefinition`对象表示，`BeanDefinition`是一个接口，内部定义了一个Bean对象的基本的属性信息，比如是否单例的、作用范围、是否延迟初始化等等。

我们之前由提到过从上下文中获取Bean的几个方法，当传入的是字符串的时候就会根据`id`或者`name`或者`alias`指定的名称进行加载。也可以通过传入类型来加载指定类型的Bean。

一个Bean只能有一个`id`和一个别名，如果同时指定了`name`和`id`，那么`name`就成了别名。`id`在Ioc容器中必须是唯一的。另外，可以指定多个name，并用`,`、`;`、` `分割。

Bean的命名遵循XML命名规范，但最好符合Java命名规范，也就是驼峰风格的命名。

### 2.2 实例化Bean

Ioc容器中的Bean使用反射来创建。可以使用无参构造函数，也可以使用含参构造函数。

#### 2.2.1 通过构造方法直接获取实例

我们在UniversalBean类中增加一个构造方法，该方法中包含一个名为`hiTo`的参数，那么我们可以使用下面的方式来通过传入该参数来获取一个Bean的实例：

    <bean id="universal2" class="me.shouheng.spring.universal.UniversalBean">
        <constructor-arg name="hiTo" value="Spring"/>
    </bean>

这里当构造函数只存在一个参数的时候，我们可以这么去写：使用`name`指定参数的名称，使用`value`指定参数的值。当构造函数存在多个参数的时候，我们就需要声明多个`constructor-arg`标签。

#### 2.2.2 通过静态工厂方法获取实例

我们在UniversalBean类增加一个工厂方法，它接受两个参数`hiTo1`和`hiTo2`，然后我们可以用下面的方式在XML中配置：

    <bean id="universal4" class="me.shouheng.spring.universal.UniversalBean" factory-method="staticFactory">
        <constructor-arg name="hiTo1" value="Spring"/>
        <constructor-arg name="hiTo2" value="Winter"/>
    </bean>

也就是相对于普通的定义方式，在bean标签中又增加了一个`factory-method`属性，并指定该属性的值是静态工厂方法的名字。然后，我们在该标签内通过两个`constructor-arg`标签指定参数的值即可。

对应于`constructor-arg`的有简化版的`c`命名空间可以使用，使用该命名空间使代码看上去更加简洁。

#### 2.2.3 通过实例工厂方法获取实例

在这种方式中，我们需要先创建一个工厂Bean，然后创建Bean的时候指定工厂Bean及其工厂方法。

    <bean id="factory" class="me.shouheng.spring.universal.FactoryBean"/>
    <bean id="universal5" factory-bean="factory" factory-method="factory">
        <constructor-arg name="hiTo1" value="Spring"/>
        <constructor-arg name="hiTo2" value="Winter"/>
    </bean>

这里，我们在FactoryBean中定义了一个工厂方法`factory`，它接受两个参数`hiTo1`和`hiTo2`。在上面的代码中，我们先创建了该工厂Bean，名为`factory`。
然后，在创建`universal5`的时候，分别使用`factory-bean`指定对应的工厂Bean和`factory-method`指定该Bean中的工厂方法。

## 3、依赖注入

除了使用构造方法向创建的实例中设置值，还可以通过`setter`方法（Spring要求方法的内容确实符合setter方法的要求）对实例的属性进行设置。
而且它的配置方式与使用构造方法大同小异，区别在于：

1. 首先它要求指定的属性确实提供了Setter方法；
2. 我们可以在bean标签中使用`property`子标签来设置属性的值，它的配置方式与`constructor-arg`非常相似。

### 3.1 注入值和引用

这里我们定义了两个类DiBean和RefBean，其中的DiBean有一个类型为String的`message`字段，我们这里使用`property`为其注入值`So what?`。
而RefBean中有一个类型为DiBean的字段`diBean`，这里我们定义一个Bean并通过`property`的`ref`属性指定它引用的Bean：

    <bean id="di" class="me.shouheng.spring.di.DiBean">
    <property name="message" value="So what?"/>
    </bean>

    <bean id="refBean" class="me.shouheng.spring.di.RefBean">
    <property name="diBean" ref="di"/>
    </bean>

### 3.2 注入列表、字典和其他类型的值
	
除了注入一些字符串常量还有引用Bean，`property`标签还为我们提供了更加丰富的功能，我们可以使用它们来完成更加多样化的注入。

为了测试Spring注入其他类型实例的能力，我们定义了一个名为DiMulti的类。它有一个名为`names`的`List<String>`类型的字段，然后我们可以像下面这样为其注入值：

    <bean id="diMulti" class="me.shouheng.spring.di.DiMulti">
      <property name="names">
        <list>
        <value>Chen</value>
        <value>Li</value>
        <value>Xu</value>
        </list>
      </property>
    </bean>

如果你是在IDEA中使用的话，那么你基本可以懂得其他的类型该如何配置了：在`property`标签中有许多其他的子标签，
我们可以通过`property`标签的`name`属性指定要注入的字段的名称，然后再使用`property`的子标签来组装我们需要注入的各种类型的值。

### 3.3 p命名空间

你可以使用p命名空间来简化setter注入，比如上面的`refBean`可以使用如下的方式进行简化：

    <bean id="refBean" class="me.shouheng.spring.di.RefBean" p:diBean-ref="di"/>

注意这里需要引用p命名空间：

    xmlns:p="http://www.springframework.org/schema/p"

我们看它的规则其实就是`p:字段名="值"`，如是字段是引用类型，那么就是`p:字段名-ref=“Bean的id”`

## 4、DI的其他知识

### 4.1 循环依赖

所谓的循环依赖就是：A创建的时候需要使用B，B需要用到C，而C又要用到A。按照注入的方式不同又分成：**构造器循环依赖**和**setter方法循环依赖**。

构造器循环依赖表示Bean在创建的时候依赖于其他的Bean而造成的循环依赖，此种循环依赖无解，只能通过抛出BeanCurrentlyInCreationException异常表示循环依赖。

对于setter方法循环依赖又分成两种情形：**单例作用域的循环依赖**和**prototype作用域的循环依赖**。前者是可以解决的，而后者无法解决。
因为单例作用域的Bean在创建之后会被放在Bean创建池中待用，而prototype作用域的Bean不会被Spring容器缓存，无法提前暴露创建完毕的Bean。

### 4.2 Bean的作用域

#### 4.2.1 Bean的基本作用域

Bean的基本作用域有下面两种：

1. **singleton**：单例作用域，表示每个Spring Ioc容器中只会存在一个实例，而且其完整生命周期完全由Spring容器管理。
Spring容器中如果没有指定作用域默认就是单例的，配置的方式也很简单就是通过在bean标签中使用`scope`属性指定值为`singleton`。
如果Bean是延迟初始化的，那么该Bean会在首次使用的时候创建并放在单例缓存池中。
2. **prototype**：原型，指每次向Spring容器请求获取Bean都返回一个全新的Bean，相对于“singleton”来说就是不缓存Bean，每次都是一个根据Bean定义创建的全新Bean。

#### 4.2.2 Bean在Web应用中的作用域

1. **request作用域**：表示每个请求需要容器创建一个全新Bean。比如提交表单的数据必须是对每次请求新建一个Bean来保持这些表单数据，请求结束释放这些数据。
2. **session作用域**：表示每个会话需要容器创建一个全新Bean。比如对于每个用户一般会有一个会话，该用户的用户信息需要存储到会话中，此时可以将该Bean配置为web作用域。
3. **globalSession**：类似于session作用域，只是其用于portlet环境的web应用。如果在非portlet环境将视为session作用域。

### 4.3 延迟初始化

延迟初始化的Bean只有在用到的时候才会被初始化，它的配置方式很简单只需在<bean>标签上指定`lazy-init`属性值为`true`即可延迟初始化Bean。

延迟初始化适用于可能需要加载很大资源，而且很可能在整个应用程序生命周期中很可能使用不到的Bean。

在`<beans>`标签中通过制定default-lazy-init="true"可以将内部全部bean延迟初始化。

### 4.4 depends-on

有两个Bean，分别是A和B，我们在A的配置中设置它的depends-on属性为B，那么A会在B初始化完成之后再被创建，而B要等到A销毁了之后才能被销毁。

### 4.5 init-method和destroy-method

在使用XML配置Bean的时候，可以在Bean标签中通过这init-method和destroy-method分别指定初始化时和销毁时调用的方法：

1. init-method ：指定初始化方法，在构造器注入和setter注入完毕后执行；
2. destroy-method：指定销毁方法，只有`singleton`作用域能销毁，`prototype`作用域的一定不能，其他作用域不一定能；
3. 可以使用@PostConstruct和@PreDestroy注解标记指定的方法来实现生命周期回调，要使用这种方式还要在XML中加入`<context:annnotation-config>`标签，或者使用`<context:component-scan/>`；
4. 实现生命周期的第三种选择是实现InitializingBean和DisposableBean接口。

关于`<context:annnotation-config>`和`<context:component-scan/>`标签的区别可以查看下文。

### 4.6 Bean重写

当使用多个配置文件定义Bean的时候，如果不同的配置文件中存在相同的名称的Bean就会发生重写，规则是最后加载的Bean会覆盖先加载的Bean.

## 5、自动装配

### 5.1 基于XML配置的自动装配

自动装配就是指由Spring来自动地注入依赖对象，无需人工参与。

Spring支持`no`、`byName`、`byType`、`constructor`和`default`5种自动装配，默认是`no`指不支持自动装配。
`byName`表示按照名称装配，`byType`表示按照类型自动装配，`constructor`也是按照类型来装配的，只是用于构造器注入方式。

我们定义如下的FirstBean，它其中有一个SecondBean类型的实例字段，并且有一个Setter方法：

```
public class FirstBean {
    
    private SecondBean secondBean;
    
    public void setSecondBean(SecondBean secondBean) {
        this.secondBean = secondBean;
    }
}
```

我们按照如下的方式进行自动装配：

    <bean name="firstBean" class="me.shouheng.spring.autowire.FirstBean" autowire="byName"/>
    <bean name="secondBean" class="me.shouheng.spring.autowire.SecondBean" />
	
测试证明这样的装配是可以成功注入进去的。我们来总结一下自动装配相关的内容：

1. 不是所有类型都能自动装配，Object、基本数据类型等无法自动装配；
2. 数组、集合、字典类型的根据类型自动装配和普通类型的自动装配是有区别的，它们需要根据泛型信息注入；
3. 自动装配的缺点就是没有了配置，在查找注入错误时非常麻烦，还有比如基本类型没法完成自动装配，所以可能经常发生一些莫名其妙的错误；
4. 自动装配注入和配置注入同时存在时，配置注入的数据会覆盖自动装配注入的数据；
5. 自动装配要求被装配的Bean中的对象实例有Setter方法。

### 5.2 自动扫描

以上我们定义Bean的时候，对于我们需要用到的每个Bean，我们都要分别使用bean标签进行定义。实际上，我们有更简单的方式来定义Bean。

如下所示，我们可以使用context命名空间中的`component-scan`来指定要扫描的包，该包下面的使用`@Component`注解生命的类会被当做Bean：

```
<context:component-scan base-package="me.shouheng.spring.componentscan"/>
```

注意，使用以上配置我们需要引用context命名空间，你可以输出前半部分之后由IDEA自动帮你补全该命名空间的定义。

当然，同样的功能你还可以通过使用基于类的配置方式实现：

```
@Configurable
@ComponentScan(value = "me.shouheng.spring.componentscan")
public class ComponentScanConfiguration { }
```

这里我们用`@Configurable`将其声明为一个配置类，然后使用`@ComponentScan`注解指定要扫描的包路径。然后，我们通过如下的方式来获取一个上下文：

```
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ComponentScanConfiguration.class);
```

对基于以上配置方式做简单总结：

1. 和`@Component`具有相似功能的还有`@Service`、`@Repository`和`@Controller`，只是针对不同的类的类型，后面三个继承自前者；
2. 自动装配可以通过为字段添加`@Autowired`注解来实现，也可以不指定注解，但是要给字段提供Setter方法和构造方法；
3. 可以通过`@Component`注解的`value`字段为Bean指定一个名称，默认将首字母小写之后作为Bean的名字；
4. 可以通过`@ComponentScan`注解的`basePackages`来指定多个扫描的包；
5. `@Autowired`注解有一个`required`的布尔类型字段，默认为true，表示如果找不到可装配的Bean就抛异常；
将其设置为false时，如果找不到也不会抛异常，但是如果最终没有注入值，又恰好用到了它，可能会在运行时抛出NPE异常。

### 5.3 基于Java的自动装配机制

我们在上面使用了`@Configurable`注解和AnnotationConfigApplicationContext来配置和获取上下文，并使用`@ComponentScan`注解指定要扫描的包路径。

使用以上注解和上下文，我们也可以直接在Java代码中实现装配和注入。

```
@Configuration
public class JavaConfig {

    @Bean("first")
    public FirstBean firstBean() {
        return new FirstBean(secondBean());
    }

    @Bean("second")
    public SecondBean secondBean() {
        return new SecondBean();
    }
}
```

以上时基于Java自动装配的机制，显然如果我们需要定义某个Bean，就要在这里单个地进行声明。你可以把`@ComponentScan`注解当作这种配置方式的一个便捷功能。

## 6、混合使用多种配置方式

以上提供了基于Java和基于XML的两种配置方式，实际上我们可以混合使用多种不同类型的配置方式。这里我们简单列举一下好了：

1. 在XML中引用基于XML配置：`<import resource="AspectConfig.xml "/>`
2. 在XML中应用基于Java的配置：`<bean class="me.shouheng.spring.javaconfig.JavaConfig"/>`，没错就是将配置类当作Bean引入；
3. 在基于Java的配置中引用基于Java的配置：`@Import(value = {JavaConfig.class})`
4. 在基于Java的配置中引用基于XML的配置：`@ImportResource(value = {"classpath:aop/AOPConfig.xml"})`

## 问题整理：

### 1、`<context:annnotation-config>`和`<context:component-scan/>`标签的区别

如果用`<context:annotation-config/>`，我们还需要配置Xml注册Bean，而使用`<context:component-scan />`的话，注册的步骤都免了。
当然前提是我们对需要扫描的类使用的注解（比如@Componet,@Service），而如果同时使用两个配置的话，`<context:annotation-config/>`会被忽略掉。

就是说`<context:component-scan/>`的功能是包含`<context:annnotation-config>`的，前者不仅可以使用注解配置，还可以扫描自动包下面的Bean。

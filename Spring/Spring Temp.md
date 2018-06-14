
除了使用构造方法向创建的实例中设置值，还可以通过`setter`方法（Spring要求方法的内容确实符合setter方法的要求）对实例的属性进行设置。
而且它的配置方式与使用构造方法大同小异，区别在于：
1).首先它要求指定的属性确实提供了Setter方法；2).然后，我们可以在bean标签中使用`property`子标签来设置属性的值，它的配置方式与`constructor-arg`非常相似。

这里我们定义了两个类DiBean和RefBean，其中的DiBean有一个`message`字段，我们这里使用`property`为其注入值`So what?`。
而RefBean中有一个类型为DiBean的字段`diBean`，这里我们定义一个Bean并通过`property`的`ref`属性指定它引用的Bean：

    <bean id="di" class="me.shouheng.spring.di.DiBean">
    <property name="message" value="So what?"/>
    </bean>

    <bean id="refBean" class="me.shouheng.spring.di.RefBean">
    <property name="diBean" ref="di"/>
    </bean>

除了注入一些字符串常量还有引用Bean，`property`标签还为我们提供了其他许多更加花哨的功能，我们可以使用它们来完成更加多样化的注入。

为了测试它注入其他类型实例的能力，我们定义了一个名为DiMulti的类。它有一个名为`names`的`List<String>`类型的参数，然后我们可以像下面这样向其注入值：

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

你可以使用p命名空间来简化setter注入，比如上面的`refBean`可以使用如下的方式进行简化：

    <bean id="refBean" class="me.shouheng.spring.di.RefBean" p:diBean-ref="di"/>

注意这里需要引用p命名空间：

    xmlns:p="http://www.springframework.org/schema/p"

我们看它的规则其实就是`p:字段名="值"`，如是字段是引用类型，那么就是`p:字段名-ref=“Bean的id”`


循环依赖：

比如A在需要使用B，B需要用到C，而C又要用到A。按照注入的方式不同又分成：构造器循环依赖和setter方法循环依赖。

构造器循环依赖表示Bean在创建的时候依赖于其他的Bean而造成的循环依赖，此种循环依赖无解，只能通过抛出BeanCurrentlyInCreationException异常表示循环依赖。

对于setter方法循环依赖又分成两种情形：单例作用域的循环依赖和prototype作用域的循环依赖，前者是可以解决的，而后者无法解决。
因为单例作用域的Bean在创建之后会被放在Bean创建池中待用，而prototype作用域的Bean不会被Spring容器缓存，无法提前暴露创建完毕的Bean。

Bean的作用域：

1. singleton：单例作用域，表示每个Spring Ioc容器中只会存在一个实例，而且其完整生命周期完全由Spring容器管理。
Spring容器中如果没有指定作用域默认就是单例的，配置的方式也很简单就是通过在bean标签中使用`scope`属性指定值为`singleton`。
如果Bean是延迟初始化的，那么该Bean会在首次使用的时候创建并放在单例缓存池中。
2. prototype：原型，指每次向Spring容器请求获取Bean都返回一个全新的Bean，相对于“singleton”来说就是不缓存Bean，每次都是一个根据Bean定义创建的全新Bean。









`property`标签中可选的子标签：

1. value
2. list
3. bean
4. ref
5. array
6. desciption
7. idref
7. map
8. null
9. meta
10. props
11. set

我们再向DiMulti中增加一个新的字段`Map<String, Integer> grades`，然后我们可以在XML中按照下面的方式来为其赋值：

    <property name="grades">
        <map key-type="java.lang.String" value-type="java.lang.Integer">
        <entry key="Wang" value="100"/>
        <entry key="Li" value="99"/>
        <entry key="Sun" value="98"/>
        </map>
    </property>

以上就是Map类型的注入方式，因为Map的键值对可以是任何的自定义类型，所以对应于属性`key`和`value`的还有`key-ref`和`value-ref`。

其他类型的注入方式，我们就不再继续深究了。从上面几个基本的配置方式，我们大致可以得出Spring注入时配置的基本的逻辑。

	
	
beans标签中可选的子标签：

1. `bean`：
2. `alias`：
3. `desciption`：
4. `import`：

bean标签中可以选择的属性：

1. `id`：
2. `abstract`：
3. `autowire`：
4. `autowire-candidate`：
5. `class`：
6. `depends-on`：
7. `destroy-method`：
8. `factory-bean`：
9. `factory-method`：
10. `init-method`：
11. `lazy-init`：
12. `name`：
13. `parent`：
14. `primary`：
15. `scope`：

bean标签内可选的子标签：

1. `constructor-arg`
2. `description`
3. `lookup-method`
4. `meta`
5. `property`
6. `qualifier`
7. `replaced-method`




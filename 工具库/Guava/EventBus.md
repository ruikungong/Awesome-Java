# Guava源码分析——EventBus

EventBus的设计理念是基于观察者模式的，可以参考[设计模式(1)—观察者模式](https://juejin.im/post/5b60659df265da0f793a85ba)先来了解该设计模式。

## 1、程序示例

EventBus的使用是非常简单的，首先你要添加`Guava`的依赖到自己的项目中。这里我们通过一个最基本的例子来说明`EveentBus`是如何使用的。

    public static void main(String...args) {
        // 定义一个EventBus对象，这里的Joker是该对象的id
        EventBus eventBus = new EventBus("Joker");
        // 向上述EventBus对象中注册一个监听对象	
        eventBus.register(new EventListener());
        // 使用EventBus发布一个事件，该事件会给通知到所有注册的监听者
        eventBus.post(new Event("Hello every listener, joke begins..."));
    }

    // 事件，监听者监听的事件的包装对象
    public static class Event {
        public String message;
        Event(String message) {
            this.message = message;
        }
    }

    // 监听者
    public static class EventListener {
        // 监听的方法，必须使用注解声明，且只能有一个参数，实际触发一个事件的时候会根据参数类型触发方法
        @Subscribe
        public void listen(Event event) {
            System.out.println("Event listener 1 event.message = " + event.message);
        }
    }

首先，这里我们封装了一个事件对象`Event`，一个监听者对象`EventListener`。然后，我们用`EventBus`的构造方法创建了一个`EventBus`实例，并将上述监听者实例注册进去。然后，我们使用上述`EventBus`实例发布一个事件`Event`。然后，以上注册的监听者中的使用`@Subscribe`注解声明并且只有一个`Event`类型的参数的方法将会在触发事件的时候被触发。

总结：从上面的使用中，我们可以看出，EventBus与观察者模式不同的地方在于：当注册了一个监听者的时候，只有当某个方法使用了`@Subscribe`注解声明并且参数与发布的事件类型匹配，那么这个方法才会被触发。这就是说，同一个监听者可以监听多种类型的事件，也可以在多次监听同一个事件。

## 2、EventBus源码分析

### 2.1 分析之前

好了，通过上面的例子，我们了解了EventBus最基本的使用方法。下面我们来分析一下在`Guava`中是如何为我们实现这个API的。不过，首先，我们还是先试着考虑一下自己设计这个API的时候如何设计，并且提出几个问题，然后带着问题到源码中寻找答案。

假如要我们去设计这样一个API，最简单的方式就是在观察者模式上进行拓展：每次调用`EventBus.post()`方法的时候，会对所有的观察者对象进行遍历，然后获取它们全部的方法，判断该方法是否使用了`@Subscribe`并且方法的参数类型是否与`post()`方法发布的事件类型一致，如果一致的话，那么我们就使用反射来触发这个方法。在观察者模式中，每个观察者都要实现一个接口，发布事件的时候，我们只要调用接口的方法就行，但是EventBus把这个限制设定得更加宽泛，也就是监听者无需实现任何接口，只要方法使用了注解并且参数匹配即可。

从上面的分析中可以看出，这里面不仅要对所有的监听者进行遍历，还要对它们的方法进行遍历，找到了匹配的方法之后又要使用反射来触发这个方法。首先，当注册的监听者数量比较多的时候，链式调用的效率就不高；然后我们又要使用反射来触发匹配的方法，这样效率肯定又低了一些。那么在`Guava`的`EventBus`中是如何解决这两个问题的？

另外还要注意下下文中的`观察者`和`监听者`的不同，监听者用来指我们使用`EventBus.register()`注册的对象，观察者是EventBus中的对象`Subscriber`，后者封装了一个监听者的所有的信息，比如监听的方法等等。
一般我们是不会直接操作`Subscriber`对象的，它的访问权限也只在EventBus的包中可访问。

### 2.2 着手分析

首先，当我们使用`new`初始化一个EventBus的时候，实际都会调用到下面的这个方法：

    EventBus(String identifier, Executor executor, Dispatcher dispatcher, SubscriberExceptionHandler exceptionHandler) {
        this.subscribers = new SubscriberRegistry(this);
        this.identifier = (String)Preconditions.checkNotNull(identifier);
        this.executor = (Executor)Preconditions.checkNotNull(executor);
        this.dispatcher = (Dispatcher)Preconditions.checkNotNull(dispatcher);
        this.exceptionHandler = (SubscriberExceptionHandler)Preconditions.checkNotNull(exceptionHandler);
    }

这里的`identifier`是一个字符串类型，类似于EventBus的id；
`subscribers`是SubscriberRegistry类型的，实际上EventBus在添加、移除和遍历观察者的时候都会使用该实例的方法，所有的观察者信息也都维护在该实例中；
`executor`是事件分发过程中使用到的线程池，可以自己实现；
`dispatcher`是Dispatcher类型的子类，用来在发布事件的时候分发消息给监听者，它有几个默认的实现，分别针对不同的分发方式；
`exceptionHandler`是SubscriberExceptionHandler类型的，它用来处理异常信息，在默认的EventBus实现中，会在出现异常的时候打印出log，当然我们也可以定义自己的异常处理策咯。

所以，从上面的分析中可以看出，如果我们想要了解EventBus是如何注册和取消注册以及如何遍历来触发事件的，就应该从`SubscriberRegistry`入手。确实，个人也认为，这个类的实现也是EventBus中最精彩的部分。

#### 2.2.1 SubscriberRegistry

根据2.1中的分析，我们需要在EventBus中维护几个映射，以便在发布事件的时候找到并通知所有的监听者，首先是`事件类型->观察者列表`的映射。
上面我们也说过，EventBus中发布事件是针对各个方法的，我们将一个事件对应的类型信息和方法信息等都维护在一个对象中，在EventBus中就是观察者`Subscriber`。
然后，通过事件类型映射到观察者列表，当发布事件的时候，只要根据事件类型到列表中寻找所有的观察者并触发监听方法即可。
在SubscriberRegistry中通过如下数据结构来完成这一映射：

    private final ConcurrentMap<Class<?>, CopyOnWriteArraySet<Subscriber>> subscribers = Maps.newConcurrentMap();

从上面的定义形式中我们可以看出，这里使用的是事件的Class类型映射到Subscriber列表的。这里的Subscriber列表使用的是Java中的CopyOnWriteArraySet集合，
它底层使用了CopyOnWriteArrayList，并对其进行了封装，也就是在基本的集合上面增加了去重的操作。这是一种适用于**读多写少**场景的集合，在读取数据的时候不会加锁，
写入数据的时候进行加锁，并且会进行一次数组拷贝。

既然，我们已经知道了在SubscriberRegistry内部会在注册的时候向以上数据结构中插入映射，那么我们可以具体看下它是如何完成这一操作的。

在分析`register()`方法之前，我们先看下SubscriberRegistry内部经常使用的几个方法，它们的原理与我们上面提出的问题息息相关。
首先是`findAllSubscribers()`方法，它用来获取指定监听者对应的全部观察者集合。下面是它的代码：

    private Multimap<Class<?>, Subscriber> findAllSubscribers(Object listener) {
        // 创建一个哈希表
        Multimap<Class<?>, Subscriber> methodsInListener = HashMultimap.create();
        // 获取监听者的类型
		Class<?> clazz = listener.getClass();
        // 获取上述监听者的全部监听方法
        UnmodifiableIterator var4 = getAnnotatedMethods(clazz).iterator(); // 1
        // 遍历上述方法，并且根据方法和类型参数创建观察者并将其插入到映射表中
        while(var4.hasNext()) {
            Method method = (Method)var4.next();
            Class<?>[] parameterTypes = method.getParameterTypes();
            // 事件类型
            Class<?> eventType = parameterTypes[0];
            methodsInListener.put(eventType, Subscriber.create(this.bus, listener, method));
        }
        return methodsInListener;
    }

这里注意一下`Multimap`数据结构，它是Guava中提供的集合结构，与普通的哈希表不同的地方在于，它可以完成一对多操作。这里用来存储事件类型到观察者的一对多映射。
注意下1处的代码，我们上面也提到过，当新注册监听者的时候，用反射获取全部方法并进行判断的过程非常浪费性能，而这里就是这个问题的答案：

这里`getAnnotatedMethods()`方法会尝试从`subscriberMethodsCache`中获取所有的注册监听的方法（即使用了注解并且只有一个参数），下面是这个方法的定义：

    private static ImmutableList<Method> getAnnotatedMethods(Class<?> clazz) {
        return (ImmutableList)subscriberMethodsCache.getUnchecked(clazz);
    }

这里的`subscriberMethodsCache`的定义是：

    private static final LoadingCache<Class<?>, ImmutableList<Method>> subscriberMethodsCache = CacheBuilder.newBuilder().weakKeys().build(new CacheLoader<Class<?>, ImmutableList<Method>>() {
        public ImmutableList<Method> load(Class<?> concreteClass) throws Exception { // 2
            return SubscriberRegistry.getAnnotatedMethodsNotCached(concreteClass);
        }
    });

这里的作用机制是：当使用`subscriberMethodsCache.getUnchecked(clazz)`获取指定监听者中的方法的时候会先尝试从缓存中进行获取，如果缓存中不存在就会执行2处的代码，
调用SubscriberRegistry中的`getAnnotatedMethodsNotCached()`方法获取这些监听方法。这里我们省去该方法的定义，具体可以看下源码中的定于，其实就是使用反射并完成一些校验，并不复杂。

这样，我们就分析完了`findAllSubscribers()`方法，整理一下：当注册监听者的时候，首先会拿到该监听者的类型，然后从缓存中尝试获取该监听者对应的所有监听方法，如果没有的话就遍历该类的方法进行获取，并添加到缓存中；
然后，会遍历上述拿到的方法集合，根据事件的类型（从方法参数得知）和监听者等信息创建一个观察者，并将`事件类型-观察者`键值对插入到一个一对多映射表中并返回。

下面，我们看下EventBus中的`register()`方法的代码：

    void register(Object listener) {
        // 获取事件类型-观察者映射表
        Multimap<Class<?>, Subscriber> listenerMethods = this.findAllSubscribers(listener);
        Collection eventMethodsInListener;
        CopyOnWriteArraySet eventSubscribers;
        // 遍历上述映射表并将新注册的观察者映射表添加到全局的subscribers中
		for(Iterator var3 = listenerMethods.asMap().entrySet().iterator(); var3.hasNext(); eventSubscribers.addAll(eventMethodsInListener)) {
            Entry<Class<?>, Collection<Subscriber>> entry = (Entry)var3.next();
            Class<?> eventType = (Class)entry.getKey();
            eventMethodsInListener = (Collection)entry.getValue();
            eventSubscribers = (CopyOnWriteArraySet)this.subscribers.get(eventType);
            // 如果指定事件对应的观察者列表不存在就创建一个新的
            if (eventSubscribers == null) {
                CopyOnWriteArraySet<Subscriber> newSet = new CopyOnWriteArraySet();
                eventSubscribers = (CopyOnWriteArraySet)MoreObjects.firstNonNull(this.subscribers.putIfAbsent(eventType, newSet), newSet);
            }
        }
    }

SubscriberRegistry中的`register()`方法与`unregister()`方法类似，我们不进行说明。下面看下当调用`EventBus.post()`方法的时候的逻辑。下面是其代码：

    public void post(Object event) {
        // 调用SubscriberRegistry的getSubscribers方法获取该事件对应的全部观察者
        Iterator<Subscriber> eventSubscribers = this.subscribers.getSubscribers(event);
        if (eventSubscribers.hasNext()) {
            // 使用Dispatcher对事件进行分发
            this.dispatcher.dispatch(event, eventSubscribers);
        } else if (!(event instanceof DeadEvent)) {
            this.post(new DeadEvent(this, event));
        }
    }

从上面的代码可以看出，实际上当调用`EventBus.post()`方法的时候回先用SubscriberRegistry的getSubscribers方法获取该事件对应的全部观察者，所以我们需要先看下这个逻辑。
以下是该方法的定义：

    Iterator<Subscriber> getSubscribers(Object event) {
        // 获取事件类型的所有父类型和自身构成的集合
        ImmutableSet<Class<?>> eventTypes = flattenHierarchy(event.getClass()); // 3
        List<Iterator<Subscriber>> subscriberIterators = Lists.newArrayListWithCapacity(eventTypes.size());
        UnmodifiableIterator var4 = eventTypes.iterator();
        // 遍历上述事件类型，并从subscribers中获取所有的观察者列表
        while(var4.hasNext()) {
            Class<?> eventType = (Class)var4.next();
            CopyOnWriteArraySet<Subscriber> eventSubscribers = (CopyOnWriteArraySet)this.subscribers.get(eventType);
            if (eventSubscribers != null) {
                subscriberIterators.add(eventSubscribers.iterator());
            }
        }
        return Iterators.concat(subscriberIterators.iterator());
    }

这里注意以下3处的代码，它用来获取当前事件的所有的父类包含自身的类型构成的集合，也就是说，加入我们触发了一个Interger类型的事件，那么Number和Object等类型的监听方法都能接收到这个事件并触发。

接下来我们看真正的分发事件的逻辑是什么样的



	
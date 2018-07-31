# Guava源码分析——EventBus

EventBus的设计理念是基于观察者模式的，可以参考[设计模式(1)—观察者模式](https://juejin.im/post/5b60659df265da0f793a85ba)来了解该设计模式。

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

假如要我们去设计这样一个API，最简单的方式就是在观察者模式上进行拓展：每次调用`EventBus.post()`方法的时候，会对所有的观察者对象进行遍历，然后获取它们全部的方法，判断该方法是否使用了`@Subscribe`并且方法的参数类型是否与`post()`方法发布的事件一致，如果一致的话，那么我们就使用反射来触发这个方法。

从上面的分析中可以看出，这里面不仅要对所有的监听者进行遍历，还要对它们的方法进行遍历，找到了匹配的方法之后又要使用反射来触发这个方法。首先，当注册的监听者数量比较多的时候，链式的调用效率就不高；然后我们又要使用反射来触发匹配的方法，这样效率肯定又低了一些。那么在`Guava`的`EventBus`中是如何解决这两个问题的。

### 2.2 着手分析






首先是用观察者模式设计，用一个队列维护观察者，然后当事件发生变化的时候通知所有的观察者。

	public class EventBus {
	    // EventBus的身份标志
		private final String identifier;
		// 执行者
	    private final Executor executor;
	    // 异常处理
		private final SubscriberExceptionHandler exceptionHandler;
	    // 用来注册和取消注册观察者
		private final SubscriberRegistry subscribers;
		// 事件分发
		private final Dispatcher dispatcher;

		// 实际内部都会调用这个构造方法来初始化所有的变量
		EventBus(String identifier, Executor executor, Dispatcher dispatcher, SubscriberExceptionHandler exceptionHandler) {
			// 使用new关键字创建一个SubscriberRegistry对象，并将当前的EventBus作为参数传入
			this.subscribers = new SubscriberRegistry(this);
			this.identifier = (String)Preconditions.checkNotNull(identifier);
			this.executor = (Executor)Preconditions.checkNotNull(executor);
			this.dispatcher = (Dispatcher)Preconditions.checkNotNull(dispatcher);
			this.exceptionHandler = (SubscriberExceptionHandler)Preconditions.checkNotNull(exceptionHandler);
		}
	}
	
EventBus注册和取消注册的方法
	
    public void register(Object object) {
		// 使用SubscriberRegistry的实例进行注册
        this.subscribers.register(object);
    }

    public void unregister(Object object) {
		// 使用SubscriberRegistry的实例取消注册
        this.subscribers.unregister(object);
    }

当使用EventBus发布一个值的时候的内部逻辑：

    public void post(Object event) {
		// 从SubscriberRegistry中获取所有的观察者
        Iterator<Subscriber> eventSubscribers = this.subscribers.getSubscribers(event);
        if (eventSubscribers.hasNext()) {
			// 使用Dispatcher进行分发事件
            this.dispatcher.dispatch(event, eventSubscribers);
        } else if (!(event instanceof DeadEvent)) {
			// 当已经不存在观察者的时候就分发一个DeadEvent
            this.post(new DeadEvent(this, event));
        }
    }


	
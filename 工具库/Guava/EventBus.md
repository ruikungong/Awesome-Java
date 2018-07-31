# EventBus

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


	
# ThreadLocal原理

防止任务在共享资源上产生冲突的一种方式是根除对变量的共享，使用线程的本地存储为使用相同变量的不同线程创建不同的存储。

下面是一个ThreadLocal的实例。这里我们使用了静态的全局变量threadLocal来保存Integer类型的值，包括在main线程。我们在不同的线程中将指定的数字传入到threadLocal中进行保存。然后，再将其读取出来：

    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();

    public static void main(String...args) {
        threadLocal.set(-1);
        ExecutorService executor = Executors.newCachedThreadPool();
        for (int i=0;i<5;i++) {
            final int ii = i; // i不能是final的，创建临时变量
            executor.submit(new Runnable() {
                public void run() {
                    threadLocal.set(ii);
                    System.out.println(threadLocal.get());
                }
            });
        }
        executor.shutdown();
        System.out.println(threadLocal.get());
    }

每个线程都正确地读取出来了保存到ThreadLocal中的数据。

## 1、ThreadLocal的作用原理

在上面的程序中，我们将ThreadLocal定义为全局的静态变量，而且每次只要在指定的线程执行的方法中，调用ThreadLocal的set()和get()方法就将值保存到了指定的线程的名下。其实，当当前正在执行的线程向ThreadLocal中读写数据的时候是都是在当前线程中完成的，每次会调用Thread.currentThread()方法获取当前的线程，然后再获取该线程中的一个哈希表，再用当前的ThreadLocal作为键从哈希表中获取值。所以，数据是以键值对的形式存储在线程中的。

### 1.1 写操作

当我们在一个线程的内部，向一个ThreadLocal中写入数据的时候（调用set()方法），它要执行的逻辑如下：

    public void set(T value) {
	    // 获取当前线程
        Thread t = Thread.currentThread();
		// 在getMap()方法中会返回Thread的threadLocals字段，该字段默认是null的
        ThreadLocalMap map = getMap(t);
        if (map != null)
          // 以键值对的形式将数据写入到哈希表中
            map.set(this, value);
        else
		    // 如果指定Thread对应的threadLocals字段为Null，就实例化一个
            createMap(t, value);
    }

当调用了ThreadLocal的`set()`方法的时候，我们会先用`Thread.currentThread()`方法获取当前线程。然后，用`getMap(Thread)`方法获取当前线程的`threadLocals`字段。这个字段是`ThreadLocalMap`的实例，它是一个哈希表，每个Entry的都是一个继承自WeakReference的实例，也就是弱引用。它的键是`ThreadLocal<?>`，值是Object类型。然后当我们调用了`ThreadLocalMap`的`set(ThreadLocal<?>, Object)`方法的时候，会将指定的键值对插入到该哈希表中。其实，这里把ThreadLocalMap的实例放在Thread中，就是要实现一种一对多的关系，即一个Thread对应多种ThreadLocal<?>。

总结：当在当前的线程中调用ThreadLocal.set()方法的时候，实际上是将该ThreadLocal作为一个键，用来在哈希表(ThreadLocalMap)中根据该键获取对应的值，而实际的数据是以键值对(ThreadLocalMap.Entry)的形式存储在Thread当中的。

### 1.2 读操作

以下是读取操作相关的逻辑：

    public T get() {
	    // 获取当前线程
        Thread t = Thread.currentThread();
		// 获取当前线程的Map
        ThreadLocalMap map = getMap(t);
        if (map != null) {
		    // 获取当前ThreadLocal对应的Entry，该Entry是一个弱引用类型，可能被回收
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
			    // 从Entry中获取结果
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

从这里看出，它的确也是，先获取调用ThreadLocal的get()方法的线程，然后获取该线程内部的Map，然后将当前的ThreadLocal作为键，从map中读取一个键值对，并从中读取保存的结果。

### 1.3 底层数据结构

上面我们说ThreadLocal的变量都是保存在Thread内部的一个变量ThreadLocalMap中，那它又是怎么存储的呢？

首先，它内部定义了类似于哈希表的结点的类，不过这里它继承了WeakReference，也就是说当内存不够用的时候该键值对可能会被回收。

    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

然后，是ThreadLocalMap的定义。从下面可以看出，它的内部定义了一个Entry类型的数组：

    static class ThreadLocalMap {

        private Entry[] table;
    
        // ....
    }

这也是它实现哈希的一种方式，与HashMap和TreeMap不同的地方在于，这里实现的哈希存储是基于线性探测法来实现的。这也是上面定义数组的结点的原因，它不是链表形式的结点，所以仅仅是作为一个普通的数组的一个元素。当我们向该数组中存储元素的时候，也就是把一个“键值对”塞到数组里的时候，它实际上执行了下面的操作：

    private void set(ThreadLocal<?> key, Object value) {

        ThreadLocal.ThreadLocalMap.Entry[] tab = table;
        int len = tab.length;
        // 这里倒是跟HashMap一样，它使用2的整数次幂来实现，将指定的ThreadLocal
        // 映射到数组的索引，这里是用来截取ThreadLocal的哈希码的后几位
        int i = key.threadLocalHashCode & (len-1);

        // 这里是一个遍历操作，目的在于寻找与当前的ThreadLocal的哈希码相等的数组元素
        // 它寻找的开始位置是前面计算得出的索引，然后在一个“键簇”中进行查找
        for (ThreadLocal.ThreadLocalMap.Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

            if (k == key) {
                e.value = value;
                return; // 注意，找到了更新之后就返回了
            }

            if (k == null) {
                replaceStaleEntry(key, value, i);
                return;
            }
        }

        // 表示没有找到，新建一个结点并赋值给数组的指定元素
        tab[i] = new ThreadLocal.ThreadLocalMap.Entry(key, value);
        // 散列表中存储的记录总数+1
        int sz = ++size;
        // 如果已经存储的容量大于我们指定的容量，那么对数组进行扩容，并重新计算哈希值
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            rehash();
    }

以上就是ThreadLocal的基本用法和底层的实现的原理。
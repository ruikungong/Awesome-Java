## 4、线程同步

### 4.1 Sychronized

要控制线程对共享资源的访问，一种方式是将需要共享的资源包装进一个对象。然后把要访问该对象的资源的方法标记为sychronized。如果某个线程处于对标记为sychronized的方法调用中，那么在这个方法返回之前，其他调用类中任何标记sychronized方法的线程都会被阻塞。

注意在使用并发时，将域设置为private是非常重要的，否则，sychronized关键字则不能防止其他任何直接访问域，这样会产生冲突。

还要注意如果我们对一个类中的两个方法f()和g()加了同步锁，那么如果我们的一个线程获取了f()的锁，并且还没有释放，那么任何其他线程都不能访问g()方法，但是可以访问h()方法：

    public synchronized void f() {}
    public synchronized void g() {}
    public void h() {}

JVM会跟踪重复加锁并计数，每当离开一个sychronized方法时，计数递减，当计数减至0的时候，锁被完全释放，此时别的任务就可以使用此资源。

### 4.2 显式的Lock对象

也可以使用Lock对象来进行加锁，Lock对象必须被显式地创建、锁定和释放。它的缺点是，相比于使用sychronized关键字加锁的方式，它不够灵活，而且容易出错。因此，通常只有在解决特殊问题时，才使用显式的Lock对象。

使用Lock的方式，如果事务失败了，那么那么我们还可以对异常进行处理。而使用sychronized的时候，只能抛出异常，而无法做任何清理工作。

我们需要使用try-finally结构来使用Lock加锁，而且如果有return语句的话，它return应该被放置在try语句块中，以确保unlock不会过早地发生，从而将数据暴露给第二个任务。

除了使用lock()方法，还可以使用tryLock()方法，它有两个重载的方法，其中的一个还可以用来设置超时的时间。它具有一个boolean类型的返回值，用来判断是否成功获取到锁。

    ReentranLock lock = new ReentrantLock();

    public int method() {
        lock.lock();
        try{
            // ...
            return ret;
        } finally {
            lock.unlock();
        }
    }

### 4.3 原子性与可见性

**原子性**可以应用于除了long和double之外的所有基本类型上的简单操作。对于long和double类型，它们的读写被当作两个分离的32位操作来执行，这就有可能导致行为的不确定性。但是，如果定义long和double类型时，使用了volatile关键字，就会获得原子性。

++和--操作虽然看起来简洁，实际上该操作不是原子的。

**可见性**：一个任务做出的修改，即使在不中断的意义上讲是原子的，但是其他任务也可能是不可见的。voliate关键字保证了应用程序中的可见性。若将域声明为**volatile**的，那么只要对这个域产生了写操作那么所有的读操作都可以看到这个修改，即使使用了本地缓存。

如果多个任务在同时访问某个域，这个域就应该是volatile的，否则，这个域应该只能由同步来访问。同步也会导致向主存中刷新，因此，如果一个域完全由sychronized方法或语句来防护，那就不必将其设置为volatile的。

如果一个域可以被多个任务同时访问，且这些任务中至少有一个是写入操作，就应该将这个域设置为volatile的。

示例代码 该程序在找到一个奇数的时候结束程序的运行：

    public static void main(String ...args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        final MyClass myClass = new MyClass();
        Runnable runnable = new Runnable() {
            public void run() {
               while (true) {
                   myClass.add();
               }
            }
        };
        executor.execute(runnable);
        executor.shutdown();
        int val;
        while (true) {
            if ((val = myClass.getValue()) % 2 != 0) {
                System.out.println(val);
                System.exit(1);
            }
        }
    }

    private static class MyClass {
        private int value = 0;
        public synchronized void add() {
            value++;
            value++;
        }
        public int getValue() {
            return value;
        }
    }

实际程序的执行结果是，输出了9719并结束了程序。按理说，add()方法中的value连续自增两次是没有奇数的，那么为什么会出现奇数呢？

出现奇数是程序中缺少同步使得其数值可以在处于不稳定的中间状态的时候被读取。解决的方法是在getValue方法上加上sychronized关键字，这样只有当一个线程释放了锁之后，另一个线程才能获取到写入的值。

### 4.4 原子类

解决上面的问题还可以使用原子类：这里使用了AtomicInteger类，并用了它的addAndGet(2)方法。这是一个原子的操作，用来取代之前的两次自增操作。而且使用了原子类之后我们就不需要在方法上面添加sychronized关键字了，因为它的每次操作都是原子的。

    public static void main(String ...args) {
        ExecutorService executor = Executors.newCachedThreadPool();
        final MyClass myClass = new MyClass();
        Runnable runnable = new Runnable() {
            public void run() {
               while (true) {
                   myClass.add();
               }
            }
        };
        executor.execute(runnable);
        executor.shutdown();
        int val;
        while (true) {
            if ((val = myClass.getValue()) % 2 != 0) {
                System.out.println(val);
                System.exit(1);
            }
        }
    }

    private static class MyClass {
        private AtomicInteger value = new AtomicInteger(0);
        public void add() {
            value.addAndGet(2);
        }
        public int getValue() {
            return value.get();
        }
    }

Atomic类被设计用来构建concurrent包中的类，因此只有在特殊情况下才在自己的代码中使用它们。对于常规编程，它们很少派上用场，但是在涉及性能调优时，它们大有用武之地。

### 4.5 临界区

**临界区**是指通过使用sychronized关键字分离出来的代码段。这里sychronized常被用来指定某个对象，此对象的锁被用来对花括号内的代码进行同步控制：

    sychronized(syncObject) {
        // ...
    }

这也被成为**同步代码块**。在进入此段代码之前，必须得到syncObject（是syncObject上面的锁）上面的锁。如果其他线程已经得到了这个锁，那么就得等到锁被释放以后，才能进入临界区。

通过使用同步代码块，而不是对整个方法进行同步控制，可以使多个任务访问对象的时间性能得到显著提高。

使用sychronized同步代码块最合理的方式是，使用其方法正在被调用的当前对象：**sychronized(this)**. 在这种方式中，如果获得了sychronized块上的锁，那么该对象其他的**sychronized方法和临界区**就不能被调用。因此，如果在this上面同步，临界区的效果就会直接缩小在同步的范围内。

    private Object syncObject = new Object();

    public synchronized void f() {
        // ...
    }

    public void g() {
        synchronized (syncObject) {
            // ...
        }
    }

如上所示，这里第一个锁加在f方法上，它会对f()方法所在的整个类进行加锁。而g()方法内部的同步代码块将锁加在syncObject对象上，它不会对g()方法所在的整个类进行加锁，仅对syncObject加锁。所以，如果我们在不同的线程中同时访问f()和g()两个方法是不会有问题的。但是，有一点格外需要注意的是，**f()方法内部不应该用到syncObject对象，因此有可能发生死锁**。

参考：
1. [Java线程的5种状态及切换(透彻讲解)](https://blog.csdn.net/pange1991/article/details/53860651)
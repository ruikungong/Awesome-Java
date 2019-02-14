# 并发编程专题 3：synchronized

根据 synchronized 可以被用来修饰的对象可以分成下面几种情形，

![synchronized 的修饰分类](res/synchronized_1.jpg)

可以通过对编译生成的代码反汇编来了解 synchronized 的作用原理，

源代码，

```java
public class SyncTest {

    static int i = 0;

    public static void main(String...args) {
        synchronized (SyncTest.class) {
            i++;
        }
    }
}
```

使用如下代码进行反汇编，

```
javap -c ./SyncTest.class
```

反汇编结果，

```
Compiled from "SyncTest.java"
public class me.shouheng.test.SyncTest {
  static int i;

  static {};
    Code:
       0: iconst_0
       1: putstatic     #10                 // Field i:I
       4: return

  public me.shouheng.test.SyncTest();
    Code:
       0: aload_0
       1: invokespecial #15                 // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String...);
    Code:
       0: ldc           #1                  // class me/shouheng/test/SyncTest
       2: dup
       3: astore_1
       4: monitorenter                      // 获取锁
       5: getstatic     #10                 // Field i:I
       8: iconst_1
       9: iadd
      10: putstatic     #10                 // Field i:I
      13: aload_1
      14: monitorexit                       // 释放锁
      15: goto          21
      18: aload_1
      19: monitorexit
      20: athrow
      21: return
    Exception table:
       from    to  target type
           5    15    18   any
          18    20    18   any
}
```

对照 Java 源代码和反汇编之后生成的代码，我们可以看出，被 `synchronized` 修饰的 `i++` 一行被 `monitorenter` 和 `monitorexit` 两个关键字包裹。它们分别用来获取`监视器 monitor` 和释放监视器 monitor. 

synchronized 先天具有重入性：在执行 sychronized 指令时，首先要尝试获取对象的锁。如果这个对象没有被锁定，或者当前线程已经拥有了该对象的锁，就把锁的计数器加 1，相应地执行 `monitorexit` 指令时会将锁的计数器减 1，当计数器为 0 时就释放锁。若获取对象锁失败，那当前线程就要阻塞等待，直到对象锁被另外一个线程释放为止。在上面的程序中存在 1 个 monitorenter 和两个 monitorexit，这说明 synchronized 关键字修饰的锁是可重入的，当获取了一次锁之后就不需要再次获取锁。

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到 BLOCKED 状态。

除了使用 sychronized，我们还可以使用 JUC 中的 ReentrantLock 来实现同步，它与 sychronized 类似，区别主要表现在以下 3 个方面：

1. 等待可中断：当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待。ReentrantLock 使用 CAS 实现，可以在获取锁的时候设置一个超时的时间，当到达了指定的时间仍然没有获取到锁可以放弃获取锁。
2. 公平锁：多个线程等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁；而非公平锁无法保证，当锁被释放时任何在等待的线程都可以获得锁。sychronized 本身时非公平锁，而 ReentrantLock 默认是非公平的，可以通过构造函数要求其为公平的。公平锁和非公平锁对应于 ReentrantLock 中的两个内部类，
3. 锁可以绑定多个条件：ReentrantLock 可以绑定多个 Condition 对象，而 sychronized 要与多个条件关联就不得不加一个锁，ReentrantLock 只要多次调用 newCondition 即可。

在 JDK1.5 之前，sychronized 在多线程环境下比 ReentrantLock 要差一些，但是在 JDK1.6 以上，虚拟机对 sychronized 的性能进行了优化，性能不再是使用 ReentrantLock 替代 sychronized 的主要因素。

缺点：synchronized 的一个问题就是效率比较低，因为每次只有一个线程能够获取到对象的监视器。所以，为了提升 synchronized 的效率，提出了 CAS、轻量级锁锁和偏向锁等概念。而 synchronized 则被称为重量级锁。对于新提出的几个概念，CAS 是基础，轻量级锁和偏向锁是在 CAS 基础之上进行的优化。

## 附录

不加锁时的代码

```java
public class SyncTest {

    static int i = 0;

    public static void main(String...args) {
//        synchronized (SyncTest.class) {
            i++;
//        }
    }
}
```

不加锁时的反汇编，

```
Compiled from "SyncTest.java"
public class me.shouheng.test.SyncTest {
  static int i;

  static {};
    Code:
       0: iconst_0
       1: putstatic     #10                 // Field i:I
       4: return

  public me.shouheng.test.SyncTest();
    Code:
       0: aload_0
       1: invokespecial #15                 // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String...);
    Code:
       0: getstatic     #10                 // Field i:I
       3: iconst_1
       4: iadd
       5: putstatic     #10                 // Field i:I
       8: return
}
```




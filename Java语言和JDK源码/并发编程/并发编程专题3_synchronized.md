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

对照 Java 源代码和反汇编之后生成的代码，我们可以看出，被 synchronized 修饰的 `i++` 一行被 monitorenter 和 monitorexit 两个关键字包裹。它们分别用来获取监视器 monitor 和释放监视器 monitor. synchronized 先天具有重入性。每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一。

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到 BLOCKED 状态。

synchronized 的一个问题就是效率比较低，因为每次只有一个线程能够获取到对象的监视器。所以，为了提升 synchronized 的效率，提出了 CAS、轻量级锁锁和偏向锁等概念。而 synchronized 则被称为重量级锁。对于新提出的几个概念，CAS 是基础，轻量级锁和偏向锁是在 CAS 基础之上进行的优化。

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




# Java 基础回顾：HashMap 源码分析

要理解 HashMap 的实现原理需要数据结构方面的知识，这里给出之前整理的一些数据结构和 `hashCode()` 码方法覆写相关的内容：[《Java 基础回顾：几个比较重要的预定义类》](https://blog.csdn.net/github_35186068/article/details/85047777) 及 [《Awesome-Java》见数据结构与算法部分](https://github.com/Shouheng88/Awesome-Java)。

Java 集合 API 中的 HashMap 是使用**拉链法**来解决碰撞冲突的。所谓拉链法就是使用一个数组存储链表的头结点。对每个键值对，我们计算键的哈希码，并对得到的哈希码取余操作，从而得到该键值对所在的链表在数组中的索引，然后将其插入到指定的链表中。不过，为了提升性能在 HashMap 的实现中对许多点进行了优化：动态调整数组的大小；当一个链表存储的元素比较多的时候，采用红黑树来提升查找的效率等。

下面我们具体来看 HashMap 的实现的逻辑，

## 1、结点的定义

首先是每个键值对结点的定义，

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
```

从这里可以看出，该结点直接实现了 Map 中定义的 Entry. 这个结点是适用于链表的，此外，还有一个适用于红黑树的结点：

```java
    private static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;  // 红黑树父结点
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
    }
```

该树结点继承了 `LinkedHashMap.Entry`，而 `LinkedHashMap.Entry` 又继承了 `HashMap.Node`. 因此，可以说 TreeNode 间接继承了 Node. 那么，`LinkedHashMap.Entry` 又是何方神圣呢？其定义如下，

```java
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

如果熟悉 LinkedHashMap 的话，你可能会直到它在 HashMap 的基础上，同时将哈希表中的元素维护到双向链表中，而上面的类就是该双向链表的结点。

有了 TreeNote 的基础关系和链表结构，所以，当链表因为长度太长而转换成红黑树之后链表之间的关系可以保留下来。

## 2、几个常量的含义 & 哈希表的计算过程

    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16

哈希表默认的长度，即所谓的桶的长度，当不显式指定哈希表的长度的时候将使用这个长度。

    static final int MAXIMUM_CAPACITY = 1 << 30;

哈希表的最大容量，需要是 2 的倍数。

    static final float DEFAULT_LOAD_FACTOR = 0.75f;

负载因子，当哈希表中已经使用的桶的数量达到了这个比例的时候就需要进行扩容。

    static final int TREEIFY_THRESHOLD = 8;

当哈希表中的一个桶中的元素的数量达到了这个数值的时候，桶的数据结构将从链表转型成为红黑树，以提升查找的性能。

    static final int UNTREEIFY_THRESHOLD = 6;

当哈希表中的桶中的元素小于这个数量的时候，桶的数据结构将从红黑树转型成为链表。

在 HashMap 的定义中，特别提到许多的常量要是 2 的倍数，这是为了提升计算的效率。下面是哈希表的计算过程，

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

Java 中的每个 Object 都有一个名为 `hashCode()` 的方法，用来获取实例的哈希值。既然每个实例都已经提供了这样一个方法，那么我们为什么还要再进行计算呢？说白了，这里就是为了提高散列码的随机性。因为插入的元素分布越均匀，我们在进行查找的时候效率就越高（链表比较短）。而至于为什么取后 16 位进行异或操作呢？首先，我们想要哈希码的高 16 位和低 16 位都参与计算，因为这可以提高散列的随机性。其次，因为我们需要使用

因为我们将链表的头结点保存在数组中，所以需要将这里的哈希码映射到指定的数组索引。我们这里使用的是除留余数法，就是将哈希码除以某个数之后的余数作为数组的索引。而在 HashMap 中实际是这样实现除法操作的：

    table[e.hash & (newCap - 1)]
    // 比如当newCap为16的时候，减1得15，即后四位全1，再与hash值取并
    // 那么就得到了hash值在数组中的索引，实际上是为了降低除法的低效率

也就是将指定的哈希码与哈希表容量减一的并作为数组的索引。我们没有使用取余的操作，因为除法和取余是现代操作系统最慢的操作。我们在计算哈希表的容量的时候故意使用2的整数次幂（减1之后就是低位全1的二进制），在计算数组时将其减1再和哈希码取并，这样就避免了使用除法。这也是为什么哈希表的容量要是2的整数次幂的原因。所以，也就是因为我们计算索引时只用后面的几位数这个原因，才将h移位之后异或的。

保证每次取得的哈希表的容量是2的整数次幂的相关代码：

    private static int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

该方法保证了每次计算返回的结果是不小于cap的最小的2的整数次幂。

HashMap在性能方面进行了许多优化，使用了哈希表和红黑树两种数据结构，因此也是面试的热点。这里是一份简单的源码分析：[HashMap的源码分析](https://github.com/Shouheng88/Java-Programming/blob/master/Java%20Language/HashMap.java)。

3.关于HashMap中的几个字段

1. loadFactor：负载因子，默认为0.75，值等于尺寸除以容量。负载因子取值0.75是空间和效率的一个折衷。毕竟但当负载因子小的时候，空间消耗会比较大；而当负载因子比较大的时候，虽然空间消耗小了，但是查询效率低了（因为要等到插入1000个元素才调整大小，这时候碰撞的几率大了，链表的长度可能会增加，降低了查找的效率）。
2. threshold：下次扩容的临界值，size>=threshold就会扩容
3. size：已存元素的个数
4. DEFAULT_INITIAL_CAPACITY：默认的哈希表的容量，值为16.

 当HashMap存放的元素越来越多，到达临界值(阀值)threshold时，就要对Entry数组扩容，HashMap在扩容时，新数组的容量将是原来的2倍，由于容量发生变化，原有的每个元素需要重新计算索引，再存放到新数组中去，也就是所谓的rehash。HashMap默认初始容量16，加载因子0.75，也就是说最多能放16*0.75=12个元素，当put第13个时，HashMap将发生rehash，rehash的一系列处理比较影响性能，所以当我们需要向HashMap存放较多元素时，最好指定合适的初始容量和加载因子，否则HashMap默认只能存12个元素，将会发生多次rehash操作。

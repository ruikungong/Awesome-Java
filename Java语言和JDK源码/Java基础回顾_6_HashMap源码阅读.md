# Java 基础回顾：HashMap 源码分析

要理解 HashMap 的实现原理需要数据结构方面的知识，这里给出之前整理的一些数据结构和 `hashCode()` 方法覆写相关的知识：[《Java 基础回顾：几个比较重要的预定义类》](https://blog.csdn.net/github_35186068/article/details/85047777) 及 [《Awesome-Java》见数据结构与算法部分](https://github.com/Shouheng88/Awesome-Java)。

Java 集合 API 中的 HashMap 是使用**拉链法**来解决碰撞冲突的。所谓拉链法就是使用一个数组存储链表的头结点。对每个键值对，我们计算键的哈希码，并对得到的哈希码取余操作，从而得到该键值对所在的链表在数组中的索引，然后将其插入到指定的链表中。不过，为了提升性能在 HashMap 的实现中对许多点进行了优化：动态调整数组的大小；当一个链表存储的元素比较多的时候，采用红黑树来提升查找的效率等。这部分内容可以参考《算法4》，因为比较基础，所以我们不详细进行说明。下面我们具体来看 HashMap 的实现的逻辑，

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

```java
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16
```

哈希表默认的长度，即所谓的桶的长度，当不显式指定哈希表的长度的时候将使用这个长度。

```java
    static final int MAXIMUM_CAPACITY = 1 << 30;
```

哈希表的最大容量，需要是 2 的倍数。

```java
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

负载因子，当哈希表中已经使用的桶的数量达到了这个比例的时候就需要进行扩容。比如当桶的数量为 16 的时候，已经使用的桶达到了 16 * 0.75 = 12 时即扩容。负载因子取值 0.75 是空间和效率的一个折衷。毕竟但当负载因子小的时候，空间消耗会比较大；而当负载因子比较大的时候，虽然空间消耗小了，但是查询效率低了（每个桶的长度增加了）。

```java
    static final int TREEIFY_THRESHOLD = 8;
```

当哈希表中的一个桶中的元素的数量达到了这个数值的时候，桶的数据结构将从链表转型成为红黑树，以提升查找的性能。

```java
    static final int UNTREEIFY_THRESHOLD = 6;
```

当哈希表中的桶中的元素小于这个数量的时候，桶的数据结构将从红黑树转型成为链表。

```java
    static final int MIN_TREEIFY_CAPACITY = 64;
```

当一个桶中的元素的数量达到了 TREEIFY_THRESHOLD 指定的值时，要判断 HashMap 中的数组的长度是否大于等于 MIN_TREEIFY_CAPACITY，只有当数组的长度不小于 MIN_TREEIFY_CAPACITY 的时候，才会对桶进行数据结构的调整。否则，会优先对哈希表进行扩容，而不是转换桶的数据结构。

在 HashMap 的定义中，特别提到许多的常量要是 2 的倍数，这是为了提升计算的效率。下面是哈希表的计算过程，

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

Java 中的每个 Object 都有一个名为 `hashCode()` 的方法，用来获取实例的哈希值。既然每个实例都已经提供了这样一个方法，那么我们为什么还要再进行计算呢？说白了，这里就是为了提高散列码的随机性。因为插入的元素分布越均匀，我们在进行查找的时候效率就越高（链表比较短）。而至于为什么取后 16 位进行异或操作呢？首先，我们想要哈希码的高 16 位和低 16 位都参与计算，因为这可以提高散列的随机性。其次，因为我们需要将每个结点分配到桶中，一种计算方式是**除留余数法**，即将哈希码除以桶的容量之后得到的余数作为桶的索引的值。但是，因为取余的操作效率比较低，所以我们使用下面的方法来获取索引，

```java
    table[e.hash & (newCap - 1)]
```

也就是将当前容量的值减 1 之后与哈希码进行与运算。比如当 `newCap` 为 16 的时候，减 1 得 15，对应的二进制的后四位全 1，再与 hash 值进行与运算，那么就得到了 hash 值在数组中的索引（值在 0 ~ 15 之间）。显然，这样做的前提是**桶的数量，也就是上面的 newCap 必须是 2 的整数倍**。因此，每次扩容的时候使用下面的方式计算，以保证每次取得的哈希表的容量是 2 的整数次幂：

```java
    private static int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

该方法保证了每次计算返回的结果是不小于 cap 的最小的 2 的整数次幂。

## 3、查找

下面是 HashMap 的查找的方法，所有的查找操作都会调用到内部定义的 `getNode()` 方法来进行查找，

```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; // 桶
        Node<K,V> first, e; // 拉链的头结点、临时的变量（结点）
        int n; // 桶的数量 
        K k; // 临时的变量，结点的键
        if ((tab = table) != null && (n = tab.length) > 0 // 桶不为空
            && (first = tab[(n - 1) & hash]) != null // 拉链的头结点不为空
        ) {
            if (first.hash == hash // 结点的哈希码相同 
                && ((k = first.key) == key || (key != null && key.equals(k))) // 结点键的值相等
            ) {
                return first;
            }
            if ((e = first.next) != null) { 
                // 这里根据头结点是树还是链表的一部分来按照两种方式遍历
                if (first instanceof TreeNode) { // 头结点是树的一部分，到红黑树中进行遍历
                    return ((TreeNode<K,V>) first).getTreeNode(hash, key);
                }
                do {
                    if (e.hash == hash // 头结点是链表的一部分，按照链表的方式遍历
                        && ((k = e.key) == key || (key != null && key.equals(k)))
                    ) {
                        return e;
                    }
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

在查找的时候先对桶进行判断，得到其头结点。因为桶的数据结构可能是树，也可能是链表，所以需要分成两种情况来进行处理。首先，我们对头结点进行判断，判断它是否是我们要找的结点。然后，我们判断头结点的类型，如果它是一棵树，我们使用 TreeNote 的 `getTreeNode()` 方法到树中进行查找。如果它是链表的一部分，那么我们就直接通过对链表进行遍历来查找对应的键。

链表的遍历比较简单，一个 while 循环就可以搞定了。下面我们来看下 HashMap 是如何在红黑树中进行遍历的：

```java
    final TreeNode<K,V> getTreeNode(int h, Object k) {
        return ((parent != null) ? root() : this).find(h, k, null);
    }

    final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
        TreeNode<K,V> p = this;
        do {
            int ph, dir; K pk;
            TreeNode<K,V> pl = p.left, pr = p.right, q;
            if ((ph = p.hash) > h) {
                p = pl; // 到左侧继续查找
            } else if (ph < h) {
                p = pr; // 到右侧继续查找
            } else if ((pk = p.key) == k || (k != null && k.equals(pk))) {
                return p; // 找到结点
            } else if (pl == null) {
                p = pr; // 左侧分支为空，找不到了
            } else if (pr == null) {
                p = pl; // 右侧分支为空，找不到了
            } else if ((kc != null || (kc = comparableClassFor(k)) != null)
                    && (dir = compareComparables(kc, k, pk)) != 0) {
                p = (dir < 0) ? pl : pr;
            } else if ((q = pr.find(h, k, kc)) != null) {
                return q;
            } else {
                p = pl;
            }
        } while (p != null);

        return null;
    }
```

红黑树的原理是在基础的二叉树之上为每个结点增加了一个红黑的属性，它本质上是一棵二叉树，只是它的左右不平衡。因此，它的查找的逻辑与基本的二叉树相同。

## 4、插入

下面是 HashMap 的插入的逻辑，所有的插入操作最终都会调用到内部的 `putVal()` 方法来最终完成。

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    private V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;

        if ((tab = table) == null || (n = tab.length) == 0) { // 原来的数组不存在
            n = (tab = resize()).length;
        }

        i = (n - 1) & hash; // 取哈希码的后 n-1 位，以得到桶的索引
        p = tab[i]; // 找到桶
        if (p == null) { 
            // 如果指定的桶不存在就创建一个新的，直接new 出一个 Node 来完成
            tab[i] = newNode(hash, key, value, null);
        } else { 
            // 指定的桶已经存在
            Node<K,V> e; K k;

            if (p.hash == hash // 哈希码相同
                && ((k = p.key) == key || (key != null && key.equals(k))) // 键的值相同
            ) {
                // 第一个结点与我们要插入的键值对的键相等
                e = p;
            } else if (p instanceof TreeNode) {
                // 桶的数据结构是红黑树，调用红黑树的方法继续插入
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            } else {
                // 桶的数据结构是链表，使用链表的处理方式继续插入
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 已经遍历到了链表的结尾，还没有找到，需要新建一个结点
                        p.next = newNode(hash, key, value, null);
                        // 插入新结点之后，如果某个链表的长度 >= 8，则要把链表转成红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) {
                            treeifyBin(tab, hash);
                        }
                        break;
                    }
                    if (e.hash == hash // 哈希码相同 
                        && ((k = e.key) == key || (key != null && key.equals(k))) // 键的值相同
                    ) {
                        // 说明要插入的键值对的键是存在的，需要更新之前的结点的数据
                        break;
                    }
                    p = e;
                }
            }

            if (e != null) { 
                // 说明指定的键是存在的，需要更新结点的值
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null) {
                    e.value = value;
                }
                afterNodeAccess(e);
                return oldValue;
            }
        }

        ++modCount;

        // 如果插入了新的结点之后，哈希表的容量大于 threshold 就进行扩容
        if (++size > threshold) {
            resize(); // 扩容
        }

        afterNodeInsertion(evict); // 回调

        return null;
    }
```

上面是 HashMap 的插入的逻辑，可以看出，它也是根据头结点的类型，分成红黑树和链表两种方式来进行处理的。对于链表，上面已经给出了具体的插入逻辑。在链表的情形中，除了基础的插入，当链表的长度达到了 8 的时候还要将桶的数据结构从链表转型成为红黑树。对于红黑树类型的数据结构，它调用 TreeNode 的 `putTreeVal()` 方法来完成往红黑树中插入结点的逻辑。

## 5、删除

下面的方法是从 HashMap 中移除一个键值对的逻辑，它内部会调用 `removeNode()` 方法来最终完成移除操作。

```java
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
    }

    private Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 // 数组不为空
            && (p = tab[index = (n - 1) & hash]) != null)  // 桶是存在的
        {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash 
                && ((k = p.key) == key || (key != null && key.equals(k)))) {
                // 头结点就是我们要移除的结点
                node = p;
            } else if ((e = p.next) != null) {
                if (p instanceof TreeNode) {
                    // 桶的数据结构是一棵红黑树，先找到这个结点
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                } else {
                    // 桶的数据结构是链表，使用循环找该结点
                    do {
                        if (e.hash == hash 
                            && ((k = e.key) == key || (key != null && key.equals(k)))) {
                            // 找到了对应的结点
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }

            if (node != null && (!matchValue || (v = node.value) == value ||
                    (value != null && value.equals(v)))) {
                if (node instanceof TreeNode) {
                    // 使用红黑树的方法移除结点
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                } else if (node == p) {
                    // 使用当前结点的下一个结点作为链表的头结点
                    tab[index] = node.next;
                } else {
                    p.next = node.next;
                }
                ++modCount;
                --size;
                // 回调
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

从上面的方法中可以看出，在删除某个结点的时候，首先也是要寻找这个结点。当找到了结点之后就根据该结点所在的数据结构是红黑树还是链表来进行处理。红黑树会调用 TreeNode 的 `removeTreeNode()` 方法来移除结点。链表会直接将当前结点的下一个结点作为当前结点，即链表的删除。



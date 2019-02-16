# Java 基础回顾-5：容器类

## 0、基础

数组的效率比容器高，但是我们应该**优先选择容器而不是数组**，只有在已证明性能成为问题时，才应该将程序重构为使用数组。

## 1、容器的框架

下面是 Java 中的容器框架的基础关系图，

![Java 容器框架的继承关系](res/java_container.png)

根据容器的存储方式，我们可以将 Java 容器类划分成下面两个大类：

1. `Collection`：一组独立元素的列表，它有三个子类型 List、Set 和 Queue. 它们之间的区别是 `List` 必须按照插入的顺序保存元素，`Set` 不能有重复元素，`Queue` 是队列性质的列表。
2. `Map`：表示存储的是键值对类型的对象。

此外，还有两个工具类可以使用：`Arrays` 和 `Collections`. 前者提供了适用于数组类型的一系列静态工具方法，后者提供了许多适用于各种类型的静态工具方法。

## 2、Collection

顾名思义，它的作用效果与数学中的 "集合" 的概念是相似的。

这里给出 Java 中 Collection 接口的定义。Collection 作为框架中顶层的基接口，规定了其实现类应该实现哪些功能，

```java
	public interface Collection<E> extends Iterable<E> {
	    int size();
	    boolean isEmpty();
	    boolean contains(Object o);
	    Iterator<E> iterator();
	
	    // 返回列表元素组成的数组
	    Object[] toArray();
	
	    // 返回列表元素组成的数组，不同的是它可以根据传入的泛型数组，返回指定类型的数组
	    <T> T[] toArray(T[] a);
	
	    boolean add(E e);
	    boolean remove(Object o);
	    boolean containsAll(Collection<?> c);
	    boolean addAll(Collection<? extends E> c);
	    boolean removeAll(Collection<?> c);
	
	    // 只保留该集合中处于c中的元素，即只保留当前集合与c的交集元素
	    boolean retainAll(Collection<?> c);
	
	    void clear();
	}
```

### 2.1 List

List 在 Collection 的基础之上又增加了几个方法。从下面的方法的定义中，我们可以看出这些方法是针对列表类型的容器定义的方法：

```java
    boolean addAll(int index, Collection<? extends E> c);
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);
    int indexOf(Object o);
    int lastIndexOf(Object o);
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);
    List<E> subList(int fromIndex, int toIndex);
```

这里的前面的几个方法从字面意思中就可以看出大致的作用。后面的 `listIterator()` 方法的作用是返回一个迭代器，这个迭代器不仅能向后遍历，还能向前遍历；而 `subList()` 的作用是返回容器中元素的一个视图（如果在视图中添加元素，添加的元素也会反应到原来的容器上）。

`subList` 视图实现插入元素的具体方式是（以 ArrayList 为例）：在创建 `SubList`（通常继承自 `AbstractList`，作为一个 List 实现）的时候，将外部类作为参数传入到 Sublist 的构造方法中，然后当用户调用视图的添加元素的方法的时候，它调用外部类 `ArrayList` 的 `add()` 方法，同时将元素添加到外部类容器中。(装饰器模式)

List 的一个顶层实现是 AbstractList，它是一个抽象的类，为我们实现了 List 中定义的一些方法。我们熟知的 ArrayList 和 LinkedList 是在它的基础之上拓展的。

#### 2.1.1 ArrayList

它内部是基于 **动态数组** 来实现的，所以它具有数组的一些特性。比如，访问固定位置的某一元素效率比较高，而对容器中的元素查找效率比较低（需要遍历数组进行查找）。

它**默认的初始容量是10**，每次加入一个新的元素的时候，会将加入元素之前的容量与新加入的元素数量的和与加入元素之前的容量的 3/2 进行对比，选择较大的一个作为将要拓充到的容量。具体的调整数组大小的操作是由 **System.arrayCopy()** 来实现的。

至于在 ArrayList 中的删除操作，也是先将元素置为 null，之后使用动态 `System.arrayCopy()` 来调整数组的大小。

在 Arrays 中还有一个 ArrayList 的实现 **Arrays.ArrayList**，它跟这里的 ArrayList 是不同的。虽然后者也继承了 AbstractList，但是它在实现的时候没有实现 `add()` 等方法，所以是不能向其中添加新元素的。（不可变）

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    private static final int DEFAULT_CAPACITY = 10;

    private static final Object[] EMPTY_ELEMENTDATA = {};

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    transient Object[] elementData; // 数据
    
    private int size;

    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }

    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    // ...
}
```

#### 2.1.2 LinkedList

它内部是使用双向链表来实现的，所以它的一些操作和双向链表逻辑相同。比如查找某个元素的时候需要对链表进行遍历，而插入元素的时候效率比较高。

我们可以很容易地将它改造成队列、栈或双端队列。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, Serializable {

    transient int size = 0; // 链表大小

    transient Node<E> first; // 链表尾

    transient Node<E> last; // 链表头

    // 结点类
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

    // ...
}
```

#### 2.1.3 Stack

队列数据结构，我们可以使用 LinkedList 来实现一个队列，但是在 Java 提供的 API 中，队列是继承自 Vector 的，而 Vector 内部使用数组来实现。这就导致了 Stack 在性能上，相比于使用链表的方式还有值得提升的地方。

```java
public class Stack<E> extends Vector<E> {

    public Stack() {
    }

    public E push(E item) {
        addElement(item);
        return item;
    }

    public synchronized E pop() {
        E       obj;
        int     len = size();
        obj = peek();
        removeElementAt(len - 1);
        return obj;
    }

    public synchronized E peek() {
        int     len = size();
        if (len == 0) throw new EmptyStackException();
        return elementAt(len - 1);
    }

    // ...
}
```

Stack 与 Vector 类似，是线程安全的，实现线程安全的方式是对所有的 API 方法进行加锁。虽然可以在多线程环境中运行，但是非多线程环境下效率比较低。

### 2.2 Set

它不允许插入两个重复的元素。

Java 中的 Set 并没有在 Collection 的基础之上再增加新的方法。它的一个抽象基类的实现是 `AbstractSet`，不过在 AbstractSet 中并没有实现太多的方法。

它有几个重要的实现：

1. TreeSet 将元素存储在红黑树数据结构中；
2. HashSet 使用的是散列函数；
3. LinkedHashList 使用了散列，但是看起来它使用了链表来维护元素的插入顺序。

#### 2.2.1 HashSet

内部是借助于 HashMap 来实现的：当向 HashSet 中插入一个元素的时候，我们将插入的元素作为 HashMap 的键，一个默认的 Object 作为值插入到 HashMap 中。因为 HashMap 中不会有两个相同的键，所以在 HashSet 中就不会有两个相同的值。

HashSet 没有提供普通的 `get()` 方法，我们只能对其进行遍历。它在遍历的时候当然返回的是HashMap 的键的迭代器。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }

    public boolean remove(Object o) {
        return map.remove(o) == PRESENT;
    }

    // ...
}
```

#### 2.2.2 TreeSet

与 HashSet 类似，它内部使用了 TreeMap 来实现自己的功能。

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    private transient NavigableMap<E,Object> m;

    private static final Object PRESENT = new Object();

    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }

    public TreeSet() {
        this(new TreeMap<E,Object>());
    }

    public boolean add(E e) {
        return m.put(e, PRESENT) == null;
    }

    public boolean remove(Object o) {
        return m.remove(o) == PRESENT;
    }

    // ...
}
```

## 3、Map

与 Collection 相似，Map 是所有的基于键值对的容器的顶层接口。下面是它内部定义的一些方法：

```java
	public interface Map<K,V> {
	    int size();
	    boolean isEmpty();
	    boolean containsKey(Object key);
	    boolean containsValue(Object value);
	    V get(Object key);
	    V put(K key, V value);
	    V remove(Object key);
	    void putAll(Map<? extends K, ? extends V> m);
	    void clear();
	    Set<K> keySet();
	    Collection<V> values();
	    Set<Map.Entry<K, V>> entrySet();
	    interface Entry<K,V> {
	        K getKey();
	        V getValue();
	        V setValue(V value);
	        boolean equals(Object o);
	        int hashCode();
	        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
	        }
	        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
	        }
	        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
	            Objects.requireNonNull(cmp);
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
	        }
	        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
	            Objects.requireNonNull(cmp);
	            return (Comparator<Map.Entry<K, V>> & Serializable)
	                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
	        }
	    }
	    boolean equals(Object o);
	    int hashCode();
	}
```

从上面我们可以看出，在 Map 接口内部，除了定义了一些常用的操作方法之外，还定义了 Map 中键值对的接口 Entry.

### 3.1 HashMap

因为 HashMap 的源码比较重要，而且需要很长的篇幅来进行分析，我们将其作为一个单独的文章：[《Java 基础回顾-6：HashMap 源码分析》](https://blog.csdn.net/github_35186068/article/details/87386841)。

### 3.2 HashTable

相比于HashMap，HashTable的实现就显得简单得多。它内部同样自己定义了一个结点：

```java
    private static class Entry<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Entry<K,V> next;
    }
```

而且也是采用了基于**拉链法**的碰撞处理机制。它定义了一个

```java
    private transient Entry<?,?>[] table;
```

下面是它的根据哈希码计算数组的索引的方法：

```java
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
```

虽然，当插入的元素的数量超过指定的阈值的时候，它也会重新调整数组的大小。但是它的插入操作中是不存在将链表改变成红黑树的优化的。并且这里直接使用了取余的方式获取索引，不存在扰动，从而也不会强制要求容量必须为2的整数次幂。

因为它的散列的均匀性没有 HashMap 调节得那么好，所以它的性能可能会比 HashMap 要差一些。而且，它的每个方法上面都加了 sychronized 关键字进行修饰，这样虽然保证了在多线程环境中的数据一致性，但是，在非多线程环境中，无疑是一种不必要的开销。

### 3.3 TreeMap

TreeMap 是基于红黑树实现的，增删改查都是对红黑树的操作。

## 4、同步容器

### 4.1 Collections.synchronizedXXX() 方法

该方法用来对指定的集合进行装饰，它使用了装饰者模式，我们看下面的一个例子

```java
    Map m = Collections.synchronizedMap(new HashMap(...));
```

这里使用了一个匿名的 HashMap，因为 HashMap 不是线程安全的，但是这封装了之后返回的集合就是线程安全的了。下面我们看该 API 究竟做了什么操作：

它返回了 SynchronizedMap：

```java
    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
        return new SynchronizedMap<>(m);
    }
```

该 Map 的定义是

```java
    private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable {
        private final Map<K,V> m;     // Backing Map
        final Object      mutex;        // Object on which to synchronize

        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }

        public V get(Object key) {
            synchronized (mutex) {return m.get(key);}
        }

        public V put(K key, V value) {
            synchronized (mutex) {return m.put(key, value);}
        }

        // ...
    }
```

在上面的代码中，我们只给出了它的一部分内容的定义，从上面我们就很容易地看出来。它实际上就是对我们定义的 Map 进行了一层包装，我们称其为装饰者模式。它的锁不是加在每个方法上面的，而是使用了粒度更小的**私有锁**——对每个操作，对一个全局的实例进行加锁。这样能够保证每个 `put()` 和 `get()` 方法本身是原子的和线程安全的，而实际的值的获取和插入，还是对原始的容器进行操作的。


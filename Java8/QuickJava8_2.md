# 五分钟学习Java8的流编程

## 1、概述

1. Java8中在Collection中增加了一个stream()方法，该方法返回一个Stream类型。我们就是用该Stream来进行流编程的；
2. 流与集合不同，流是只有在按需计算的，而集合是已经创建完毕并存在缓存中的；
3. 流与迭代器一样都只能被遍历一次，如果想要再遍历一遍，则必须重新从数据源获取数据；
4. 外部迭代就是指需要用户去做迭代，内部迭代在库内完成的，无需用户实现；
5. 可以连接起来的流操作称为中间操作，关闭流的操作称为终端操作（从形式上看，就是用`.`连起来的操作中，中间的那些叫中间操作，最终的那个操作叫终端操作）。

## 2、筛选

### 2.1 过滤

    Stream<T> filter(Predicate<? super T> predicate);

filter通过指定一个Predicate类型的行为参数对流中的元素进行过滤，最终还是会返回一个流，因为它是中间操作。中间操作返回的结果都是一个流，所以，如果我们想要得到一个集合或者其他的非流类型，就需要使用终端操作来获取。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<Integer> filter = list.stream().filter(integer -> integer > 3).collect(Collectors.toList());
    // [4, 5, 5, 6, 7, 8, 9]
	
### 2.2 去重

    Stream<T> distinct();
	
上面就是去重的方法的定义，它会按照流中的元素的equal()和hashCode()方法进行去重。去重之后将继续返回一个流，所以它也是中间操作。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<Integer> filter = list.stream().filter(integer -> integer > 3).distinct().collect(Collectors.toList());
	// [4, 5, 6, 7, 8, 9]
	
### 2.3 限制

    Stream<T> limit(long maxSize);

就像是SQL里面的limit语句，在流中也有类似的limit()方法。它用于限制返回的结果的数量，将会从流的头开始取固定数量的元素，也是中间操作，使用完之后仍然会返回一个流。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<Integer> filter = list.stream().filter(integer -> integer > 3).limit(3).collect(Collectors.toList());
    // [4, 5, 5]
	
### 2.4 跳过

    Stream<T> skip(long n);

这个方法的定义和limit()几分相似。它也是中间操作，用于跳过从流的头开始指定数量的元素，使用完之后仍然会返回一个流。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<Integer> filter = list.stream().filter(integer -> integer > 3).skip(3).collect(Collectors.toList());
	// [6, 7, 8, 9]
	
## 3、映射

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

还记得Function函数接口的方法吗？它允许你把输入的类型转换成另一种类型。上面就是它在map()方法中的应用。在流操作中使用了该方法之后，流就会尝试将当前流中所有的元素转换成另一种类型。当你调用终端操作collect()的时候，自然也就得到了另一种类型的集合。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<String> filter = list.stream().map((integer -> String.valueOf(integer) + "-")).collect(Collectors.toList());
    // 结果：[1-, 1-, 2-, 3-, 4-, 5-, 5-, 6-, 7-, 8-, 9-]

## 4、查找

    Optional<T> findFirst();
    Optional<T> findAny();

在指定的流中查找元素的时候可以用这两个方法，它们是Stream接口中的方法，返回的已经不再是Stream类型了，这可以说明它们是终端操作。所以，通常也是用来放在终端，继续操作的话就要使用Optional接口的方法了。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    Optional<Integer> optionalInteger = list.stream().filter(integer -> integer > 10).findAny();
    Optional<Integer> optionalInteger = list.stream().filter(integer -> integer > 10).findFirst();

上面是使用的两个示例，这里返回的结果是Optional类型的。Optional的设计借鉴了Guava中的Optional。使用它的好处是你不需要像以前一样将返回的结果与null进行判断，并在结果为null的时候通过`=`赋值一个默认值了。使用Optional中的方法，你可以更优雅地完成相同的操作。下面我们列出Optional中的一些常用的方法：

|编号|方法|说明|
|:-:|:-:|:-:|
|1|isPresent()|判断值是否存在，存在的话就返回true，否则返回false|
|2|isPresent(Consumer<T> block)|在值存在的时候执行给定的代码|
|3|T get()|如果值存在，那么返回该值；否则，抛出NoSuchElement异常|
|4|T orElse(T other)|如果值存在，那么返回该值；否则，则返回other|
	
## 5、匹配

    boolean allMatch(Predicate<? super T> predicate);
    boolean noneMatch(Predicate<? super T> predicate);
    boolean anyMatch(Predicate<? super T> predicate);
	
从定义上面来看，上面的三个方法也是终端操作。它们分别用来判断：流中的数据是否全部匹配指定的条件，流中的数据是否全部不匹配指定的条件，流中的数据是否存在一些匹配指定的条件。下面是一些示例：
	
    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    boolean allMatch = list.stream().allMatch(integer -> integer < 10);
    boolean anyMatch = list.stream().anyMatch(integer -> integer > 3);
    boolean noneMatch = list.stream().noneMatch(integer -> integer > 100);

## 6、归约

    Optional<T> reduce(BinaryOperator<T> accumulator);
    T reduce(T identity, BinaryOperator<T> accumulator);

Stream接口中的reduce方法共有三个重载版本，上面我们给出常用的两个的定义。它们基本是类似的，只是第二个方法参数列表中多了个初始值，而没有初始值的那个，返回了Optinoal类型；所以，区别不大，我们只要搞明白它的行为就可以了。下面是归约的例子：

    List<String> list = Arrays.asList("a", "b", "c", "d", "e", "f");
    String ret = list.stream().reduce("-", (a, b) -> a + b);

它的输出结果是`-abcdef`，显然它的效果就是：假如，`$`是某种操作，List是某个"数列"，那么归约的意义就是`初始值$n[0]$n[1]$n[2]$...$n[n-1]`。





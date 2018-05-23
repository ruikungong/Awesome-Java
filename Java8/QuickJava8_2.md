# 五分钟学习Java8的流编程

## 1、概述

1. Java8中在Collection中增加了一个stream()方法，该方法返回一个Stream类型。我们就是用该Stream来进行流编程的；
2. 流与集合不同，流是只有在按需计算的，而集合是已经创建完毕并存在缓存中的；
3. 流与迭代器一样都只能被遍历一次，如果想要再遍历一遍，则必须重新从数据源获取数据；
4. 外部迭代就是指需要用户去做迭代，内部迭代在库内完成的，无需用户实现；
5. 可以连接起来的流操作称为中间操作，关闭流的操作称为终端操作（从形式上看，就是用`.`连起来的操作中，中间的那些叫中间操作，最终的那个操作叫终端操作）。

## 2、筛选

### 2.1 按条件过滤

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
	
### 2.3 限制返回结果

    Stream<T> limit(long maxSize);

就像是SQL里面的limit语句，在流中也有类似的limit()方法。它用于限制返回的结果的数量，将会从流的头开始取固定数量的元素，也是中间操作，使用完之后仍然会返回一个流。

    List<Integer> list = Arrays.asList(1, 1, 2, 3, 4, 5, 5, 6, 7, 8, 9);
    List<Integer> filter = list.stream().filter(integer -> integer > 3).limit(3).collect(Collectors.toList());
    // [4, 5, 5]
	
### 2.4 跳过元素

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





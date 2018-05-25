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

## 7、数值流

同样是因为装箱的性能原因，Java8中为数值类型专门提供了数值流：IntStream DoubleStream和LongStream。Stream接口提供了三个中间方法来完成从任意流映射到数值流的操作：

    IntStream mapToInt(ToIntFunction<? super T> mapper);
    LongStream mapToLong(ToLongFunction<? super T> mapper);
    DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
	
所以你可以用上面三个方法从任意流中获取数值流。然后，再利用数值流的方法来完成其他的操作。上面三个数值流和Stream接口都继承子BaseStream，所以它们包含的方法还是有区别的，但总体上来说大同小异。Stream比较具有一般性，上面三个数值流更有针对性，后者也提供了许多便利的方法。如果想要从数值流中获取对象流，你可以调用它们的`boxed()`方法，来获取装箱之后的流。

这里稍提及一下，对于Optional，Java8也为我们提供了对应的数值类型：OptionalInt OptionalDouble OptionalLong。

在上面的三种数值流中还有几个静态方法用于获取指定数值范围的流：

    public static LongStream range(long startInclusive, final long endExclusive)
    public static LongStream rangeClosed(long startInclusive, final long endInclusive)

上面是用于获取指定范围的LongStream的方法，一个对应于数学中的开区间，一个对应于数学中的闭区间的概念。
	
## 8、构建流

上面我们在获取流的时候，实际上都是从Collection的默认方法`stream()`中获取的流，这有些笨拙。实际上，Java8为我们提供了一些创建流的方法。这里，我们列举一下这些方法：

    public static<T> Builder<T> builder() // 1
    public static<T> Stream<T> empty() // 2
    public static<T> Stream<T> of(T t) // 3
    public static<T> Stream<T> of(T... values) // 4
    public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f) // 5
    public static<T> Stream<T> generate(Supplier<T> s) // 6 
    public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b) // 7

上面的方法都是Stream接口中的静态方法，我们可以用这些方法来获取到流。下面我们对每个方法做一些简要的说明：

1. 从名称上就可以看出这里使用了构建者模式，你可以每次调用Builder的`add()`方法插入一个元素来创建流；
2. 用来创建一个空的流
3. 创建一个只包含一个元素的流
4. 使用不定参数创建一个包含指定元素的流
5. 弄清楚它的原理关键是要搞明白后面的UnaryOperator的含义，这是一个函数式接口，并且继承自Function，不同之处在于它的入参和回参类型相同。这个方法的原理是从某个种子值开始，按照后面的函数的规则进行计算，每次是在之前的值的基础上执行某个函数的。所以`Stream.iterate(2, n -> n * n).limit(3)`将返回由`2 4 16`构成的流。
6. 这里的Supplier也是一个函数接口，它只有一个get()方法，无参，只接受指定类型的返回值。所以，这个方法需要你提供一个用于生成数值的函数（或者说规则），比如Math.random()等等。
7. 这个比较容易理解，就是通过将两个流合并来得到一个新的流。
	
## 9、收集器

上面我们已经见识过了流的规约操作，但是那些操作还比较幼稚。Java8的收集器为我们提供了更加强大的规约功能。

说起收集器，肯定绕不过两个类Collector和Collectors，它俩有啥关系呢？其实Collector只是一个接口；Collectors是一个类, 其中的静态内部类CollectorImpl实现了该接口，并且被Collectors用来提供一些功能。Collectors中有许多的静态方法用于获取Collector的实例，使用这些实例我们可以完成复杂的功能。当然，我们也可以通过实现Collector接口来定义自己的收集器。

Stream的collect()方法有3个重载的版本。我们就是通过其中的一个来使用收集器的，这是它的定义：

    <R, A> R collect(Collector<? super T, A, R> collector);
	
我们注意一下这个方法的参数和返回类型. 从上面我们可以看出传入的Collector有3个泛型,其中的最后一个泛类型R与返回的类型是一致的. 这很重要——可以预防你调用了某个方法却不知道最终返回的是什么类型。

我们先来看一些简单的例子，这里的stream是由Student对象构成的流：

    Optional<Student> student = stream.collect(Collectors.maxBy(comparator))  // 需要传入一个比较器到maxBy()方法中
    long count = stream.collect(Collectors.counting())

上面的两种方式比较鸡肋，因为你可以使用count()和max()方法来替代它们。下面我们再看一些收集器的其他例子，注意在这些例子中，我并没有使用lambda简化函数式接口，是因为想要你更清楚地看到它的泛类型和方法定义。这可能有助于你理解这些方法的作用机理。
	
### 9.1 计算平均值和总数

下面的语句用于计算平均值，类似的还有summingInt()用于计算总数。它们的用法是相似的。
	
    Double d = stream.collect(Collectors.averagingInt(new ToIntFunction<Student>() {
        @Override
        public int applyAsInt(Student value) {
            return value.getGrade();
        }
    }));
 
从上面我们看出，调用averagingInt()方法的时候需要传入一个ToIntFunction函数式接口，用于根据指定的类型返回一个整数值。

### 9.2 连接字符串

joining()工厂方法是专门用来连接字符串的，它要求流是字符串流，所以在对Student流进行拼接之前，需要先将其映射成字符串流：

    String members = stream.map(new Function<Student, String>() {
        @Override
        public String apply(Student student) {
           return student.getName();
        }
    }).collect(Collectors.joining(", ")); // 使用','将字符串拼接起来
	
### 9.3 广义的规约汇总

    Optional<Student> optional = stream.collect(Collectors.reducing(new BinaryOperator<Student>() {
        @Override
        public Student apply(Student student, Student student2) {
            return student.getGrade() > student2.getGrade() ? student : student2;
        }
    }));

上面的就是用来规约的函数。我们用了reducing工厂方法，并向其中传入一个BinaryOperator类型。这里我们指定最终的返回类型是Student。所以，上面的代码的效果是获取成绩最大的学生。

### 9.4 分组

Collectors中的分组还是比较有意思的。我们先看groupingBy方法的定义：

    Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier)
    Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier, Collector<? super T, A, D> downstream)

groupingBy方法有3个重载的版本，这里我们给出其中常用的两个。第一个方法是通过指定规则对流进行分组的，而第二个方法先通过classifier指定的规则对流进行分组，然后用downstream的规则对分组后的流进行后续的操作。注意第二个参数仍然是Collector类型，这说明我们仍然可以对分组后的流再次收集，比如再分组、求最大值等等。

    Map<Integer, List<Student>> map = stream.collect(Collectors.groupingBy(new Function<Student, Integer>() {
        @Override
        public Integer apply(Student student) {
           return student.getClazz();
        }
    }));
	
以上是groupingBy()方法的第一个例子。注意这里我们是通过将Student通过'班级字段'映射成一个整数来进行分组的。下面是一个二次分组的例子。这里的用了上面的第二个groupingBy()方法，并在downstream中指定了另一个分组操作。

    Map<Integer, Map<Integer, List<Student>>> map = stream.collect(Collectors.groupingBy(new Function<Student, Integer>() {
        @Override
        public Integer apply(Student student) {
           return student.getClazz();
        }
    }, Collectors.groupingBy(new Function<Student, Integer>() {
        @Override
        public Integer apply(Student student) {
            return student.getGrade() == 100 ? 1 : student.getGrade() > 90 ? 2 : student.getGrade() > 80 ? 3 : 4;
        }
    })));
	
### 9.5 分区

与分组类似的还有一个分区的操作，分区只是分组的一种特例。它们的使用方式也基本一致，它的方法签名与上面的groupingBy方法类似。我们直接看它的一个使用的方式好了：

    Map<Boolean, List<Student>> map = stream.collect(Collectors.partitioningBy(new Predicate<Student>() {
        @Override
        public boolean test(Student student) {
            return student.getGrade() > 90;
        }
    }));

这就是分区的使用方式。它通过一个指定的函数式接口，将指定的类型映射到一个布尔类型。所以，它类似与分组，只不过它分组的结果只有两种，要么true，要么false。当然，类似于分组，你也可以在partitioningBy()方法的第二个参数中再指定一个收集器，这样就可以对分区后的流进行后续的操作了。

## 总结：

以上就是Java8中的流的常见的用法，这里只是列举了一些常见的、Java8 API中提供的一些类和方法。重点仍然是搞清楚其中的设计的原理，不要盲目记忆。学习的时候结合JDK源码进行，看到方法的定义就大致了解了它的设计原理。最后，不得不说的是，使用流编程确实很简洁和优雅。
 
相关代码：
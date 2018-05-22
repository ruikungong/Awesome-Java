# Quick Java 8 （一）

## 1、概览

Java8的改进比历史上任何一次改变都比较深远。Java在改进其实也是编程语言生态变化的原因。诸如大数据需要在多核上面运行，而Java此前是不支持这种操作的。

在Java8之前，如果想要利用多个计算机的内核，你要使用线程，并且要处理复杂的同步逻辑。但是在Java8中，你可以很容易地使用流让自己的代码在多个内核上面执行。

此外，它还借鉴了其他语言和开源库的内容，比如Scala、Guava等。我们总结一下Java8的主要几个改进：

1. 函数式编程和Lambda表达式；
2. 流(Stream)编程；
3. 时间API的改进；
4. 默认方法

## 2、行为参数化

“行为”就是指方法，“行为参数化”就是指将方法作为参数传入，说白了就是指**策略模式**。而Java8只是使用了Lambda表达式简化了匿名类的代码，使匿名类看起来更加简洁。在这块内容上，你所需要掌握的东西并不多。

### 2.1 Lambda表达式基本语法

    (parameters) -> expression     // 表达式
    (parameters) -> {statements;}  // 语句，语句尾带分号的且要用花括号括起来

上面是Lambda的基本语法，第一行中是Lambda中使用表达式的情况，第二行中式Lambda中使用语句的情况，只要注意一下语句需要用花括号括起来即可。

下面是一些接口的定义，以及使用Lambda表达式

    public static void main(String...args) {
	    // 创建对象
        ICreateObject createObject = Employee::new;
        IExpression expression = employees -> employees.get(0);
        // 可以进一步简化为 IExpression expression2 = List::isEmpty;
        IExpression expression2 = employees -> employees.isEmpty(); 
        IConsumeObject consumeObject = employee -> System.out.println(employee.name);
        IAdd add = (a, b) -> a + b;
        IAdd add1 = Java8LambdaExample::cal;
        // 会报出不是Function接口异常
        //        Object object = Employee::new;
    }

    private static int cal(int a, int b) {
        return a + b;
    }
	
	@FunctionalInterface
    public interface ICreateObject {
        Employee create();
		default void method() {}
    }

    public interface IExpression {
        void filter(List<Employee> employees);
    }

    public interface IConsumeObject {
        void consume(Employee employee);
    }

    public interface IAdd {
        int add(int a, int b);
    }

    private static class Employee {
        String name;
    }

从上面的示例代码，我们可以总结出一些结论：

1. 所谓的函数接口就是指只包含一个非默认方法的接口，可以用`@FunctionalInterface`注解标明指定的接口是函数接口；
2. 如果Lambda中的`->`后面的是语句，并且当该语句只有一行的时候，我们可以将花括号去掉；
3. 想要将Lambda表达式赋值给一个对象的时候，如果这个对象不是`函数接口`，那么IDEAs会给提示；
4. 还要注意函数式接口是不允许抛出受检异常的。

下面我们总结一些常见的方法引用的示例：

上面代码中的`Employee::new`就是所谓的方法引用，下面是常见的方法引用的例子：

|编号|Lambda|等效的方法引用|
|:-:|:-:|:-:|
|1|`(Employee e)->e.getName()`|`Employee::getName`|
|2|`(String s) -> System.out.println(s)`|`System.out::println`|
|3|`(str, i) -> str.substring(i)`|`String::substring`|

所以，我们总结下来的三种方法引用的情形

|编号|Lambda|等效的方法引用|
|:-:|:-:|:-:|
|1|`(参数) -> 类名.静态方法(参数)`|`类名::静态方法`|
|2|`(参数1, 其他参数) -> 参数1.实例方法(其他参数)`|`类名::实例方法`|
|3|`(参数) -> 表达式.实例方法(参数)`|`表达式::实例方法`|

### 2.2 Java API 中的函数式接口

Java8的API中为我们提供了几个函数式接口，这些接口有必要了解一下。因为自从Java8开始接口可以定义默认方法了，所以这些接口里面又提供了一些有意思的默认方法。这可能对我们编程比较有帮助。

    public interface Predicate<T> {
        boolean test(T t);
	}
	
    public interface Consumer<T> {
        void accept(T t);
    }

    public interface Function<T, R> {
        R apply(T t);
    }

上面就是这三个接口的定义。你只要记下它们只有返回的值是不同的就可以了。第一个用来判断的，大致用来实现过滤的效果；第二是没有返回类型，只能用来对传入的参数进行处理；第三个是用来映射的，也就是说，当你想要实现的行为的参数和返回是不同的类型的时候可以用它。

因为对于数值类型，Java需要做额外的装箱和拆箱的操作，这是需要成本的。所以，对于上面的三个接口（其他的接口也是），Java8中提供了不需要装箱的版本，也就是从泛型变成了数值类型而已。以IntPredicate为例吧：

    public interface IntPredicate {
        boolean test(int value);
    }

### 2.3 复合Lambda表达式
	
Java8中提供的一些接口还是可以复合操作的。使用复合操作可以实现更复杂的逻辑。这些复合操作时以默认方法的形式定义的，每个函数式接口略有不同。所以，我们这里只列举出部分用于复合的方法。在实际的开发过程中，你可以直接进入到指定的函数式接口中查看这些方法的定义。

#### 2.3.1 比较器Comparator<T>

假设有一个数据列表employees，其中的对象是Employee，它有getName()和getAge()两个方法。

    employees.sort(Comparator.comparing(Employee::getName));
    employees.sort(Comparator.comparing(Employee::getName).reversed().thenComparing(Employee::getAge));

上面的两行代码中，第一行实现对employees按照getName()的结果进行排序。第二行代码对employees，先按照getName()的结果进行排序，然后将返回的结果逆序，再按照getAge()的结果进行排序。

#### 2.3.2 谓词复合 negate()、or()和and()

    Predicate<Employee> employeePredicate = (employee -> employee.getAge() > 13)
	employeePredicate.negate()
    employeePredicate.and(employee -> employee.getAge() <= 15).or(employee -> "LiHua".equals(employee.getName()))

这里首先定义了employeePredicate，它可以用来过滤“年龄大于13的雇员”。对其调用了negate()方法将返回一个Predicate，可以用来过滤“年纪小于等于13的雇员”。然后，最后的复合操作表示“年龄大于13并且小于15的雇员或者名字为LiHua的雇员”。注意，这里的and和or操作的顺序是从左向右的，也就是`a.or(b).and(c)`被看作`(a || b) and c`。
	
#### 2.3.3 函数Function<T>复合
	
Function有andThen和compose两个默认方法，它们都会返回一个Function实例。

    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    Function<Integer, Integer> h1 = f.andThen(g); // h1(x) = g(f(x)) = (x + 1) * 2
    Function<Integer, Integer> h2 = f.compose(g); // h2(x) = f(g(x)) = (x * 2) + 1
    System.out.println(h1.apply(1));
    System.out.println(h2.apply(1));

上面是Function的复合操作的示例，其实它的效果就相当于数学中的复合函数。不过，应当注意一下两个方法的实际的复合效果。
	
	
# 1、Java 基础回顾：基本知识总结

> 本篇文章对 Java 中的语言相关的基础知识进行了总结，其中部分内容参考了阿里的 Java 开发规范

## 1、术语

|No|结论|
|:-:|:-|
|1|**JDK** 是编写Java程序的程序员使用的软件|
|2|**JRE** 是运行Java程序的用户使用的软件|
|3|Java的执行模式是**编译和解释型**，Java程序首先由编译器转换为标准字节代码，然后由JVM来解释执行，**JVM**将字节代码程序同操作系统和硬件分开，使Java程序独立于平台运行|
|4|**垃圾回收机制**是一个系统线程，对内存使用进行跟踪|
|5|Class 类对象由 Java 编译器自动生成，隐藏在 .class 文件中|
|6|JVM 运行 Java 代码的步骤: **装载-链接-初始化**（运行时不需要加载代码）|
|7|**动态绑定**: 在运行时能够自动地选择调用哪个方法的现象|
|8|**Jar** 是一种压缩的文件，使用了 ZIP 格式，其中包含一个清单文件 MANIFEST.MF 位于 META-INF 目录|
|9|**JavaScript** 是网页中使用的脚本语言，语法类似于Java，除此之外无任何联系|
|10|Java 大小写敏感|

## 2、Java程序的基本结构

### 程序结构

	package me.shouheng;
	
	import java.util.ArrayList;
	import java.util.List;
	
	public class Test {
	    public static void main(String ...args) {
	        System.out.println("Hello world!");
	        List<String> s = new ArrayList<String>();
	    }
	}

简单说明：

|No|说明|
|:-:|:-|
|1|main 方法用于控制程序的开始和结束，main 方法包含以下要素：`public`、`static`、`void` 和 `String[] args`|
|2|可以使用 `new` 关键字创建类的实例对象，可以直接通过类名调用类所提供的静态成员|

### 包

|No|说明|
|:-:|:-|
|1|包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。【规范】|
|2|`package` 语句必须是源程序文件中第一个非注释、非空白语句|
|3|包的成员可以包含子包、类、接口。如果源代码中没有指定包，则使用默认包|
|4|一个包（子包）的成员不能重名，即使不同的类型|

包的访问方式：

1. 完全限定名称方式访问：`java.net.InetAddress hostAdd`
2. 导入包成员：`import java.net.InetAddress`
3. 导入整个包：`import java.net.*`
4. 导入**类**的静态成员：`import static java.lang.Math.*`
5. 访问包成员名称冲突：需要使用完全限定名称方式

## 3、标识符

由**字符、数字、下划线、美元符号($)组成，且开头不能为数字，分大小写，没有长度限制，不能与关键字相同**. 约定如下

|No|元素|约定|
|:-:|:-|:-|
|1|包|常用名词，全部小写|
|2|类、接口|常用名词，每个单词首字母大写；（PascalCase规则）|
|3|方法|常用动词，首字母小写；（camelCase规则）|
|4|常量|常用名词，全部大写，单词之间用下划线(_)分隔|
|5|变量|成员名称，首字母小写. （camelCase规则）|

规范：

|No|规范|
|:-:|:-|
|1|【强制】代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束|
|2|【强制】代码中的命名严禁使用拼音与英文混合的方式，更不允许直接使用中文的方式|
|3|【强制】类名使用 UpperCamelCase 风格，必须遵从驼峰形式，但以下情形例外：DO/BO/DTO/VO/AO|
|4|【强制】常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长|
|5|【强制】抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类命名以它要测试的类的名称开始，以Test结尾|
|6|【强制】POJO 类中布尔类型的变量，都不要加is ，否则部分框架解析会引起序列化错误|
|7|【推荐】如果使用到了设计模式，建议在类名中体现出具体模式|
|8|【推荐】 如果是形容能力的接口名称，取对应的形容词做接口名（ 通常是–able的形式）|
|9|【强制】对于 Service 和 DAO 类，基于 SOA 的理念，暴露出来的服务一定是接口，内部的实现类用 Impl 的后缀与接口区别|

此外，Service / DAO 层方法命名规约

1. 获取单个对象的方法用 `get` 做前缀。
2. 获取多个对象的方法用 `list` 做前缀。
3. 获取统计值的方法用 `count` 做前缀。
4. 插入的方法用 `save`（ 推荐 ） 或 `insert` 做前缀。
5. 删除的方法用 `remove`（ 推荐 ） 或 `delete` 做前缀。
6. 修改的方法用 `update` 做前缀。

## 4、程序流程

### 4.1 基本程序结构

1. 顺序结构
2. 选择结构：`if` 语句和 `switch` 语句
3. 循环结构：`for` 循环、`while` 循环、`do...while` 循环、`for each` 循环
4. 跳转语句：`break` 语句、`continue` 语句、`return` 语句

#### 两个需要注意的地方

1. 在 Java 中 `for each` 循环体语句序列中，数组或集合的元素是只读的，其值不能改变，若要改变可用常规 `for` 循环.
2. 应该强制在使用 `for` 循环的时候加上 `{}`。下面的程序示例：

假如有一段程序：

    for (int i=0;i<10;i++) int k = 5;
    for (int i=0;i<10;i++) {int k = 5;}

上面的两种写法：第一种是错误的，`for` 循环可以不加 `{}`，但是那仅限于执行语句，不能包含变量声明语句。第一段代码定义了 `k`，它的作用域是包含 `for` 循环的整个方法，因此会造成重复定义的编译错误。

#### foreach

`foreach` 循环使用了**迭代器设计模式**，它的基本使用方法：如果想要自己的数据集合能够使用 foreach 的方式遍历集合的元素，那么我们需要自己的容器实现 Iterable 接口。下面是在为背包变成可迭代的列子：

	public class Bag<E> implements Iterable<E> {
	    private Node<E> first;
	
	    private static class Node<E> {
	        E element;
	        Node<E> next;

	        Node(E element, Node<E> next) {
	            this.element = element;
	            this.next = next;
	        }
	    }
	
	    public void add(E element) {
	        first = new Node<E>(element, first);
	    }
	
	    public Iterator<E> iterator() {
	        return new ListIterator();
	    }
	
	    private class ListIterator implements Iterator<E> {
	        private Node<E> current = first;
	
	        @Override
	        public boolean hasNext() {
	            return current != null;
	        }
	
	        @Override
	        public E next() {
	            E element = current.element;
	            current = current.next;
	            return element;
	        }
	
	        @Override
	        public void remove() {}
	    }
	}

### 4.2 异常控制结构

异常为我们提供了一种解决问题的机制，有了它我们就可以实现将业务逻辑和异常处理相分离，从而使我们的代码更加易于维护和理解。

#### Java 异常的结构层次

![Java异常的结构层次](res/exceptions.png)

1. 所有的异常类是从 `java.lang.Exception` 类继承的子类；
2. `Exception` 类是 `Throwable` 类的子类。除了 `Exception` 类外，`Throwable` 还有一个子类 `Error`，用来指示运行时环境发生的错误，我们最好不要在程序中定义 `Error` 的子类；
3. Java 程序通常不捕获错误。错误一般发生在严重故障时，它们在 Java 程序处理的范畴之外；
4. 异常类有两个主要的子类：`IOException` 类和 `RuntimeException` 类；
5. Java 提供了三种可抛出结构：受检异常、运行时异常和错误；
6. 每个抛出的异常都要有文档。

对于使用受检的异常还是未受检的异常的原则：**对于可恢复的情况，使用受检的异常；对于程序错误，使用运行时异常。**

运行时异常（可以参考上图）通常意味着程序不符合API规范，继续运行下去也无济于事。而受检异常则意味着，对出现了异常进行正确的处理之后，程序仍然能够继续执行。
不过这只是一种规范，但实际上不论运行时异常还是受检异常都是可以捕获并处理的。

#### 异常处理程序的结构

    try {
        int k = 1/0;
    } catch (ArithmeticException e) {
        e.printStackTrace();
    } finally {
        System.out.println("finally");
    }

1. 异常处理系统会按照“就近”原则匹配异常，如果满足了匹配就不再继续向下匹配。因此，在 `catch` 语句中捕获异常的时候，要按照从子类到基类的顺序捕获（从具体错误到顶层错误）。
2. **异常链返回**：`finally` 语句总是会执行，所以在一个方法中可以从多个点返回，却仍然能够保证清理工作的进行。

一个异常终止的示例程序：

    public static void main(String ...args) {
        for (int i=1; i>=0; i--) {
            System.out.println(f(i));;
        }
    }

    private static int f(int i) {
        try {
            int k = 1 / i;
            return 0;
        } catch (ArithmeticException e) {
            System.out.println("catch");
            return 1;
        } finally {
            try {
                Thread.sleep(1000);
                System.out.println("finally");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

以上程序的输出结果是：
	
	finally
	0
	catch
	finally
	1

可见，`finally` 总是会被调用，而且 `return` 语句要等待 `finally` 执行完毕才会调用。

#### 异常使用指南

建议使用异常的情况：

1. 异常应该只用于异常的情况；它们永远不应该用于控制正常的控制流。
1. 如果在 `try` 语句块中已经出现了异常，而我们在 `finally` 语句块中进行异常处理的时候又抛出了异常，或者使用了 `return` 语句，那么这个时候我们的第一个抛出的异常会丢失。
丢失的后果就是当程序出现了错误的时候，我们没有办法对错误发生的位置进行定位。
1. 在恰当的级别处理问题（知道该如何处理的情况下捕获异常）；
2. 解决问题并且重新调用产生异常的方法；
3. 进行少许修补，然后绕过异常发生的地方继续执行；
4. 用别的数据进行计算，以代替方法预计会返回的值；
5. 把当前运行环境下能做的事情尽量做完，然后把相同的异常抛到高层；
6. 把当前运行环境下能做的事情尽量做完，然后把不同的异常抛到高层；
7. 终止程序；
8. 进行简化；
9. 让类库和程序更安全。

## 5、运算符

### 5.1 运算符（基本同C，特别说明几个）

|No|运算符|说明|
|:-:|:-|:-|
|1|自增(`++`)/自减(`--`)运算符|注意它本质上是分3步执行的(所以非原子的)，运算符放在前面和后面会有区别|
|2|算术运算符(`+、-、*、/`)|当参与 `/` 运算的两个操作数都是表示整数除法，否则是浮点数除法|
|3|比较运输符(`>、<、>=、<=、!=、==`)|参考总结1|
|4|字符串运算符(`+`)|运算符用于串联两个字符串，对非字符串类型的通过调用从Object类继承的toString()方法将其转换为字符串形式|
|5|赋值运算符(`+=  -=  *=  /=  %=  <<=  >>=  &=  &#124;=  ^=`)|要求右边的操作数可以隐式地转换为左边的操作数或类型相同|

### 5.2 总结1：关于比较运算符

|No|总结|
|:-:|:-|
|1|形如 `a<b<c` 的复合比较式要写成 `(a<b)&&(b<c)` 而不能是 `a<b<c`|
|2|简单数据类型比较的是它的值|
|3|引用类型比较的是两个引用是否指向同一对象实例，即是否指向同一内存位置|
|4|引用类型的比较中，参与比较的两个引用类型必须能够转换为同一引用类型|
|5|`null` 常量表示非空对象，可以与任何引用类型变量比较|
|6|如果参与比较的两个操作数，一个为常量，则采用值比较方式，而非引用地址|
|7|引用类型的比较可以使用对应类的 `equals(Object obj)` 方法实现|
|8|比较运算符具有截断性，如果前面的能够得出表达式的结果就不再继续比较了，比如 `if(obj != null && ebj.equals(obj2))`，这里如果 ebj 是 null 的话是不会进行 equals 比较的|
|9|`equals()` 方法默认比较引用，要想实现自己的比较逻辑必须覆写该方法。同时要注意 `equals()` 和 `hashCode()` 方法需要同时覆写|
|10|对浮点数的比较是非常严格的，即使一个数仅在小数部分与另一个数存在极微小的差异，仍然认为它们是不相等的。即使一个数比零大一点点，它仍然是非零的值|

### 5.3 总结2：关于字符串运算符

1. 错误的例子 `String s0 = true + false + “abc”;` // 错误(true+false)运算错误
2. 对于字符串，`+` 运算是每次创建一个 `StringBuilder` 对象，并调用它的 `append` 方法添加字符串，所以使用`StringBuilder` 比使用 `+` 的效率更高

### 5.4 总结3：关于移位

 `>>` 和 `>>>` 的区别：

    System.out.println(Integer.toBinaryString(Integer.MAX_VALUE));         // 输出结果：1111111111111111111111111111111
	System.out.println(Integer.toBinaryString(Integer.MAX_VALUE >> 10));   // 输出结果：111111111111111111111
    System.out.println(Integer.toBinaryString(Integer.MAX_VALUE >>> 10));  // 输出结果：111111111111111111111
    System.out.println(Integer.toBinaryString(-1));                        // 输出结果：11111111111111111111111111111111
    System.out.println(Integer.toBinaryString(-1 >> 10));                  // 输出结果：11111111111111111111111111111111
    System.out.println(Integer.toBinaryString(-1 >>> 10));                 // 输出结果：1111111111111111111111
    System.out.println(-1 >> 1);                                           // 输出结果：-1
    System.out.println(-1 >>> 1);                                          // 输出结果：4194303

1. `32>>32` 相当于 `32>>0`，而 `32>>33` 相当于`32>>1`；
3. 左移 (`<<`) 能按照操作符左侧指定的位数将操作符左边的操作数向左移动（在低位补0）；
4. “有符号”右移操作符(>>)按照操作符右侧指定的位数将从操作符左边的操作数向右移动；
5. “有符号”右移操作符使用 “符号拓展”：若符号位为证，则在高位插入0，若符号位为负，则在高位插入1；
6. “无符号”右移操作符(`>>>`)，无论正负都在高位插入0.

### 5.5 问题

#### 5.5.1 问题1：以下程序的输出结果是？

    int j = 0;
    for (int i = 0; i < 100; i++) {
        j = j++;
    }
    System.out.println(j);

输出结果是 0，这是因为 `j=j++` 操作等价于 `int temp=j; j=j+1; j=temp;`

#### 5.5.2 问题2: 以下两个表达式，哪个正确？

    short s = 1; s = s + 1;
    short s = 1; s += 1;

第二个，因为 `s+1` 为 int 型，不能赋值给 short 类型。第二个表达式中会被强制转型为 short 类型。

#### 5.5.3 问题3：以下程序的输出结果是

    char x = 'x';
    int i = 10;
    System.out.println(false ? i : x);
    System.out.println(false ? 10 : x);

120 和 x，这是因为第一个输出中，i 为 int 类型，第一个输出被提升为 int 型；第二个输出中，10 是常量，常量 10 可以被 char 表示，故输出 char 型。

## 6、基本数据类型

1. Java中的数据类型分成：**引用类型和简单类型** 两个大类；
2. 引用类型包括：**类、接口、数组和 null 类型**；
3. 简单类型包括：**数值类型和布尔类型**。

### 6.1 整数

#### 6.1.1 取值范围

Java 中整型的范围与运行 Java 的机器无关。它只有有符号的整数，没有无符号的。下面是它们的范围：

|数据类型|字节|范围|
|:-|:-:|:-|
|byte|1 字节|-128 ~ 127|
|short|2 字节| -32768 ~ 32767 （正负3万）|
|int|4 字节|-2 147 483 648 ~ 2 147 483 647 （正负20亿）|
|long|8 字节|-9 223 372 036 854 775 808 ~ 9 223 372 036 854 775 807|

其他：

|No|总结|
|:-:|:-|
|1|若要表示 long 类型需要在数字后加 L 或 l，同样的还有 float 和 double|
|2|二进制前加 0b，八进制前加 0，十六进制前加 0x 或 0X|
|3|Java 中没有 unsigned 类型|
|4|1_000_000 即一百万，可以加下划线，使数字更易读，没有实际意义|

#### 6.1.2 整数的取值范围与二进制表示

Java 中用补码表示二进制数，补码的最高位是符号位，最高位为 “0” 表示正数，最高位为 “1” 表示负数。

正数补码为其本身；负数补码为其绝对值各位取反加 1. 例如：

    +21，其二进制表示形式是 00010101，则其补码同样为 00010101
    -21，按照概念其绝对值为 00010101，各位取反为 11101010，再加 1 为 11101011，即 -21 的二进制表示形式为 11101011

计算个整数类型的取值范围的步骤：

- Step 1：byte 为一字节 8 位，最高位是符号位，即最大值是 01111111，因正数的补码是其本身，即此正数为 01111111，十进制表示形式为 127
- Step 2：最大正数是 01111111，那么最小负是 10000000 (最大的负数是 11111111，即 -1)
- Step 3：10000000 是最小负数的补码表示形式，我们把补码计算步骤倒过来就即可。10000000 减 1 得01111111 然后取反 10000000
- Step 4：因为负数的补码是其绝对值取反，即 10000000 为最小负数的绝对值，而 10000000 的十进制表示是 128，所以最小负数是 -128
- Step 4：由此可以得出byte的取值范围是 -128 到 +127
- 另外，整数类型包装对象提供了 `MAX_VALUE` 和 `MIN_VALUE` 两个字段，表示指定类型的最大值和最小值，还有从字符串中获取整数的方法 `valueOf(String)` 等

### 6.2 浮点数

#### 6.2.1 float 和 double

**float**

- float 数据类型是单精度、32位、符合 IEEE 754 标准的浮点数
- 浮点数不能用来表示精确的值，如货币
- 例子：float f1 = 234.5f，末尾要加 f

**double**

- double 数据类型是双精度、64 位、符合 IEEE 754 标准的浮点数
- 浮点数的默认类型为 double 类型
- double 类型同样不能表示精确的值，如货币
- 例子：double d1 = 123.4

#### 6.2.2 浮点数的存储

![浮点数的存储](res/float.gif)

也就是

    一个float类型 = 1bit（符号位）+ 8bits（指数位）+ 23bits（尾数位）
    一个double包括 = 1bit（符号位）+ 11bits（指数位）+ 52bits（尾数位）  

于是，float 的指数范围为 -128~+127，而 double 的指数范围为 -1024~+1023，并且指数位是按补码的形式来划分的（和上面的 byte 一样）。因此，float 的范围为 -2^128 ~ +2^127，也即 -3.40E+38 ~ +3.40E+38；double 的范围为 -2^1024 ~ +2^1023，也即 -1.79E+308 ~ +1.79E+308。

#### 6.2.3 精度问题

float 和 double 的精度是由尾数的位数来决定的。浮点数在内存中是按科学计数法来存储的，其整数部分始终是一个隐含着的“1”，由于它是不变的，故不能对精度造成影响。

float：2^23 = 8388608，一共七位，由于最左为 1 的一位省略了，这意味着最多能表示 8 位数： 2*8388608 = 16777216 。有 8 位有效数字，但绝对能保证的为 7 位，也即 float 的精度为 7~8 位有效数字；double：2^52 = 4503599627370496，一共 16 位，同理，double 的精度为 16\~17 位。

在使用浮点类型的时候很容易出现意想不到的结果（[了解更多](http://blog.csdn.net/zq602316498/article/details/41148063)），所以在精度要求比较高的时候建议使用 BigDecimal.

### 6.3 布尔类型

|No|总结|
|:-:|:-|
|1|boolean 数据类型表示一位的信息|
|2|这种类型只作为一种标志来记录 `true/false` 情况|
|3|只有两个取值：true 和 false|
|4|默认值是 false|
|5|Java 中 0 不能代表 false，1 也不能代表 true|

### 6.4 字符类型

|No|总结|
|:-:|:-|
|1|char 类型是一个单一的 16 位 Unicode 字符（双字节）|
|2|最小值是 \u0000（即为 0）|
|3|最大值是 \uffff（即为 65,535）|
|4|char 数据类型可以储存任何字符|

### 6.5 装箱

1. Java中存在8种简单类型：boolean, byte, short, int, long, float和double.
2. 对应的有8种包装类：Boolean, Byte, Character, Short, Integer, Long, Float和Double. 
3. Java 无论装箱和拆箱都存在显示和隐式转换两种方式. 
4. 虽然很多场合下基本数据类型和它的包装类能达到相同的效果，但是使用包装类对基本数据类型进行包装会有额外的开销。所以，能不使用包装类时，尽量使用基本数据类型。

示例：

	public static void main(String[] args) {
		int var1=10;
		Integer obj1=var1;  			// 隐式装箱
		Integer obj2=(Integer)var1;  	// 显示装箱
		int var2=obj1;  				// 隐式拆箱
		int var3=(int)obj2;  			// 显式拆箱
	}

### 6.6 类型转换

![类型转换](res/transfer.jpg)

1. 自动类型转换（隐式转换）：隐式转换只允许发生在从小的值范围类型到大的值范围类型的转换，转换后数值大小不受影响. 但可能会导致精度降低.
2. 强制类型转换（显式转换）

关于强制类型转换的总结：

|No|总结|
|:-:|:-|
|1|强制类型转换有时候会发生截断，比如 300 转换为 byte 类型会得到 44|
|2|不存在到 char 类型的隐式转换|
|3|布尔类型不允许进行任何的类型转换|
|4|整数+浮点数时，浮点数为 double 则另一个是double，浮点数为 float 则另一个是 float，整数是long 则另一个是 long，否则两个都是 int|
|5|上面的转换图实线箭头表示无信息丢失的转换，虚线箭头表示可能有精度损失的转换。可见，从字节小的到字节大的没有损失，从字节大的到字节小的会有精度损失|
|6|将 float 和 double 转换成整数时，对该数字进行截尾，即舍弃小数部分，0.7 将会被转换成 0|
|7|表达式中最大的数据类型决定了表达式的结果，如果将 float 与 double 相乘，结果就是 double 如果将 int 和 long 相乘，结果就是 long|

### 6.7 值类型和引用类型

值类型和引用类型的区别的示例代码：

    public static void main(String ...args) {
        ValueHolder holderPositive = new ValueHolder(1);
        ValueHolder holderNegative = new ValueHolder(-1);
		
        swap(holderNegative, holderPositive); // 交换引用，无法达到交换效果
        System.out.println("holderPositive.value=" + holderPositive.value + ", " + "holderNegative.value=" + holderNegative.value);
        
        swapValue(holderNegative, holderPositive); // 交换引用的字段，可以达到交换效果
        System.out.println("holderPositive.value=" + holderPositive.value + ", " + "holderNegative.value=" + holderNegative.value);
        
        int positive = 1, negative = -1;
        swap(positive, negative); // 交换值类型的值，无法达到交换效果
        System.out.println("positive=" + positive + ", " + "negative=" + negative);
        
        Integer objPos = 1, objNeg = -1;
        swap(objPos, objNeg); // 使用整数类型的包装类型来交换，无法达到交换效果
        System.out.println("objPos=" + objPos + ", " + "objNeg=" + objNeg);

        System.out.println("------------------------");
        positive = negative;
        positive = 2; // 修改一个值类型不会影响另一个
        System.out.println("positive=" + positive + ", " + "negative=" + negative);
		
        holderPositive = holderNegative;
        holderPositive.value = 2; // 修改一个引用会影响到另一个
        System.out.println("holderPositive.value=" + holderPositive.value + ", " + "holderNegative.value=" + holderNegative.value);
    }

    /**
     * 交换两个引用类型的字段：对各个引用的字段进行了修改（相当于修改了内存块上的记录） 
	 */
    private static void swapValue(ValueHolder holder1, ValueHolder holder2) {
        int value = holder1.value;
        holder1.value = holder2.value;
        holder2.value = value;
    }

    /**
     * 交换两个引用类型的引用：仅仅交换了传入的应用的副本（副本指向了其他地方） 
	 */
    private static void swap(ValueHolder holder1, ValueHolder holder2) {
        ValueHolder value = new ValueHolder(holder1.value);
        holder1 = holder2;
        holder2 = value;
    }

    /**
     * 使用整数类型的包装类型来进行交换：这里相当于重新装箱，不会修改原始对象的字段（副本指向了其他地方） 
	 */
    private static void swap(Integer integer1, Integer integer2) {
        int value = integer1.intValue();
        integer1 = integer2.intValue();
        integer2 = value;
    }

    /**
     * 交换两个值类型：交换的只是副本的值，原来的值不会变化
	  */
    private static void swap(int positive, int negative) {
        int value = positive;
        positive = negative;
        negative = value;
    }

    public class ValueHolder {
        public int value;

        public ValueHolder(int value) {
            this.value = value;
        }
    }


输出结果：

	holderPositive.value=1, holderNegative.value=-1
	holderPositive.value=-1, holderNegative.value=1
	positive=1, negative=-1
	objPos=1, objNeg=-1
	------------------------
	positive=2, negative=-1
	holderPositive.value=2, holderNegative.value=2

**结论**：

|No|结论|
|:-:|:-|
|1|在 Java 中对基本数据类型，Java 传递值的副本；对一切引用类型，Java 都传递引用的副本|
|2|对于引用类型，两个变量可能引用同一对象，因此对一个变量的操作可能影响另一个变量所引用的对象|


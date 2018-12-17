# Java 基础回顾：几个比较重要的预定义类

> 这篇文章中梳理了 Java 中几个常见的预定义类：字符串类型 String 类、Object 类、枚举类型 Enum 类以及数组。

## 1、字符串 String 类

### 1.1 字符串相关问题总结

String 的每一个看起来会修改 String 的方法实际都是创建一个全新的 String 对象，以包含修改后的字符串的内容，而最初的 String 对象则丝毫未动；

String 使用 `char value[]` 存储字符，所以 String 对象在创建之后就不能修改对象存储的字符串内容，正因如此才说 String 类型是不可变的；

String 使用正则表达式的方式，应该注意一下下面的这些方法参数是正则表达式而不是普通的字符串：

	1. 匹配验证操作：`"asd".matches("[0-9]")`
	2. 分割操作：`"0as0d".split("[0-9]")`
	3. 替换操作：`"0a0sd".replaceAll("[0-9]", "09")`


除了调用 String 对象的方法，还可以使用 `Pattern` 和 `Matcher` 两个类来使用正则表达式：

    private static Pattern imagePattern;

    public static Uri getPreviewImage(String noteContent) {
        if (imagePattern == null) {
            imagePattern = Pattern.compile(Constants.REGEX_NOTE_PREVIEW_IMAGE);
        }
        Matcher matcher = imagePattern.matcher(noteContent);
        if (matcher.find()) {
            String str = matcher.group();
            if (!TextUtils.isEmpty(str)) {
                // do something...
            }
        }
        return null;
    }

String 类有一个特殊的创建方法，就是使用 `""` 创建。比如 `new String("str")` 实际上创建了两个对象：一个是通过 `""` 创建的，另一个是使用 `new` 创建。只是创建的时期不同：一个是在编译器，一个是在运行期。

运行期间调用 String 的 `intern` 方法可以向 String Pool 中动态添加对象。如

    String s1 = "strs";
    String s2 = s1.intern();
	
    这个时候，如果我们使用`s1 == s2`进行判断的话会得到什么结果呢？    
    答案是 true. 参考intern方法的注释：如果 Pool 中存在一个与当前 String 相等（所谓的相等是指使用 equals 方法判断时相等）的对象的时候，就返回 Pool 中的那个对象。否则，就将当前的 String 添加到 Pool 中。

注意字符串拼接操作和 `==` 操作的优先级：

    String s = "str";
    System.out.println("s == s " + s == s);
    System.out.println("s == s " + (s == s));

    输出的结果是：
        false
        s == s true

    这是因为不管 + 号是用作加法还是用作连接字符串，它的优先级都要比 == 号要高。

使用字符串拼接操作符 `+` 来拼接字符串，不适合运用在大规模的场景中，这是因为当两个字符串拼接到一起时，它们的内容都要被拷贝。

char 类型是采用 `UTF-16` 编码的 Unicode 代码点，大多数 Unicode 字符用一个代码单元，辅助字符需要两个代码单元，使用 charAt 的时候获取的是指定位置的代码单元，所以当字符串中存在需要两个代码单元的字符时，就容易出现错误。一般可以使用下面的形式获取每个代码点

    int cp = sentence.codePointAt(n);
    If(Character.isSupplementaryCodePoint(cp)) i += 2;
    else i++;

### 1.2 其他

1. String 类型为 0 或多个双字节 Unicode 字符组成的序列，默认为 null，这与 `””` 不同。所以，要判断一个字符串是否为空，可以使用下面的形式 `if (str != null && str.length != 0)`，即说明字符串不是 `null` 也不是 `””`；
2. 字符串不是字符数组，不能按照数组的方式访问；
3. 比较字符串是否相等要用 `equals` 方法（C++可以用 ==，C语言使用 strcmp）;
4. 要由许多小段的字符串构建字符串，使用 `StringBuilder` 效率更高；
5. 如果要修改字符串指定位置的字符，使用 `subString()` 截取然后再使用 `+` 或者 `StringBuilder` 进行拼接即可；
6. 文件路径中的反斜号前要增加一个额外的反斜号，如 `”c:\\myDir\\myFile.txt”`；
7. Java 中字符串的长度是不可变的，这样设计是为了使字符串共享；
9. 使用 `String.format()` 静态方法可以创建一个格式化的字符串，而不输出，比如：`String msg = Sting.format(“Age is %d”,age);`。

## 2、Object 类

所有类型都隐式的派生于 `java.lang.Object` 类，其主要用于两个目的：

1. 使用 `Object` 引用绑定任何数据类型的对象；
2. `Object` 类型执行许多基本的一般用途的方法，包括 `equals()`, `finalize()`, `hashCode()`,  `getClass()`, `toString()`, `notify()`, `notifyAll()` 和 `wait()` 等.

### 2.1 finilize() 方法

1. 它是不可预测的，也是很危险的，一般情况下是不必要的；  
2. 它的出现只是对 C++ 中的析构函数的一个妥协；
3. 如果想要在类结束的时候释放占用的资源，可以使用 try-catch 结构来完成。

### 2.2 equals() 方法

equals() 方法需要遵循的规范：

1. `自反性`：当 x!=null, 有 x.equals(x) 为 true；
2. `对称性`：当 x!=null 且 y!=null, 有 x.equals(y) 与 y.equals(x) 结果相同；
3. `传递性`：当 x!=null, y!=null, z!=null, x.equals(y) 且 y.equals(z)， 那么 x.equals(z)；
4. `一致性`：当 x!=null，如果 x 和 y 没有修改过，那么 x.equals(y) 结果不变；
5. 对任何 x!=null，有 x.equals(nyll) 为 false.

覆写的诀窍：

1. 使用 `==` 操作检查 “参数是否为这个对象的引用”；
2. 使用 `instanceOf` 检查 “参数是否为正确的类型”；
3. 把参数转换成正确的类型；
4. 对于该类中的每个“关键”域，检查参数中的域是否与该对象中的域相匹配；
	1. 对于非 `float` 和 `double` 的基本类型域，使用 `==` 判断两个值是否相等；
	2. 对于引用类型的域，可以使用 `equals` 方法判断两者是否相等；
	3. 对于 `float` 域，可以使用 `Float.compare` 方法；
	4. 对于 `double` 域，可以使用 `Double.compare` 方法；
	5. 对于数组域，使用上述原则到每个元素，如果数组每个元素都很重要，可用 `Arrays.equals` 方法；
5. 为了获得最佳性能，应该先比较最可能不一致的域，或者开销最低的域；
6. 编写完 `equals` 方法之后，检查它们是否是：对称的、传递的、一致的；
7. 覆盖 `equals` 方法时总要覆写 `hashCode` 方法；
8. 不要将 `equals` 方法中的 `Object` 替换成其他类型（那就不是覆写了）。

下面是一个示例程序，其中也包含了 hashCode 方法

    private static class Person {
        private long number;
        private int age;
        private String name;
        private float wage;
        private int[] id;

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof Person)) return false;

            Person person = (Person) o;

            if (number != person.number) return false;
            if (age != person.age) return false;
            if (Float.compare(person.wage, wage) != 0) return false;
            if (name != null ? !name.equals(person.name) : person.name != null) return false;
            return Arrays.equals(id, person.id);
        }

        @Override
        public int hashCode() {
            int result = (int) (number ^ (number >>> 32));
            result = 31 * result + age;
            result = 31 * result + (name != null ? name.hashCode() : 0);
            result = 31 * result + (wage != +0.0f ? Float.floatToIntBits(wage) : 0);
            result = 31 * result + Arrays.hashCode(id);
            return result;
        }
    }

### 2.3 hashCode() 方法

`hashCode()` 方法需要遵循的规范：

1. 只要用于 `equals()` 方法比较的信息没有修改，`hashCode()` 多次调用应返回同样的值；
2. 两个对象的 `equals()` 方法返回结果相同，那么它们的 `hashCode()` 方法的返回结果也应该相同；
3. 两个对象的 `equals()` 方法返回结果不同，它们的 `hashCode()` 不一定要不同，但是如果不同的话，可以提高散列表的性能。

计算 `hashCode()` 的简单方法：

1. 把某个非零的常数值，比如 `17`，保存在名为 `result` 的 `int` 型变量中；
2. 对对象中的每个关键域 `f`（用在 `equals()` 方法中的域），完成以下步骤：
	1. 计算该域 `f` 的散列码 `c`；
		1. 若 `f` 为 `boolean` 型，计算 `f ? 1 : 0`；
		2. 若 `f` 为 `byte`, `char`, `short` 或 `int` 型，计算 `(int) f`；
		3. 若 `f` 为 `long` 型，计算 `(int) (f >>> 32)`；
		4. 若 `f` 为 `float` 型，计算 `Float.floatToIntBits(f)`；
		5. 若 `f` 为 `double` 型，计算 `Double.doubleToLongBits(f)`，然后按 long 型处理；
		6. 若 `f` 为对象引用，调用对象的 `hashCode()` 方法，如果对象为 `null` 返回 0 ；
		7. 若 `f` 为数组，对数组每个元素当作单独的域处理，如果数组所有元素都有意义，则用`Arrays.hashCode()` 方法；
	2. 按照下面公式将 `c` 合并到 `result`，`result = 31 * result + c`；
3. 返回 `result`

示例代码可以参考 `equals` 方法，它是由 IDEA 自动生成的。

### 2.4 clone() 方法：对象克隆

当我们使用 `=` 将一个引用类型赋值给另一个引用类型的时候，会使得两个引用类型共享一份资源的拷贝。
这时候修改一个引用会使的两个引用的内容都发生变化，而使用 `clone()` 就可以解决这个问题。Clone 之后的两个引用类型各自有自己的一份内容，不会相互影响。

但是，有一点值得注意的是，下面的代码中的 `CloneableClass` 中不存在引用类型，如果其中仍然存在引用类型的话，我们需要在 `clone()` 方法中也将该引用类型 clone 一份。

    public static void main(String[] args) throws CloneNotSupportedException {
        // 测试1
        CloneableClass test1 = new CloneableClass("Test Class-01");
        CloneableClass cloned = test1.clone();
        // 修改了克隆对象的字段会反映到被克隆对象上
        cloned.inner.name = "clone2.inner.name"; 
        System.out.println(test1.inner.name);
        // 测试2
        AnotherCloneableClass test2 = new AnotherCloneableClass("Test Class-02");
        AnotherCloneableClass anotherCloned = test2.clone();
        anotherCloned.inner.name = "anotherClone2.inner.name";
        System.out.println(test2.inner.name);
    }

    private static class InnerClass implements Cloneable{
        public String name;

        public InnerClass clone() throws CloneNotSupportedException{
            return (InnerClass)super.clone();
        }
    }

    private static class CloneableClass implements Cloneable{
        private String name;
        public InnerClass inner;

        public CloneableClass(String name){
            this.name = name;
            inner = new InnerClass();
        }

        public CloneableClass clone() throws CloneNotSupportedException{
            return (CloneableClass)super.clone();
        }
    }

    private static class AnotherCloneableClass extends CloneableClass{
        public AnotherCloneableClass(String name) {
            super(name);
        }

        public AnotherCloneableClass clone() throws CloneNotSupportedException{
            AnotherCloneableClass cloned = (AnotherCloneableClass) super.clone();
            cloned.inner = inner.clone();
            return cloned;
        }
    }

输出结果：

    clone2.inner.name
    null

## 3、枚举类 Enum

### 2.1 枚举的示例

#### 2.1.1 基本使用示例

下面是枚举的一个使用示例：

    public enum ProductType {
        NORMAL(0, "普通品"),
        SPECIAL(1, "特殊品"); // 这里的分号是必不可少的

        public final String name;
        public final int no;

        ProductType(int id, String name) {
		    this.id = id;
            this.name = name;
        }

        public static ProductType getPortraitById(String name) {
            for (ProductType type : values()){
                if (type.name.equals(name)){
                    return type;
                }
            }
            throw new IllegalArgumentException("illegal argument");
        }
    }

上面是一个枚举的示例，注意这里面的一些细节：

1. 首先，定义枚举的时候要用 `enum` 关键字，这只是替代了定义类的时候的 `class` 关键字；
2. 实际上枚举是隐式继承 `Enum.class` 的，所以，枚举本身也是一个类，并且上面用到的 `values()` 方法就来自于 `Enum`；
3. 枚举通常用来表示那些不会进行修改的对象，所以我们可以根据需要向枚举中添加一些字段；
4. 枚举的字段可以是 `public` 的，因为它同时也是 `final` 的，所以，不用担心暴露得太多而无法控制的问题；
5. 因为所有的枚举类型都默认继承了 `Enum` 类，所以自定义枚举类型就不能再继承其他类了；
6. 枚举有一个 `ordinal` 字段，它表示的是指定的枚举值在所有枚举值中的位置（从 `0` 开始）。不过，我们通常倾向于自己实现自己的 `id` 来给枚举值标序，因为这样更有利于维护。

#### 2.1.2 使用接口组织枚举

虽然，枚举没有办法继承新的类，但是却可以实现接口。这里我们在接口内部定义一组枚举，用来表示同一个大类中的一些分组，然后每个枚举内部再定义一些具体的枚举值：

    public interface City {
        enum ChineseCity implements City {
            BEIJING, SHANGHAI, GUANGZHOU;
        }

        enum AmericanCity implements City {
            NEW_YORK, HAWAII, IOWA, WASHINGTON;
        }

        enum EnglishCity implements City {
            BRISTOL, CAMBRIDGE, CHESTER, LIVERPOOL;
        }
    }

这里我们定义的是一个城市的接口，借口内部定义了三个枚举，分别枚举了中国、美国和英国的城市。可以看出在这里我们使用接口定义了“城市”的抽象概念，然后在接口的内部定义了三个枚举，来对应三个不同的国家。这样就相当于在 `City` 到具体的枚举之间又增加了一个新的层次。定义完毕之后，我们可以这么使用：

    public static void main(String ...args) {
        City city = City.AmericanCity.IOWA;
        System.out.println(city);
        System.out.println(City.ChineseCity.BEIJING);
        System.out.println(City.AmericanCity.NEW_YORK);
        System.out.println(City.EnglishCity.LIVERPOOL);
    }

#### 2.1.3 使用枚举增强枚举的扩展性

如下面的代码所示，我们定义了一个接口类型 `Operation` 来表示一些操作，其内部定义了一个执行的方法（需要注意的是，我们在使用 `enum` 定义枚举的时候，实现接口的方法的操作是在各个枚举值上面实现的）。我们可以先定义一些基本的枚举类型，如果我们要在原来的基础之上进行拓展的话，那么我们只需要实现 `Operation` 并添加新的枚举即可：

    public interface Operation {
        double apply(int num1, int num2);
    }

    public enum BasicOperation implements Operation {
        ADD() {
            public double apply(int num1, int num2) {
                return num1 + num2;
            }
        },
        MINUS() {
            public double apply(int num1, int num2) {
                return num1 - num2;
            }
        },
        TIMES() {
            public double apply(int num1, int num2) {
                return num1 / num2;
            }
        },
        DIVIDE() {
            public double apply(int num1, int num2) {
                return num1 * num2;
            }
        };
    }

    public enum ExtendedOperation implements Operation {
        EXP() {
            public double apply(int num1, int num2) {
                return Math.exp(num1);
            }
        };
    }

对以上定义的方法的一个调用：

    Operation operation = BasicOperation.ADD;
    System.out.println(operation.apply(7, 8));

使用上面的两行代码，我们可以轻易地得出结果为 `15`. 这是没有问题的，而且我们可以看出这里借助于枚举实现了策略模式。

### 2.2 EnumSet 和 EnumMap

这是两个适用于枚举类型的容器类型，略。

## 3、数组

### 3.1 一维数组

#### 3.1.1 一维数组的声明

    类型[] 数组名; 或 类型 数组名[];

声明和创建分别进行

    类型[] 数组名;
    数组名 = new 类型[元素个数];

声明和创建同时进行

    类型[] 数组名 = new 类型[元素个数];

#### 3.1.2 一维数组的实例化

    数组名 = new 类型[]{ 元素0,元素1,......,元素n };
    类型[] 数组名 = new 类型[]{ 元素0,元素1,......,元素n };
    类型[] 数组名 = { 元素0,元素1,......,元素n };

注意：

1. 如果通过 `{}` 初始化数组元素，则用 new 关键字创建数组不需要也不能指定数组的元素个数，编译器会自动推断元素个数. 即 `int []arr = new int[5]{1,2,3,4,5};` 是错误的. 
2. 就是如果指定了数组的内容就不能指定数组的大小。指定数组大小只能在仅仅定义数组的时候使用。
3. 创建一个数组而没有给其元素赋值时，数字数组所有元素初始化为 `0`，`boolean` 数组的所有元素初始化为 `false`，对象数组的所有元素初始化为 `null`；
4. 可以将 `类型[]` 当作一个整体，不要往 `[]` 中添加数字;
5. 常见的定义数组的错误形式：

        int[4][2] arr = new int[][];
        int[][] arr; arr = new int[4][2]{........};
        int[][] arr = new int[4][2]{........};

#### 3.1.3 一维数组的访问

    数组名[下标];

可以通过数组的 `length` 属性获得数组的长度，即

    数组名.length;

### 3.2 二维数组

#### 3.2.1 二维数组声明

    类型[][] 数组名; 或 类型 数组名[][];

声明和创建分别进行：

    类型[][] 数组名;
    数组名 = new 类型[元素个数1][元素个数2];

声明和创建同时进行：

    类型[][] 数组名 = new 类型[元素个数1][元素个数2];

#### 3.2.2 二维数组的实例化

    数组名 = new 类型[] { 元素0, 元素1, ......, 元素n };
    类型[][] 数组名 = new 类型[][] { 元素0, 元素1, ......, 元素n };
    类型[][] 数组名 = { 元素0, 元素1, ......, 元素n };

#### 3.2.3 二维数组访问

    数组名[下标1][下标2];

譬如 `arr[2][3];` 可以理解为包含两个数组的数组. 故：

    数组名.length         // 返回 数组名 的元素个数
    数组名[下标].length   // 返回 数组[下标] 的元素个数

### 3.3 不规则数组

#### 3.3.1 声明并初始化

    int[][] jaggedArray = { {1,3,5,7,9}, {0,2,4,6,8}, {11,22} };
    int[][] jaggedArray = { new int[]{1,3,5,7,9}, 
        new int[]{0,2,4,6,8}, new int[]{11,22} };
    int[][] jaggedArray = new int[][]{ new int[]{1,3,5,7,9}, new int[]{0,2,4,6,8}, new int[]{11,22} };
    int[][] jaggedArray = new int[3][ ];
    jaggedArray[0] = new int[]{1,3,5,7,9}; jaggedArray[1] = new int[]{0,2,4,6,8};; jaggedArray[2] = new int[]{11,22};

### 3.4 数组的工具类

#### 3.4.1 Java.util.Arrays

`Java.util.Arrays` 提供了一些提供了一系列的方法，可以用来对数组进行操作：

1. `sort()`：对数组进行排序；
2. `binarySearch()`：使用二分法查找指定的键值（注意查找之前需要先排序）；
3. `copyOf()` 和 `copyOfRange()`：复制数组，截取或使用默认值填充；
4. `toString()`：返回指定数组内容的字符串表示形式；
5. `fill()`: 使用指定的值类型填充数组；
6. `hashCode()`：产生数组的散列码；
7. `asList()`：将指定数组转换成 `List` 类型，注意 `asList()` 方法返回的 `ArrayList` 是 `Arrays` 的私有静态内部类，它不允许我们向返回的容器中加入或者移除元素。

#### 3.4.2 System.arraycopy()

`System.arraycopy()` 提供了拷贝数组的高效方法，它的定义是：`System.arraycopy(src, scrPos, dest, destPos, length)`。注意，该方法不会自动包装和自动拆包，所以 `Integer[]` 和 `int[]` 是不能相互复制的。

使用示例：

    int[] arr = new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    int[] dest = new int[5];
    System.arraycopy(arr, 4, dest, 2, 3);
    System.out.println(Arrays.toString(dest));

输出结果是：

    [0, 0, 4, 5, 6]
	
可以看出它的效果是：将 `arr` 中从索引 4 开始的连续 3 个元素拷贝到 dest 中从 2 开始向后的所有元素中。并且这里要求复制到 dest 中的元素个数必须小于 dest 定义的个数。

#### 3.4.3 数组克隆

`clone()` 方法不能直接用于多维数组，要用的话也要在每一维上使用 `clone()` 方法：

    int[] arr = {1,2,3,4,5}; int[] arr2 = arr.clone();

常用的复制数组方法总结：

1. 使用循环；
2. 使用数组变量的 `clone()` 方法，简单但是不灵活；
3. 使用 `System.arraycopy()` 方法. 简单灵活；
4. `Java,util.Arrays.copyOf` / `copyOfRange()` 方法. 简单灵活。

#### 3.4.5 数组与泛型

不能实例化具有参数化类型的数组，如下面的两种实例化方式中第一种是合法的，而第二种是不合法的：

    Bag<Integer>[] bags = new Bag[5]; // 合法
    // Bag<Integer>[] bags = new Bag<Integer>[5]; // 不合法
    for (int i=0;i<5;i++) {
        bags[i] = new Bag();
        bags[i].value = i;
    }

不能创建泛型数组，但是可以创建 `Object[]` 数组，然后将其强制转换成泛型数组：

    private static class Bag<T> {
        // T[] ts = new T[5]; // 非法
        T[] ts = (T[]) new Object[5]; // 合法，但是会在编译期得到“不受检查”的警告信息
    }



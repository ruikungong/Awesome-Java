# Java 基础回顾：泛型和 Class 类

## 1、泛型

以 ArrayList 为例，在范型出现之前，ArrayList 的实现机制是内部管理一个 `Object[]` 类型的数组。比如` add` 方法以前是 `add(Object obj)`，现在是 `add(E e)`。那么以前的时候显然如果你定义一个 String 类型的 ArrayList，传入 File 类型也是可以的，因为它也继承自 Object。这显然就会出现错误！但是有了泛型之后，传入的只能是E类型的，不然会报错。也就是说，泛型给我们提供了一种类型检查的机制。

Java泛型是使用类型擦除来实现的。它的好处只是提供了编译器类型检查机制。在泛型方法内部无法获取任何关于泛型参数类型的信息，泛型参数只是起到了占位的作用。如 `List<String>` 在运行时实际上是 List 类型，普通类型被擦除为 Object。因为泛型是擦除的，所以泛型不能用于显式地引用运行时类型的操作中，例如：转型、`instanceof` 操作或者 `new` 表达式。

另外，

1. 使用泛类型的代码意味着可以被很多不同类型的对象重用；
2. Java 编译器最终将泛型版本编译为无泛型版本；
3. 使用 extends 的意义就在于指定了擦除的上界。

### 1.2 泛型类

泛型类的声明与一般类的声明语法一致，但需要在声明的泛型类名称后使用 `<>` 指定一个或多个类型参数，如：

```java
    class MyClass <E> { } 或 class MyClass <K,V> { }
```

### 1.3 泛型接口

泛型接口的声明与一般接口的声明语法一致，但需要在声明的泛型接口名称后使用 `<>` 指定一个或多个类型参数，如：

```java
    interface IMyMap <E> 或 intetface IMyMap <K,V> 
```

### 1.4 泛型方法

从下面的示例中，我们可以看出当定义了一个泛型方法的时候，可以提高代码的复用性。如：

    public void method(T e) { } 或 public void method(List<?> list)

不过通常我们需要为泛型指定一个擦除上界来对泛型的范围进行控制。

### 1.5 泛型参数的约束

```java
    <T extends 基类或接口> 或 <T extends 基类或接口1 & 基类或接口2>
```

前面的形式表示 `T` 需要是指定的基类或者接口的子类，后面的形式表示 `T` 需要是指定的接口或者基类 1 的子类并且是基类或者接口 2 的子类。

### 1.6 泛型的补偿

#### 1.6.1 创建实例

使用类型标签来获取指定类型的实例：

```java
    try {
        Person person = Person.class.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
```

但是使用上面的方式，要求指定的类型标签必须有默认的构造器。

#### 1.6.2 创建数组

可以使用`类型标签+Array.newInstance()`的方式实现：

```java
    int[] arr = (int[]) Array.newInstance(int.class, 5);
```

下面的这种方式在运行时不会出错，但是在 main 方法中强制进行类型转换的时候会出错：

```java
    private static  <T> T[] createArray(T t) {
        T[] arr = (T[]) new Object[5];
        arr[0] = t;
        return arr;
    }

    public static void main(String ...args) {
        Integer[] array = createArray(5); // ClassCastException
        System.out.println(Arrays.toString(array));
    }
```

这是因为它的运行时类型仍然是Object[]，写成下面的形式就不会错了：

```java
    public static void main(String ...args) {
        Object[] array = createArray(5); // 不会出错
        System.out.println(Arrays.toString(array));
        Integer integer = (Integer) array[0]; // 也不会错
        System.out.println(integer);
    }
```

因为有了擦除，数组的运行时类型只能是 `Object[]`。

#### 1.6.3 自限定的类型

```java
    private static class SelfBounded<T extends SelfBounded<T>> {}

    private static class A extends SelfBounded<A> {}
    
    private static class B extends SelfBounded<A> {}

    private static class C extends SelfBounded {}

    // private static class D extends SelfBounded<C> {}  // 错误!

    // private static class E extends SelfBounded<B> {}  // 错误!

    // private static class F extends SelfBounded<D> {}  // 错误!
```

可以看出，当定义了 `class M extends SelfBounded<N>` 的时候，这里对N的要求是它的必须实现了 `SelfBounded<N>`。

#### 1.6.4 泛类与子类

虽然 `Object obj = new Integer(123);` 是可行的，但是 `ArraylList<Object> ao = new ArrayList<Integer>();` 是错误的。因为 `Integer` 是 `Object` 的派生类，但是 `ArrayList<Integer>` 不是 `ArraylList<Object>` 的派生类。

#### 1.6.5 通配符

根据上面的泛类与子类的关系，如果要实现一个函数，如

```java
    void PrintArrayList(ArrayList<Object> c){
        for(Object obj:c){}
    }
```

那么 `ArrayList<Integer>(10)` 的实例是无法传入到该函数中的，因为 `ArrayList<Integer>` 和`ArrayList<Object>` 是没有继承关系的。在这种情况下就可以使用通配符解决这个问题。我们可以定义上面的函数为如下形式，这样就可以将泛型传入了。

```java
    PrintArrayList(ArrayList<?>c) {
        for(Object obj:c){}
    }
```

当然，也可以指定通配符 `?` 的约束，即将其写成下面的形式

```java
    <? extends 基类>
    <? super 派生类>
```

就是在运行时指定擦除的边界。

## 2、Class 类

Class 类包含了与类某个类有关的信息，Class 也支持泛型。它们的效果是相同的，只是使用泛型具有编译器类型检查的效果，相对更加安全。

以下是使用Class类的一个测试例子。在这里我们可以通过反射来获取并修改 `private方法` 和 `private字段` 的`accessible` 属性。当设置为 `true` 时，我们就可以对其进行调用或者修改。

此外，我们还可以进行自定义注解以及获取类、方法和字段的注解等信息。

**示例 1：获取 Class 对象的属性信息，修改 private 类型的字段，调用 private 方法：**

```java
    public static void main(String ...args) {
        Class<SubClass> subClass = SubClass.class;

        try {
            // 如果SubClass是private类型的，那么会抛出以下异常：
            // java.lang.IllegalAccessException: Class me.shouheng.rtti.RttiTest can not access a member of class
            // me.shouheng.rtti.RttiTest$SubClass with modifiers "private"
            SubClass sub = subClass.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }

输出 Class 类的方法，这里输出的结果中包含了超类的方法：

        System.out.print("\nMethods:\n");
        System.out.println(Arrays.toString(subClass.getMethods()));

输出 Class 内部定义的方法，它只输出了在 SubClass 中新加入的方法的定义：

        System.out.print("\nDeclaredMethods:\n");
        System.out.println(Arrays.toString(subClass.getDeclaredMethods()));

        System.out.print("\nMethodInformation:\n");
        printMethodInfo(subClass);

输出 Class 的字段，输出的结果为空：

        System.out.print("\nFields:\n");
        System.out.println(Arrays.toString(subClass.getFields()));

输出 Class 中定义的字段：

        System.out.print("\nDeclaredFields:\n");
        System.out.println(Arrays.toString(subClass.getDeclaredFields()));

        System.out.print("\nFieldInformation:\n");
        printFieldInfo(subClass);

        System.out.print("\nClassInformation:\n");
        printClassInfo(subClass);
    }

    private static void printMethodInfo(Class<?> c) {
        SubClass sub = new SubClass();
        sub.setName("My Simple Name");
        // 打印方法信息
        Method[] methods = c.getDeclaredMethods();
        Method method = methods[0];
        System.out.println(method.getName());
        System.out.println(method.getReturnType());

输出方法的注解信息，对于能够输出的注解信息是有要求的：

        // 输出注解信息
        Annotation[] annotations = method.getDeclaredAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }

可以在获取了方法的Method对象之后调用它的invoke方法进行触发：

        // 触发方法
        try {
            System.out.println(method.invoke(sub));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

修改方法的属性，这里我们是已经知道out方法是第二个方法的前提下进行的。还有直接使用Class对象的getMethod尝试使用方法名获取方法是不行的：

        try {
            // 设置out方法的accessible为true，这样可以从外部调用该方法
            method = methods[1];
            method.setAccessible(true);
            // 直接根据名称来获取out方法是不行的
    //      method = c.getMethod("out");
    //      method.setAccessible(true);
            method.invoke(sub);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }

    private static void printFieldInfo(Class<?> c) {
        SubClass sub = new SubClass();
        sub.setName("My Simple Name");
        Field[] fields = c.getDeclaredFields();
        Field field = fields[0];
        try {
            System.out.println(field.get(sub));
        } catch (IllegalAccessException e) {
            System.out.println(e);
        }
        Annotation[] annotations = field.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }
        System.out.println(field.isAccessible());
        System.out.println(field.getName());

我们可以通过修改一个字段的访问性来修改这个字段的属性：

        field.setAccessible(true);
        try {
            field.set(sub, "Shit");
            System.out.println(field.get(sub));
        } catch (IllegalAccessException e) {
            System.out.println(e);
        }
    }

    private static void printClassInfo(Class<?> c) {
        System.out.println(Arrays.toString(c.getAnnotations()));
        System.out.println(c.getCanonicalName());
        System.out.println(c.getClass());
        System.out.println(c.getGenericSuperclass());
        System.out.println(Arrays.toString(c.getGenericInterfaces()));
        System.out.println(Arrays.toString(c.getConstructors()));
        System.out.println(c.getPackage());
    }

    private static class SuperClass {}

    // private 类型的不行
    @Deprecated
    public static class SubClass extends SuperClass implements Interface {
        @me.shouheng.rtti.Field
        @NotNull
        private String name;
        private SuperClass sup;

        @Deprecated
        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public SuperClass getSup() {
            return sup;
        }

        public void setSup(SuperClass sup) {
            this.sup = sup;
        }

        private void out() {
            System.out.println("private method in SubClass");
        }
    }

    private interface Interface {}
```

输出结果：

```java
	Methods:
	[public java.lang.String me.shouheng.rtti.RttiTest$SubClass.getName(), public void me.shouheng.rtti.RttiTest$SubClass.setName(java.lang.String), public me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.getSup(), public void me.shouheng.rtti.RttiTest$SubClass.setSup(me.shouheng.rtti.RttiTest$SuperClass), public final void java.lang.Object.wait() throws java.lang.InterruptedException, public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]
	
	DeclaredMethods:
	[public java.lang.String me.shouheng.rtti.RttiTest$SubClass.getName(), private void me.shouheng.rtti.RttiTest$SubClass.out(), public void me.shouheng.rtti.RttiTest$SubClass.setName(java.lang.String), public me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.getSup(), public void me.shouheng.rtti.RttiTest$SubClass.setSup(me.shouheng.rtti.RttiTest$SuperClass)]
	
	MethodInformation:
	getName
	class java.lang.String
	@java.lang.Deprecated()
	My Simple Name
	private method in SubClass
	
	Fields:
	[]
	
	DeclaredFields:
	[private java.lang.String me.shouheng.rtti.RttiTest$SubClass.name, private me.shouheng.rtti.RttiTest$SuperClass me.shouheng.rtti.RttiTest$SubClass.sup]
	
	FieldInformation:
	java.lang.IllegalAccessException: Class me.shouheng.rtti.RttiTest can not access a member of class me.shouheng.rtti.RttiTest$SubClass with modifiers "private"
	@me.shouheng.rtti.Field()
	false
	name
	Shit
	
	ClassInformation:
	[@java.lang.Deprecated()]
	me.shouheng.rtti.RttiTest.SubClass
	class java.lang.Class
	class me.shouheng.rtti.RttiTest$SuperClass
	[interface me.shouheng.rtti.RttiTest$Interface]
	[public me.shouheng.rtti.RttiTest$SubClass()]
	package me.shouheng.rtti
```

**示例2：泛型参数的获取：**

下面是在实际的框架设计中会用到的一些方法，它尝试从类的泛型中获取泛型的名称：

```java
    private static class Model { }

    private static class Product extends Model { }

    private static class Store<T extends Model> {

        public Store() {
            Class cls = this.getClass();
            ParameterizedType pt = (ParameterizedType) cls.getGenericSuperclass();
            Type[] args = pt.getActualTypeArguments();

            System.out.println(cls);
            System.out.println(pt);
            System.out.println(Arrays.toString(args));
            System.out.println(((Class) args[0]).getSimpleName());
        }
    }

    private static class ProductStore extends Store<Product> { }

    public static void main(String ...args) {
        ProductStore store = new ProductStore();
    }
```

输出结果：

```java
	class me.shouheng.rtti.RttiTest$ProductStore
	me.shouheng.rtti.RttiTest.me.shouheng.rtti.RttiTest$Store<me.shouheng.rtti.RttiTest$Product>
	[class me.shouheng.rtti.RttiTest$Product]
	Product
```

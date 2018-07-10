# 常用的Java第三方库JodaTime

Java8之前的时间库中存在一些设计不好的地方，导致用起来非常地不方便，又容易出错。比如，要实现在指定的日期的基础上面增加指定的时间的操作，你需要些大量的样板代码；而它的月份从0开始，稍有不慎就会掉入坑中。所以，通常我们使用第三方库Joda Time来进行时间相关的操作。

## 1、使用JodaTime

JodaTime在Github上面的主页：[JodaTime](https://github.com/JodaOrg/joda-time)

使用JodaTime的时候的两种配置方式：

在Maven中：

    <dependency>
      <groupId>joda-time</groupId>
      <artifactId>joda-time</artifactId>
      <version>2.9.9</version>
    </dependency>

在Gradle中：

    compile 'joda-time:joda-time:2.9.9'

## 2、获取DateTime实例

当使用JodaTime的时候，首先你要获取一个DateTime实例，然后用它的其他方法串联起来实现强大的功能。想要获取一个DateTime实例，你有很多种方式。下面列出常见的几种方式：

方式1：使用系统时间构造DateTime实例

    DateTime dateTime = new DateTime();

方式2：使用具体的时间构造DateTime实例，该方法有许多重载版本

    DateTime dateTime1 = new DateTime(
    2000, // year
    1,    // month
    1,    // day
    0,    // hour (midnight is zero)
    0,    // minute
    0,    // second
    0     // milliseconds
    );

方式3：使用Calendar构造DateTime实例

    DateTime dateTime2 = new DateTime(Calendar.getInstance());

方式4：使用其他DateTime实例构造DateTime实例

    DateTime dateTime3 = new DateTime(dateTime);

方式5：使用字符串构造DateTime实例

    DateTime dateTime4 = new DateTime("2006-01-26T13:30:00-06:00");
    DateTime dateTime5 = new DateTime("2006-01-26");

## 3、使用DateTime的方法

DateTime中有许多的方法，这里我们将常用的方法分成两类。一类是在方法中返回DateTime的那种，一类是在方法中返回Property类型的那种。显然，后面的那种继续串联操作的话，就需要调用Property的实例方法了。

这里，我们先给出DateTime中的第一类方法。

    // 指定的时间单位上面增加指定的值
    DateTime dateTime0 = dateTime.plusDays(1);
    System.out.println(dateTime0);

    // 指定的时间单位上面减少指定的值
    DateTime dateTime6 = dateTime.minusDays(1);
    System.out.println(dateTime6);

    // 除了增减日期还可以直接指定它的指定时间单位上面的值
    DateTime dateTime7 = dateTime.withYear(2020);
    System.out.println(dateTime7);

    // 按照指定的格式输出日期
    System.out.println(dateTime.toString("E MM/dd/yyyy HH:mm:ss.SSS"));

在上面的代码中，我们只给出了其中的一部分方法的实例。实际上，在DateTime内部有许多的方法，只是它们的原理基本类似。

上面的一些方法，如果涉及的时间发生了变化（具体是指时间对应的毫秒数发生了变化），就会调用DateTime实例的withMillis()方法。在该方法中，如果发现传入的毫秒数与当前的毫秒数不一样就会新建一个DateTime实例，并将其返回。所以，上面的plusDays(1)和minusDays(1)返回的DateTime实际上已经是另一个实例了。

## 4、使用Property的

可以通过DateTime实例的millisOfDay() dayOfYear() minuteOfDay()等一些列方法可以获取到该DateTime的一个Property实例，然后可以通过调用Property的方法再获取一个DateTime实例。也就是说，实际上调用DateTime的方法获取Property实例是为了对指定的时间位置的信息进行修改。比如，对“日”进行修改，对“年”进行修改等等。修改了之后还是要获取一个DateTime实例，然后再继续进行后续的操作。

实际上每次调用DateTime的方法获取Property实例的时候，都会将当前的DateTime作为参数传入。然后当调用了指定的方法之后又会调用DateTime实例的withMillis()方法判断时间是否发生变化，如果发生了变化就创建一个新实例并返回。

下面是它的一些示例：

    // 这里先用dayOfMonth获取一个Property实例，然后调用它的withMaximumValue方法
    // 它的含义是指定日期的其他日期不变，月份变成最大的之后返回一个DateTime，即如果传入的是2018年5月1日，将返回2018年5月31日，
    // 年，月，秒等位置不变，日变成该月最大的。
    DateTime dateTime0 = dateTime.dayOfMonth().withMaximumValue();
    DateTime dateTime1 = dateTime.dayOfMonth().withMinimumValue();
		
## 5、其他的静态方法
		
除了上面的一些类之外，JodaTime还有许多的静态方法供我们使用。比如：

    System.out.println(Days.daysBetween(dateTime1, dateTime).getDays());
    System.out.println(Months.monthsBetween(dateTime1, dateTime).getMonths());
    System.out.println(Years.yearsBetween(dateTime1, dateTime).getYears());

当然，这里我们只列出了对两个DateTime实例的“日” “月”和“年”单位的操作，还有许多类似的类可以用来对“毫秒”“秒”等操作。

## 结语

这里只是通过JodaTime的一些常用的方法的实例来说明其设计的基本原理，重点在于理清其中的逻辑，明白每个被串联的操作究竟做了什么。

相关代码：[Java-advanced](https://github.com/Shouheng88/Java-advanced)
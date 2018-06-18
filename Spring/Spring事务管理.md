# Spring的事务管理

## 1、数据库事务

事务的ACID特性：原子性 隔离性 一致性 持久性

四种隔离界别：未提交读 提交读 可重复读 序列化

数据库事务类型有本地事务和分布式事务：

1. 本地事务：就是普通事务，能保证单台数据库上的操作的ACID，被限定在一台数据库上；
2. 分布式事务：涉及两个或多个数据库源的事务，即跨越多台同类或异类数据库的事务（由每台数据库的本地事务组成的），
分布式事务旨在保证这些本地事务的所有操作的ACID，使事务可以跨越多台数据库；

Java事务类型有JDBC事务和JTA事务：

1. JDBC事务：就是数据库事务类型中的本地事务，通过Connection对象的控制来管理事务；
2. JTA事务：JTA指Java事务API(Java Transaction API)，是Java EE数据库事务规范，JTA只提供了事务管理接口，
由应用程序服务器厂商（如WebSphere Application Server）提供实现，JTA事务比JDBC更强大，支持分布式事务。

Java EE事务类型有本地事务和全局事务：

1. 本地事务：使用JDBC编程实现事务；
2. 全局事务：由应用程序服务器提供，使用JTA事务；

按是否通过编程实现事务有声明式事务和编程式事务；

1. 声明式事务：通过注解或XML配置文件指定事务信息；
2. 编程式事务：通过编写代码实现事务。

## 2、Spring事务管理框架

使用Spring事务管理框架需要使用下面的依赖：

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${spring.version}</version>
</dependency>
```

该框架提供了事务策略，它是一个接口，定义如下。针对每种具体的事务类型或者数据库类型，有不同的实现方式：

```
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition var1) throws TransactionException;
    void commit(TransactionStatus var1) throws TransactionException;
    void rollback(TransactionStatus var1) throws TransactionException;
}
```

这里还要注意一下每个方法中都会抛出一个异常TransactionException，它是非受检异常，也就是继承自RuntimeException。

`getTransaction()`方法会根据传入的TransactionDefinition返回一个TransactionStatus。TransactionStatus代表一个事务，包含了事务的状态的信息。

而TransactionDefinition接口则是用来指定事务的传播行为、隔离级别、超时和是否只读等信息的。









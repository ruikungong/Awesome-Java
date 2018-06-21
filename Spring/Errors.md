## Problem 1

Cannot create PoolableConnectionFactory (Could not create connection to database server.)
 
MySQL version 8.0.11 
 
Update MySQL Connector to : 
 
         <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.11</version>
        </dependency>
 
## Problem 2

Caused by: java.sql.SQLException: Access denied for user 'shouh'@'localhost' (using password: YES) 

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="root" />
        <property name="password" value="${password}" />
    </bean>
	
之前使用占位符把用户名写在属性文件里面，调试的时候发现注入的值就是错误的，后来将占位符的内容直接写在bean的属性里面就没有问题了。

MYSQL启动的时候需要手动输入`MySQL -uroot -password`才行，否则会报错:`Access denied for user 'ODBC'@'localhost' (using password: NO)`

## Problem 3

继续调试出现异常：The server time zone value '?D1ú±ê×?ê±??' is unrecognized or represents more than one time zone.

按照下面步骤执行：

![](res/time_zone_err.png)

参考：

[mysql运行报The server time zone value '?D1ú±ê×?ê±??' is unrecognized or represents more than one time zone的解决方法](https://www.cnblogs.com/ljy-20180122/p/9157912.html)


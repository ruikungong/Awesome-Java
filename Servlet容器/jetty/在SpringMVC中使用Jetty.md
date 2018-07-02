# 在Spring MVC中使用Jetty

## 1、关于Jetty

之前我们讨论Tomcat的时候，没有涉及Servlet容器的内容。实际上，在我们开发的时候使用的是Tomcat，这是一个比较主流的Servlet容器。
除了Tomcat，Jetty也是一种Servlet容器，虽然地位现在不及Tomcat。

Jetty与Tomcat的几点区别：

1. 相对Tomcat，Jetty更轻量级。由于Tomcat除了遵循JavaServlet规范之外，自身还扩展了大量JEE特性以满足企业级应用的需求，
所以Tomcat是较重量级的，而且配置较Jetty亦复杂许多。
但对于大量普通互联网应用而言，并不需要用到Tomcat其他高级特性，所以在这种情况下，使用Tomcat是很浪费资源的。
这种劣势放在分布式环境下，更是明显。
换成Jetty，每个应用服务器省下那几兆内存，对于大的分布式环境则是节省大量资源。
而且，Jetty的轻量级也使其在处理高并发细粒度请求的场景下显得更快速高效。
2. Jetty更灵活，体现在其可插拔性和可扩展性，更易于开发者对Jetty本身进行二次开发，定制一个适合自身需求的Web Server。
相比之下，重量级的Tomcat原本便支持过多特性，要对其瘦身的成本远大于丰富Jetty的成本。用自己的理解，即增肥容易减肥难。
3. 然而，当支持大规模企业级应用时，Jetty也许便需要扩展，在这场景下Tomcat便是更优的。

## 2、安装Jetty

我们可以到地址：[下载地址](https://www.eclipse.org/jetty/download.html)获取Jetty的安装版本。这里我们选择zip格式，将其下载下来解压到指定的目录即可。

然后，我们进入到Jetty的目录下面，在命令行种执行下面的命令：

```
cd $JETTY_HOME
java -jar start.jar
```

便可以启动Jetty。由于我们没有部署任何web应用，所以当我们通过http://localhost:8080访问的时候会得到404的错误。
这时候，我们可以进入jetty目录下面的`demo-base`目录，执行下面的命令：

```
cd demo-base
java -jar ../start.jar
```

这样当我们再次打开http://localhost:8080的时候就进入了Jetty的欢迎页面。如果你已经看到了欢迎页面，那么说明Jetty也已经安装成功了。

## 3、使用Jetty

### 3.1 创建Jetty基目录

在任意文件夹下面创建一个新的文件夹JettyDemo，然后进入JettyDemo目录，并在命令行中执行：

```
java -jar D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar --add-to-startd=http,deploy
```

这里的`D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar`是我的电脑中的Jetty的安装目录。

这样就创建好了Jetty的基目录，不过此时你执行`java -jar D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar`仍然会看到404，因为该目录中暂时还没有项目。

### 3.2 改变Jetty的端口

在Jetty基目录中执行

```
java -jar D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar jetty.http.port=8081
```

这样可以修改使用的端口。除了使用命令行指令，也可以到`start.ini`或者`start.d/http.ini`文件中进行修改。

### 3.3 为HTTPS & HTTP2增加SSL

在Jetty基目录中执行

```
java -jar D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar --add-to-startd=https,http2
```

### 3.4 修改Jetty的HTTPS端口

在Jetty基目录中执行

```
java -jar D:\jetty\jetty-distribution-9.4.11.v20180605\start.jar jetty.ssl.port=8444
```

## 4、在Intellij中使用Jetty

我们可以在Intellij中直接使用Jetty部署我们的SpringMVC项目。在Maven中的配置方式也比较简单，只要在`pom.xml`的`plugins`中加入如下的插件即可：

```
<!--jetty插件-->
<plugin>
     <groupId>org.eclipse.jetty</groupId>
     <artifactId>jetty-maven-plugin</artifactId>
     <version>9.4.11.v20180605</version>
     <configuration>
         <stopPort>9988</stopPort>
         <stopKey>foo</stopKey>
         <scanIntervalSeconds>5</scanIntervalSeconds>
         <webAppConfig>
              <contextPath>/</contextPath>
         </webAppConfig>
    </configuration>
</plugin>
```

这样配置完成之后在Maven Project标签页下面的Plugins下面就会出现一个Jetty的目录，我们直接双击`jetty:run`即可运行我们的项目

# Tomcat是如何加载Spring和SpringMVC及Servlet

# 概述

大家是否清楚，Tomcat是如何加载Spring和SpringMVC，今天我们就弄清下这个过程（记录最关键的东西）

其中会涉及到大大小小的知识，包括加载时候的设计模式，Servlet知识等，看了你肯定有所收获~

# Tomcat

tomcat是一种Java写的Web应用服务器，也被称为Web容器，专门运行Web程序

## tomcat启动

tomcat启动了之后会在操作系统中生成一个Jvm（Java虚拟机）的进程，从配置监听端口（默认8080）监听发来的HTTP/1.1协议的消息

默认配置文件这样

```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

 当Tomcat启动完成后，它就会加载其安装目录下webapps里的项目（放war包会自动解压成项目）

#### *小提问：webapps里多个项目，是运行在同一个JVM上吗*

是运行在同一个JVM上的（Tomcat启动时创建的那个），多个项目就是多个线程，之所以项目间数据不共享，是因为类加载器不一样的缘故  参考：

[tomcat 与 jvm的关系]: https://blog.csdn.net/u010325193/article/details/81156813

## 加载Web程序（Spring+SpringMVC框架）

tomcat启动完毕后，最关键的是生成了ServletContext（Tomcat的上下文），然后会根据webapps项目里的web.xml进行加载项目

下面是一个SpringMVC+Spring项目的部分web.xml



```xml
<!--以下为加载Spring需要的配置-->
<!--Spring配置具体参数的地方-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>
    classpath:applicationContext.xml
  </param-value>
</context-param>
<!--Spring启动的类-->
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

 <!--以下为加载SpringMVC需要的配置-->
<servlet>
  <servlet-name>project</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>   <!--servlet被加载的顺序，值越小优先级越高（正数）-->

  <servlet-mapping>
        <servlet-name>project</servlet-name>
        <url-pattern>*.html</url-pattern>
  </servlet-mapping>
</servlet>
```

### 初始化Spring

tomcat首先会加载进ContextLoaderListener，然后将applicationContext.xml里写的参数注入进去，来完成一系列的Spring初始化（如各种bean，数据库资源等等）

这里就是经常听到的Ioc容器的初始化了，我们搜索这个类发现以下代码



```java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    //省略其他方法

    /**
     * Initialize the root web application context.
     */
    @Override
    public void contextInitialized(ServletContextEvent event) {
        initWebApplicationContext(event.getServletContext());
    }

    //省略其他方法
}
```

这里最重要的是通过ServletContext，初始化属于Spring的上下文WebApplicationContext，并将其存放在ServletContext

WebApplicationContext多重要老铁们都懂得，我们经常用webApplicationContext.getBean()来获取被Spring管理的类，所以这也是IOC容器的核心

Spring采用这种监听器来启动自身的方法，也是一种设计模式，叫观察者模式：

整个过程是这样的，Tomcat加载webapps项目时，先通过反射加载在web.xml标明的类（通通放入一个数组）

到某个时刻，我tomcat（事件源，事件的起源）会发起一个叫ServletContextEvent的事件（里面带着各种参数）

凡是实现了ServletContextListener接口的类，我都会调用里面的contextInitialized方法，并把这个事件参数传进去

咳咳，现在我看看这个数组里有没符合条件的（遍历），发现真有实现这个接口的（类 instanceof 接口），就调用contextInitialized方法

于是Spring就被动态加载进来了~~   

参考：

[通过tomcat容器启动spring容器的启动过程]: https://blog.csdn.net/qq_31854907/article/details/86300901



#### *题外话：*

加载一个类，可以用用完整的类名，通过java反射加载，Class.forName(类名)

也能直接new 一个类 来加载

### 初始化SpringMVC

看配置文件，标签是servlet，我们得首先了解下servlet是什么东东

### Servlet简介

Servlet是一个接口，为web通信而生（说白了就是一堆sun公司的大佬们开会，拍板造出的类，有固定的几个方法）

[![img](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208181143156-765773601.png)](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208181143156-765773601.png)

tomcat有一套定义好的程序（其实不只是tomcat，能跑java写的web应用服务器如Jetty等，都有这固定程序）

1.当tomcat加载进来一个类时，如果它实现了Servlet接口，那么会记载到一个Map里，然后执行一次init()方法进行Servlet初始化

2.当tomcat收到浏览器的请求后，就会在Map里找对应路径的Servlet处理，路径就是写在<url-pattern>标签里的参数，调用service()这个方法

3.当Servlet要被销毁了，就调用一次destroy()方法

各位看到这是不是感觉相识，跟Spring加载差不多嘛，都是实现了一个接口后就被命运（tomcat）安排~~

当然，我们自己实现Servlet接口太鸡儿麻烦了，于是有HttpServlet（一个抽象类）帮我们实现了大部分方法（包含http头的设置，doXXX方法判断等等）

[![img](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208182617222-1653086969.png)](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208182617222-1653086969.png)

所以我们只要继承HttpServlet就实现几个方法就能用啦

[![img](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208182659719-991860520.png)](https://img2020.cnblogs.com/blog/1764312/202012/1764312-20201208182659719-991860520.png)

[Servlet在何时加载(tomcat源码)]: https://www.cnblogs.com/hvicen/p/6010688.html

 

### SpringMVC加载

为什么要讲Servlet，因为SpringMVC的核心就是DispatcherServlet（前置控制器），如图

![](pic\dispatcherServlet .PNG)

 

 DispatcherServlet由SpringMVC的实现，已经实现的很棒棒了，我们不需要再动它

tomcat从web.xml中加载DispatcherServlet，然后会调用它的init()方法

Servlet配置文件默认在/WEB-INF/<servlet-name>-servlet.xml，所以现在默认叫project-servlet.xml

当然，也能自己指定文件



```xml
 <!--以下为加载SpringMVC需要的配置-->
<servlet>
  <servlet-name>project</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <load-on-startup>1</load-on-startup>

   <!--指定配置文件-->
  <init-param>
                   <param-name>contextCOnfigLocation</param-name>
                   <param-value>classPath:spring-servlet.xml</param-value>
  </init-param>

  <servlet-mapping>
        <servlet-name>project</servlet-name>
        <url-pattern>*.html</url-pattern>
  </servlet-mapping>
</servlet>
```

当SpringMVC加载好后，浏览器有请求过来，如果是.html结尾，tomcat就会交给DispatcherServlet处理

而DispatcherServlet会根据路径找到对应的处理器处理，可以理解为我们写的Controller接收到了（具体SpringMVC处理流程会写一篇博文）

至此，浏览器发送请求，到执行我们写的代码这个流程就结束了 ~~撒花~~

参考：https://www.cnblogs.com/top-housekeeper/p/14105297.html
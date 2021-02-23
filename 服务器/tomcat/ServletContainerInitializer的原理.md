# ServletContainerInitializer的原理

最近在项目启动过程遇到一个错误：

java.io.IOException: java.lang.ClassCastException: Cannot cast org.springframework.web.SpringServletContainerInitializer to javax.servlet.ServletContainerInitializer

不能将SpringServletContainerInitializer转化成ServletContainerInitializer，即向上转型失败。

出现以上问题一般是maven项目里面引用了带有ServletContainerInitializer这个类的jar包。

网上说是因为项目中的jar包与tomcat的jar包都有javax.servlet导致的，只要将项目中导入的依赖标注为编译时使用就可以了。即多加 <scope>provided</scope>：

> ```xml
> <dependency>
>     <groupId>javax.servlet</groupId>
>     <artifactId>javax.servlet-api</artifactId>
>     <version>3.1.0</version>
>     <scope>provided</scope>
> </dependency>
> ```

但是我看了一下项目中的依赖早就已经有加这个了。

无奈请教师傅，师傅叫我把tomcat换成7版本，神奇的居然可以了。。。难道是版本冲突问题？带着这个疑问我们来看看ServletContainerInitializer的原理。



我们知道tomcat启动之后会去加载spring容器。但是加载之前我们需要做一些初始化的工作比如：注册servlet或者filtes等。servlet规范中通过ServletContainerInitializer实现此功能。

每个框架（Spring）要使用ServletContainerInitializer就必须在对应的jar包的META-INF/services 目录创建一个名为javax.servlet.ServletContainerInitializer的文件：

![](E:\gitbook\服务器\tomcat\pic\SpringServletContainerInitializer.png)

文件内容指定具体的ServletContainerInitializer实现类：

```
org.springframework.web.SpringServletContainerInitializer
```

那么，当web容器启动时就会运行这个初始化器做一些组件内的初始化工作。

一般伴随着ServletContainerInitializer一起使用的还有HandlesTypes注解，通过HandlesTypes可以将感兴趣的一些类注入到ServletContainerInitializerde的onStartup方法作为参数传入。

```java
@HandlesTypes({WebApplicationInitializer.class})
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    public SpringServletContainerInitializer() {
    }
     public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
      ...
    }
}
```

SpringServletContainerInitializer实现了ServletContainerInitializer，ServletContainerInitializer只有一个方法onStartup，这个方法在tomcat容器启动后执行，那么相对应的框架（Spring）就会去执行其实现类实现的方法进行加载spring容器前的初始化工作。

```java
package javax.servlet;

public interface ServletContainerInitializer {
    void onStartup(Set<Class<?>> var1, ServletContext var2) throws ServletException;
}
```

详细请参考：https://blog.csdn.net/j080624/article/details/80016905



我们康康SpringServletContainerInitializer的onStartup到底做了什么

```java
public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext) throws ServletException {
        List<WebApplicationInitializer> initializers = new LinkedList();
        Iterator var4;
        if (webAppInitializerClasses != null) {
            var4 = webAppInitializerClasses.iterator();
   // webAppInitializerClasses 就是servlet3.0规范中为我们收集的 WebApplicationInitializer 接口的实现类的class
        // 从webAppInitializerClasses中筛选并实例化出合格的相应的类
            while(var4.hasNext()) {
                Class<?> waiClass = (Class)var4.next();
                if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) && WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
                    try {
                        initializers.add((WebApplicationInitializer)waiClass.newInstance());
                    } catch (Throwable var7) {
                        throw new ServletException("Failed to instantiate WebApplicationInitializer class", var7);
                    }
                }
            }
        }

        if (initializers.isEmpty()) {
            servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
        } else {
            servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
              // 这行代码说明我们在实现WebApplicationInitializer可以通过继承Ordered, PriorityOrdered来自定义执行顺序
            AnnotationAwareOrderComparator.sort(initializers);
            var4 = initializers.iterator();
   			// 迭代每个initializer实现的方法
            while(var4.hasNext()) {
                WebApplicationInitializer initializer = (WebApplicationInitializer)var4.next();
                initializer.onStartup(servletContext);
            }

        }
    }
```

SpringServletContainerInitializer 由支持Servlet3.0+的Servlet容器实例化并调用.
Servlet容器还会查询classpath下SpringServletContainerInitializer类上修饰的@HandlesTypes注解所标注的WebApplicationInitializer接口的实现类. 这一步也是容器帮我们完成的.
**SpringServletContainerInitializer通过实现ServletContainerInitializer将自身并入到Servlet容器的生命周期中, 并通过自身定义的WebApplicationInitializer将依赖于Spring框架的系统初始化需求与Servlet容器解耦. 即依赖于spring的系统可以通过实现WebApplicationInitializer来实现自定义的初始化逻辑. 而不需要去实现ServletContainerInitializer**

参考：https://blog.csdn.net/chuixue24/article/details/103462458
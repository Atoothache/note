# 静态变量注入bean，@Autowire的注入时期

## @Autowire的注入顺序

​        一起探讨@Autowired属性注入,设值注入和构造注入的注入时机: https://blog.csdn.net/snailmann/article/details/87930143 

​		以上文章得到的结论是： 注入顺序为 构造注入 > 属性注入 > 设值注入。@Autowired的注入顺序其实是根据类与其对象的生命周期决定的。

​		对象生命周期：一个类在实例化时会先进入实例化阶段所以构造方法会被调用，最先进行构造注入，接着会进入填充属性阶段，进行变量的初始化，这时就会属性注入，最后接着set注入。

​		类生命周期：但是，如果说注入的变量为静态变量那结果就不一样了，类在加载时就会进行静态属性的初始化，这时候就只进行静态属性注入，然而由于静态属性注入在spring容器初始化之前完成，所以静态属性注入是永远不可能实现的。

## 问题例子：

```java
public class ReadResultExcel {
    @Autowired
    private static SyncService syncService;
    public static void parseExcel(Workbook wb){
        syncService.sync(true);
    }
}
```

​		该段代码晃眼一看没啥问题，但是运行就会null异常，因为此处注入的SyncService为null，这是因为SyncService为静态变量，它在ReadResultExcel类加载时就会初始化并通过@Autowired注入，但是这时候spring容器还没有形成，更不用说形成SyncService这个bean了，因此为null。

知道了问题所在，我们就可以分析解决方式了！

## 解决问题：

#### 1.不使用@Autowired，使用SpringContextUtil.getBean手动注入

我们最直接的目的是要在静态方法里面使用bean，那我们就直接在静态方法里面手动注入bean就好啦！不需要了解自动注入顺序啦。注意：静态方法是程序启动时自动放在内存中了，在调用之前不会去做任何操作，所bean是在调用时才去注入的，这时候spring容器和bean实例早就准备好了。

```java
public class ReadResultExcel {
    public static void parseExcel(Workbook wb){
        SyncService syncService = SpringContextUtil.getBean(SyncService.class);
        syncService.sync(true);
    }
}

```

#### 2. @Autowired 用在构造函数上

​		如果非要用自动注入也是可以的。
代码参考：

```java
@Component
public class ReadResultExcel {
    private static SyncService syncService;
    Autowired
    public ReadResultExcel(SyncService syncService）{
        ReadResultExcel.syncService = syncService;
    }
    public static void parseExcel(Workbook wb){
        syncService.sync(true);
    }
}
```

​		这里使用构造注入，有人可能会疑惑，我调用静态方法根本就不要实例化啊，所以构造方法肯定不会执行，何来构造注入一说。没错如果直接调用静态方法确实不会执行构造方法，所以在这里@Component是关键，他表示将ReadResultExcel实例化为spring的一个bean，也就是spring容器启动时就会去实例化ReadResultExcel调用它的构造方法，进而构造注入。

​		注意：一个类里可以使用自动注入其他bean的前提是这个类本身的管理也是要交给spring容器的（线程内部和使用静态方法的工具类都是不交给spring容器的）。你调用这个方法所在的类可能并不是由spring来管理的，也就是说采用@Autowired这种自动注入应该是无效的，在针对这种情形，spring确实提供这样一种途径，就是在无法自动注入的情况下，直接调用beanfactory去拿某个bean的实例，调用这样方法得到的实例是跟自动注入得到的实例是一样的。

#### 3.@Autowired用在属性上   使用 @PostConstruct 注解    

​		@PostConstruct是Java EE 5引入来影响Servlet生命周期的注解，被用来修饰非静态的void()方法，@PostConstruct在构造函数和@Autowired之后执行。		

​		其实从依赖注入的字面意思就可以知道，要将对象p注入到对象a，那么首先就必须得生成对象a和对象p，才能执行注入。所以，如果一个类A中有个成员变量p被@Autowried注解，那么@Autowired注入是发生在A的构造方法执行完之后的。如果想在生成对象时甚至生成对象之前完成某些初始化操作，而偏偏这些初始化操作又依赖于依赖注入，那么就无法在构造函数中跟静态方法中实现。

​		以上这段话可可能有点拗口，看下面例子：如果MyUtils需要实例化成bean，那么它经历的生命周期为，静态属性初始化->调用构造方法实例化->普通属性初始化。也就是说在构造方法里面是没法使用普通属性的，因为它还没初始化，我们只能给他赋值而不能使用。

​		为此，可以使用@PostConstruct注解一个方法来完成初始化，@PostConstruct注解的方法将会在依赖注入完成后被自动调用。Constructor >> @Autowired >> @PostConstruct


代码参考：

```java
@Component
public class MyUtils {

 	private static MyUtils staticInstance = new MyUtils();
 	
	@Autowired
 	private MyMethorClassService myService;
 	
	@PostConstruct
 	public void init(){
  		 staticInstance.myService = myService;   
 	}

	public static Integer invokeBean(){      
  		return staticInstance.myService.add(10,20);
	}
}
```

​	感觉有点投机取巧，人家java语言设计成静态方法调用不了实例变量，但是我们转了一圈似乎又间接调用到了！

<script type="text/javascript">
window.addEventListener("load", function() {
  var click_handle = function() {
    if (this.href.substr(-5) == ".html") {
      location.href = this.href;
    } else {
      location.href = "./index.html";
    }
  };
  var as = document.querySelectorAll(".chapter a, .navigation-prev, .navigation-next");
  for (var i = 0; i < as.length; i++) {
    as[i].addEventListener("click", click_handle, true);
    as[i].title = as[i].innerText;
  }
});
</script>

<script type="text/javascript">
window.addEventListener("load", function() {
  var click_handle = function() {
    if (this.href.substr(-5) == ".html") {
      location.href = this.href;
    } else {
      location.href = "./index.html";
    }
  };
  var as = document.querySelectorAll(".chapter a, .navigation-prev, .navigation-next");
  for (var i = 0; i < as.length; i++) {
    as[i].addEventListener("click", click_handle, true);
    as[i].title = as[i].innerText;
  }
});
</script>


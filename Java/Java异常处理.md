# Java异常处理

参考文献：https://www.cnblogs.com/zsh-blogs/p/10459035.html

### try-catch语句执行过程

```java
try{
//正常执行的代码
}catch (Exception e){
//出错后执行的代码
}finally{
//无论正常执行还是出错,之后都会执行的代码
}
//跟上面try catch无关的代码
```

正常执行的代码如果出现异常,就不会执行出现异常语句后面的所有正常代码。
异常可能会被捕获掉,比如上面catch声明的是捕获Exception,那么所有Exception包括子类都会被捕获,但如Error或者是Throwable但又不是Exception(Exception继承Throwable)就不会被捕获。如果异常被捕获,就会执行catch里面的代码.如果异常没有被捕获,就会往外抛出,相当于这整个方法出现了异常。
finally中的代码只要执行进了try catch永远都会被执行.执行完finally中的代码,如果异常被捕获就会执行外面跟这个try catch无关的代码.否则就会继续往外[抛出异常](https://www.baidu.com/s?wd=抛出异常&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1YLP1fduAPWPWTduyR3ny7-0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPHD4P1bzPWc4PHmkP1D3Pjfd)，不会执行外面跟这个try catch无关的代码。

```java
        try {
            //2.自定义异常
            throw new FdException(ResultEnum.UNKNOWN_ERROR);
        }catch (FdException f){
            f.printStackTrace();
        }
        System.out.println("异常被捕获不受异常影响继续执行" );

    打印：
        com.faduit.riskwarn.exception.FdException: 未知错误
        异常被捕获不受异常影响继续执行
```

```java
        try {
            //1.空指针异常
            Map map = new TreeMap();
            map.putAll(null);
        }catch (FdException f){
            f.printStackTrace();
        }
        System.out.println("异常没被捕获，抛出到外层方法，代码受影响不执行" );

    打印：
        java.lang.NullPointerException
            at java.util.TreeMap.putAll(TreeMap.java:313)
```
return无论在哪里,只要执行到就会返回,但唯一一点不同的是如果return在try或者catch中,即使返回了,最终finally中的代码都会被执行.这种情况最常用的是打开了某些资源后必须关闭,比如打开了一个OutputStream,那就应该在finally中关闭,这样无论有没有出现异常,都会被关闭。

### 不捕获异常的情况

如果方法内部没有捕获异常，那么就会抛出到外层方法，外层再没有捕获就继续往外抛出。。。那么最终到哪里呢？

按照方法调用一层一层的向上传递，如果到达main方法还得不到处理，就被jvm捕获，则程序终止。

当Java虚拟机追溯到调用栈的底部的方法时，如果仍然没有找到处理该异常的代码块，将按以下步骤处理：
1、调用异常对象的printStackTrace()方法，打印来自方法调用栈的异常信息。
2、如果该线程不是主线程，那么终止这个线程，其他线程继续正常运行。如果该线程是主线程（即方法调用栈的底部为main()方法），那么整个应用程序被终止。

### throws作用

当一个方法产生一个它不处理的异常时，那么就需要在该方法的头部声明这个异常，以便将该异常传递到方法的外部进行处理。使用 throws 声明的方法表示此方法不处理异常。使用 throws 声明抛出异常的思路是，当前方法不知道如何处理这种类型的异常，该异常应该由向上一级的调用者处理。强制上一级捕获异常或者继续向上抛异常。

## throw 拋出异常

与 throws 不同的是，throw 语句用来直接拋出一个异常，后接一个可拋出的异常类对象，其语法格式如下：

```
throw ExceptionObject;
```


其中，ExceptionObject 必须是 Throwable 类或其子类的对象。如果是自定义异常类，也必须是 Throwable 的直接或间接子类。例如，以下语句在编译时将会产生语法错误：

```
throw new String("拋出异常");    // String类不是Throwable类的子类
```


当 throw 语句执行时，它后面的语句将不执行，此时程序转向调用者程序，寻找与之相匹配的 catch 语句，执行相应的异常处理程序。如果没有找到相匹配的 catch 语句，则再转向上一层的调用程序。这样逐层向上，直到最外层的异常处理程序终止程序并打印出调用栈情况。

throw 关键字不会单独使用，它的使用完全符合异常的处理机制，但是，一般来讲用户都在避免异常的产生，所以不会手工抛出一个新的异常类的实例，而往往会抛出程序中已经产生的异常类的实例。



**throws 关键字和 throw 关键字在使用上的几点区别如下**：

- throws 用来声明一个方法可能抛出的所有异常信息，表示出现异常的一种可能性，但并不一定会发生这些异常；throw 则是指拋出的一个具体的异常类型，执行 throw 则一定抛出了某种异常对象。
- 通常在一个方法（类）的声明处通过 throws 声明方法（类）可能拋出的异常信息，而在方法（类）内部通过 throw 声明一个具体的异常信息。
- throws 通常不用显示地捕获异常，可由系统自动将所有捕获的异常信息抛给上级方法； throw 则需要用户自己捕获相关的异常，而后再对其进行相关包装，最后将包装后的异常信息抛出。
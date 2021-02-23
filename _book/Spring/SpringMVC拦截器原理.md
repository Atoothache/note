# SpringMVC 拦截器原理

### 前言

```
SpringMVC 拦截器也是Aop(面向切面)思想构建，但不是 Spring Aop 动态代理实现的，
主要采用责任链和适配器的设计模式来实现，直接嵌入到 SpringMVC 入口代码里面。
```

### 流程分析

```
Copy浏览器请求

DispatcherServlet 执行调用 doService(request, response) 作为 Servlet 主要执行者，
doService(request, response) 通过调用 doDispatch(request, response) 来真正执行请求处理

doDispatch(request, response) 中完成拦截器的添加和拦截器拦截处理
    通过 getHandler(HttpServletRequest request) 获取到 HandlerExecutionChain 处理器执行链，
    将拦截器注入到 HandlerExecutionChain 的属性中。
    分别调用 HandlerExecutionChain 的三个方法，applyPreHandle、applyPostHandle、triggerAfterCompletion，
    实现前置拦截/请求提交拦截和请求完成后拦截。
    使用责任链的设计模式，实际调用的是HandleInterceptor的三个接口，分别对应preHandle、postHandle、afterCompletion
```

### HandlerExecutionChain 源码分析

```java
Copypublic class HandlerExecutionChain {

    private final Object handler;
    @Nullable
    private HandlerInterceptor[] interceptors;
    @Nullable
    private List<HandlerInterceptor> interceptorList;
    private int interceptorIndex;

    /**
      按照列表中interceptor的顺序来执行它们的preHandle方法，直到有一个返回false。
      true：表示继续流程（如调用下一个拦截器或处理器）
      返回false后：表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，
      调用triggerAfterCompletion方法，此时this.interceptorIndex指向上一个返回true的interceptor的位置，
      所以它会按逆序执行所有返回true的interceptor的afterCompletion方法。
    */
    boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = 0; i < interceptors.length; this.interceptorIndex = i++) {
                HandlerInterceptor interceptor = interceptors[i];
                if (!interceptor.preHandle(request, response, this.handler)) {
                    this.triggerAfterCompletion(request, response, (Exception)null);
                    return false;
                }
            }
        }

        return true;
    }

    /**
      按照逆序执行所有interceptor的postHandle方法
    */
    void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = interceptors.length - 1; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];
                interceptor.postHandle(request, response, this.handler, mv);
            }
        }

    }

    /**
      从最后一次preHandle成功的interceptor处逆序执行afterCompletion方法。
    */
    void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) throws Exception {
        HandlerInterceptor[] interceptors = this.getInterceptors();
        if (!ObjectUtils.isEmpty(interceptors)) {
            for(int i = this.interceptorIndex; i >= 0; --i) {
                HandlerInterceptor interceptor = interceptors[i];

                try {
                    interceptor.afterCompletion(request, response, this.handler, ex);
                } catch (Throwable var8) {
                    logger.error("HandlerInterceptor.afterCompletion threw exception", var8);
                }
            }
        }

    }
}
```
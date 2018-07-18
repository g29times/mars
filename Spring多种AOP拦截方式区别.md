---
title: (原)Spring多种AOP拦截方式区别
date: 2018-06-26 21:32:16
tags:
---
![design_pattern&frame](1.png)

## 总览
| 方式 | 子类 | 场景 | 
| - | - | 
| 过滤器（Filter）      |  | 可以拿到原始的http请求，但是拿不到你请求的控制器和请求控制器中的方法的信息。 | 
| 拦截器（Interceptor） |  | 可以拿到你请求的控制器和方法，却拿不到请求方法的参数。 | 
|                     | HandlerInterceptor | 针对请求地址做一些验证、预处理等 | 
|                     | MethodInterceptor | 对一些普通的方法上的拦截 | 
| ControllerAdvice    |  |  | 
| 切片  （Aspect）     |  | 可以拿到方法的参数，但是却拿不到http请求和响应的对象。 | 

## 应用
当我们希望记录请求方法的参数，会想到用Interceptor拦截，通过preHandle方法的request是可以获取到请求信息的，但是一旦request.getInputStream()被读取，postHandle将会报错，因为request中的流已被释放。

关于InputStream为什么不能被重复读取？可以看看[这篇文章](http://blog.csdn.net/dreamtdp/article/details/26733563)

## 参考
[谈谈spring中的拦截器](https://blog.csdn.net/hongxingxiaonan/article/details/48090075)
[Spring 拦截器—HandlerInterceptor—xml方式](https://www.cnblogs.com/gl-developer/p/5997508.html)
[Spring 拦截器—HandlerInterceptor—注解方式](https://www.jianshu.com/p/dc5cc2e25ab2)
[Java实现inputstream流的复制](https://blog.csdn.net/qq_25646191/article/details/78856639)
[java对象克隆以及深拷贝和浅拷贝](https://www.cnblogs.com/xuanxufeng/p/6558330.html)
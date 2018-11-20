---
layout:     post
title:      "spring-面向切面AOP"
subtitle:   " \"其实就是拦截器\""
date:       2018-11-08 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - spring
    - spring AOP
    - spring 面向切面
---


# 什么是面向切面
拦截器。

# 怎么使用？
1.管理类 //Advisor（切面）  
管理2和3。  
2.哪些类的哪些方法 //Pointcut（切点）  
3.在方法的前面、后面还是前面/后面 //Advice（增强）  


以上三个问题，在AOP中占用重要的地位，因为Spring AOP的主要工作就是围绕以上三点展开：Spring AOP通过Pointcut（切点）指定在哪些类的哪些方法上织入横切逻辑，通过Advice（增强）描述横切逻辑和方法的具体织入点（方法前、方法后、方法的两端等）。此外，Spring通过Advisor（切面）将Pointcut和Advice两者组装起来。有了Advisor的信息，Spring就可以利用JDK或CGLib的动态代理技术采用统一的方式为目标Bean创建织入切面的代理对象了。 

# 底层实现原理
动态代理。

---
有2种技术，  
1.jdk  //面向接口。      
1）代理类Proxy  
2）业务类InvokerHandler  

2.第三方开源-CGLib  //可以是类。

# 面向切面和控制反转有什么关系吗
没有关系。2个独立的核心模块。1个是注入数据，1个是拦截器什么的。但是二者可以结合使用。

1.注入数据  
思想沿袭自最早的MVC框架-struts。  
2.拦截器  
思想沿袭自j2ee里的servlet规范-过滤器，只不过spring的拦截器框架封装的更完善更好用而已。

# 应用场景
但凡是类似拦截器这样的功能，即拦截每个类做点什么事情，都是面向切面，  
1.日志 //进入方法和退出方法  
2.执行时间 //性能监控  
3.事务 

# 工作应用


# 参考
http://www.iteye.com/topic/1123293

---
layout:     post
title:      "spring-控制反转IoC"
subtitle:   " \"其实就是注入数据\""
date:       2018-11-07 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - spring
    - spring IoC
    - spring 控制反转
---

# 是什么
注入数据。

早在struts框架的时候，struts2也是注入数据。

所以，所谓的控制反转，就是注入数据。不管它叫什么，是叫控制反转，还是依赖注入，本质上都是注入数据。

叫这么一个拗口的名字，不能望文生义的名字都不是好名字。

# 底层实现原理？

# 容器
类继承图  
接口  
1.BeanFactory  
BeanFactory就是代表容器。而且是一个最简单的容器。  
2.ApplicationContext  
继承了BeanFactory。扩充了一些功能，具备应用程序的功能。//一般都是使用这个类的子类，因为支持：1.注入数据2.面向切面3.事务。而BeanFactory都不支持。

实现类  
3.web应用程序
WebApplicationContext   
1）继承了ApplicationContext
2）作用：web应用程序

4.java程序  
类路径XmlApplicationContext/磁盘文件XmlApplicationContext

---
作用  
管理和创建bean。

# bean
BeanDefinition

---
怎么实例化bean？  
有好几种方法。一般使用默认的构造方法。

---
何时初始化？  
1.默认
程序启动/启动spring容器的时候。

2.第一次使用时再实例化  
第一次请求时。怎么做？懒初始化配置。

---
生命周期？  
1.初始化  
程序启动的时候。  
2.销毁  
程序关闭的时候。

---
是否单例？  
默认单例。

---
怎么注入数据？  
1.set方法。   
2.注解 //@Autowire //一般使用以上2种   
3.构造器。  


---
作用域  
1.单例    
单例。  
适用于无状态的类。  
2.原型  
每次请求。  
适用于service类，每次注入的数据(比如，注入到控制器)都是一个新的实例。  
3.请求、会话、上下文  
只能在web应用程序里使用。

区别  
原型和单例的区别？  
单例和多例的区别。

上下文和单例的区别？  
单例也是单例。上下文也是单例。区别是作用域范围不同，单例是spring容器，上下文是web容器。

原型和请求的区别？  
都是多例。但是，原型是service类；而请求是控制器类。

# 架构图
各个模块的关系。

# 如何处理一个请求
就是跟一般的MVC处理流程差不多。

# 工作实践


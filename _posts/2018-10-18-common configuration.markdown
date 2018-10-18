---
layout:     post
title:      "common configuration"
subtitle:   " \"读配置文件\""
date:       2018-10-18 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 支付
---



# 作用

读配置文件。



# jdk Properties也可以读配置文件

1.新建文件流

根据文件名字



2.读配置文件

.load(流)方法



# 那为什么还要使用common 读配置文件包呢

因为更方便。只需要一步即可。

1.设置文件名字.setFileName()。//不需要新建文件流对象，这个开源包的作用就是封装好了这一步。





# 应用场景

最常用的是读.properties配置文件。



# 发展史

1.common configuration 1.0

有.setFileName()方法。



2.common configuration 2.0

使用方式不一样了。



---

具体改变了哪些地方？完善了哪些地方？



# 与jdk Properties的区别？


# common 1.0底层实现

数据存储这一块，就是封装了Map。



jdk-Properties类，继承了Hashtable。

![image.png](https://upload-images.jianshu.io/upload_images/6367548-70774eaacb1fa338.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 最佳实践
1.jdk Properties  
麻烦。

2.开源项目 common configuration  
方便。  
推荐使用。


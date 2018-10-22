---
layout:     post
title:      "java IO和NIO"
subtitle:   " \"文件File、网络包java.net.Socket\""
date:       2018-10-30 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - IO
---

# IO
#### IO
1.按字节和字符
InputStream
OutputStream

2.按写和读
Writer
Reader

3.按数据类型  
1）字节  
2）文件、基于文件的套接字  
3）对象

4.从哪里写和读  
1）磁盘(即文件)  
可以字节和字符。  
2）网络  
只能字节。为什么？


类的继承图基本上也是按上面的1234来的。

---
类继承图  
![](http://pg60ucix6.bkt.clouddn.com/image002.png)

---
文件操作  
1.流程  
物理文件——》文件抽象：文件对象File——》File流(包含文件描述符FileDescriptor)——》读写操作  
                      
                      
2.文件描述符FileDescriptor是什么？作用？

                      
---
字节和字符互相转换  
有专门的类，  
InputStreamReader  
OutputStreaWriter
                      
                      

#### 文件

#### 套接字

# 缓冲区
1.普通类读写慢。  
2.使用缓冲区。

缓冲区为什么快？底层原理？


# 编码和解码
1.写  
编码

2.读  
解码

# 阻塞和非阻塞

# IO的问题

# 随机访问文件
从文件的任意位置开始写读数据。其他的都是顺序写读。

https://docs.oracle.com/javase/tutorial/essential/io/rafs.html

# 为什么需要NIO
包含以下步骤，  
1.具体的流  
文件流  
绑定TCP套接字

2.通道  
网络套接字通道  
本地磁盘文件通道

3.缓冲区  
ByteBuffer  
其他各种基本数据类型缓冲区

4.读写到其他地方

# 什么是NIO
NIO stands for non-blocking I/O.
N是非阻塞的缩写。

# NIO与IO的区别
1.IO慢，哪怕是缓冲类也慢。  
基于流。一次读写一个字节。

2.NIO快。为什么？快在哪里？底层原理？  
基于通道channel。一次读写一个块的数据。


# 参考
https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html
https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html#ma

官方教程  
https://docs.oracle.com/javase/tutorial/essential/io/index.html  
https://docs.oracle.com/javase/7/docs/technotes/guides/io/index.html  
http://openjdk.java.net/projects/nio/


# 应用场景

# 工作实践

# 开源项目
web服务器都是使用了NIO，

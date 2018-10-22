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
如何使用？  
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

代码示例
```
//网络套接字
public void selector() throws IOException {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        Selector selector = Selector.open();
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);//设置为非阻塞方式
        ssc.socket().bind(new InetSocketAddress(8080));
        ssc.register(selector, SelectionKey.OP_ACCEPT);//注册监听的事件
        while (true) {
            Set selectedKeys = selector.selectedKeys();//取得所有key集合
            Iterator it = selectedKeys.iterator();
            while (it.hasNext()) {
                SelectionKey key = (SelectionKey) it.next();
                if ((key.readyOps() & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT) {
                    ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();
                 SocketChannel sc = ssChannel.accept();//接受到服务端的请求
                    sc.configureBlocking(false);
                    sc.register(selector, SelectionKey.OP_READ);
                    it.remove();
                } else if 
                ((key.readyOps() & SelectionKey.OP_READ) == SelectionKey.OP_READ) {
                    SocketChannel sc = (SocketChannel) key.channel();
                    while (true) {
                        buffer.clear();
                        int n = sc.read(buffer);//读取数据
                        if (n <= 0) {
                            break;
                        }
                        buffer.flip();
                    }
                    it.remove();
                }
            }
        }
}
```

```
//三个容易的步骤
//第一步是获取通道。我们从 FileInputStream 获取通道：
FileInputStream fin = new FileInputStream( "readandshow.txt" );
FileChannel fc = fin.getChannel();

//下一步是创建缓冲区：
ByteBuffer buffer = ByteBuffer.allocate( 1024 );

//最后，需要将数据从通道读到缓冲区中，如下所示：
fc.read( buffer );
```

---
底层原理和优点  
基于事件监听。

异步 I/O 的一个优势在于，它允许您同时根据大量的输入和输出执行 I/O。同步程序常常要求助于轮询，或者创建许许多多的线程以处理大量的连接。使用异步 I/O，您可以监听任何数量的通道上的事件，不用轮询，也不用额外的线程。


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

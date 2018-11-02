---
layout:     post
title:      "线程通信之wait和notify"
subtitle:   " \"Object.wait()/notify()\""
date:       2018-10-07 08:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - multi thread
---


# 前言
1.每个对象有一个监视器。  
2.每个对象还有一个线程集合。线程集合只能通过锁对象.wait/notify来操作。

# 作用
可以线程通信的问题。线程通信需要处理同步问题，怎么处理？有2种方法：  
1.wait/notify  
2.同步机制

# 怎么用
1.当前线程必须获取/拥有锁对象  
哪个线程要调用锁对象的wait/notify，它必须要先获取锁对象。

线程对象调用锁对象的wait/notify之前，必须确保当前线程拥有锁对象。
所以，这就导致必须在同步代码块里调用锁对象的wait/notify。因为只有在同步代码块里调用锁对象的wait/notify，当前线程才会拥有锁对象，也才可能调用锁对象的wait/notify。

2.同一个对象  
锁对象和锁对象.wait/notify，必须是同一个对象。否则会报错-非法监视器状态异常。

3.锁对象.wait/notify，必须在同步代码块/方法里  
因为锁对象/监视器 和 锁对象.wait/notify 里的锁对象，必须是同一个对象。即2里说的。

```
//调用wait
synchronized (obj) { //当前线程必须获取锁对象 
         while (<condition does not hold>)
             obj.wait(timeout); //调用wait/notify的对象和锁对象是同一个对象 //调用wait/notify的代码必须在同步代码块里
         
         ... // Perform action appropriate to condition //读线程：读共享数据
     }

//调用notify
synchronized（对象）{
    改变条件  //写线程：写共享数据
     对象.notifyAll();  //激活读线程
}
```

# wait方法
wait()等同于wait(0)。

参数为0，表示无限等待，即一直等待下去，直到被另外一个线程通知。

# what is the difference between wait and sleep、yield？
共同点  
都是停止当前线程执行。

不同点  
1.是否放弃锁
wait //线程A会放弃锁。什么时候恢复？1.在指定时间到之前，有另外一个线程B修改数据之后即完成任务之后调用notify，线程A重新获取锁重新执行。2.一直没有别的线程调用notify，时间到，线程A继续获取锁继续执行。

sleep //线程A不会放弃锁。yield也是，只是交出CPU。在是否放弃锁方面，sleep和yield是一样的，都不放弃锁，而是只放弃CPU。

2.停止时间是否确定
sleep的停止时间是确定的，就是参数指定的时间。什么时候恢复？时间到了就恢复。

yield的停止时间是不确定的，它只是交出CPU，正在等待执行的线程集合里取一个同样优先级的线程来执行。也有可能是再次取的自己。什么时候恢复？时间不确定，依赖CPU和线程调度器。

https://javarevisited.blogspot.com/2011/12/difference-between-wait-sleep-yield.html
https://www.jianshu.com/p/25e959037eed

# wait和join的区别
1.锁对象.wait/notify  
2.线程对象.join——》this.wait——》锁对象是线程对象  
作用？让当前主线程等待，直到对象锁线程1( 指的是主线程调用线程1.join() )执行完毕。接着启动线程2。这样就确保了线程1和线程2按顺序执行。

http://www.importnew.com/14958.html
http://www.java67.com/2017/11/difference-between-wait-and-join-method-of-thread-java.html


# 线程执行完了，意味着什么？
```
public final synchronized void join(long millis)

    throws InterruptedException {

        long base = System.currentTimeMillis();

        long now = 0;



        if (millis < 0) {

            throw new IllegalArgumentException("timeout value is negative");

        }



        if (millis == 0) {

            while (isAlive()) { //对象锁线程1执行完毕，接着执行线程2——这是怎么实现的？看这里代码知道，线程1执行完毕，线程死亡，循环结束，线程1.join()方法执行完毕，继续执行主线程里的代码，即启动线程2。 //这里的问题是主线程在线程1对象锁上等待，而且是无线等待，除非有另外一个线程调用线程1.notify()，否则主线程应该还是等待。线程死了，join()方法执行完毕，这个可以理解——但是为什么线程1死了，join()执行完毕了，为什么主线程重新开始执行了？没有另一个线程去调用线程1.notify()，主线程怎么会活过来呢？线程1退出的时候即exit()的时候会notifyAll()。https://juejin.im/post/5b3054c66fb9a00e4d53ef75 //还有一个问题，调1次wait()不就行了吗？为什么要循环调用？因为调1次也是让主线程等待，调n次还是让主线程等待，所以循环调用的作用是什么？防止虚假唤醒，看API-Object.wait()。至于什么是虚假唤醒，还需要再看维基百科。

                wait(0);

            }

        } else {

            while (isAlive()) {

                long delay = millis - now;

                if (delay <= 0) {

                    break;

                }

                wait(delay);

                now = System.currentTimeMillis() - base;

            }

        }

    }


``` 
https://juejin.im/post/5b3054c66fb9a00e4d53ef75

https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#wait()


# 线程通信的各种场景分析
http://wingjay.com/2017/04/09/Java%E9%87%8C%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%BA%BF%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%EF%BC%9F/


# 参考
<a>https://www.cnblogs.com/stateis0/p/9061611.html</a>

https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html

https://www.kancloud.cn/digest/java-thread/107464









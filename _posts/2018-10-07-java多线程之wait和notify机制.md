---
layout:     post
title:      "java多线程之wait/notify机制"
subtitle:   " \"1.wait引起当前执行线程等待 2.notify唤醒线程 \""
date:       2018-10-07 08:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - multi thread
---


# 应用场景
#### 控制多个线程的执行顺序
多个线程的应用程序，线程执行顺序是没有顺序的，随机的。

如果你想要按指定顺序执行线程，为什么想要按指定顺序执行线程呢？假设2个线程访问共享数据，一个线程依赖另外一个线程的执行结果，所以必须一个先一个后。具体怎么实现一个先一个后？看下文。

---
代码

```
线程1

线程2

主线程{
    线程1.start();
    线程1.join(); //确保线程1和线程2按顺序执行
    线程2.start();
}
```

具体是怎么确保的呢？  
1.线程1.start();  
线程1执行。

2.线程1.join();  
引起主线程等待。

3.线程1执行完成  //线程执行完成之后，jvm会自动调用线程.exit()  
线程.exit()。//唤醒notifyAll所有其他线程

//Thread.exit()
```
/**
     * This method is called by the system to give a Thread
     * a chance to clean up before it actually exits.
     */
    private void exit() {
        if (group != null) {
            group.threadTerminated(this);
            group = null;
        }
        /* Aggressively null out all reference fields: see bug 4006245 */
        target = null;
        /* Speed the release of some of these resources */
        threadLocals = null;
        inheritableThreadLocals = null;
        inheritedAccessControlContext = null;
        blocker = null;
        uncaughtExceptionHandler = null;
    }
    
/**
     * Notifies the group that the thread {@code t} has terminated.
     *
     * <p> Destroy the group if all of the following conditions are
     * true: this is a daemon thread group; there are no more alive
     * or unstarted threads in the group; there are no subgroups in
     * this thread group.
     *
     * @param  t
     *         the Thread that has terminated
     */
    void threadTerminated(Thread t) {
        synchronized (this) { //先获取当前对象的内置锁
            remove(t);

            if (nthreads == 0) {
                notifyAll(); //再唤醒所有线程
            }
            if (daemon && (nthreads == 0) &&
                (nUnstartedThreads == 0) && (ngroups == 0))
            {
                destroy();
            }
        }
    }
```

4.线程2.start();  
线程2执行。

#### 启动程序，并且一直保持运行状态
有的系统比较复杂，比如第三方支付系统，有很多个微服务，有的是web程序，有的是java程序，java程序怎么运行呢？1.使用while(true) 2.使用wait，引起当前主线程一直等等。

---
代码

```
主线程{
    //加载所有spring配置文件：同时，启动了很多的spring线程
    ... //多线程运行：spring容器提供了很多bean服务供本地或外界调用

    //
    while(true){
        MainTest.class.wait(); //引起主线程等待，使得程序一直处于运行状态
    }
}
```


# wait/notify和阻塞的关系
一个线程等待，直到另外一个线程唤醒这个线程；否则，一直阻塞。

# Object.wait与Thread.join的区别
1.当前线程调用wait()，引起当前线程等待 //当前线程就是当前执行线程  
2.thread对象.join()，引起当前线程等待 //当前线程指的是当前执行线程，而不是thread对象.join()的thread对象

#### Object.wait

#### Thread.join
封装了wait()。

//Thread
```
/**
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final synchronized void join(long millis) //先获取当前线程的内置锁
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0); //再引起当前执行线程(即主线程)等待，而不是调用join()的线程对象等待
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

# notify和notifyAll
1.notify //随机唤醒一个线程  
2.notifyAll //唤醒所有其他线程

# wait/notify和对象的内置锁之间的关系
不管是wait还是notify，都必须先获取某个对象的内置锁。看官方文档https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#wait()。

---
官方文档   
17.2. Wait Sets and Notification  
**Every object, in addition to having an associated monitor, has an associated wait set. A wait set is a set of threads**.

When an object is first created, its wait set is empty. Elementary actions that add threads to and remove threads from wait sets are atomic. Wait sets are manipulated solely through the methods Object.wait, Object.notify, and Object.notifyAll.

Wait set manipulations can also be affected by the interruption status of a thread, and by the Thread class's methods dealing with interruption. Additionally, the Thread class's methods for sleeping and joining other threads have properties derived from those of wait and notification actions.

https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html
---
代码

//Thread.join()
```
/**
     * Waits at most {@code millis} milliseconds for this thread to
     * die. A timeout of {@code 0} means to wait forever.
     *
     * <p> This implementation uses a loop of {@code this.wait} calls
     * conditioned on {@code this.isAlive}. As a thread terminates the
     * {@code this.notifyAll} method is invoked. It is recommended that
     * applications not use {@code wait}, {@code notify}, or
     * {@code notifyAll} on {@code Thread} instances.
     *
     * @param  millis
     *         the time to wait in milliseconds
     *
     * @throws  IllegalArgumentException
     *          if the value of {@code millis} is negative
     *
     * @throws  InterruptedException
     *          if any thread has interrupted the current thread. The
     *          <i>interrupted status</i> of the current thread is
     *          cleared when this exception is thrown.
     */
    public final synchronized void join(long millis) //同步方法，先获取当前对象的内置锁
    throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (millis == 0) {
            while (isAlive()) {
                wait(0); //再引起当前执行线程等待
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

---
为什么要必须先获取对象的内置锁，然后再wait/notify呢？  
不是的话，会报错-当前线程不拥有对象的内置锁。

---
jvm为什么要这么设计呢？理由是什么？  
确保同步访问同一个对象的数据。

---
参考  
https://www.jianshu.com/p/f4454164c017

# 参考
https://juejin.im/post/5b3054c66fb9a00e4d53ef75


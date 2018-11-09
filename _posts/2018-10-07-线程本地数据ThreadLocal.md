---
layout:     post
title:      "线程本地数据ThreadLocal"
subtitle:   " \"每个线程都有自己的数据，互不干扰。\""
date:       2018-10-07 09:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - multi thread
---

# 是什么 
顾名思义，线程本地类。

# 作用  
在多线程环境下，每个线程都有一个副本。

如果是普通数据，那么所有线程操作的都是同一份数据，会出现线程同步的问题。

所以，每个线程都有一个副本的好处就是避免线程同步的问题。

# 应用场景  
每个线程都有自己的数据，而不是多线程之间共享这个数据。

1.userId  
2.transactionId  
3.threadId

# 具体怎么使用
```
//jdk7-API官方文档
import java.util.concurrent.atomic.AtomicInteger;

//获取线程id：不是系统默认的线程id，是自定义线程id
 public class ThreadId {
     // Atomic integer containing the next thread ID to be assigned
     private static final AtomicInteger nextId = new AtomicInteger(0); //初始值是0

     // Thread local variable containing each thread's ID
     private static final ThreadLocal<Integer> threadId =
         new ThreadLocal<Integer>() { //匿名对象
             @Override protected Integer initialValue() {
                 return nextId.getAndIncrement(); //第一个线程调用的时候，得到的值是1；第二个线程调用的时候，得到的值是2；后面以此类推
         }
     };

     // Returns the current thread's unique ID, assigning it if necessary
     public static int get() { //获取当前线程的线程id
         return threadId.get(); //每个线程第一次调用ThreadLocal.get()方法时，因为没有初始值，所以都会调用initialValue()方法，得到一个初始值
     }
 }
 ```
 

总结  
1.类  
自定义工具类

2.类包含了线程本地数据  
确保每个线程都有自己的副本数据。//数据属于每个线程自己

3.每个线程获取数据的时候，是通过线程安全的方法去获取数据的   比如，使用并发包-原子类。这样就确保了获取的数据是唯一的。//数据唯一性


# 底层实现  
第一次获取值  
第一次调用get()方法获取值的时候，会调用initialValue()方法。所以，一般需要重写initialValue()方法。

```
//jdk7-ThreadLocal
/**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) { //第一次调用get()，map为null
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null)
                return (T)e.value;
        }
        return setInitialValue(); //第一次调用get()，会调用初始化方法initialValue()得到一个初始值
    }
    
    /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
        T value = initialValue(); //调用初始化方法，得到初始值
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) 
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
    
    /**
     * Returns the current thread's "initial value" for this
     * thread-local variable.  This method will be invoked the first
     * time a thread accesses the variable with the {@link #get}
     * method, unless the thread previously invoked the {@link #set}
     * method, in which case the <tt>initialValue</tt> method will not
     * be invoked for the thread.  Normally, this method is invoked at
     * most once per thread, but it may be invoked again in case of
     * subsequent invocations of {@link #remove} followed by {@link #get}.
     *
     * <p>This implementation simply returns <tt>null</tt>; if the
     * programmer desires thread-local variables to have an initial
     * value other than <tt>null</tt>, <tt>ThreadLocal</tt> must be
     * subclassed, and this method overridden.  Typically, an
     * anonymous inner class will be used.
     *
     * @return the initial value for this thread-local
     */
    protected T initialValue() {
        return null; //默认的初始化方法，值是null。所以需要重写。
    }
```

---
数据保存在哪里？  
有一个数据保存每个线程自己的数据。这个数据使用的数据结构是映射map，jdk自定义了一个新的映射map-ThreadLocalMap。

这个map与其他jdk-map类不同的地方在于，  
1.在哪里？  
这个map类是在ThreadLocal里面，也就是说，这个数据是与每个线程紧密相关的，只是供内部使用。  
2.key？  
基于1的原因，导致ThreadLocalMap实现的时候，它的key是什么呢？是线程本身吗？或者换个问法：

为什么不用个全局的map来实现threadlocal，key为当前threalocal+thread，value为其对应的值。毕竟这个实现简单又易于理解。

这里涉及到对象回收的问题。

所以，jdk使用的是哪一种解决方案呢？   
1》Thread.map map就是ThreadLocalMap。我们说，每个线程都有自己的数据，为什么这么说呢？原因就在于每个线程都有一个自己的数据ThreadLocalMap。ThreadLocalMap存放了真正的数据value。

2》map(ThreadLocal,value) key是ThreadLocal，而ThreadLocal是final，系统全局唯一，即所有线程使用的是同一个key。同一个key，那怎么找到对应线程自己的数据？虽然key是一样的，但是map不一样，这里说的不一样，是说每个线程都有自己的map，每个线程取数据的时候，也是分2步：1.先取自己线程的map 2.再取map的value。所以哪怕key一样，也没有关系。

```
/**
     * ThreadLocalMap is a customized hash map suitable only for
     * maintaining thread local values. No operations are exported
     * outside of the ThreadLocal class. The class is package private to
     * allow declaration of fields in class Thread.  To help deal with
     * very large and long-lived usages, the hash table entries use
     * WeakReferences for keys. However, since reference queues are not
     * used, stale entries are guaranteed to be removed only when
     * the table starts running out of space.
     */
    static class ThreadLocalMap {

        /**
         * The entries in this hash map extend WeakReference, using
         * its main ref field as the key (which is always a
         * ThreadLocal object).  Note that null keys (i.e. entry.get()
         * == null) mean that the key is no longer referenced, so the
         * entry can be expunged from table.  Such entries are referred to
         * as "stale entries" in the code that follows.
         */
        static class Entry extends WeakReference<ThreadLocal> {
            /** The value associated with this ThreadLocal. */
            Object value; //值

            Entry(ThreadLocal k, Object v) { //键是线程本身ThreadLocal
                super(k);
                value = v;
            }
        }
        
    
    /**
         * Set the value associated with key.
         *
         * @param key the thread local object
         * @param value the value to be set
         */
        private void set(ThreadLocal key, Object value) { //写方法

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
    
    /**
         * Get the entry associated with key.  This method
         * itself handles only the fast path: a direct hit of existing
         * key. It otherwise relays to getEntryAfterMiss.  This is
         * designed to maximize performance for direct hits, in part
         * by making this method readily inlinable.
         *
         * @param  key the thread local object
         * @return the entry associated with key, or null if no such
         */
        private Entry getEntry(ThreadLocal key) { //读方法
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }
```

# 工作应用
支付

---
电商

# 参考
https://www.jianshu.com/p/9c03c85db06e

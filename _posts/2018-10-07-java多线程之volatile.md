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



# 同步的几种解决方法
#### synchronized
#### 并发包
#### volatile
没有对象锁。

同步关键字。但又不能真正的同步，也就是说不能排他锁。多个线程可以同时读写。

那有什么作用呢？可以确保线程可见性，一个线程在修改数据，另一个线程始终可以读到最新被修改的值，而不是旧值。

```
volatile 数据;

线程1 修改数据;

线程2 读数据; //读到的始终是最新被线程1修改的值。而如果使用对象锁，可能读到的是旧的值，因为线程1执行过程当中或者修改数据期间，线程2读的数据是旧的。
```



# 很少使用
1.工作使用很少  
2.jdk使用很少  
3.应用场景很少  

# 工作使用

```
package com.gzh.dpp.csp.instruction;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;

import com.gzh.commons.context.AppContext;
import com.gzh.commons.http.HttpClientUtils;

public class MainCSPI {
	
	private static Log log = LogFactory.getLog(MainCSPI.class);
	private static volatile boolean running = true; //作为程序开关
	
	public static void main(String[] args) {
		
		Runtime.getRuntime().addShutdownHook(new Thread() {
			public void run() {
				
				try {					
					
					HttpClientUtils.getInstance().shutdown();
					
					AppContext.stop();
					
					log.info(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss]").format(new Date()) + " Main server stopped!");
				}catch (Throwable t) {
                    log.error("Main stop error", t);
                }
				
				synchronized (MainCSPI.class) {
	                running = false;
	                MainCSPI.class.notify();
	            }
			}
		});
		try {
			AppContext.start();
			//初始化HttpClient连接
			HttpClientUtils httpClient = HttpClientUtils.getInstance();
			httpClient.setConnectionTimeout(8*1000);
			httpClient.setReadTimeout(15*1000);
		} catch (RuntimeException e) {
			log.error(e.getMessage(), e);
			throw e;
		}
		log.info(new SimpleDateFormat("[yyyy-MM-dd HH:mm:ss]").format(new Date()) + " Main server started!");
		
		synchronized (MainCSPI.class) {
			while (running) { //让程序一直处于运行的主状态
	            try {
	            	MainCSPI.class.wait();
	            } catch (Throwable e) {
	            }
	        }
		}
		
	}

}

```

# jdk使用
并发包。

# 应用场景
#### 作为程序开关(启动/关闭)的布尔值

#### 满足以下2个条件，可以使用
volatile看起来简单，但是要想理解它还是比较难的，这里只是对其进行基本的了解。volatile相对于synchronized稍微轻量些，在某些场合它可以替代synchronized，但是又不能完全取代synchronized，只有在某些场合才能够使用volatile。

使用它必须满足如下两个条件：  
1.对变量的写操作不依赖当前值；  
2.该变量没有包含在具有其他变量的不变式中。

volatile经常用于两个两个场景：状态标记两、double check

# 面试题
8. Java中的Volatile关键是什么作用？怎样使用它？在Java中它跟synchronized方法有什么不同？

自从Java 5和Java内存模型改变以后，基于volatile关键字的线程问题越来越流行。应该准备好回答关于volatile变量怎样在并发环境中确保可见性、顺序性和一致性。

---
总结  
1.优点  
确保读线程始终可以读到写线程的最新值。即确保所谓的线程可见性，线程执行顺序。

2.缺点  
没有锁，不能同步。即不能确保所谓的数据一致性。

# 参考
http://cmsblogs.com/?p=2092

https://www.ibm.com/developerworks/cn/java/j-jtp06197.html

https://docs.oracle.com/javase/specs/jls/se7/html/jls-8.html#jls-8.3.1.4


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

上面讲的是只有一个线程本地数据的情况下，key其实是一样的，因为是同一个线程本地数据。所有的线程使用的也是同一个线程本地数据。

那么能不能定义多个线程本地数据？可以。如果定义了多个线程本地数据，key又是什么样的？每个线程本地数据都有不同的key，因为现在是多个线程本地数据，所有的线程都可以访问多个线程本地数据，获取的时候，获取到的就是不同的线程本地数据，因为写的时候是根据当前ThreadLocal.set(this,value)写进去的，所以ThreadLocal.get()的时候获取到的就是当前ThreadLocal。

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
#### 支付
所有的服务/业务，只有1个地方使用到了线程本地数据。

//定义：类-静态数据(线程安全数据)
```


package com.gzh.dpp.dts.manage;



public class BusinessIDManager {

	

	private static ThreadLocal<String> businessID = new ThreadLocal<String>(); //线程安全数据 //这个数据为什么要线程安全？这个数据是分布式事务的流水id，每个用户的每个请求都是一个单独的线程，所以应该作为线程本地数据。

	

	public static void setBusinessId(String id) {

		businessID.set(id);

	}

	

	public static String getBusinessId() {

		return businessID.get();

	}

	

	public static void remove() {

		businessID.remove();

	}



}


```

//写数据
```
public String beginTransaction(MainBusinessActivity businessActivity) {		
		try {
			String transactionId = businessService.createBusinessActivity(businessActivity);
			BusinessIDManager.setBusinessId(businessActivity.getTransactionId()); //写数据
			
			return transactionId;
		} catch(Exception ex) {
			log.error("DTS:begin business transaction exception.", ex);
			return null;
		}		
	}
```

//读数据
```
/**
	 * 重新提交业务活动
	 * @param transactionId
	 * @return
	 */
	public boolean reconfirmTransaction(String transactionId) {
		log.info("DTS:reconfirm transaction:"+transactionId);
		
		try {
			MainBusinessActivity activity = businessService.getBusinessActivity(transactionId, true);
			log.info(activity.toString());
			
			if(!HandleFailureMode.RECONFIRM.getMode().equals(activity.getHandleFailure())) {
				log.warn("Handle mode of business failured is not reconfirm,ID is "+transactionId);
				return false;
			}
			
			//重新提交分支业务
			List<SubBusinessAction> reconfirmList = new ArrayList<SubBusinessAction> ();
			for(SubBusinessAction subAction : activity.getActionList()) {
				//查询执行结果
				String serviceName = subAction.getServiceName();
				Object serviceObj = AppContext.getBean(serviceName);
				CurrentSubAction currentAction = (CurrentSubAction)MethodUtils.invokeMethod(serviceObj, "getCurrentAction", new String[] {transactionId, serviceName});
				if(currentAction == null) {
					reconfirmList.add(subAction);
				}					
			}
			//没有执行成功的分支服务
			if(activity.getActionList().size() == reconfirmList.size()) {
				endTransaction(transactionId);
				return false;
			}
			
			for(SubBusinessAction subAction : reconfirmList) {
				String serviceName = subAction.getServiceName();
				Object serviceObj = AppContext.getBean(serviceName);
				
				String reconfirmMethodName = subAction.getMethodName();
				ISerializer serializer = SerializerFactory.getSerializer();
				
				String serializerStr = subAction.getParamsValue();				
				Object paramsValue = serializer.mergeFrom(serializerStr);
				
				if(BusinessIDManager.getBusinessId() == null) {
					BusinessIDManager.setBusinessId(transactionId);
				} else if(!transactionId.equals(BusinessIDManager.getBusinessId())) {
					log.warn("transactionId:"+transactionId + " ThreadLocal:"+BusinessIDManager.getBusinessId());
					BusinessIDManager.setBusinessId(transactionId);
				}
				
				Object resultObj = MethodUtils.invokeMethod(serviceObj, reconfirmMethodName, paramsValue);
				log.info(new StringBuilder("reconfirm transaction[id=").append(transactionId).append(",serviceName=").append(serviceName)
						.append("] execute result is").append(resultObj).toString());
			}			
			
		} catch(Exception ex) {
			log.error("DTS:reconfirm transaction["+transactionId+"] exception.", ex);			
			
			return false;
		} finally {
			BusinessIDManager.remove();
		}
		
		//重新提交成功，结束业务活动
		endTransaction(transactionId);		
		
		return true;
	}
```

注：写数据和读数据的完整代码。
```


package com.gzh.dpp.dts.manage;



import java.util.ArrayList;

import java.util.List;



import org.apache.commons.lang.reflect.MethodUtils;

import org.apache.commons.logging.Log;

import org.apache.commons.logging.LogFactory;



import com.gzh.commons.serializer.ISerializer;

import com.gzh.commons.serializer.SerializerFactory;

import com.gzh.dpp.dts.enums.ActionStatus;

import com.gzh.dpp.dts.enums.ActivityStatus;

import com.gzh.dpp.dts.enums.HandleFailureMode;

import com.gzh.dpp.dts.model.CurrentSubAction;

import com.gzh.dpp.dts.model.MainBusinessActivity;

import com.gzh.dpp.dts.model.SubBusinessAction;

import com.gzh.dpp.dts.service.IBusinessService;

import com.gzh.commons.context.AppContext;



public class TransactionManager implements ITransactionManager, IExceptionTransactionService {

	private static Log log = LogFactory.getLog(TransactionManager.class);

	

	private IBusinessService businessService;

	public void setBusinessService(IBusinessService businessService) {

		this.businessService = businessService;

	}	

	

	public String beginTransaction(MainBusinessActivity businessActivity) {		

		try {

			String transactionId = businessService.createBusinessActivity(businessActivity);

			BusinessIDManager.setBusinessId(businessActivity.getTransactionId());

			

			return transactionId;

		} catch(Exception ex) {

			log.error("DTS:begin business transaction exception.", ex);

			return null;

		}		

	}

	

	public boolean endTransaction(String transactionId) {

		BusinessIDManager.remove();	

		

		try {

			return businessService.updateBusinessActivityEnd(transactionId);

		} catch(Exception ex) {

			log.error("DTS:end business transaction exception.", ex);

			return false;

		}		

	}

	

	/**

	 * 取消回滚业务活动

	 * @param transactionId  业务活动唯一标识

	 * @return

	 */

	public boolean rollBackTransaction(String transactionId) {

		log.info("DTS:rollback transaction:"+transactionId);

		

		BusinessIDManager.remove();		

		

		try {

			MainBusinessActivity activity = businessService.getBusinessActivity(transactionId, true);

			log.info(activity.toString());

			

			if(!HandleFailureMode.CANCEL.getMode().equals(activity.getHandleFailure())) {

				log.warn("Handle mode of business failured is not cancel,ID is "+transactionId);

				return false;

			}

			

			//失败回滚分支业务

			for(SubBusinessAction subAction : activity.getActionList()) {

				//查询执行结果

				String serviceName = subAction.getServiceName();

				Object serviceObj = AppContext.getBean(serviceName);

				CurrentSubAction currentAction = (CurrentSubAction)MethodUtils.invokeMethod(serviceObj, "getCurrentAction", new String[] {transactionId, serviceName});

				if(currentAction != null && currentAction.getActionStatus().equals(ActionStatus.CONFIRM.getStatus())) {

					String cancelMethodName = subAction.getMethodName();

					ISerializer serializer = SerializerFactory.getSerializer();

					

					String serializerStr = subAction.getParamsValue();					

					Object paramsValue = serializer.mergeFrom(serializerStr);

					String[] txParams = new String[] {transactionId, serviceName};

					Object resultObj = MethodUtils.invokeMethod(serviceObj, cancelMethodName, new Object[] {paramsValue, txParams});

					log.info(new StringBuilder("rollback transaction[id=").append(transactionId).append(",serviceName=").append(serviceName)

							.append("] execute result is").append(resultObj).toString());

				}					

			}			

			

		} catch(Exception ex) {

			log.error("DTS:rollback transaction["+transactionId+"] exception.", ex);

			return false;

		}

		

		//回滚成功，结束业务活动

		endTransaction(transactionId);

		

		return true;

	}

	

	/**

	 * 重新提交业务活动

	 * @param transactionId

	 * @return

	 */

	public boolean reconfirmTransaction(String transactionId) {

		log.info("DTS:reconfirm transaction:"+transactionId);

		

		try {

			MainBusinessActivity activity = businessService.getBusinessActivity(transactionId, true);

			log.info(activity.toString());

			

			if(!HandleFailureMode.RECONFIRM.getMode().equals(activity.getHandleFailure())) {

				log.warn("Handle mode of business failured is not reconfirm,ID is "+transactionId);

				return false;

			}

			

			//重新提交分支业务

			List<SubBusinessAction> reconfirmList = new ArrayList<SubBusinessAction> ();

			for(SubBusinessAction subAction : activity.getActionList()) {

				//查询执行结果

				String serviceName = subAction.getServiceName();

				Object serviceObj = AppContext.getBean(serviceName);

				CurrentSubAction currentAction = (CurrentSubAction)MethodUtils.invokeMethod(serviceObj, "getCurrentAction", new String[] {transactionId, serviceName});

				if(currentAction == null) {

					reconfirmList.add(subAction);

				}					

			}

			//没有执行成功的分支服务

			if(activity.getActionList().size() == reconfirmList.size()) {

				endTransaction(transactionId);

				return false;

			}

			

			for(SubBusinessAction subAction : reconfirmList) {

				String serviceName = subAction.getServiceName();

				Object serviceObj = AppContext.getBean(serviceName);

				

				String reconfirmMethodName = subAction.getMethodName();

				ISerializer serializer = SerializerFactory.getSerializer();

				

				String serializerStr = subAction.getParamsValue();				

				Object paramsValue = serializer.mergeFrom(serializerStr);

				

				if(BusinessIDManager.getBusinessId() == null) {

					BusinessIDManager.setBusinessId(transactionId);

				} else if(!transactionId.equals(BusinessIDManager.getBusinessId())) {

					log.warn("transactionId:"+transactionId + " ThreadLocal:"+BusinessIDManager.getBusinessId());

					BusinessIDManager.setBusinessId(transactionId);

				}

				

				Object resultObj = MethodUtils.invokeMethod(serviceObj, reconfirmMethodName, paramsValue);

				log.info(new StringBuilder("reconfirm transaction[id=").append(transactionId).append(",serviceName=").append(serviceName)

						.append("] execute result is").append(resultObj).toString());

			}			

			

		} catch(Exception ex) {

			log.error("DTS:reconfirm transaction["+transactionId+"] exception.", ex);			

			

			return false;

		} finally {

			BusinessIDManager.remove();

		}

		

		//重新提交成功，结束业务活动

		endTransaction(transactionId);		

		

		return true;

	}



	@Override

	public boolean handleExceptionTransaction(String transactionId) {

		log.info("DTS:handle exception transaction:"+transactionId);

		

		try {

			MainBusinessActivity activity = businessService.getBusinessActivity(transactionId, true);

			log.info(activity.toString());

			if(!ActivityStatus.BEGIN.getStatus().equals(activity.getStatus())) {

				log.warn("transaction status is not 'begin':"+transactionId);

				return false;

			}

			

			

			//已执行的分支服务

			List<CurrentSubAction> currentActionList = new ArrayList<CurrentSubAction>();

			//未执行的分支服务

			List<SubBusinessAction> unexecutedActionList = new ArrayList<SubBusinessAction>();

			for(SubBusinessAction subAction : activity.getActionList()) {

				//查询执行结果

				String serviceName = subAction.getServiceName();

				Object serviceObj = AppContext.getBean(serviceName);

				CurrentSubAction currentAction = (CurrentSubAction)MethodUtils.invokeMethod(serviceObj, "getCurrentAction", new String[] {transactionId, serviceName});

				if(currentAction != null) {

					currentActionList.add(currentAction);

				} else {

					unexecutedActionList.add(subAction);

				}

			}

			

			//若无任何分支服务，结束业务活动

			if(currentActionList.size() == 0) {

				return endTransaction(transactionId);

			}

			//若各分支服务都有执行并状态一致，结束业务活动

			if(activity.getActionList().size() == currentActionList.size()) {

				String tmpStatus = null;

				boolean flag = true;

				for(CurrentSubAction currentAction : currentActionList) {

					String status = currentAction.getActionStatus();

					if(tmpStatus !=null && !status.equals(tmpStatus)) {

						flag = false;

						break;

					}

					tmpStatus = status;

				}

				if(flag) {

					return endTransaction(transactionId);

				}

			}

			

			//失败处理方式为CANCEL，依次将状态为CONFIRM的分支服务回滚

			if(HandleFailureMode.CANCEL.getMode().equals(activity.getHandleFailure())) {

				for(CurrentSubAction currentAction : currentActionList) {

					if(currentAction != null && currentAction.getActionStatus().equals(ActionStatus.CONFIRM.getStatus())) {

						String serviceName = currentAction.getServiceName();

						Object serviceObj = AppContext.getBean(serviceName);

						

						SubBusinessAction subAction = getSubActionForList(transactionId, serviceName, activity.getActionList());

						

						String cancelMethodName = subAction.getMethodName();

						ISerializer serializer = SerializerFactory.getSerializer();						

						String serializerStr = subAction.getParamsValue();					

						Object paramsValue = serializer.mergeFrom(serializerStr);

						String[] txParams = new String[] {transactionId, serviceName};

						Object resultObj = MethodUtils.invokeMethod(serviceObj, cancelMethodName, new Object[] {paramsValue, txParams});

						log.info(new StringBuilder("handle rollback transaction[id=").append(transactionId).append(",serviceName=").append(serviceName)

								.append("] execute result is").append(resultObj).toString());						

					}

				}

			}

			

			//失败处理方式为RECONFIRM，依次尝试提交未完成的分支服务

			if(HandleFailureMode.RECONFIRM.getMode().equals(activity.getHandleFailure())) {

				for(SubBusinessAction subAction : unexecutedActionList) {

					String serviceName = subAction.getServiceName();

					Object serviceObj = AppContext.getBean(serviceName);

					

					String reconfirmMethodName = subAction.getMethodName();

					ISerializer serializer = SerializerFactory.getSerializer();

					

					String serializerStr = subAction.getParamsValue();				

					Object paramsValue = serializer.mergeFrom(serializerStr);

					

					if(BusinessIDManager.getBusinessId() == null) {

						BusinessIDManager.setBusinessId(transactionId);

					} else if(!transactionId.equals(BusinessIDManager.getBusinessId())) {

						log.warn("transactionId:"+transactionId + " ThreadLocal:"+BusinessIDManager.getBusinessId());

						BusinessIDManager.setBusinessId(transactionId);

					}

					

					Object resultObj = MethodUtils.invokeMethod(serviceObj, reconfirmMethodName, paramsValue);

					log.info(new StringBuilder("handle reconfirm transaction[id=").append(transactionId).append(",serviceName=").append(serviceName)

							.append("] execute result is").append(resultObj).toString());					

				}

			}			

			

		} catch(Exception ex) {

			log.error("DTS:handel exception transaction["+transactionId+"] exception.", ex);

			

			return false;

		} finally {

			BusinessIDManager.remove();

		}		

		

		return endTransaction(transactionId);

	}

	

	private SubBusinessAction getSubActionForList(String transactionId,String serviceName, List<SubBusinessAction> actionList) {

		for(SubBusinessAction subAction : actionList) {

			if(subAction.getTransactionId().equals(transactionId) && subAction.getServiceName().equals(serviceName)) {

				return subAction;

			}

		}

		

		return null;

	}

	

	



}




```

#### 电商  
有2个地方用到了线程本地数据。


1.Invoke类
类struts框架，线程本地数据是invoke。  
invoke作用是，调用过滤器链——》控制器类XXXAction。

---
invoke为什么要线程本地？  
1）过滤器  
2）过滤器里的数据invoke
过滤器是单例，如果里面有数据，则需要使用线程本地数据，避免线程安全问题。

---
代码


//写数据和读数据-过滤器
```
package com.bigning.fantastic.filter;

import java.io.IOException;
import java.util.Date;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.log4j.Logger;

import com.bigning.encrypt.PropertyEncrypter;
import com.bigning.fantastic.action.ActionInvoker;
import com.bigning.fantastic.action.ActionMngr;
import com.bigning.fantastic.action.FantasticActionInvoker;
import com.bigning.fantastic.action.FantasticForwardController;
import com.bigning.fantastic.action.LicenseExpiredActionInvoker;
import com.bigning.fantastic.log.Log4jMngr;
import com.bigning.util.AppSetting;
import com.bigning.util.DateFormatFactory;
/**
 * Fantastic 控制中心
 * @author adm
 *
 */
public class FantasticFilter implements Filter{
	// 许可证格式： loveNicoleForever.yyyy-MM-dd
	private static Pattern licenseReg=Pattern.compile("^loveNicoleForever\\.(\\d{4}\\-\\d{1,2}-\\d{1,2})$",Pattern.CASE_INSENSITIVE);
	private static Logger logger;
	private static final ThreadLocal<ActionInvoker> INVOKER=new ThreadLocal<ActionInvoker>(); //定义线程本地数据
	ServletContext ctx;
	ActionInvoker invoker;
	public void destroy() {
		ActionMngr.destroy();
		ctx=null;
		logger.info(getClass().getName()+" is destroyed");
	}
	public void doFilter(ServletRequest arg0, ServletResponse arg1,final FilterChain arg2) throws IOException, ServletException {
		ActionInvoker inst;
		synchronized(INVOKER){
			inst=INVOKER.get(); //如果存在，就直接使用
			if(inst==null){ //如果不存在，就写数据
				inst=(ActionInvoker)invoker.clone(); //init方法里：invoker=new FantasticActionInvoker();
				INVOKER.set(inst); 
			}
		}
		inst.invoke(ctx, (HttpServletRequest)arg0, (HttpServletResponse)arg1,new FantasticForwardController(){

			public void forward(HttpServletRequest request,HttpServletResponse response, String uri) throws IOException, ServletException {
				logger.info("Filter: Use Default processer for "+uri);
				arg2.doFilter(request, response);				
			}
			});
		
	}
	

	public void init(FilterConfig arg0) throws ServletException {
		// 初始化日志记录
		Log4jMngr.init();
		logger=Logger.getLogger("com.bigning.fantastic.filter.FantasticFilter");
		ctx=arg0.getServletContext();
		// 处理许可证
		String license=AppSetting.get("fantastic.license.key");
		boolean valid=false;
		try{
			PropertyEncrypter encrypter=PropertyEncrypter.getInstance();
			String desc=encrypter.decrypt(license);
			Matcher m=licenseReg.matcher(desc);
			if(m.find()){
				String dd=m.group(1);
				Date d=DateFormatFactory.getInstance("yyyy-MM-dd").parse(dd);
				if(d.after(new Date())){
					valid=true;
					logger.info("License will be expired on "+dd);
				}else{
					logger.fatal("License is already expired on "+dd);
				}
			}
						
		}catch(Exception e){
			logger.error("Cannot restore license key:"+license, e);
		}
		// 设置 Invoker
		if(valid){
			ActionMngr.init();
			invoker=new FantasticActionInvoker(); 
		}else{
			invoker=new LicenseExpiredActionInvoker();
		}
		// 清除登录记录
		logger.info(getClass().getName()+" is initialized.");
	}

}


```






2.作用域对象  
为什么要线程本地？  
1.请求  
2.响应  
3.上下文  

为了在控制器类之外的类，比如service和util类里面访问servlet作用域对象，就定义了类.静态数据，以方便在任何地方都可以访问静态类的数据。静态类包含了线程本地数据，线程本地数据封装了上面的3个对象。静态类的数据为什么要是线程本地数据？因为作用域对象是属于当前请求的，每次请求都会分配一个新的线程，所以作用域对象数据要弄成线程本地数据。

在哪里写？invoke的时候写数据。  
在哪里读？service和util的时候，读数据。

除了这种方式，还有其他方法写读作用域对象吗？有。而且一般的MVC框架，比如struts，还有电商项目自己实现的框架-类struts，都是这么实现的：1.BaseAction：封装了作用域对象 2.业务Action继承BaseAction，就可以在控制器Action里访问作用域对象了。

如果想要在service和util类访问作用域对象，可以把数据作为参数传过去，不过这样就是有点麻烦，每次需要传参，而且方法的定义也多了一个参数。

如果是线程本地数据的这种解决方法，那么不管在任何地方，直接使用就可以了，类.静态数据-线程本地数据.作用域对象。

---
代码
//线程本地数据定义的地方
```
package com.bigning.fantastic.action;

import javax.servlet.ServletContext;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class FantasticAppContext {
	final static ThreadLocal<FantasticContext> session=new ThreadLocal<FantasticContext>(); //线程本地数据，它封装了servlet作用域对象
	
	static void set(FantasticContext ctx){
		session.set(ctx);
	}
	public static void set(ServletContext sc, HttpServletRequest request, HttpServletResponse response){
		set(new FantasticContext(sc,request,response));
	}
	public static HttpServletRequest getRequest(){
		return session.get().request;
	}
	public static HttpServletResponse getResponse(){
		return session.get().response;
	}
	public static ServletContext getServletContext(){
		return session.get().sc;
	}
	public static void remove(){
		session.remove();
	}
}
```

//线程本地数据类
```
package com.bigning.fantastic.action;

import javax.servlet.ServletContext;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class FantasticContext {
	final ServletContext sc;
	final HttpServletRequest request;
	final HttpServletResponse response;
	FantasticContext(ServletContext sc, HttpServletRequest request, HttpServletResponse response){
		this.sc=sc;
		this.request=request;
		this.response=response;
	}
	public ServletContext getServletContext() {
		return sc;
	}
	public HttpServletRequest getRequest() {
		return request;
	}
	public HttpServletResponse getResponse() {
		return response;
	}
	
}

```

//写数据-FantasticActionInvoker
```
private void invoke()throws IOException,ServletException{
		if(filtered()){
			return;
		}
		request.setCharacterEncoding("UTF-8");
		response.setCharacterEncoding("UTF-8");
		HttpServletRequest req=null;
		try{
			req=processMultipart(request);
		}catch(Exception e){
			throw new ServletException("Error in processing Multipart form data for "+request.getRequestURI(),e);
		}
		try{
			FantasticAppContext.set(ctx, req, response); //每次请求每次都有一个线程，每次都要到这里写数据
			invokeImpl(req);
		}
		finally{
			FantasticAppContext.remove();
		}
```


//读数据-service类
```
package com.ppet.ord.intf.impl;

import javax.servlet.http.HttpServletRequest;

import com.bigning.fantastic.action.FantasticAppContext;
import com.ppet.attachment.Attached;
import com.ppet.attachment.AttachmentUtil;
import com.ppet.ord.AbstractOrderService;
import com.ppet.ord.SalesOrder;
import com.ppet.ord.intf.IOrderService;
import com.ppet.ord.so.AttachmentUploadForm;

public class UploadAttachmentService  extends AbstractOrderService implements IOrderService<AttachmentUploadForm>{

	public void service(SalesOrder var, AttachmentUploadForm model, HttpServletRequest request) throws Exception {
		Boolean updated=Boolean.FALSE;
		if( model.getAttachment() != null ){
			Attached []files={
				new Attached(model.getAttachment(), model.getAttachmentFilename())
			};
			String path=AttachmentUtil.getUploadPath(var.getCustCode())+var.getOrderNo()+"/";
			AttachmentUtil.store(FantasticAppContext.getServletContext(), var, files, path, null); //读作用域对象
			updated=Boolean.TRUE;
		}
		request.setAttribute("FLAG", updated);
		request.setAttribute("form", var);
	}

}

```


# 开源项目
struts2框架。

和电商项目自己实现的MVC框架差不多。

---
参考
《深入剖析struts2原理》-作者陆周

# 参考
https://www.jianshu.com/p/9c03c85db06e

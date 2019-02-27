---
layout:     post
title:      "spring之控制反转IoC"
subtitle:   " \"其实就是注入数据\""
date:       2018-11-07 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - spring
---

# 是什么
注入数据。

早在struts框架的时候，struts2也是注入数据。

所以，所谓的控制反转，就是注入数据。不管它叫什么，是叫控制反转，还是依赖注入，本质上都是注入数据。

叫这么一个拗口的名字，不能望文生义的名字都不是好名字。

# 底层实现原理？
注入数据是如何实现的？最终还是要看spring源码的。  
反射。

---
步骤  
1.启动  

2.加载配置文件  
解析配置文件。怎么解析？使用jdk-xml工具类。    

3.创建和加载bean  
创建bean，并且放到容器里。  

如何使用/访问bean？1.容器.getBean(name)；//因为启动的时候已经创建了bean，所以用的时候直接从map里取bean即可。2.注入数据。//注入的数据相当于有2个bean，把1个bean注入到另外一个bean，怎么注入？也是从容器里取即可，因为所有的bean在启动程序的时候都已经完成创建，并且存起来了(存在map里)，用的时候取即可，注入数据也是这么取的。  

怎么知道这个时候已经创建bean？使用jdk自带的监控工具jvisualVM，可以看到自定义bean已经被创建了对象，而且只有1个对象。这也刚好证明了默认是单例。

---
数据具体是怎么存放的？
```
	/** Map from bean name to merged RootBeanDefinition */
	private final Map<String, RootBeanDefinition> mergedBeanDefinitions =
			new ConcurrentHashMap<String, RootBeanDefinition>(64); //这里的map只是存放的BeanDefinition，这个时候bean实例对象还没有创建
```

```
/**
 * Generic registry for shared bean instances, implementing the
 * {@link org.springframework.beans.factory.config.SingletonBeanRegistry}.
 * Allows for registering singleton instances that should be shared
 * for all callers of the registry, to be obtained via bean name.
 *
 * <p>Also supports registration of
 * {@link org.springframework.beans.factory.DisposableBean} instances,
 * (which might or might not correspond to registered singletons),
 * to be destroyed on shutdown of the registry. Dependencies between
 * beans can be registered to enforce an appropriate shutdown order.
 *
 * <p>This class mainly serves as base class for
 * {@link org.springframework.beans.factory.BeanFactory} implementations,
 * factoring out the common management of singleton bean instances. Note that
 * the {@link org.springframework.beans.factory.config.ConfigurableBeanFactory}
 * interface extends the {@link SingletonBeanRegistry} interface.
 *
 * <p>Note that this class assumes neither a bean definition concept
 * nor a specific creation process for bean instances, in contrast to
 * {@link AbstractBeanFactory} and {@link DefaultListableBeanFactory}
 * (which inherit from it). Can alternatively also be used as a nested
 * helper to delegate to.
 *
 * @author Juergen Hoeller
 * @since 2.0
 * @see #registerSingleton
 * @see #registerDisposableBean
 * @see org.springframework.beans.factory.DisposableBean
 * @see org.springframework.beans.factory.config.ConfigurableBeanFactory
 */
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {


	/** Cache of singleton objects: bean name --> bean instance */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64); //bean实例对象存放在这个map里
	
	/**
	 * Return the (raw) singleton object registered under the given name,
	 * creating and registering a new one if none registered yet.
	 * @param beanName the name of the bean
	 * @param singletonFactory the ObjectFactory to lazily create the singleton
	 * with, if necessary
	 * @return the registered singleton object
	 */
	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "'beanName' must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while the singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
				beforeSingletonCreation(beanName);
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<Exception>();
				}
				try {
					singletonObject = singletonFactory.getObject(); //创建bean对象
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
					afterSingletonCreation(beanName);
				}
				addSingleton(beanName, singletonObject);
			}
			return (singletonObject != NULL_OBJECT ? singletonObject : null);
		}
	}
	
	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return (singletonObject != NULL_OBJECT ? singletonObject : null);
	}

```

```
public abstract class BeanUtils {

/**
	 * Convenience method to instantiate a class using the given constructor.
	 * As this method doesn't try to load classes by name, it should avoid
	 * class-loading issues.
	 * <p>Note that this method tries to set the constructor accessible
	 * if given a non-accessible (that is, non-public) constructor.
	 * @param ctor the constructor to instantiate
	 * @param args the constructor arguments to apply
	 * @return the new instance
	 * @throws BeanInstantiationException if the bean cannot be instantiated
	 */
	public static <T> T instantiateClass(Constructor<T> ctor, Object... args) throws BeanInstantiationException {
		Assert.notNull(ctor, "Constructor must not be null");
		try {
			ReflectionUtils.makeAccessible(ctor);
			return ctor.newInstance(args); //使用反射技术，创建bean对象
		}
		catch (InstantiationException ex) {
			throw new BeanInstantiationException(ctor.getDeclaringClass(),
					"Is it an abstract class?", ex);
		}
		catch (IllegalAccessException ex) {
			throw new BeanInstantiationException(ctor.getDeclaringClass(),
					"Is the constructor accessible?", ex);
		}
		catch (IllegalArgumentException ex) {
			throw new BeanInstantiationException(ctor.getDeclaringClass(),
					"Illegal arguments for constructor", ex);
		}
		catch (InvocationTargetException ex) {
			throw new BeanInstantiationException(ctor.getDeclaringClass(),
					"Constructor threw exception", ex.getTargetException());
		}
	}
```

```
/**
	 * Instantiate the given bean using its default constructor.
	 * @param beanName the name of the bean
	 * @param mbd the bean definition for the bean
	 * @return BeanWrapper for the new instance
	 */
	protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
		try {
			Object beanInstance;
			final BeanFactory parent = this;
			if (System.getSecurityManager() != null) {
				beanInstance = AccessController.doPrivileged(new PrivilegedAction<Object>() {
					public Object run() {
						return getInstantiationStrategy().instantiate(mbd, beanName, parent);
					}
				}, getAccessControlContext());
			}
			else {
				beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent); //创建bean对象成功。但是这个时候还没有注入数据，那什么时候注入的数据呢？
			}
			BeanWrapper bw = new BeanWrapperImpl(beanInstance);
			initBeanWrapper(bw);
			return bw;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
		}
	}
```

4.注入数据  
注入数据的底层原理？数据虽然都是从容器取的，但是具体是怎么把一个bean注入到另外一个bean里的？源码是怎么实现的？先创建bean对象，如果bean对象有依赖数据(即有属性)，那么创建依赖数据的对象。从这里也可以知道，bean对象的创建顺序是什么样的——先创建数据的对象(放到容器里)，再创建依赖数据的对象(放到容器里)，并且把依赖数据赋值给数据的属性。

```
/**
	 * Apply the given property values, resolving any runtime references
	 * to other beans in this bean factory. Must use deep copy, so we
	 * don't permanently modify this property.
	 * @param beanName the bean name passed for better exception information
	 * @param mbd the merged bean definition
	 * @param bw the BeanWrapper wrapping the target object
	 * @param pvs the new property values
	 */
	protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
		if (pvs == null || pvs.isEmpty()) {
			return;
		}

		MutablePropertyValues mpvs = null;
		List<PropertyValue> original;

		if (System.getSecurityManager() != null) {
			if (bw instanceof BeanWrapperImpl) {
				((BeanWrapperImpl) bw).setSecurityContext(getAccessControlContext());
			}
		}

		if (pvs instanceof MutablePropertyValues) {
			mpvs = (MutablePropertyValues) pvs;
			if (mpvs.isConverted()) {
				// Shortcut: use the pre-converted values as-is.
				try {
					bw.setPropertyValues(mpvs);
					return;
				}
				catch (BeansException ex) {
					throw new BeanCreationException(
							mbd.getResourceDescription(), beanName, "Error setting property values", ex);
				}
			}
			original = mpvs.getPropertyValueList();
		}
		else {
			original = Arrays.asList(pvs.getPropertyValues());
		}

		TypeConverter converter = getCustomTypeConverter();
		if (converter == null) {
			converter = bw;
		}
		BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);

		// Create a deep copy, resolving any references for values.
		List<PropertyValue> deepCopy = new ArrayList<PropertyValue>(original.size());
		boolean resolveNecessary = false;
		for (PropertyValue pv : original) {
			if (pv.isConverted()) {
				deepCopy.add(pv);
			}
			else {
				String propertyName = pv.getName();
				Object originalValue = pv.getValue();
				Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue); //创建属性(即依赖数据)的对象
				Object convertedValue = resolvedValue;
				boolean convertible = bw.isWritableProperty(propertyName) &&
						!PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
				if (convertible) {
					convertedValue = convertForProperty(resolvedValue, propertyName, bw, converter);
				}
				// Possibly store converted value in merged bean definition,
				// in order to avoid re-conversion for every created bean instance.
				if (resolvedValue == originalValue) {
					if (convertible) {
						pv.setConvertedValue(convertedValue); //把依赖数据的对象赋值给数据属性
					}
					deepCopy.add(pv);
				}
				else if (convertible && originalValue instanceof TypedStringValue &&
						!((TypedStringValue) originalValue).isDynamic() &&
						!(convertedValue instanceof Collection || ObjectUtils.isArray(convertedValue))) {
					pv.setConvertedValue(convertedValue);
					deepCopy.add(pv);
				}
				else {
					resolveNecessary = true;
					deepCopy.add(new PropertyValue(pv, convertedValue));
				}
			}
		}
		if (mpvs != null && !resolveNecessary) {
			mpvs.setConverted();
		}

		// Set our (possibly massaged) deep copy.
		try {
			bw.setPropertyValues(new MutablePropertyValues(deepCopy));
		}
		catch (BeansException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Error setting property values", ex);
		}
	}
```

```
/**
	 * Create a new ClassPathXmlApplicationContext with the given parent,
	 * loading the definitions from the given XML files.
	 * @param configLocations array of resource locations
	 * @param refresh whether to automatically refresh the context,
	 * loading all bean definitions and creating all singletons.
	 * Alternatively, call refresh manually after further configuring the context.
	 * @param parent the parent context
	 * @throws BeansException if context creation failed
	 * @see #refresh()
	 */
	public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh(); //加载配置文件；加载bean
		}
	}
```

```
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); //加载配置文件

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory); //加载bean

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}
		}
	}
```

```
/**
	 * Load bean definitions from the specified resource location.
	 * <p>The location can also be a location pattern, provided that the
	 * ResourceLoader of this bean definition reader is a ResourcePatternResolver.
	 * @param location the resource location, to be loaded with the ResourceLoader
	 * (or ResourcePatternResolver) of this bean definition reader
	 * @param actualResources a Set to be filled with the actual Resource objects
	 * that have been resolved during the loading process. May be {@code null}
	 * to indicate that the caller is not interested in those Resource objects.
	 * @return the number of bean definitions found
	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
	 * @see #getResourceLoader()
	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource)
	 * @see #loadBeanDefinitions(org.springframework.core.io.Resource[])
	 */
	public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			throw new BeanDefinitionStoreException(
					"Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			// Resource pattern matching available.
			try {
				Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location); //加载配置文件 //[file [/Users/gongzhihao/src/gzh/spring/spring/target/classes/applicationContext-gzh1.xml]]
				int loadCount = loadBeanDefinitions(resources); //加载bean
				if (actualResources != null) {
					for (Resource resource : resources) {
						actualResources.add(resource);
					}
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");
				}
				return loadCount;
			}
			catch (IOException ex) {
				throw new BeanDefinitionStoreException(
						"Could not resolve bean definition resource pattern [" + location + "]", ex);
			}
		}
		else {
			// Can only load single resources by absolute URL.
			Resource resource = resourceLoader.getResource(location);
			int loadCount = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			if (logger.isDebugEnabled()) {
				logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");
			}
			return loadCount;
		}
	}
```

//加载bean
```
/**
	 * Actually load bean definitions from the specified XML file.
	 * @param inputSource the SAX InputSource to read from
	 * @param resource the resource descriptor for the XML file
	 * @return the number of bean definitions found
	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
	 */
	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			int validationMode = getValidationModeForResource(resource);
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware()); //加载配置文件里的内容-bean
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```



//日志
```
十一月 19, 2018 9:36:58 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@5f2108b5: startup date [Mon Nov 19 21:36:58 CST 2018]; root of context hierarchy
十一月 19, 2018 9:36:58 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from file [/Users/gongzhihao/src/gzh/spring/spring/target/classes/applicationContext-gzh1.xml] //加载配置文件
十一月 19, 2018 9:37:05 下午 org.springframework.beans.factory.support.DefaultListableBeanFactory preInstantiateSingletons
信息: Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@49ec71f8: defining beans [service1]; root of factory hierarchy //加载bean

十一月 19, 2018 9:44:19 下午 gzh.spring.Main main
信息: [2018-11-19 21:44:19] Main server started!

```


# 容器
类继承图  
接口  
1.BeanFactory  
BeanFactory就是代表容器。而且是一个最简单的容器。  

2.ApplicationContext  
继承了BeanFactory。扩充了一些功能，具备应用程序的功能。//一般都是使用这个类的子类，因为支持：1.注入数据2.面向切面3.事务。而BeanFactory都不支持。

实现类  
3.web应用程序接口
WebApplicationContext   
1）继承了ApplicationContext
2）作用：web应用程序

4.java程序  
类路径XmlApplicationContext/磁盘文件XmlApplicationContext。

ApplicationContext——》ConfigurableApplicationContext——》ClassPathXmlApplcationContext。  

【工作应用】  
支付-微服务，很多服务不是web应用程序，怎么启动？入口类，即包含main方法的类。  
怎么加载bean？ClassPathXmlWebApplication。顾名思义，就是从类路径加载applicationContext-XXX.xml这些配置文件。

【代码】


```
/*    */ package com.gzh.commons.context;
/*    */ 
/*    */ import org.apache.commons.lang.StringUtils;
/*    */ import org.springframework.context.ApplicationContext;
/*    */ import org.springframework.context.support.AbstractApplicationContext;
/*    */ import org.springframework.context.support.ClassPathXmlApplicationContext;
/*    */ 
/*    */ public class AppContext
/*    */ {
/*    */   private static volatile AbstractApplicationContext appContext;
/*    */   private static final String XML_EXPRESSION = "classpath*:applicationContext*.xml";
/*    */   
          //初始化
/*    */   public static synchronized void initConfig(String regularExpression)
/*    */   {
/* 15 */     if (appContext == null) {
/* 16 */       if (StringUtils.isEmpty(regularExpression)) {
/* 17 */         appContext = new ClassPathXmlApplicationContext("classpath*:applicationContext*.xml");
/*    */       } else {
/* 19 */         appContext = new ClassPathXmlApplicationContext(regularExpression.split("[,\\s]+"));
/*    */       }
/*    */     }
/*    */   }
/*    */   
/*    */   public static void setAppContext(ApplicationContext applicationContext) {
/* 25 */     appContext = (AbstractApplicationContext)applicationContext;
/*    */   }
/*    */   
/*    */   public static AbstractApplicationContext getAppContext() {
/* 29 */     if (appContext == null) {
/* 30 */       initConfig(null);
/*    */     }
/* 32 */     return appContext;
/*    */   }
/*    */   
/*    */   public static Object getBean(String name) {
/* 36 */     if (appContext == null) {
/* 37 */       initConfig(null);
/*    */     }
/*    */     
/* 40 */     return appContext.getBean(name);
/*    */   }
/*    */   
/*    */   public static void start() {
/* 44 */     if (appContext == null) {
/* 45 */       initConfig(null);
/*    */     }
/*    */     
/* 48 */     appContext.start(); //启动应用程序，加载配置文件里的bean
/*    */   }
/*    */   
/*    */   public static synchronized void stop() {
/* 52 */     if (appContext != null) {
/* 53 */       appContext.stop();
/* 54 */       appContext.close();
/* 55 */       appContext = null;
/*    */     }
/*    */   }
/*    */ }

```



---
作用  
管理和创建bean。并且包含了注入数据，即所谓的控制反转。

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

# 注入数据
1.何时创建bean？启动应用程序的时候就创建和加载了。  
2.何时注入数据？也是启动应用程序的时候创建，不过是先创建bean对象，如果bean对象有依赖数据，那么创建依赖数据的对象。

# 流程

# bean是否单例
1.默认 //单例  
2.原型 //多例  
3.web作用域范围  

---
工作使用  
一般都是单例。//控制器、service、dao都是单例。

注：struts2控制器是多例，每个请求/线程都有自己的数据。

---
如何确保线程安全？  
1.一般都不需要使用多例    
2.如果使用多例，如何确保线程安全？  
1）线程本地数据  
工作使用，分布式事务id。  
2）加锁  
不建议使用。


# 参考

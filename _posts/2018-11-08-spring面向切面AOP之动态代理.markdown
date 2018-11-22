---
layout:     post
title:      "spring面向切面AOP之java动态代理"
subtitle:   " \"拦截器包含2个核心：1.代理对象(基于反射技术，构造器.创建对象() ) 2.拦截器功能(jvm支持或CGLib支持)\""
date:       2018-11-08 06:00:00
author:     "青乡"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - spring
    - spring AOP
    - spring 面向切面
    - java动态代理
---



# 为什么要学动态代理技术
因为spring的面向切面(就是拦截器)基于动态代理。

# 动态代理是什么
1.要想实现拦截器的功能  
2.就必须使用动态代理技术  
3.动态代理就是一个拦截器类  
1）方法之前干点什么  
2）业务类.业务方法()  
3）方法之后干点什么

# 为什么要使用动态代理
要实现拦截器的功能，有2种方法，  
1.耦合  
把业务代码和拦截器代码混在一起  
即把拦截器代码硬编码到业务代码里面去，进行拦截。  
2.解耦  
把拦截器代码从业务代码里挪出来，这就是解耦。怎么挪？使用动态代理技术。

# 应用场景
应用极其广泛，看一下官网就知道了https://github.com/cglib/cglib。但凡是涉及到创建对象(基于反射创建对象，1.类.创建对象() 2.构造器.创建对象() )、字节码技术、反编译(反编译就是针对字节码.class文件——》.java文件)等等这些技术，通通使用的都是jdk-动态代理和CGLib。

# 工作应用

# 底层原理
核心技术  
1.拦截器      
1）调用业务类.业务方法() //业务类的代理商。怎么生成代理类？基于反射技术，构造器.创建对象()。  
2）之前和之后干点什么事情 //这是拦截器/面向切面的唯一作用

2.注入数据  
也是基于反射技术，所以在基于反射这一块，面向切面和注入数据是一样的。不一样的是，面向切面还多了一个拦截器的功能。

注入数据，先读配置文件，根据配置文件的bean定义，创建对象——基于反射技术，即构造器.创建对象()。

拦截器功能里的代理类(具体的说，就是业务类的代理类)，创建对象——也是基于反射技术，即构造器.创建对象()。为什么不直接调用业务类.业务方法？如果你要实现拦截器功能的话，可以在拦截器里拦截嘛(具体是调用业务类.业务方法() )，这样不行吗？不行，业务拦截器和业务类的代码耦合了，所以不行，所以解耦，要使用代理技术，即就是所谓的动态代理技术，动态的意思是，业务类不止一个嘛，随时可以为任何业务类动态创建代理对象，而且代码不耦合。



#### jdk-动态代理
怎么使用？  
1.代理类Proxy  
作用  
获取实际业务类的一个代理。怎么获取？底层技术是反射，即构造器.newInstance(拦截器类)。可以看到，与普通的构造器.newInstance()不一样，多了一个参数拦截器，作用就是对业务类进行拦截。怎么拦截？就是调用proxy.业务方法()的时候，先进入/调用拦截器的invoke()方法，这个调用是由jvm实现的，业因为构造器.newInstance(拦截器类)的时候，把拦截器类作为参数传入进去了，所以jvm知道先去调用拦截器.invoke()方法，而实际的业务类.业务方法()是在invoke()方法里调用的，通过Method.invoke(业务类,方法参数)这种方式进行调用。

2.拦截器类InvocationHandler  
invoke()方法  
1）业务方法之前执行的拦截  
2）实际业务类.业务方法()  
3）业务方法之后执行的拦截  

自定义拦截器类需要继承jdk-拦截器类InvocationHandler接口，并且实现接口的唯一invoke方法。

```
package com.baobaotao.proxy;
import java.lang.reflect.Proxy;
public class TestForumService {
	public static void main(String[] args) {
               
//               //①希望被代理的目标业务类
//		ForumService target = new ForumServiceImpl(); 
//		
//               //②将目标业务类和横切代码编织到一起
//		PerformanceHandler handler = new PerformanceHandler(target);
//		
//                //③根据编织了目标业务类逻辑和性能监视横切逻辑的InvocationHandler实例创建代理实例
//		ForumService proxy = (ForumService) Proxy.newProxyInstance(  //jdk-动态代理：只能是接口  //底层实现：jvm实现
//				target.getClass().getClassLoader(),
//				target.getClass().getInterfaces(), //这里决定了jdk-动态代理，只能是接口
//				handler);
//
//                //④调用代理实例
//		proxy.removeForum(10);   
//		proxy.removeTopic(1012);
		
		
		CglibProxy proxy = new CglibProxy();  
	      ForumServiceImpl forumService = //①   //CGLib：可以不是接口
	                (ForumServiceImpl )proxy.getProxy(ForumServiceImpl.class);  
	      forumService.removeForum(10);  
	      forumService.removeTopic(1023); 
		
	}
}
```

#### CGLib
不管是jdk-动态代理，还是CGLib，底层的创建对象，都是使用的构造器.创建对象()。

jdk-动态代理的缺点是，只能是接口。怎么解决这个问题？CGLib。CGLib底层使用了字节码技术，使得可以是非接口。为什么字节码就可以是非接口呢？因为jdk-动态代理的Proxy.创建对象(只能是接口)，而CGLib可以是非接口——底层使用的是字节码技术，允许是非接口，但是从字节码得到类之后，最终还是从构造器.创建对象()得到对象，spring面向切面、jdk-动态代理、CGLib的最后一步都是一样的，都是使用的构造器.创建对象()得到对象。

```
abstract public class AbstractClassGenerator
implements ClassGenerator
{

protected Object create(Object key) {
        try {
        	Class gen = null;
        	
            synchronized (source) {
                ClassLoader loader = getClassLoader();
                Map cache2 = null;
                cache2 = (Map)source.cache.get(loader);
                if (cache2 == null) {
                    cache2 = new HashMap();
                    cache2.put(NAME_KEY, new HashSet());
                    source.cache.put(loader, cache2);
                } else if (useCache) {
                    Reference ref = (Reference)cache2.get(key);
                    gen = (Class) (( ref == null ) ? null : ref.get()); 
                }
                if (gen == null) {
                    Object save = CURRENT.get();
                    CURRENT.set(this);
                    try {
                        this.key = key;
                        
                        if (attemptLoad) {
                            try {
                                gen = loader.loadClass(getClassName());
                            } catch (ClassNotFoundException e) {
                                // ignore
                            }
                        }
                        if (gen == null) {
                            byte[] b = strategy.generate(this); //得到类的字节码
                            String className = ClassNameReader.getClassName(new ClassReader(b));
                            getClassNameCache(loader).add(className);
                            gen = ReflectUtils.defineClass(className, b, loader); //从字节码得到类
                        }
                       
                        if (useCache) {
                            cache2.put(key, new WeakReference(gen));
                        }
                        return firstInstance(gen); //调用构造器.创建对象()，即通过反射技术得到对象
                    } finally {
                        CURRENT.set(save);
                    }
                }
            }
            return firstInstance(gen);
        } catch (RuntimeException e) {
            throw e;
        } catch (Error e) {
            throw e;
        } catch (Exception e) {
            throw new CodeGenerationException(e);
        }
    }
```

```
//ReflectUtils
public static Object newInstance(final Constructor cstruct, final Object[] args) {
            
        boolean flag = cstruct.isAccessible();
        try {
            cstruct.setAccessible(true);
            Object result = cstruct.newInstance(args); //构造器.创建对象()
            return result;
        } catch (InstantiationException e) {
            throw new CodeGenerationException(e);
        } catch (IllegalAccessException e) {
            throw new CodeGenerationException(e);
        } catch (InvocationTargetException e) {
            throw new CodeGenerationException(e.getTargetException());
        } finally {
            cstruct.setAccessible(flag);
        }
                
    }
```

# spring面向切面
CGLib的缺点是，业务类的每个方法，都被拦截器拦截了。怎么解决？spring面向切面是基于CGLib，解决了这个问题，即针对每个方法是否拦截处理。

# 参考
http://www.iteye.com/topic/1123293




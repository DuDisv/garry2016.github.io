---
layout: post
title: 设计模式-单例模式
categories: [设计模式]
description: 设计模式中最常见最简单的模式
keywords: 设计模式, 单例模式,Singleton
---

### 1.介绍
>确保某个类只有一个实例，并且自行实例化并向整个系统提供这个实例。

实现单例模式的关键点：

* 构造函数不对外开放，一般为private
* 通过一个静态方法或者枚举返回单例类对象
* 确保单例类的对象有且只有一个，尤其在多线程的环境下
* 确保单例类不会在反序列化时不会重新构建对象 

### 2. 实现方式

##### 1.恶汉式

```
//恶汉式

public class Singleton {
	private static final Singleton mSingleton = new Singleton();
 
    private Singleton(){
    }
 
    public static Singleton getInstance(){
        return mSingleton;
    }
}
```

##### 2.懒汉式

```
/**
* 懒汉式
*/
public class Singleton{
    private static Singleton mSingleton = null;
 
    private Singleton(){
 
    }
 
    public static synchronized Singleton getInstance(){
        if(mSingleton == null){
            mSingleton = new Singleton();
        }
        return mSingleton;
    }
}
```

优点：使用时才实例化，一定程序减少了资源。

缺点：每次调用时都会进行同步操作，造成不必要的开销。


##### 3.Double check lock 实现单例(推荐)

```
/**
* DCL模式
*/
public class Singleton{
    private static Singleton mSingleton = null;
 
    private Singleton(){
 
    }
 
    public static Singleton getInstance(){
        if(mSingleton == null){
            synchronized (Singleton.class){
                if(mSingleton == null){
                    mSingleton = new Singleton();
                }
            }
        }
        return mSingleton;
    }
}
```

优点：资源利用率高，第一次执行时对象才会被实例化，效率高。

缺点：第一次加载反应稍慢，java内存模型的原因偶尔会失败，高并发的情况下有一定的缺陷。



##### 4.静态内部类的单例模式(推荐)

```
/**
* 内部静态类
*/
public class Singleton{
    private Singleton(){
 
    }
 
    public static Singleton getInstance(){
        return SingletonHolder.sInstance;
    }
 
    private static class SingletonHolder{
        private static final Singleton sInstance = new Singleton();
    }
}
```


##### 5.枚举单例

```
/**
* 枚举单例
*/
public enum Singleton{
    INSTANCE;
 
    public doSomething(){
 
    }
}
//call :  Singleton.INSTANCE.doSomething();
```

注意：java1.5以上才能生效



##### 6.容器单例

```
/**
* 容器单例
*/
public class SingletonManager{
    private static Map<String,Object> objMap = new HashMap<>();
 
    private SingletonManager(){
 
    }
 
    public static void registerService(String key, Object instance){
        if(!objMap.containsKey(key)){
            objMap.put(key,instance);
        }
    }
 
    public static Object getService(String key){
        return objMap.get(key);
    }
}
```



### 3.Andorid源码中的单例

```
/*package*/ static class ServiceFetcher {
    int mContextCacheIndex = -1;

    /**
     * Main entrypoint; only override if you don't need caching.
     */
    //通过getService获取服务
    public Object getService(ContextImpl ctx) {
        ArrayList<Object> cache = ctx.mServiceCache;
        Object service;
        synchronized (cache) {
            if (cache.size() == 0) {
                // Initialize the cache vector on first access.
                // At this point sNextPerContextServiceCacheIndex
                // is the number of potential services that are
                // cached per-Context.
                for (int i = 0; i < sNextPerContextServiceCacheIndex; i++) {
                    cache.add(null);
                }
            } else {
                //从cache中获取服务
                service = cache.get(mContextCacheIndex);
                if (service != null) {
                    return service;
                }
            }
            service = createService(ctx);
            cache.set(mContextCacheIndex, service);
            return service;
        }
    }

    /**
     * Override this to create a new per-Context instance of the
     * service.  getService() will handle locking and caching.
     */
    //子类复写该方法创建服务。
    public Object createService(ContextImpl ctx) {
        throw new RuntimeException("Not implemented");
    }
}
```


1.service容器

```
private static final HashMap<String, ServiceFetcher> SYSTEM_SERVICE_MAP =
        new HashMap<String, ServiceFetcher>();

private static int sNextPerContextServiceCacheIndex = 0;
```



2.注册服务

```
private static void registerService(String serviceName, ServiceFetcher fetcher) {
   if (!(fetcher instanceof StaticServiceFetcher)) {
      fetcher.mContextCacheIndex = sNextPerContextServiceCacheIndex++;
   }
   SYSTEM_SERVICE_MAP.put(serviceName, fetcher);
}
```

3.静态代码库，第一次加载该类时执行

```
static{
   registerService(LAYOUT_INFLATER_SERVICE, new ServiceFetcher() {
       public Object createService(ContextImpl ctx) {
           return PolicyManager.makeNewLayoutInflater(ctx.getOuterContext());
       }});
}
```



4.根据Key获取对应的服务

```
@Override
public Object getSystemService(String name) {
    ServiceFetcher fetcher = SYSTEM_SERVICE_MAP.get(name);
    return fetcher == null ? null : fetcher.getService(this);
}
```

### 4.总结

##### 优点：

* 单例模式在内存中只有一个实例，减少了内存开支，特别是一个对象需要频繁的创建、销毁时，而且创建和销毁时性能又无法优化，单例模式的优势就很明显。

* 单例模式减少了系统性能开销。

* 单例模式可以避免对资源的多重占用。

* 单例模式可以在系统设置全局的访问点，优化和共享资源访问。

##### 缺点：

* 单例模式一般没有接口，扩展很难。若要扩展，只能修改代码。

* 单例模式如果引用了Context，很容易引起内存泄漏，注意传递给单例对象的Context最好是Application Context。 


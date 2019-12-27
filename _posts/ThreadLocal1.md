---
title: ThreadLocal再探
layout: post
categories: Java
tags: java基础 多线程
---
* content
{:toc}

上一次写了ThreadLocal的功能和基本用法，它能够在多线程访问同一资源时避免不同步的问题，并且知道ThreadLocal是通过每一个线程有一个自己的副本来实现的。那么ThreadLocal的原理究竟是什么呢？





## 1 ThreadLocal、ThreadLocalMap和Thread

首先我们考虑这个问题，多个线程访问同一个变量，每个线程有自己的一个副本，怎么做呢。一般应该会想到使用map的数据结构，key是线程名，value是变量。那么ThreadLocal是怎么做的呢？

### 1.1 ThreadLocal.set()方法

首先来看threadlocal的set方法：

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
    }
```

代码逻辑十分清晰：首先通过当前线程获取一个ThreadLocalMap对象的引用，如果此ThredLocalMap不为空就将键值对插入，如果为空就创建ThreadLocalMap并插入键值对。但是在这段代码中可以发现两个重要的信息：

- ThreadLocalMap是存放在Thread类中的变量。
- ThreadLocalMap使用ThreadLocal作为key而非Thread。

我们来看一下getMap方法：

```java
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```
验证了第一条。

现在知道set方法就是将<this,value>插入当前Thread的ThreadLocalMap中，如果map为空就创建一个map再插入。

### 1.2 ThreadLocal.get()方法

```java
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

可以看到get方法也是从当前Thread对象中取出ThreadLocalMap，然后根据this这个ThreadLocal对象从ThreadLocalMap中取出一个Entry对象，那么Entry是什么类型呢，根据

```java
T result = (T)e.value;
```

这一句推测应该是个键值对类型，因为我们返回了它的值。至于具体是什么一会再探究。

最后看到如果map为空则返回setInitialValue(),来看下这个是什么：

```java
private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
        if (this instanceof TerminatingThreadLocal) {
            TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
        }
        return value;
    }
```

它又创建了一个initialValue(),其实就是个null:

```java
protected T initialValue() {
        return null;
    }
```

然后在当前Thread对象的ThreadLocalMap中插入<this,null>这个键值对，最后返回null。

好了现在知道get方法的流程了，从当前线程的ThreadLocalMap中根据键this取出键值对，返回值，如果map为空就创建map并插入<this,null>，返回null。

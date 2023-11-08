---
layout: post
title: "设计模式-单例模式(c++实现)"
date:   2020-03-01
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

1. 简单介绍单例模式, `c++`单例模式的实现, 以及通用实现如继承`noncopyable`

2. `c++`单例模式的线程安全问题

3. 用`thread_local`实现的线程级别的单例模式

### 1. 简单介绍单例模式	& `c++`实现

单例模式一般用来用来创建全局唯一的对象, 例如:在系统中，我们只有一个配置文件，当配置文件被加载到内存之后，以对象的形式存在，也理所应当只有一份。

C++简单的单例如下:

```c++
class Singleton{
public:
    static Singleton* GetInstance(){
        static Singleton instance;
        return &instance;
    }

    Singleton(const Singleton&) = delete;           // copy ctor
    Singleton(Singleton&&) = delete;                // move ctor
    Singleton& operator=(const Singleton&)=delete;  // copy assign
    Singleton& operator=(Singleton&&)=delete;       // move assign

    void print(){cout<<"this is singleton"<<endl;}

private:   // not protected不然子类可以创建对象了
    Singleton(){cout<<"ctor"<<endl;}
};
```

使用实例如下:

``` c++
    Singleton::GetInstance()->print();
```

上面的代码问题在于, 单例模式的代码无法服用了, 可以用模板的方式实现:
```c++
class noncopyable{
public:
    noncopyable(const noncopyable&) = delete;
    noncopyable& operator=(const noncopyable&) = delete;
protected:
    noncopyable() = default;
    ~noncopyable() = default;
};

template<typename T>
class Singleton : public noncopyable{
public:
    using value_type = T;
    using pointer    = T*;

    static pointer GetInstance(){
        static T obj;
        return &obj;
    }
};
```
使用如下:

```c++
class Config : public Singleton<Config>{
public:
    int getAppid(){...}
}

// 一般可以定义一个宏
#define CONF_INS Config::getInstance()

// 使用
CONF_INS->getAppid();

```

2. `c++`单例模式的线程安全问题

问题上面的单例模式，在多线程环境下，线程安全吗?
C++11以及之后是安全的, 对于静态变量的创建编译器会插入静态变量的锁.
```c++
Singleton& Singleton::instance(){
    // check to see if we need to create the Singleton
    EnterCriticalSection( &instance_lock);
    static Singleton s;
    LeaveCriticalSection( &instance_lock);

    return s;
}
```

4. 用`thread_local`实现的线程级别的单例模式

有些时候我们需要线程级别的单例模式, 例如使用 `mysql client` 时候(非线程安全的), 这个时候可以考虑写通用线程安全级别的单例模式, 将上面的`static` 改为`c++`的 `thread_local` 即可

```c++
template<typename T>
class ThreadSingleton : public noncopyable{
public:
    static T* GetInstance(){
        static thread_local T obj;
        return &obj;
    }
};
```

使用如下：
```c++
class DbHandle : public ThreadSingleton<DbHandle>{
public:
    DbHandle(){
        init();
    }

    int query(){
        client_.select(....);
        ....
    }

    void init(){
        client_.connect(ip, port....);
    }
private:    
    MysqlClient client_;
}
```


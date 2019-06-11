---
title: ThreadLocal类
layout: post
categories: Java
tags: java基础 多线程
---
* content
{:toc}

在很多时候，需要使用多个线程访问同一个资源，这时候可能会出现不同步问题，当然我们可以通过设置同步的方式来解决，但是会造成更大的开销，比如在某个线程获取链接时其他线程就得等待。如果此资源对于各个线程来说需要拥有不同的值，那么可以用ThreadLocal类解决这个问题。 





假设这样一个场景，有多个线程需要向屏幕上打印数据，我们用DataManager类来管理数据，所有线程都要使用这个类来收发数据，Data类就是收发的数据类型。我们在主方法中开始三个线程，每个线程向屏幕打印一条数据，数据格式为“Thread_0设置数据A”。

我们先看这样一段代码：

```java
package javademo;

public class JavaDemo {
    public static void main(String args[]) {
        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据A");
            DataManager.setData(data);
            DataManager.showData();
        }).start();

        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据B");
            DataManager.setData(data);
            DataManager.showData();
        }).start();

        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据C");
            DataManager.setData(data);
            DataManager.showData();
        }).start();
    }
}

//数据的管理类，所有线程通过这个类来存取数据
class DataManager {
    private static Data data = new Data();

    public static Data getData() {
        return data;
    }

    public static void setData(Data d) {
        data = d;
    }

    public static void showData() {
        System.out.println(Thread.currentThread().getName() + data.getString());
    }
}

//线程需要取用的信息类
class Data {
    public String getString() {
        return string;
    }

    public void setString(String string) {
        this.string = string;
    }

    private String string;
}
```

我们将程序运行三次，结果如下：  

Thread-0设置数据C  
Thread-2设置数据C  
Thread-1设置数据C  

Thread-2设置数据B  
Thread-1设置数据B  
Thread-0设置数据B  

Thread-0设置数据B  
Thread-2设置数据C  
Thread-1设置数据C  

我们想要的结果应该是(顺序可以打乱，但是需要要对应上)：  
Thread-0设置数据A  
Thread-1设置数据B  
Thread-2设置数据C  

这当然是由于不同步的问题，例如：某个线程准备打印,而另一个线程正在存放，将准备打印的数据覆盖掉了等等。

ThreadLocal类常用的有三个方法。

方法名|作用
:-|:-
void set(T value)|设置数据
T get()|返回存放的数据
void remove()|清除数据

下面使用ThreadLocal类来解决问题，只需要修改一下DataManager类，使用ThreadLocal类来存放数据。

```java
package javademo;

public class JavaDemo {
    public static void main(String args[]) {
        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据A");
            DataManager.setData(data);
            DataManager.showData();
        }).start();

        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据B");
            DataManager.setData(data);
            DataManager.showData();
        }).start();

        new Thread(() -> {
            Data data = new Data();
            data.setString("设置数据C");
            DataManager.setData(data);
            DataManager.showData();
        }).start();
    }
}

//数据的管理类，所有线程通过这个类来存取数据
class DataManager {
    //修改此处，用ThreadLocal来存放数据
    private static final ThreadLocal<Data> THREAD_LOCAL = new ThreadLocal<Data>();

    //存取和打印都使用ThreaLocal的方法
    public static Data getData() {
        return THREAD_LOCAL.get();
    }

    public static void setData(Data d) {
        THREAD_LOCAL.set(d);
        ;
    }

    public static void showData() {
        System.out.println(Thread.currentThread().getName() + THREAD_LOCAL.get().getString());
    }
}

//线程需要取用的信息类
class Data {
    public String getString() {
        return string;
    }

    public void setString(String string) {
        this.string = string;
    }

    private String string;
}
```

将程序运行三次，结果如下：  

Thread-1设置数据B  
Thread-2设置数据C  
Thread-0设置数据A  

Thread-2设置数据C  
Thread-0设置数据A  
Thread-1设置数据B  

Thread-2设置数据C  
Thread-1设置数据B  
Thread-0设置数据A  

可以看到已经解决了不同步问题，这就是ThreadLocal最基本也是最常用的用法。ThreadLocal是个非常重要的类，它的原理下一次再写吧。
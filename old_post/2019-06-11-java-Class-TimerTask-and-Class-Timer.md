---
title: 定时器任务，TimerTask和Timer类
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

在java中使用Timer类和TimerTask类可以实现定时器效果的操作，比如每间隔1秒钟播放一声“滴”等，下面来看怎么用吧。



## Timer类
首先来看Timer类，它提供了一系列的schedule方法，也就是计划方法，表示按各种不同的方式设定定时任务。

```java
void schedule​(TimerTask task, long delay)
void schedule​(TimerTask task, long delay, long period)
void schedule​(TimerTask task, Date time)
void schedule​(TimerTask task, Date firstTime, long period)
void scheduleAtFixedRate​(TimerTask task, long delay, long period)
void scheduleAtFixedRate​(TimerTask task, Date firstTime, long period)
```

可以看到每个schedule方法都需要一个TimerTask参数，它表示要定时执行的任务。

## TimerTask类

我们来看看TimerTask类，首先看到它的声明：  

```java
    public abstract class TimerTask
    extends Object
    implements Runnable
```

它是一个抽象类，并且需要实现Runnable接口。  
再看它的方法：

```java
protected  TimerTask()//保护类型的构造方法，只能在本类或派生类中调用

boolean cancel()//取消
abstract void run()//要实现的任务
long scheduledExecutionTime()//返回此任务最后一次执行的时间
```

也就是说TimerTask实现计划任务的方法实际上是通过多线程来实现的，将要执行的任务放在Runnable接口的run()方法中，然后启动一个线程来执行这个任务。

## 举栗子

下面来使用一下这两个类实现一个简单的任务，每间隔1s输出一下当前时间，正好复习一下Data类。

```java
package javademo;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;

public class JavaDemo {
    public static void main(String args[]) {
        Timer timer = new Timer();
        MyTask myTask=new MyTask();
        timer.scheduleAtFixedRate(myTask, 100, 1000);//100ms后开始执行，每隔1000ms执行一次
    }
}

//任务类
class MyTask extends TimerTask {
    public static final SimpleDateFormat SDF_DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH-mm-ss-SSS");

    @Override
    public void run() {
        // TODO Auto-generated method stub
        // 输出当前时间
        Date date = new Date();
        System.out.println(SDF_DATE_FORMAT.format(date));
    }
}
```

输出结果：  
2019-06-11 14-03-21-154  
2019-06-11 14-03-22-155  
2019-06-11 14-03-23-154  
2019-06-11 14-03-24-154  
2019-06-11 14-03-25-155  
2019-06-11 14-03-26-154  
2019-06-11 14-03-27-154  
2019-06-11 14-03-28-155  
2019-06-11 14-03-29-154  
2019-06-11 14-03-30-155  
2019-06-11 14-03-31-155  
2019-06-11 14-03-32-154  

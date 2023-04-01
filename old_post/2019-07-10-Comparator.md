---
title: java比较器
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

今天学习一下java的比较器的含义和用法。




## 1 为什么要有比较器

在很多的算法中需要对两个对象进行比较大小，最典型就是排序算法。可以理解，既然要对一些对象进行排序，首先必须知道怎么定义两个对象的大小。对于java内置类型，比如int、double、String等，标准库中已经定义好了比较方法。那么对于用户自定义的类型，如果要进行涉及到比较大小的操作，用户就必须自定义比较方法，也就是必须要有比较器。  
这与C++标准库类似，C++标准库的sort方法有两种提供比较器的方法，一是重载类型的"<"运算符，另一种是额外提供一个比较器（常见的是提供lambda表达式，传入两个对象传出一个bool值）。在java中采用了类似的解决方法，分别是Comparable接口和Comparator比较器。

## 2 Comparable接口

先看下面一个例子，要对Person这个类型进行排序的时候，根据比较标准的不同结果也不同，例如可以根据身高、体重等进行排序。

```java
package javademo;

import java.util.Arrays;

public class JavaDemo {
    public static void main(String[] args) {
        Person[] persons = { new Person("小凯", 185, 70, 20), new Person("小菲", 48.44, 160, 23),
                new Person("小杰", 165, 60, 19) };
        Arrays.sort(persons);
        System.out.println(Arrays.toString(persons));
    }
}

class Person implements Comparable<Person> {
    @Override
    public int compareTo(Person o) { // 按照体重进行比较
        // TODO Auto-generated method stub
        return (int) (this.height - o.height);
    }

    Person(String name, double height, double weight, int age) {
        this.name = name;
        this.weight = weight;
        this.height = height;
        this.age = age;
    }

    @Override
    public String toString() {
        // TODO Auto-generated method stub
        return this.name;
    }

    private String name;
    private double height;
    private double weight;
    private int age;
}
```

输出结果：[小菲, 小杰, 小凯]

## 3 Comparator接口

下面使用Comparator按照年龄进行排序，Comparator接口由于只有一个方法，因此可以使用lambda表达式来代替。

```java
package javademo;

import java.util.Arrays;

public class JavaDemo {
    public static void main(String[] args) {
        Person[] persons = { new Person("小凯", 185, 70, 20), new Person("小菲", 48.44, 160, 23),
                new Person("小杰", 165, 60, 19) };
        Arrays.sort(persons, (Person a, Person b) -> {
            return a.getAge() - b.getAge(); // 按照年龄进行排序
        });
        System.out.println(Arrays.toString(persons));
    }
}

class Person implements Comparable<Person> {
    @Override
    public int compareTo(Person o) { // 按照体重进行比较
        // TODO Auto-generated method stub
        return (int) (this.height - o.height);
    }

    Person(String name, double height, double weight, int age) {
        this.name = name;
        this.weight = weight;
        this.height = height;
        this.age = age;
    }

    @Override
    public String toString() {
        // TODO Auto-generated method stub
        return this.name;
    }

    public int getAge() {
        return age;
    }

    private String name;
    private double height;
    private double weight;
    private int age;
}
```

输出结果：[小杰, 小凯, 小菲]

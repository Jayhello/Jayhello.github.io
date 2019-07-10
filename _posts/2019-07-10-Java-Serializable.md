---
title: Java 对象的序列化与反序列化
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

对象的序列化也就是将对象保存为二进制字节码的形式，可以保存为文件或者通过网络传输等。




## 1 基本介绍  

由于java的对象序列化似乎找不出什么缺点，所以对象序列化尽量不要自己动手。要使对象能够序列化，首先需要将该类声明实现Serializable接口。Serializable接口不需要重写任何方法，它只是描述类的一种能力。在java中描述类的能力的接口有两个，还有一个是Cloneable。  
除此之外需要考虑的一个问题是，我们通常在对象中会保存对其他对象的引用，那么我们序列化的时候，一定是希望把这个引用所指的对象也序列化进来，而不是序列化这个引用。因此，要序列化的类中所有的引用的类型也要声明Serializable接口。java在序列化的时候会将所有引用到的对象都序列化出来。

## 2 举例

在下面的代码中有一个Person类声明为Serializable的，每个Person类对象中还包含了一个CellPhone类对象。  
在主方法中，声明了一个Person[]数组，将这数组序列化，再反序列化，打印反序列化的数组。  
可以看到在序列化的过程中，persons对象的中的三个Person对象都被序列化了，Person对象中的phone对象也成功被序列化。

```java
package javademo;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.Arrays;

public class JavaDemo {
    public static void main(String[] args) throws Exception {
        Person[] persons = { new Person("小凯", 185, 70, 20), new Person("小菲", 48.44, 160, 23),
                new Person("小杰", 165, 60, 19) };

        // 序列化对象
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("D:\\person.out"));
        oos.writeObject(persons);
        oos.close();

        // 反序列化对象
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("D:\\person.out"));
        Person[] readPersons = (Person[]) ois.readObject();
        ois.close();
        System.out.println(Arrays.toString(readPersons));
    }
}

@SuppressWarnings("serial")
class Person implements Comparable<Person>, Serializable {
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
        this.phone = new CellPhone(name + "'s phone");
    }

    @Override
    public String toString() {
        // TODO Auto-generated method stub
        return this.name + "  " + this.age + "  " + this.height + "  " + this.weight + "  " + this.phone;
    }

    public int getAge() {
        return age;
    }

    private String name;
    private double height;
    private double weight;
    private int age;
    private CellPhone phone;
}

class CellPhone implements Serializable { // 此类型也要声明Serializable接口
    private String name;

    @Override
    public String toString() {
        return name;
    }

    public CellPhone(String name) {
        this.name = name;
    }
}
```

输出结果：[小凯  20  185.0  70.0  小凯's phone, 小菲  23  48.44  160.0  小菲's phone, 小杰  19  165.0  60.0  小杰's phone]

## 3 transient关键字  

如果不希望某个字段被序列化就将它声明为transient的，通常，通过计算出来的值不需要序列化，密码等保密性内容不应被序列化。

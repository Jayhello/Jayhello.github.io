---
title: BigInteger和BigDecimal类
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

有些时候我们会遇到long和double提供的精度不够的情况（long大约为19位十进制有效数字，double大约为15~16位十进制有效数字），为了解决这个问题，java类库提供了BigInteger类和BigDecimal类。这两个类理论上能够表示任意精度的数字，并且提供计算方法，但是实际使用依赖于具体计算机的性能。




要明白精度和范围的概念是不同的。对于long来说，它的范围和精度是一致的，在它范围内的每个数字它都能够准确表示。然而对于double类型来说，熟系IEEE754标准的朋友知道，它的64位是由1个符号位，11个指数位，和52个尾数位来表示的，这52个尾数位就代表了它的精度。也就是说double的精度比long还要低一些。举个例子，像1234567890987654.123这个数小数点前面已经有16位了，虽然这个数在double的范围内，但是小数点后面这个.123是表示不出来的，也就是精度损失了。看下面一个例子：

```java
public class k {
    public static void main(String args[]) {
        double d=1234567890987654.123;
        System.out.println(d);
    }
}
```

输出结果为：**1.234567890987654E15**
可以看出小数点后面的精度是损失了的，如果使用BigDecimal类再试试。

```java
package jju;
import java.math.BigDecimal;
public class k {
    public static void main(String args[]) {
        BigDecimal bigD = new BigDecimal(1234567890987654.123);
        System.out.println(bigD);
    }
}
```

输出结果为：**1234567890987654**
为什么精度还是损失了呢，这是由于在构造BigDecimal的时候输入了1234567890987654.123，它默认是一个**double**类型，所以在传入之前就已经损失精度了。正确的做法是用String进行构造。

```java
package jju;
import java.math.BigDecimal;
public class k {
    public static void main(String args[]) {
        BigDecimal bigD = new BigDecimal("1234567890987654.123");
        System.out.println(bigD);
    }
}
```

输出结果为:**1234567890987654.123**。与预期一致。

BigInteger的用法与BigDecimal类似。

需要注意的是这两个类是不可以使用+-*/等运算符的，需要使用类中的方法进行运算。此外，这两个类在计算效率方面要比基本类型低得多，所以如果不是必须的话，还是少用这两个类。

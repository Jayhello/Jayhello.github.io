---
title: java.lang.Math类
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

java.lang.Math提供了一些常用的数学函数，这些都是静态方法，可以直接通过类名调用。只记一下原来不知道的以及认为可能用到的。






```java
    public static double atan2(double f1, double f2);//反三角函数，，f1表示临边长度，f2表示对边长度。

    // 角度和弧度互相转换
    public static double toRadians(double angdeg);
    public static double toDegrees(double angrad);
```

其中IEEEremainder(double f1, double f2)这个方法需要注意，3.0对2.0取余等于-1.0，5.0对2.0取余等于1.0。这是由于按照 IEEE 754 标准的规定，对两个参数进行余数运算。余数的算术值等于 f1 - f2 × n，其中 n 是最接近商 f1/f2 准确算术值的整数，如果两个整数都同样接近 f1/f2，那么 n 是其中的偶数。如果余数是 0，那么它的符号与第一个参数的符号相同。

```java
    public static double cbrt(double a);//开三次方
    public static double IEEEremainder(double f1, double f2);//根据IEEE 754标准求f1对f2取余
    public static double ceil(double a);//向上取整，1.1->2, -1.1->-1;
    public static double floor(double a);//向下取整
    public static double rint(double a);//返回最接近的整数，如果同样接近返回偶数那个。(但是为什么返回类型不是int或者long?)
    public static int round(float a);//四舍五入，实际上是把原来的数+0.5再向下取整。
    public static long round(double a);
    public static long multiplyHigh(long x, long y);//这个函数把两个64位的整数在128位内相乘，再返回前64位，减少了乘法的精度损失。




```

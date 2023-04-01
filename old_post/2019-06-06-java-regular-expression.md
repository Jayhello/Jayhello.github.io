---
title: 正则表达式
layout: post
categories: Java
tags: java基础 正则表达式 String
---
* content
{:toc}

正则表达式很早就接触过，也在书籍上看到过，但是由于它需要记忆的内容较多，所以总是有点抗拒去仔细的学习，今天总结一下关于正则表达式的内容吧。




## 1 正则表达式基础

在java.util.regex包中找到Class Pattern，其中介绍了正则表达式的规则，我挑一些比较常用的记录下来。

1.匹配单个字符

结构      |     匹配
:-|:-
x|单个字符对应匹配
\\\\|匹配\
\\0n或\\0nn或\\0mnn|将字符转换成八进制数字后与该数字匹配(0<=m<=3>,0<=n<=7,也就是在十进制0-255之间)
\\xhh 或 \\uhhhh 或 \\x{h...h}|将字符转换成16进制数字与该十六进制数字匹配
\\t|匹配制表符
\\n|匹配换行符

2.按照范围进行匹配

结构   |   匹配
:-|:-
[abc]|匹配a或b或c
[^abc]|不是a、b、c
[a-zA-Z]|匹配大小写字母
[a-z&&[^bc]]|a-z除了b、c

3.转义字符匹配

结构|匹配
:-|:-
.|匹配任意字符
\\d 或 \\D|匹配一个数字[0-9] 或 非数字[^0-9]
\\s 或 \\S|匹配空白字符（换行符、制表符等） 或 非空白字符
\\W 或 \\W|匹配字母、数字、下划线 或 非字母、数字、下划线

4.量词匹配

结构|匹配
:-|:-
X?|X出现0次或1次
X*|X出现0次或多次
X+|X出现1次或多次
X{n}|X出现n次
X{n,}|X至少出现n次
X{n,m}|X出现n到m次

5.逻辑匹配

结构|匹配
:-|:-
XY|X表达式后接着Y表达式
X\|Y|满足X表达式或Y表达式
(X)|X表达式作为一个整体

## 2 正则表达式在String类中的使用

正则表达式最常用的用法也就是对String进行验证，比如验证一个字符串是否是一个合法的网址、电话号码、地址、日期等，也就是说对**格式**进行验证。参考API文档，在String类的方法中，凡是参数名为'regex'的说明此处可以填入一个正则表达式。主要有如下几个方法。

```java
boolean maches(String regex);//判断是否匹配
String replaceAll(String regex, String replacement)//将所有匹配regex的字串替换为replacement，返回替换后的字符串
String replaceFirst(String regex, String replacement)//替换第一个匹配的子串
String[] split(String regex);//按照regex进行拆分
String[] split(String regex, int limit);//按照regex进行拆分，至多拆分limit次，如果limit为负值，则尽可能多的拆分。
```

## 2 java.util.regex开发包的正则处理类

虽然大部分情况下使用String类就能满足对正则表达式的使用，但是java.util.regex包中也提供了对正则表达式的支持。这个包中主要包含了Matcher类和Pattern类。

1.Pattern类

Pattern类主要提供了对正则表达式的编译功能，也就是说能够将String形式的正则表达式以另一种私有的形式包装在pattern里面，其实查看源码可以发现，它内部也是用一个String来存储正则表达式。  
Pattern类主要提供一下几个方法：

```java
public static Pattern compile(String regex);//将regex编译成Pattern类型对象
public static Pattern compile(String regex, int flags);

public String[] split(CharSequence input);//根据对象内保存的正则表达式拆分input字符串
public String[] split(CharSequence input, int limit);

public Matcher matcher(CharSequence input);//返回一个Matcher类对象，该对象利用此Pattern类对象内的正则表达式对input进行匹配
```

2.Matcher类

Matcher类的构造函数被私有化了，并且没有提供静态方法获得Matcher类对象，也就是说Matcher类对象必须依靠Pattern类进行实例化。Matcher主要提供了以下几个方法：

```java
public boolean matches();//返回是否匹配
public String replaceAll();//替换

public boolean find();//查找下一个匹配的子串
public String group();//返回下一个匹配的字串
```

其中主要需要关注的就是find()和group()方法，因为这两个方法是String类中不具备的。举个例子:

```java
package javademo;

import java.util.regex.Matcher;
import java.util.regex.Pattern;


public class JavaDemo {
    public static void main(String args[]) {
        String str = "INSERT INTO dept(deptno,dname,loc) VALUES (#{deptno},#{dname},#{loc}";
        //要取出str中所有#{}中包含的内容
        Pattern pat = Pattern.compile("#\\{.+\\}");//先取出所有满足#{}格式的字串
        Matcher mat = pat.matcher(str);
        while(mat.find())
        {
            System.out.println(mat.group().replaceAll("#|\\{|\\}", ""));//再去除#、{、}
        }
    }
}
```

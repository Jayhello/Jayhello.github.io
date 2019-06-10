---
title: UUID类和Optional类
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

UUID类可以用来随机生成无重复的字符串。  
Optional类可以用来避免if(obj != null)这样的判断。



## 1 UUID类

UUID类可以根据时间戳来生成一个随机的字符串，并且我们可以认为生成的字符串是无重复的（只有千亿分之一的概率重复），它在需要生成随机无重复字符串的场合非常有用，例如给一批文件命名。用法如下：

```java
package javademo;

import java.util.UUID;

public class JavaDemo {
    public static void main(String args[]) {
        for (int i=0; i<3; ++i) {
        UUID uuid = UUID.randomUUID();
        System.out.println(uuid);
        }
    }
}
```

输出结果：  
0077959f-1a47-4447-8b7a-ebe9a6d698c4  
37611a89-84dc-4c06-b066-54ffec5c7371  
0c35e5c5-8b5d-41fb-a5e7-9526d591beca  

## 2 Optional类

Optional类主要对null进行处理，先看下面一段代码。

```java
package javademo;

import java.util.Optional;

public class JavaDemo {
    public static void main(String args[]) {
        String str = "123";
        printString(str);
    }

    static void printString(String str) {
        if (str != null) {//此处必须判断str是否为空
            System.out.println(str);
        }
    }
}
```

在使用方法接受某个对象并对此对象进行处理的时候，我们不可避免的需要判断此引用是否为空，在以往的代码中随处可见这样形式的判断，在时间和空间上都造成了一定的开销。因此Optional类就提供了解决这个问题的方法。  
Optional类提供的方法主要是基于两个思路：  

- 不允许传入空对象
- 允许传入空对象，但在传入空对象时使用默认对象代替

Optional类主要是在对象的传递时对对象进行包装，在包装的过程中就可禁止对空对象进行包装，或者对空对象用默认对象代替，我们仍以String为例：

```java
package javademo;

import java.util.Optional;

public class JavaDemo {
    public static void main(String args[]) {
        // of()方法+get()方法，不允许传入空对象
        Optional<String> notNull = Optional.of("123");
        System.out.println(notNull.get().toString());

        // ofNullable()方法+orElse()方法，允许传入空对象,对空对象使用其他对象代替
        Optional<String> canBeNull = Optional.ofNullable("123");
        System.out.println(canBeNull.orElse("456"));

        Optional<String> canBeNull1 = Optional.ofNullable(null);
        System.out.println(canBeNull1.orElse("456"));
    }
}
```

示例中使用方法名和作用如下：  

方法名     |    作用
:-|:-
static \<T> Optional\<T> of​(T value) | 返回一个Optional\<T>对象，不允许传入空值，如果传入空值则抛出异常
T get()    |    返回包装的T对象，如果为空则抛出一个异常
static \<T> Optional\<T> ofNullable​(T value)    |    允许传入一个空值
T orElse​(T other)    |    返回包装的对象，如果为空则返回other

通过以上示例可以发现，一般可以将of()和get()配合使用，即包装时不允许直接传入空对象，拆包时直接拆包。将ofNullable()和orElse()配合使用，即包装时可以传入空对象，拆包时使用默认对象来代替空对象。或者在任何时候都用orElse方法来拆包。

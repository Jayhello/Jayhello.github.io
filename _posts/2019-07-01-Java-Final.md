---
title: Java final关键字
layout: post
categories: Java
tags: java基础
---
* content
{:toc}

今天总结一下final关键字的作用。可能在三种地方使用final关键字，数据、方法、类。




## 数据

对数据使用final关键字意味着不想让这个数据发生改变，这又分两种情况，一是在编译完成时就不再改变，另一种是在运行时被初始化后不再改变。区分这两种情况的方法是关键字static。

当static和final组合使用时，变量在装载时被初始化，不使用static的final值在创建对象时初始化。

使用final定义的引用不能改变引用的绑定，即这个引用不能再绑定到其它对象，但该对象是可以改变的。**在java中没有给出让对象无法改变的方法，除非你自己定义类来实现。**

还有一种使用final关键字的方法称为“空白final”，即在定义变量时不给出初值，例如：

```java
final int i;
```

但是必须在构造器中对i进行初始化，这样就使得该变量的值与每个对象相关，而不是类。

也可以将final关键字用在方法的参数上，这意味着在方法体内不可改变该参数的值或引用到的对象。

## final方法

使用final声明的方法有两个作用。

- 一是不希望该方法被覆盖。
- 二是将针对该方法的调用都转为内嵌调用，（也就是C++中内联函数的概念）。目前已经不建议这种使用方法了。

这里稍微思考一下，为什么把final方法设计成内嵌调用呢。试想，如果对非final进行内嵌调用是行不通的。因为内嵌调用是在编译时就将代码展开，如果是非final方法可能被复写，存在多态的情况，需要到运行时才能确定调用的具体是哪个方法。因此，非final方法是不可以内嵌的。但这只是说明final方法可以内嵌，但不是必须内嵌。

由此可以联想到，在C++中虚函数应该是不可以声明成内联函数的，理由同上。试验了一下：

```cpp
class Foo
{
public:
    virtual void vFunc() const;
private:
};

class Child: public Foo
{
public:
    virtual void vFunc() const;
};

inline
void Foo::vFunc() const
{
    cout<<"class Foo"<<endl;
}

inline
void Child::vFunc() const
{
    cout<<"class Child"<<endl;
}

int main()
{
    Foo* pCh = new Child();
    pCh->vFunc();

    return 0;
}
```

输出结果为：class Child

也就是说编译器自动无视了内联机制。

使用final可以让方法无法被覆盖，还有一种让方法无法被覆盖的方法是将方法声明为private的，这时候派生类无法访问这个方法自然也就无法覆盖它。需要注意的是，这时候在派生类中声明同名方法编译器是不报错的，但区别在于这样做实际上是声明了一个新的方法而不是覆盖了基类的方法。为了确保程序的行为使我们想要的，应该在需要覆盖的方法前加上@Override注释，这样编译器就会检查这个方法是不是成功覆盖了基类的方法。

## final类

使用final声明的类意味着这个类无法被继承。那么它的方法自然也就没有机会被覆盖，因此将它的方法声明为final是无意义的。

在使用final声明方法或类的时候应该十分谨慎，因为在最初设计类的时候是很难预测这个类日后的使用情况的，也很难排除它需要被继承和覆盖的情况。

## final与安全发布

先看下面一个例子:

```java
package javademo;

class Base {
    public Base() {
        draw();
    }

    public void draw() {
        System.out.println("Base draw");
    }
}

class Son extends Base {
    int i = 1;

    @Override
    public void draw() {
        // TODO Auto-generated method stub
        System.out.println("Son draw i = " + i);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Son son = new Son();
    }
}
```

输出结果：Son draw i = 0  
在基类的构造函数中调用了draw()方法，draw()方法被重写了，因此会调用派生类的draw()方法，输出i，输出结果为0。这是由于java先为对象开辟内存空间，将所有字段赋为0，等到赋值的时候再将0改为要赋的值。在父类构造函数调用draw()的时候，i只被分配了空间，还没有正确赋值，因此输出0。

```java
package javademo;

class Base {
    public Base() {
        draw();
    }

    public void draw() {
        System.out.println("Base draw");
    }
}

class Son extends Base {
    final int i = 1;

    @Override
    public void draw() {
        // TODO Auto-generated method stub
        System.out.println("Son draw i = " + i);
    }
}

public class JavaDemo {
    public static void main(String[] args) {
        Son son = new Son();
    }
}
```

输出：Son draw i = 1
可以看到将i声明为final字段，同样的代码输出i=1。这是由于final声明的字段必须被正确的初始化，虚拟机在i被分配空间之后就立即将其赋为正确的值。所以在调用draw()的时候i已经被正确赋值了。

关于安全发布，在多线程情境下，很有可能在一个字段被分配空间之后，但还未正确初始化时就被其他线程使用，也就是不同步。根据以上的final特性，如果某一字段在初始化后就不需要被修改，就可以将其设为final，使内存的分配与初始化同时进行，不可以在中间被其他线程打断，这也就是安全发布的意义。  

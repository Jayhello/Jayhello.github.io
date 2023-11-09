---
layout: post
title: "设计模式-创建者模式(c++实现)"
date:   2020-04-12
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

1. 简单说明创建者模式的实例, 以及`c++`写的时候注意事项

2. 带有director的构建者模式

### 1.  创建者模式的实例, 以及`c++`写的时候注意事项

建造者模式或者构建者模式也是创建型模式，建造者模式的原理和代码实现非常简单，掌握起来并不难，难点在于应用场景。

为什么需要建造者模式？

1.对象的构造参数很多, 部分参数其实我们也用不到

如下第二种方式构建对象语义更加清晰点尤其是对于参数特别多的对象

```c++
MyClass o = new MyClass(5, 5.5, 'A', var, 1000, obj9, "hello");
MyClass o = MyClass.builder().id(5).avr(5.5).level('A').d(var).e(1000).f(obj9).g("hello");
```

下面给出 person类示例，一堆信息那么构建的时候就很麻烦了需要传递一堆参数:

```c++
class Person{
    std::string m_name, m_street_address, m_post_code, m_city; // 地址信息
    std::string m_company_name, m_position, m_annual_income;   // 公司信息
    ....
};    
```

下面给出一个简单的创建者模式:

1. 创建者模式需要和原有的类尽量不耦合, 否则就会使得原有的person类变得不够单一

2. 为了让类的构建使用构建者模式可以考虑将类的构造函数设置为private，当然建议还是public，然后复杂的构造optional

3. Person的getter/setter不要返回Person&(不符合get/set语义), 这个留给构建者模式.

4. 必填选项传入构造函数, 选填作为函数

```c++
class Person{
    std::string m_name, m_street_address, m_post_code, m_city;  // Personal Detail
    std::string m_company_name, m_position, m_annual_income;    // Employment Detail
    Person(string name):m_name(name){}  // 构造函数private, 简单的保留名称
    ...getter/setter还是保留为public
public:
    friend ostream& operator<<(ostream&, Person);

    friend class PersonBuilder;   // 建造者为友元类, 方便访问成员函数
    static PersonBuilder Create(string name);  // 方便调用
};
```

构建者类如下(构建者类因与原来分离, 不然代码过于臃肿):

```c++
class PersonBuilder{
public:
    PersonBuilder(string name):person_(name){}

    PersonBuilder& Lives(){return *this;}  // 返回构建者的引用
    PersonBuilder& At(string address){
        person_.m_street_address = address;
        return *this;
    }

    PersonBuilder& In(string city){
        person_.m_city = city;
        return *this;
    }

    PersonBuilder& WithPost(string post_code){
        person_.m_post_code = post_code;
        return *this;
    }

    Person Build(){
        return person_;
    }
    operator Person() const { return move(person_); }
    Person person_;
};
```

使用如下:

```c++
PersonBuilder pb("xy");
Person person = pb.Lives()
        .At("恒大花园")
        .WithPost("52000")
        .In("广州番禺")
        .Build();

cout<< person <<endl;
```

构建者模式使用函数组合构建的语义明显优于参数的传递.
也可以用更简单的方式创建:

```c++
PersonBuilder Person::Create(string name){return PersonBuilder(name);}
```

使用如下:

```c++
Person person = Person::Create("xy").Lives()
        .At("东华花园")
        .WithPost("52000")
        .In("广州番禺");    // builder对于调用者是透明的
```

1. 建造者既可以放在类内也可以类外, 还是放在类外吧，不然影响代码可读性

2. Builder可以作为类的友元函数这样可以直接访问类的成员了无需定义一推set, get

3. c++的circular引用问题, 可以返回类的指针

### 2. 带有director的构建者模式

下面以汽车的构造示例说明, 汽车类定义如下:

```c++
class Car{
public:
    Car(string name):name_(name){}

    void SetName(string name){name_ = name;}
    void SetSeats(int s){s_ = s;}
    void SetEngine(int e){e_ = e;}
    void SetDoor(int d){d_ = d;}

    void Info(){
        printf("name: %s, seats: %d, engine: %d, door: %d\n", name_.c_str(), s_, e_, d_);
    }
string name_;
int s_; e_;d_;
};
```

各种汽车的`Build`定义如下:

```c++
class CarBuilder{
public:
    virtual Car* Build() = 0;
};

class SuvBuilder : public CarBuilder{
public:
    virtual Car* Build(){
        Car* p_car = new Car("SUV");
        p_car->SetDoor(1);
        p_car->SetEngine(2);
        p_car->SetSeats(3);

        return p_car;
    }
};

class MiniBuilder: public CarBuilder{
public:
    virtual Car* Build(){
        Car* p_car = new Car("mini");
        p_car->SetDoor(1);
        p_car->SetEngine(1);
        p_car->SetSeats(1);

        return p_car;
    }
};
```

`director`定义如下:

```c++
class CarDirector{
public:
    CarDirector(CarBuilder* p_build):p_build_(p_build){}

    Car* Build(){
        return p_build_->Build();
    }

CarBuilder* p_build_;
};

```

使用如下:

```c++
CarDirector cd(new SuvBuilder);
Car* pc = cd.Build();
pc->Info();
```

### 引用

https://refactoring.guru/design-patterns/builder

https://time.geekbang.org/column/article/199674

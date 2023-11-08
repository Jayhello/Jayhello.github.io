---
layout: post
title: "设计模式-工厂模式(c++实现)"
date:   2020-03-02
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

1. 通过实例简单说明下工厂模式的使用

2. 通过实际开发使用的实例一步步完善工厂模式, 支持动态注册(免去了一堆 `if-else`)

### 1. 简单说明下工厂模式的使用

简单工厂（Simple Factory）

看个示例, 我们根据不同的配置文件使用不同的解析器将文件解析到内存. 不同的解析器定义如下(具体实现省略了):

```c++
class IConfig{   // 定义配置base子类是为了factory统一返回
public:
    virtual int Parse(string file) = 0;
};
class JsonConfig : public IConfig{
public:
    virtual int Parse(string file)override{return 0;}
};
class XmlConfig : public IConfig{
public:
    virtual int Parse(string file)override{return 0;}
};
class YamlConfig : public IConfig{
public:
    virtual int Parse(string file)override{return 0;}
};
```


使用如下:
```c++
class Application{
public:
    int Load(string file){
        IConfig* p_config;

        if(file.find("json")){
            p_config = new JsonConfig();
        }else if(file.find("xml")){
            p_config = new XmlConfig();
        }else if(file.find("yaml")){
            p_config = new YamlConfig();
        }

        p_config->Parse(file);
        return 0;
    }
};
```

可以考虑将上面配置文件的创建使用工厂模式创建出来
```c++
class ConfigFactory{
public:
    static IConfig* GetInstance(string file){ // 统一返回基类指针
        IConfig* p_config;

        if(file.find("json")){
            p_config = new JsonConfig();
        }else if(file.find("xml")){
            p_config = new XmlConfig();
        }else if(file.find("yaml")){
            p_config = new YamlConfig();
        }

        return p_config;
    }
};
```

使用如下:

```c++
class Application{
public:
    int Load(string file){
        IConfig* p_config = ConfigFactory::GetInstance(file); // 这里的逻辑更加清晰明了
        // 判断什么的忽略了先...

        p_config->Parse(file);
        return 0;
    }
};
```

上述工厂模式的优点是: 简单,易理解; 缺点是新的对象需要改factory加case。

### 2. 动态注册工厂模式

1. factory的模板参数是基类，和参数

2. 通过static的map(key为字符串, fun可以创建类)记录所有注册的类

```c++
template<class Base, typename... Args>
class Factory:public boost::noncopyable{
public:
    typedef Base* PBase;
    typedef std::unique_ptr<Base> UpBase;
    typedef std::vector<UpBase> VecUpObj;
    typedef std::function<UpBase(const Args& ...args)> AllocFun;

    static UpBase CreateObj(const std::string& key, const Args& ...args);

    static void Register(const std::string& key, AllocFun alloc);

    static Factory& GetInstance();

private:
    static std::unordered_map<std::string, AllocFun> m_key_alloc;
};
```

注上面的使用`&&完美转发std::forward`更好点

```c++
typedef std::function<BaseClassPtr(Args&& ...)> AllocFun;

static BaseClassPtr CreateObj(const Key& key, Args&& ...args){
    return m_alloc_fun_[key](std::forward<Args>(args)...); // 完美转发
}
```

改为如下, 不然有左值转右值问题:

```c++
typedef std::function<BasePtr(const Args& ...)> AllocFun;
static BasePtr Create(const Key& key, const Args& ...args){
    if(0 == m_key_alloc.count(key)){
        return nullptr;
    }

    return m_key_alloc[key](args...);
}
```

函数实现如下:

1.模板的static成员变量需要在类外再次定义, 否则会报错未定义的引用，而且需要加上typename
```c++
template<class Base, typename... Args>
std::unordered_map<std::string, typename Factory<Base, Args...>::AllocFun> Factory<Base, Args...>::m_key_alloc;

template<class Base, typename... Args>
Factory<Base, Args...>& Factory<Base, Args...>::GetInstance(){
    static Factory factory;
    return factory;
}

template<class Base, typename... Args>
void Factory<Base, Args...>::Register(const std::string& key, typename Factory<Base, Args...>::AllocFun alloc){
    m_key_alloc[key] = alloc;
}
template<class Base, typename... Args>
typename Factory<Base, Args...>::UpBase Factory<Base, Args...>::CreateObj(const std::string& key, const Args& ...args){
    if(m_key_alloc.count(key)){
        return m_key_alloc[key](args...);
    }
}
```

使用如下：

```c++
typedef Factory<TemplateMsgBase, Key> TemplateMsgFactory;

TemplateMsgFactory::GetInstance().Register("base", [](const Key& key){
    return TemplateMsgFactory::UpBase(new TemplateMsgBase(key));
});

TemplateMsgFactory::GetInstance().Register("sub", [](const Key& key){
    return TemplateMsgFactory::UpBase(new TemplateMsgComment(key));
});

Key key;
key.set_name("sub");
auto p_msg = TemplateMsgFactory::GetInstance().CreateObj("base", key);
p_msg->printf();
```

上述调用方式不太好，不如直接继承Factory, 然后再初始化:

```c++
class MsgFactory: public Factory<TemplateMsgBase, Key>{
public:
    static MsgFactory& GetInstance(){
        static MsgFactory factory;
        return factory;
    }

protected:
    typedef Factory<TemplateMsgBase, Key> BaseFactory;
    MsgFactory():BaseFactory(){
        BaseFactory::GetInstance().Register("base", [](const Key& key){
            return BaseFactory::UpBase(new TemplateMsgBase(key));
        });

        BaseFactory::GetInstance().Register("sub", [](const Key& key){
            return BaseFactory::UpBase(new TemplateMsgComment(key));
        });
    }
};
```

使用如下：

```c++
Key key;
key.set_name("sub");
auto p_msg = MsgFactory::GetInstance().CreateObj("base", key);
p_msg->printf();
```

当然上面的方法注册lambda函数也不太优雅。
下面给出另一个方法, 支持任意的key作为键(对象的创建使用creator而不是上面的lambda, 记得creatorbase这样的继承体系方便扩展):

```c++
template<typename Base, typename ...Args>
class CreatorBase{
public:
    virtual Base* Create(const Args&... args) = 0;
};

template<typename Base, typename ...Args>
class CreatorSub: public CreatorBase<Base, Args...>{
    public:
    virtual Base* Create(const Args&... args){return new Base(args...);}
};
```

`Factory` 定义如下:

```c++
template<typename Key, typename Base, typename ...Args>
class Factory{
public:
    typedef Base* PBase;
    typedef CreatorBase<Base, Args...>* PCreator;
    typedef std::unique_ptr<CreatorBase<Base, Args...>> PuCreator;

    static PBase CreateObj(const Key& key, const Args& ...args);

    static void Register(const Key& key, PCreator p_alloc);

    static Factory& GetInstance();

    ~Factory();
protected:
    static std::unordered_map<Key, PuCreator> m_key_pcreator;
};
创建对象
template<typename Key, typename Base, typename ...Args>
typename Factory<Key, Base, Args...>::PBase Factory<Key, Base, Args...>::CreateObj(const Key& key, const Args& ...args){
    return m_key_pcreator[key]->Create(args...);
}
当然上面也可以抛出异常：
template<typename Key, typename Base, typename ...Args>
typename Factory<Key, Base, Args...>::PBase Factory<Key, Base, Args...>::CreateObj(const Key& key, const Args& ...args){
    if(0 == m_key_pcreator.count(key)){
        throw std::invalid_argument("key not exist");
    }

    return m_key_pcreator[key]->Create(args...);
}
注册：
template<typename Key, typename Base, typename ...Args>
void Factory<Key, Base, Args...>::Register(const Key& key, typename Factory<Key, Base, Args...>::PCreator p_alloc){
    typename Factory<Key, Base, Args...>::PuCreator p(p_alloc);
    m_key_pcreator[key] = std::move(p);
}

template<typename Key, typename Base, typename ...Args>
Factory<Key, Base, Args...>& Factory<Key, Base, Args...>::GetInstance(){
    static Factory factory;
    return factory;
}
测试类：
class Base{
public:
    Base(string name="base"):name_(name){}
    virtual void print(){cout<<"base name: "<<name_<<endl;}
    string name_;
};

class Sub:public Base{
public:
    Sub(string name="sub"):Base(name){}
    virtual void print(){cout<<"sub name: "<<name_<<endl;}
};
```

使用如下(由于使用了creator继承, 且有虚函数，因此register那里可以直接传入子类的指针)：

```c++
try {
    BaseFactory::GetInstance().CreateObj(2, "11")->print();
}catch (const std::invalid_argument& ia){
    std::cerr << "Invalid argument: " << ia.what() << '\n';
}
```

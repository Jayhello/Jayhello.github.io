---
layout: post
title: "结构型模式-职责链模式(c++实现)"
date:   2020-04-05
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

`职责链模式`也是开发过程中常用的一种设计模式, 相对比较简单, 本文给出实际使用的简化版的实例。

`职责链模式` 将请求的发送和接收解耦，让多个接收对象都有机会处理这个请求。将这些接收对象串成一条链，并沿着这条链传递这个请求，直到链上的某个接收对象能够处理它为止。

下面给出 根据账号, ip信息做反垃圾的示例:

```c++
struct User{
    std::string name;
    std::string pwd;
    std::string ip;
};
```

过滤器基类设计如下(简化版本):

```c++
class FilterBase{
public:
    virtual std::string Name() = 0;
    virtual int DoCheck(User user, bool& is_spam) = 0;
};
```

账号过滤器如下:

```c++
class AccountFilter:public FilterBase{
public:
    virtual std::string Name(){return "check name password";}
    virtual int DoCheck(User user, bool& is_spam){
        is_spam = (user.name == "xy" and user.pwd == "xy_pwd");
        return 0;
    }
};
```

Ip过滤器如下:

```c++
class IpFilter:public FilterBase{
public:
    virtual std::string Name(){return "ip white list";}
    virtual int DoCheck(User user, bool& is_spam){
        is_spam = (user.ip == "black ip");
        return 0;
    }
};
```

反垃圾链如下:

```c++
class FilterChain{
public:
    int DoCheck(User user, bool& is_spam){
        for(auto plugin : m_vec_plugin){
            if(0 == plugin->DoCheck(user, is_spam) and is_spam){
                printf("plugin %s hit\n", plugin->Name().c_str());
                break;
            }
        }
    }

    FilterChain& AddFilter(FilterBase* p_plugin){
        m_vec_plugin.push_back(p_plugin);

        return *this;
    }

private:
    std::vector<FilterBase*> m_vec_plugin;
};
```

使用如下(支持按需动态的增加职责链过滤器):

```
void test_chain(){
    User user1{"xy", "xy_pwd", ""};
    User user2{"xy2", "xy_pwd2", "black ip"};

    FilterChain fc;
    fc.AddFilter(new AccountFilter).AddFilter(new IpFilter);

    bool is_spam;
    fc.DoCheck(user1, is_spam);
    fc.DoCheck(user2, is_spam);
}
```

### 职责链模式 vs 装饰器模式

`职责链模式`: 只关心被调用者之间的调用传递。

`装饰器模式`: 它不关心外界如何调用，只注重对对象功能的加强，装饰后还是对象本身。

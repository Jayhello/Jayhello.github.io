---
layout: post
title: "设计模式-状态机模式(c++实现)"
date:   2020-05-10
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

1. 简单介绍下状态机模式

2. 给出基于`c++`的状态机模式过滤代码中的 // 注释


### 1. 什么是状态机 ？ 

`有限状态机`（英语：`finite-state machine`，缩写：`FSM`）又称有限状态自动机，简称状态机，是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。

由三个部分组成: `状态（State）`、`事件（Event）`、`动作（Action）`其中，事件也称为转移条件（Transition Condition）。事件触发状态的转移及动作的执行。

状态机模式在软件开发中不是那么常用但是在其使用的场景中, 能将繁琐的if-else代码变得清晰可读、易于扩展、DEBUG, 主要应用于游戏, 工作流引擎等。

### 2. 代码注释示例

示例 1, 删除 //相关的注释 常规思路如下:

1. 以`\n`将代码分成`vector<string> lines`

2. 如果每一line中没有/则append到code, 有一个 / 则非法, 2个以上//取首个//位置然后将后面字符全部删掉

`if-else`实现实例如下:

```c++
int GetCodes(const string& code, string& real_code){
    int state = 0;

    for(char ch : code){
        if(0 == state){
            if('/' == ch){
                state = 1;  //  1 个 /
            }else{
                real_code.push_back(ch);  // 正常code
            }
        }else if(1 == state){ // 1个 /
            if('/' == ch){ // 后面必须紧跟 /
                state = 2; // 转移到状态 2 , 两个 /
            }else{
                return -2; // 后面不是 / 则错误
            }
        }else if(2 == state){
            if('\n' == ch){    // 两个 / 遇到 \n换行, 转移状态 0
                real_code.push_back('\n');
                state = 0;
            }// 否则忽略注释代码
        }
    }

    return 0;
}
```

状态机模式代码

```c++
class ICodeState{    // 基类接口
public:
    virtual void EatChar(char ch) = 0;  // eat字符 驱动转态转移
};

typedef std::shared_ptr<ICodeState> CodeStatePtr;

// 状态实例如下:

class CodeLineFsm;
typedef CodeLineFsm* CodeLineFsmPtr;

class CodeStateNormal : public ICodeState{
public:
    CodeStateNormal(CodeLineFsmPtr ptr):ptr_(ptr){}
    virtual void EatChar(char ch) override;

    CodeLineFsmPtr ptr_;   // 状态机 context
};

class CodeStateOneSlash : public ICodeState{
public:
    CodeStateOneSlash(CodeLineFsmPtr ptr):ptr_(ptr){}
    virtual void EatChar(char ch) override;
    CodeLineFsmPtr ptr_;
};

class CodeStateTwoSlash : public ICodeState{
public:
    CodeStateTwoSlash(CodeLineFsmPtr ptr):ptr_(ptr){}
    virtual void EatChar(char ch) override;
    CodeLineFsmPtr ptr_;
};

// 状态机类定义如下:

class CodeLineFsm{
public:
    CodeLineFsm(){
        state_ptr_ = std::make_shared<CodeStateNormal>(this); // 初始化正常代码状态
        real_codes_.clear();
    }

    string ExtractCode(const string& code);  // 对外接口提取正常代码

    void AppendChar(char ch);   // 状态迁移时候 append char
    void ResetState(CodeStatePtr state_ptr);// 状态迁移重置 state

protected:
    CodeStatePtr state_ptr_;  // 状态指针
    string real_codes_;       // 状态迁移是append正常代码
};
```

下面看下状态迁移实例的eatChar代码

```c++
void CodeStateNormal::EatChar(char ch){
    if(ch == '/'){  // 迁移到单个slash
        ptr_->ResetState(std::make_shared<CodeStateOneSlash>(ptr_));
    }else{
        ptr_->AppendChar(ch);  // 正常则append代码字符
    }
}

void CodeStateOneSlash::EatChar(char ch){
    if(ch == '/'){// 迁移到 2 /
        ptr_->ResetState(std::make_shared<CodeStateTwoSlash>(ptr_));
    }else{ // 注释错误了
        throw std::invalid_argument("/ must follow with /");
    }
}

void CodeStateTwoSlash::EatChar(char ch){
    if(ch == '\n'){ // 遇到\n, 注释结束了
        ptr_->AppendChar('\n');
        ptr_->ResetState(std::make_shared<CodeStateNormal>(ptr_));
    }
}
```

状态机实现定义如下:

```c++
string CodeLineFsm::ExtractCode(const string& code){
    for(char ch : code){
        state_ptr_->EatChar(ch);
    }

    return real_codes_;
}

void CodeLineFsm::AppendChar(char ch){
    real_codes_.push_back(ch);
}

void CodeLineFsm::ResetState(CodeStatePtr state_ptr){
    state_ptr_.swap(state_ptr);
}
```

使用实例如下:

```c++
httplib::CodeLineFsm fsm;
string real_code = fsm.ExtractCode(codes);
```

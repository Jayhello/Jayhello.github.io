---
layout: post
title: "设计模式-备忘录模式(c++实现)"
date:   2020-05-01
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

备忘录模式，也叫快照（Snapshot）模式，英文翻译是 Memento Design Pattern。

应用场景也比较明确和有限，主要是用来防丢失、撤销、恢复等。简单的应用比如：游戏存档、浏览器中的后退、word中的撤销。都是应用了这种设计模式。

下面用 `c++` 的备忘录模式实现简化的文本备份恢复。

```c++
class TextBase{
public:
    const string &getText() const {return text_;}
    void setText(const string &text) {text_ = text;}
    // .....

    std::string text_;
    int version_;
};
```

备忘录类定义如下:

```c++
class Memento : public TextBase{   // 备忘录定义如下：
};

class TextEdit : public TextBase{
public:
    void Save(Memento* memento){
        memento->setText(text_);
        memento->setVersion(version_);
    }

    void Restore(const Memento& memento){
        text_ = memento.getText();
        version_ = memento.getVersion();
    }

    void Print(){printf("text: %s, version: %d\n", text_.c_str(), version_);}
};
```

管理类如下:

```c++
typedef TextEdit* TextEditPtr;
class TextManager{
public:
    TextManager(TextEditPtr ptr):ptr_(ptr){}

    void Save(){
        Memento memento;
        ptr_->Save(&memento);
        backups_.push_back(memento);
    }

    void Restore(){
        const Memento& memento = backups_.back();
        ptr_->Restore(memento);
        backups_.pop_back();
    }

    int Size()const{return backups_.size();}

    TextEditPtr ptr_;
    std::vector<Memento> backups_;
};
```

使用如下:

```c++
TextEditPtr p_edit = new TextEdit;  // 里面先忽略指针释放的问题, 主要关注实现以及使用
TextManager manager(p_edit);        // 文本对象给manager管理

p_edit->setText("version 1");
p_edit->setVersion(1);
p_edit->Print();

manager.Save();     // 存储上个版本(实际使用的时候也可以异步定时存储)
printf("now back up size: %d\n", manager.Size());

p_edit->setText("version 2");
p_edit->setVersion(2);

p_edit->Print();

manager.Restore();  // 恢复上个版本
printf("now restore size: %d\n", manager.Size());

p_edit->Print();
```

---
layout: post
title: "设计模式-装饰器模式(c++实现)"
date:   2020-03-17
tags: [设计模式]
comments: true
author: xy
# toc: true  # 这里加目录导致看到地方太小了
---

本文主要内容如下:

1. 简单说明装饰器模式示例及其优点

2. `c++`文件压缩加密的实例

3. `c++ html`格式的示例

### 1. 简单说明装饰器模式示例及其优点

装饰器模式是为了运行时动态的扩展一个类的功能。它谨遵开闭原则，它实现的关键在于继承和组合的结合使用，解耦对象之间的关系。 

在Java标准库中，InputStream是抽象类，FileInputStream、ServletInputStream、Socket.getInputStream()这些InputStream都是最终数据源。

现在，如果要给不同的最终数据源增加缓冲功能、计算签名功能、加密解密功能，那么，3个最终数据源、3种功能一共需要9个子类。如果继续增加最终数据源，或者增加新功能，子类会爆炸式增长，这种设计方式显然是不可取的。

例如我们可以按照下面的方式动态添加功能:

```
// 创建原始的数据源:
InputStream fis = new FileInputStream("test.gz");
// 增加缓冲功能:
InputStream bis = new BufferedInputStream(fis);
// 增加解压缩功能:
InputStream gis = new GZIPInputStream(bis);

// 或者简单点一次性写
InputStream input = new GZIPInputStream( // 第二层装饰
    new BufferedInputStream( // 第一层装饰
        new FileInputStream("test.gz") // 核心功能
    ));
```

Decorator模式有什么好处？它实际上把核心功能和附加功能给分开了。核心功能指FileInputStream这些真正读数据的源头，附加功能指加缓冲、压缩、解密这些功能。如果我们要新增核心功能，就增加Component的子类。

### 2. `c++`文件压缩加密的实例

    下面简单给出 `c++` 实例,  接口如下:

```c++
class IDataSource{
public:
    virtual string ReadData() = 0;
    virtual int WriteData(string data) = 0;
};
```

实现如下:

```c++
class FileDataSSource : public IDataSource{
public:
    FileDataSSource(string path):path_(path){}

    virtual string ReadData(){
        File f(path_);
        f.Open();
        return f.Read();
    }

    virtual int WriteData(string data){
        File f(path_);
        f.Open();
        return f.Write(data);
    }

    string path_;
};
```

我们现在要给类新增读取的加解密, 压缩功能, 使用装饰器模式:

```c++
typedef IDataSource* DataSourcePtr;
class DataSourceDecorate : public IDataSource{
public:
    DataSourceDecorate(DataSourcePtr ptr):ptr_(ptr){}

    DataSourcePtr ptr_;
};
```

加解密(当然加解密的类也可以依赖注入, 这里为了简化问题):

```c++
class EncryptionDecorate : public DataSourceDecorate{
public:
    EncryptionDecorate(DataSourcePtr ptr):DataSourceDecorate(ptr){}

    virtual string ReadData(){
        string buff = Encryption::Decry(ptr_->ReadData());// 读取之后解密
        printf("EncryptionDecorate read: %s\n", buff.c_str());
        return buff;
    }

    virtual int WriteData(string data){
        string buff = Encryption::Encry(data);加密后写入
        printf("EncryptionDecorate write: %s\n", buff.c_str());
        return ptr_->WriteData(buff);
    }
};
```

压缩:

```c++
class CompressDecorate : public DataSourceDecorate{
public:
    CompressDecorate(DataSourcePtr ptr):DataSourceDecorate(ptr){}

    virtual string ReadData(){
        string buff = Gzip::DeCompress(ptr_->ReadData()); // 读取后解压缩
        printf("CompressDecorate read: %s\n", buff.c_str());
        return buff;
    }

    virtual int WriteData(string data){
        string buff = Gzip::Compress(data);  // 压缩后写入
        printf("CompressDecorate write: %s\n", buff.c_str());
        return ptr_->WriteData(buff);
    }
};
```

使用如下:

```c++
string path = "/tmp/xy.txt";
DataSourcePtr p_base = new FileDataSSource(path);
DataSourcePtr p_encrypt = new EncryptionDecorate(p_base);
DataSourcePtr p_compress = new CompressDecorate(p_encrypt);

p_compress->WriteData("123");
string buff = p_compress->ReadData();
cout<<buff<<endl;
```

### 3. `c++ html`格式的示例

为了加深理解, 下面给出文本字体的示例。假设我们需要渲染一个HTML的文本，但是文本还可以附加一些效果，比如加粗、变斜体、加下划线等。为了实现动态附加效果，可以采用Decorator模式。

```c++
class Text{      // 接口预留
public:
    virtual std::string Show()const = 0;
};
class PureText : public Text{
public:
    virtual std::string Show()const{
        return text_;
    }

    void SetText(std::string text){
        text_ = text;
    }

protected:
    std::string text_;
};
```

装饰器，给文字加粗、加span

```c++
class TextDecorator:public Text{
public:
    TextDecorator(Text* p):p_obj(p){
    }

protected:
    Text* p_obj;     // 对所有的装饰类加工
};
class BoldTextDecorator:public TextDecorator{
public:
    using::TextDecorator::TextDecorator;

    virtual std::string Show()const{
        return "<br>" + p_obj->Show() + "</br>";  // 现有的类装饰
    }
};
class SpanTextDecorator: public TextDecorator{
public:
    SpanTextDecorator(Text* p):TextDecorator(p){
    }

    virtual std::string Show()const{
        return "<span>" + p_obj->Show() + "</span>";
    }
};
```

使用如下：

```c++
void test_decorator(){
    PureText pure;
    pure.SetText("pure");

    Text* p_bold = new BoldTextDecorator(&pure);
    std::cout<<p_bold->Show()<<std::endl;

    Text* p_bold_span = new SpanTextDecorator(p_bold);
    std::cout<<p_bold_span->Show()<<std::endl;
}
```

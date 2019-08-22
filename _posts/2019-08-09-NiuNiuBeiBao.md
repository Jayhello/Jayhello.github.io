---
title: 牛客网编程题《牛牛的背包问题》
layout: post
categories: 算法题
tags: 回溯
---
* content
{:toc}

牛牛准备参加学校组织的春游, 出发前牛牛准备往背包里装入一些零食, 牛牛的背包容量为w。 牛牛家里一共有n袋零食, 第i袋零食体积为v[i]。牛牛想知道在总体积不超过背包容量的情况下,他一共有多少种零食放法(总体积为0也算一种放法)。  





# 1 题目描述  

## 输入描述：

输入包括两行
第一行为两个正整数n和w(1 <= n <= 30, 1 <= w <= 2 * 10^9),表示零食的数量和背包的容量。
第二行n个正整数v[i](0 <= v[i] <= 10^9),表示每袋零食的体积。

## 输出描述：

输出一个正整数, 表示牛牛一共有多少种零食放法。

## 示例：
输入：  
3 10  
1 2 4  

输出：  
8  

说明：  
三种零食总体积小于10,于是每种零食有放入和不放入两种情况，一共有2\*2\*2 = 8种情况。  

# 2 分析

这题这一看和背包问题的描述很类似，惟一的区别在于要求输出是放法的种数，但是实际上没有办法用背包问题的解法来解决。  
要想到解法也很容易，对于每种零食有放和不放两种情况，那么一共有n种零食，也就是有2的n次方种方法，用回溯法对每种情况验证，总体积小于背包容量就将结果+1。根据这个思路可以写出如下解法。

## 初始思路：

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

vector<int> v; //  每袋零食的体积
int n;  //  零食的袋数
// 用于累计放法种数的函数，i表示进行到i袋零食的抉择，w表示剩余背包容量
int accumulate(int i, int w)
{
    if(w<0)
    {
        return 0;
    }
    
    // i超过零食的袋数，说明这是一种可行的办法
    if(i >= n)
    {
        return 1;
    }
    
    // 对于每袋零食，有放和不放两种情况，然后递归的对下一代零食调用本函数
    return accumulate(i+1,w-v[i]) + accumulate(i+1, w);
    
}

int main()
{
    int w;
    cin>>n;
    cin>>w;
    v = vector<int>(n, 0);
    for(int i = 0; i<n; ++i)
    {
        cin>>v[i];
    }
    
    cout<<accumulate(0,w);
    return 0;
}
```

结果：  
不通过  
您的代码已保存  
运行超时:您的程序未能在规定时间内运行结束，请检查是否循环有错或算法复杂度过大。  
case通过率为80.00%  

纯暴力的解法是通过不了所有的测试用例的，因为最多要进行2的30次方的函数调用，因此必须对回溯进行剪枝。  

在题目的说明中已经给出了提示，三袋零食的总体积小于背包容量，因此一共有8种放法。也就是说，当处理到第i袋零食时，如果剩余的零食总体积小于背包容量的话，就可以直接累加2的（n-i）次方种放法。下面根据这个思路对方法进行优化。  

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

vector<int> v; // 每袋零食的体积

// 新增
vector<int> iSum; // iSum[i]表示从第i个到最后一个的和

int n;  //  零食的袋数
// 用于累计放法种数的函数，i表示进行到i袋零食的抉择，w表示剩余背包容量
int accumulate(int i, int w)
{
    if(w<0)
    {
        return 0;
    }
    
    // i超过零食的袋数，说明这是一种可行的办法
    if(i >= n)
    {
        return 1;
    }
    
    // 新增 
    if(iSum[i] <= w)
    {
        return 1<<(n-i);
    }
    
    // 对于每袋零食，有放和不放两种情况，然后递归的对下一代零食调用本函数
    return accumulate(i+1,w-v[i]) + accumulate(i+1, w);
    
}

int main()
{
    int w;
    cin>>n;
    cin>>w;
    v = vector<int>(n, 0);
    for(int i = 0; i<n; ++i)
    {
        cin>>v[i];
    }
    
    // 新增
    iSum = v;
    for(int i = n-2; i >= 0; --i)
    {
        iSum[i] += iSum[i+1];
    }
    
    cout<<accumulate(0,w);
    return 0;
}
```

结果：  
您的代码已保存
答案错误:您提交的程序没有通过所有的测试用例
case通过率为50.00%

用例:
21 1165911996
842104736 130059605 359419358 682646280 378385685 622124412 740110626 814007758 557557315 40153082 542984016 274340808 991565332 765434204 225621097 350652062 714078666 381520025 613885618 64141537 783016950

对应输出应该为:

703

你的输出为:

2097152  

输出结果错误了，而且差的很离谱，为什么呢。观察给出的输出样例，发现给的零食体积都在10的9次方这个数量级，那么累加之后是会超过int类型的范围的，因此要把累加数组改为long long类型的。这是一个很容易犯但又难以发现的错误，就是要考虑数据类型的范围。修改后：  

```cpp
#include<iostream>
#include<vector>
#include<algorithm>
using namespace std;

vector<int> v; // 每袋零食的体积

// 新增
vector<long long> iSum; // iSum[i]表示从第i个到最后一个的和

int n;  //  零食的袋数
// 用于累计放法种数的函数，i表示进行到i袋零食的抉择，w表示剩余背包容量
int accumulate(int i, int w)
{
    if(w<0)
    {
        return 0;
    }
    
    // i超过零食的袋数，说明这是一种可行的办法
    if(i >= n)
    {
        return 1;
    }
    
    // 新增 
    if(iSum[i] <= w)
    {
        return 1<<(n-i);
    }
    
    // 对于每袋零食，有放和不放两种情况，然后递归的对下一代零食调用本函数
    return accumulate(i+1,w-v[i]) + accumulate(i+1, w);
    
}

int main()
{
    int w;
    cin>>n;
    cin>>w;
    v = vector<int>(n, 0);
    for(int i = 0; i<n; ++i)
    {
        cin>>v[i];
    }
    
    // 新增
    iSum = vector<long long>(v.begin(), v.end());
    for(int i = n-2; i >= 0; --i)
    {
        iSum[i] += iSum[i+1];
    }
    
    cout<<accumulate(0,w);
    return 0;
}
```

结果：
通过
您的代码已保存
答案正确:恭喜！您提交的程序通过了所有的测试用例

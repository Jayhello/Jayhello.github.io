---
title: Create Your Blog on Github
layout: post
categories: jekyll
tags: jekyll github
---
* content
{:toc}

这是我在github上创建博客后写的第一篇博文，就分享一下怎么在github上创建自己的博客吧。在自己搭建还是fork别人的项目这个问题上我也纠结了很久，最后还是决定先fork别人的项目吧，毕竟可以快速搭建好并投入使用。

## 1 寻找一个钟意的主题
首先要有一个github账号，这个不赘述。
首先在[themes](http://jekyllthemes.org/)这个网站上寻找一个自己钟意的主题，点击demo可以预览该主题的博客，点击homepage可以进入repostory。

![themes](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/themes.png)

![themes](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/homepageAndDemo.png)

## 2 fork主题
选好主题后，点击homepage进入repostory，点击右上角的fork标志。

![fork](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/fork.png)

fork完成后，点击你头像右边的小箭头，查看你的repostories，就看看到你刚刚fork的项目了。点开这个项目，点击settings。修改项目的名称为xxxx.github.io。注意：xxxx为你的github账号名（最好按照这个格式改，否则你的博客名会很长），点击Rename修改成功。

![name](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/name.png)

往下翻，找到GitHub Pages这一项，下面那个链接就是你的博客网址了,点进去看一下吧。

![link](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/link.png)

## 3 修改主题
这时候博客已经搭建好了，但是博客里的内容还完全和你无关，这时候需要做一些修改。
打开项目目录中的_config.yml来配置你的个人信息,打开你的博客，将两者的信息对照，应该就能知道要改哪些项了。注意baseurl这一项，如果你的项目名称是按照xxxx.github.io俩命名的话就设置为空。（baseurl: ‘’）
![link](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/config.png)
还有一些其他的信息可能在不同的文件里，比如index.html等，如果需要修改，就去这些文件里找，不会很多。

## 4 写博客
你的博文一般是放在_posts这个文件夹中，如果是fork的博客的话里面应该有一些文件了，可以看到后缀名是.md格式，md就是markdown的意思。写博客一般用的是markdown语法来写。可以百度一下，或者到github里搜一下。这里推荐两个写博客的工具:jekyllwriter和visual studio code。VSCode就不用介绍了，支持markdown语法的编辑，还有很多的插件可以用。jekyllwriter是一个专门用来管理github博客的软件，在[jekyllwriter](http://jekyllwriter.com)可以下到，下载后直接解压就可以使用。
![link](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/jekyllwriter.png)

用法很简单：首先在Account选项卡中点击左边的GitHub配置GitHub帐号，点击Add Account，然后点Creat new token,会跳转到github页面。勾选repo然后点下面的generate token，复制生成的token到jekyllwriter。

![link](https://raw.githubusercontent.com/HUSTHuangKai/HUSTHuangKai.github.io/master/_images/CreateYourBlogOnGithub/account.png)
将你的账户添加好后，点击账户名旁边的刷新按钮，就可以显示你的博客repository，如果使用的是默认名称就只有一个\。这时候在file->open post list中可以看到已经有的博文。新建一篇博文，编辑好后点击Publish就可以推送到网页博客中了。
---
title: 在hexo中加入liveRe(来必力)评论功能
date: 2018-03-02 16:31:51
categories: 
tags:
	- hexo
	- liveRe
---


本计划用畅言，但要进行备案，直觉告诉我此处有坑，发现另外一个不错的评论系统——“来必力”——界面Q弹清爽，更重要的是：**在next主题的5.1.4版本中已集成liveRe插件**，只需简单一个步骤即可添加liveRe评论功能
————————————————————————————————————————————————————————————
## 注册并安装City版 ##


![看这猥琐又不失可爱的画风](http://p7wcdketk.bkt.clouddn.com/18-4-30/29128334.jpg)

在“管理页面”中选择安装City版

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/47859752.jpg)

## 配置livere_uid ##

一系列的设置后，在“管理页面”->“代码管理”中的“data-uid”变量就是我们需要的关键值，**将该值复制到Next主题配置文件中的livere_uid:即可**

![](http://p7wcdketk.bkt.clouddn.com/18-4-30/76153538.jpg)
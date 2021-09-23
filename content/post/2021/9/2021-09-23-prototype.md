---
title: 设计模式-Prototype原型模式
author: ymkNK
categories: [Note,Tech,DesignPattern]
tags: [Note,Tech,DesignPattern]
date: 2021-09-23 15:21:54 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/5.jpeg
slug: 2021/9/prototype
toc: true
---
## 场景
有一个object，想要生成与之完全相同的复制品，该如何是实现？
问题：
- 私有数据
- 不想包含具体的包
- 如果原始的对象只是一个接口，具体类都不知道，怎么处理呢？

## 方案
将克隆过程，委派给被克隆的实际对象，由实际对象负责clone
因此，只要实现了clone()方法的，就是原型模式

## 实现
1. 原型（Prototype）接口对clone方法做一个申明，绝大部分情况下，里面只有一个名为clone的方法
2. 具体原型（ConcretePrototype）类实现接口
3. 客户端（Client）可以复制实现了原型接口的任何对象

## 应用
> 将自身作为构造函数的参数传入
> 复制数组

1. 如果需要复制一些对象，同时又希望代码独立于这些对象所属的具体类
2. 如果子类的区别仅在于对象的初始化方式，那么可以使用该模式减少子类的数量，别人创建这些子类的目的可能只是为了创建特定的对象。


## 优点
- 克隆时 无需与所属的具体类耦合
- 克隆原型 防止反复运行初始化
- 更方便的生成复杂对象
- 可以用继承以外的方式来处理复杂对象的不同配置

## 缺点
- 克隆博涵循环引用的复杂对象可能会非常麻烦

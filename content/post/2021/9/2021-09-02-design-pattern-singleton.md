---
title: 设计模式-Singleton单例模式学习笔记
author: ymkNK
categories: [Note,Tech,DesignPattern]
tags: [Note,Tech,DesignPattern]
date: 2021-09-02 15:50:47 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/27.jpeg
slug: 2021/9/design-pattern-singleton
toc: true
---
## 概要
保证一个类只有一个实例，保证一个进程中，某个类有且仅有一个实例

## 方案：
1. 构造函数私有化
2. 新建静态构造方法作为构造函数

## 饿汉式
类加载的时候，就实例化。

## 懒汉式
第一次使用的时候，再实例化。节省资源，但是需要注意线程安全问题

## 总结
### 优点
1. 只有一个实例，避免了重复的内存开销
2. 避免对资源的重复占用 
   
### 缺点
可以理解为一个全局变量，没有接口，不能继承。违背了SRP职责
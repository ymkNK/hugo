---
title: 设计模式-Observer观察者模式
author: ymkNK
categories: [DesignPattern]
tags: [DesignPattern]
date: 2021-11-25 15:21:43 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/5.jpeg
slug: 2021/11/observer
toc: true
---
## 简介
行为型设计模式

## 组成部分
- 发布者 接口
- 订阅者 接口
- 具体订阅者 实现
- 客户端

## 使用场景
需要监听某种事件的变化，然后通知会对这种变化感兴趣的对象

## 优缺点
优点
- 减少发布者和订阅者的耦合
- 支持开闭原则

缺点
- 没有完全解耦，可能会存在循环依赖
- 性能损耗
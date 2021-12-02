---
title: 设计模式-Facade门面模式
author: ymkNK
categories: [DesignPattern]
tags: [DesignPattern]
date: 2021-12-02 15:55:10 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/17.jpeg
slug: 2021/12/design-pattern-facade
toc: true
---
## 简介
结构型设计模式

## 组成部分
- 门面接口
- 复杂子系统

## 真实场景
- 银行柜员

## 使用场景
- 底层是复杂的子系统，对外暴露提供的就只是一个简单的接口
- 构建多层系统结构，利用门面模式最为每一层的入口，简化层间调用

## 实现方式
- 新增
- 维护、变更

## 优缺点
优点
- 简化调用逻辑
- 高内聚低耦合
- 更好的划分层次，提高了安全性
- 符合迪米特法则，最少知道原则

缺点
- 有可能成为所有类型都耦合的上帝对象
- 不符合开闭原则：增加子系统需要对原有系统进行修改
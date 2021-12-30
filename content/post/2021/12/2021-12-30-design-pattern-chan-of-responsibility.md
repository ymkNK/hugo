---
title: 设计模式-Chain of Responsibility享元模式
author: ymkNK
categories: [DesignPattern]
tags: [DesignPattern]
date: 2021-12-30 15:43:25 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/15.jpeg
slug: 2021/12/design-pattern-chain-of-responsibility
toc: true
---
## 核心思想
一条链，有多个对象，都可以去处理一个请求

## 举例
- 去医院问诊
- java中的interceptor拦截器

## 实现
最标准的实现用链表来实现，也可以用数组等等
- 定义接口
- base handler 共通的逻辑
- 具体的实现逻辑

但是具体如何配置链，放在数据库、配置中心都是ok的，

### 优点
- 符合单一职责原则
- 开闭原则
- 比较灵活，支持运行时变更

### 缺点
- 链特别长的时候，可能会有很多链都走不动，造成资源浪费
- 可能有部分消息未处理
- 太灵活，也可能会有缺点，比如重复处理（可以放一个context，共享一个状态）
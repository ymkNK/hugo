---
title: 设计模式-Composite组合模式
author: ymkNK
categories: [DesignPattern]
tags: [DesignPattern]
date: 2021-11-11 15:32:55 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/1.jpeg
slug: 2021/11/design-pattern-composite
toc: true
---
## 前言
树状组合模式，结构型设计模式

## 例子
纸盒箱
->可以包含产品
->可以包含更小的盒子

## 解决方案
组合模式建议使用一个通用接口来和产品盒子进行交互

## 实现方式
1. 模型能够以树状展示
2. 申明组件接口
3. 创建叶子节点（简单元素）
4. 创建容器节点（复杂元素）
5. 容器可以添加和删除元素

## 代码演示
### 画图功能

- 通用接口定义：draw() move()
- 叶子节点实现接口
- 容器节点实现接口，并且支持add()、remove()

### 文件系统
- 文件（叶子节点，能够执行）
- 文件夹（容器节点，可以包含文件和文件夹）

## 适用场景
- 树状场景
- 希望客户端代码以相同方式处理简单和复杂元素

## 优缺点
优点：
- 可以使用多态、递归
- 开闭原则 无需更改代码

缺点：
- 对于功能差异较大的类，提供公共接口或许会有困难
- 在特定情况下，需要过度一般化组件接口，使其变得令人难以理解
- 不容易限制容器中的构件


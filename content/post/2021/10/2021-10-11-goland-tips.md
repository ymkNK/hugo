---
title: Goland全局替换正则技巧
author: ymkNK
categories: [Note]
tags: [Note]
date: 2021-10-11 11:33:32 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/20.jpeg
slug: 2021/10/goland-tips
toc: true
---
# 使用Goland的全局替换
1. 单文件
使用`Command+R`进行替换
   
2. 多文件
使用`Shift+Command+R`进行替换
   
# 正则替换
可以使用正则表达式进行替换
例如，我们想为DoFunction这个方法添加一个ctx作为第一个参数
- 打开Regex的开关
- 搜索：`DoFunction((.*?)\)`，其中 `(.*?)` 表示正则 group，会取到这个函数的参数
- 替换：`DoFunction(ctx, $1)`，其中 $1 表示上一步匹配到的第一个参数
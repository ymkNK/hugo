---
title: 常用命令行统计指令
author: ymkNK
categories: [Tech]
tags: [Tech]
date: 2021-12-21 14:12:16 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/20.jpeg
slug: 2021/12/awk-tool-analysis
toc: true
---
## 背景
使用awk快速统计查看自己的常用命令

## 指令
```
history --i | awk '{print $4}' | sort | uniq -c | sort -k1,1nr | head -20
``` 

### 解析
1. `history -i` 查看所有的历史记录 
2. `awk '{print $4}'` 使用awk分词整理，取到每一行的第四个参数，也就是我们常用的命令
3. `sort` 排序
4. `uniq -c` 去重计数
5. `sort -k1,1nr` 排序
6. `head -20` 取前20

### 效果
```
 ~  history --i | awk '{print $4}' | sort | uniq -c | sort -k1,1nr | head -20
4570 git
 578 rake
 174 ****test
 132 ls
 122 cd
 118 go
  80 testcover
  61 sshymk
  52 vim
  35 source
  28 kinitymk
  27 goimports
  22 gofumpt
  20 sudo
  17 brew
  17 kinit
  17 open
  16 echo
  15 ssh
```
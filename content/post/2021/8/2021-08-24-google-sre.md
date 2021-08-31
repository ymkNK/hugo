---
title: Google SRE解密笔记(一)
author: ymkNK
categories: [Note,Tech]
tags: [Note,Tech]
date: 2021-08-24 21:55:29 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/8.jpeg
slug: 2021/8/google-sre
toc: true
---
# 拥抱风险
百分百可靠的服务，是不现实也是没有必要的。
可靠性到达一定值之后，会带来成本的大幅度提升：过分要求稳定性会限制新产品的开发速度和交付速度。

用于对于高可靠性和极端可靠性的感知几乎是没有的。

基于这一点，SRE追求快速创新和高效的服务运营业务之间的风险的平衡，而不是简单的将服务在线时间最大化。

平衡系统的**功能、服务和性能**
## 管理风险                                                                                                                                        
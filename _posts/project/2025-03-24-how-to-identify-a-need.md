---
layout: post
title:  "如何识别一个需求"
tags: 工程能力
category: 工程能力 
---

# 如何识别一个需求

[toc]

方案的合理性要能 MATCH 到需求才好评估。可以按照：场景 -> 现状 -> 需求 -> 方案结构整理一下。我举个例子：（纯粹的例子，不代表当前这个需求的实际情况）

# 场景

在大板上默认带了集控和 launcher，其中 launcher 需要支持通过集控做相关的配置，并且要求 launcher 的设置限定在当前会话，不影响到其他用户。

# 现状

1. 集控是一个后台服务
2. launcher 是一个用户进程，每一个会话都会有自己进程
3. 用户不会同时运行多个 launcher（或者会）
4. launcher 通过 UDI 接口暴露自己的设置接口
5. UdiServer 不支持多个 launcher 示例同时注册接口

# 需求

1. UdiServer 需要支持同一个 appid（launcher），注册多个实例
2. 客户端需要能够指定把消息放到某个 appid 的特定实例

# 方案一
优缺点

# 方案二
优缺点

# 方案三
优缺点 


##  总结

其中场景的整理可以帮助我们圈定我们的目标，现状分析可以帮助我们找到当前和目标之间的差距，这些差距是我们需要识别并转化出来的实际技术需求点。要解决这些技术点，我们可以有多种可能的方案，这些方案首先应该都能实现我们的目标，只是彼此之间存在各自的侧重点。如果能找到一个完美的方案最好，如果找不到就通过我们的自己取舍，选择当前最合适的方案。

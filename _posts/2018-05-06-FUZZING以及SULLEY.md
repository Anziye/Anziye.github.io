---
layout:   post
title:    FUZZING以及SULLEY
subtitle: 什么是FUZZING以及安装SULLEY过程和注意事项
date:     2018-05-06
author:   ZY
catalog:  true
tags:
    - 流量
    - 安全
    - 网络
---

## 什么是FUZZING
Fuzzing 就是向目标程序发送畸形或者半畸形的数据以引发错误，从而找出漏洞的一种方法。
Fuzzers基本分成两大类：generation（创建）和mutation（变异）。Generation fuzzers创建数据，
然后发送到到目标程序， mutation fuzzers并不创建数据，而是截获程序接收的数据，然后修改数据。

## 什么是SULLEY
Sulley是一款Fuzzer工具，有优秀的崩溃报告，自动虚拟化技术。在fuzzing的过程中，可以任意时刻，
甚至是目标程序崩溃的时候，重新启动程序到前一个状态。
Sulley最主要的是4大模块，协议构建的过程是先进行数据报文建模，然后再连接每一个数据报文组成状
态机，至于其他的模块是辅助fuzzing，来让用户更好地了解fuzzing的状态。

## 安装SULLEY
sulley的安装在官网上有详细的[安装教程](https://github.com/OpenRCE/sulley/wiki/Windows-Installation)，但是我在安装的这个过程当中遇见了很多问题。

1、版本太新的impacket会因为依赖项太多而安装不上，在所用的教程里面，尽量选用示例的库版本；

2、还有一个问题估计只有我才会遇上，我的电脑用户名是中文的，因为python的编码问题，在安装过程中解析各个文件路径时出现编码错误。
这个需要将代码中的默认编码改一下，不是常见的“utf-8”，而是中文的“gbk”。

3、在导入pcapy时，一直找不到指定模块，但是winpcap的dll是存在的。一直没有办法解决，重新安装了2.7版本的python，立刻解决了这个问题，所以版本真的很重要。

以上

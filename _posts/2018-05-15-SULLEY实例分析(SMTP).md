---
layout:   post
title: Sulley实例分析之SMTP
subtitle: FUZZ的重点是对于目标的了解，对目标分析得越清晰，Request越明确，效果越好。
date: 2018-05-15
author:   ZY
catalog:  true
tags:
 - 协议
 - 安全
 - 网络
---

## SMTP的连接和发送过程

（a）建立TCP连接

（b）客户端发送HELO命令以标识发件人自己的身份，然后客户端发送MAIL命令；

         服务器端正希望以OK作为响应，表明准备接收

（c）客户端发送RCPT命令，以标识该电子邮件的计划接收人，可以有多个RCPT行；

         服务器端则表示是否愿意为收件人接收邮件

（d）协商结束，发送邮件，用命令DATA发送

（e）以.表示结束输入内容一起发送出去

（f）结束此次发送，用QUIT命令退出

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

# SMTP的连接和发送过程

（a）建立TCP连接

（b）客户端发送HELO命令以标识发件人自己的身份，然后客户端发送MAIL命令；

         服务器端正希望以OK作为响应，表明准备接收

（c）客户端发送RCPT命令，以标识该电子邮件的计划接收人，可以有多个RCPT行；

         服务器端则表示是否愿意为收件人接收邮件

（d）协商结束，发送邮件，用命令DATA发送

（e）以.表示结束输入内容一起发送出去

（f）结束此次发送，用QUIT命令退出

# 一个SMTP示例

    C: telent SMTP.163.com 25  //以telenet方式连接163邮件服务器  
    S: 220 163.com Anti-spam GT for Coremail System //220为响应数字，其后的为欢迎信息  
    C: HELO SMTP.163.com //除了HELO所具有的功能外，EHLO主要用来查询服务器支持的扩充功能   
    S: 250-mail  
    S: 250-AUTH LOGIN PLAIN  
    S: 250-AUTH=LOGIN PLAIN  
    S: 250 8BITMIME //最后一个响应数字应答码之后跟的是一个空格，而不是'-'   
    C: AUTH LOGIN   //请求认证  
    S: 334 dxNlcm5hbWU6  //服务器的响应——经过base64编码了的“Username”=  
    C: Y29zdGFAYW1heGl0Lm5ldA==  //发送经过BASE64编码了的用户名  
    S: 334 UGFzc3dvcmQ6  //经过BASE64编码了的"Password:"=  
    C: MTk4MjIxNA==  //客户端发送的经过BASE64编码了的密码  
    S: 235 auth successfully  //认证成功   
    C: MAIL FROM: bripengandre@163.com  //发送者邮箱  
    S: 250 … .  //“…”代表省略了一些可读信息  
    C: RCPT TO: bripengandre@smail.hust.edu.cn　//接收者邮箱  
    S: 250 … .    // “…”代表省略了一些可读信息  
    C: DATA //请求发送数据  
    S: 354 Enter mail, end with "." on a line by itself  
    C: Enjoy Protocol Studing  
    C: .  
    S: 250 Message sent  
    C: QUIT //退出连接   
    S: 221 Bye  
    
在这个过程当中还会有其他的命令，比如VRFY：验证给定用户邮箱是否存在，以及接受关于该用户的详细信息；EXPN：扩充邮件列表。

# SMTP常用命令（看懂SC之间会话过程，命令一般是客户端给服务器端发送的）

SMTP命令不区分大小写，但参数区分大小写。常用命令如下：

HELO <domain> <CRLF>——向服务器标识用户身份发送者能欺骗、说谎，但一般情况下服务器都能检测到

RCPT TO: <forward-path> <CRLF>——<forward-path>用来标志邮件接收者的地址，常用在MAIL FROM后，可以有多个RCPT TO

DATA <CRLF>——将之后的数据作为数据发送，以<CRLF>.<CRLF>标志数据的结尾

REST <CRLF>——重置会话，当前传输被取消

NOOP <CRLF>——要求服务器返回OK应答，一般用作测试

QUIT <CRLF>——结束会话

VRFY <string> <CRLF>——验证指定的邮箱是否存在，由于安全方面的原因，服务器大多禁止此命令

EXPN <string> <CRLF>——验证给定的邮箱列表是否存在，由于安全方面的原因，服务器大多禁止此命令

HELP <CRLF>——查询服务器支持什么命令
 
# SMTP常用的响应

501——参数格式错误

502——命令不可实现

503——错误的命令序列

504——命令参数不可实现

211——系统状态或系统帮助响应

214——帮助信息

220<domain>——服务器就绪

221<domain>——服务关闭

421<domain>——服务器未就绪，关闭传输信道

250——要求的邮件操作完成

251——用户非本地，将转发向<forward-path>

450——要求的邮件操作未完成，邮箱不可用

550——要求的邮件操作未完成，邮箱不可用

451——放弃要求的操作，处理过程中出错

551——用户非本地，请尝试<forward-path>

452——系统存储不足，要求的操作未执行

552——过量的存储分配，要求的操作未执行

553——邮箱名不可用，要求的操作未执行

354——开始邮件输入，以“.”结束

554——操作失败

# 浏览器发送邮件的过程：

例如：xxx@126.com可通过登陆126服务器来收发邮件

xxx@126.com在126邮箱提供的邮件页面上填写的相应信息（如发信人邮箱、收信人邮箱等），通过http协议被提交给126服务器；

126服务器根据这些信息组装一封符合邮件规范的邮件；

然后smtp.126.com通过SMTP协议将这封邮件发送到接收端邮件服务器。

# SULLEY示例

在Sulley的Readme是已经给出了一次很完整的Fuzz testing过程，包括对于程序crash之后coredump的分析，有兴趣的可以参考。

一般来讲，使用Sulley的基本流程如下：
- 分析目标协议，了解会话过程
- 针对目标协议的会话过程构造协议包，比如这里的SMTP的各种请求和响应包
- 根据需要配置Sulley提供的Agent，把测试目标与Sulley的Agent连接起来，这样Agent可以在Fuzz开始以后帮助我们自动化测试过程
- 写python脚本，构造好Session
- 开始Fuzz,检查结果

对于这个例子而言，SMTP会话过程简单的可以描述成

    S: 220 foo.com Simple Mail Transfer Service Ready
    C: EHLO bar.com
    S: 250-foo.com greets bar.com
    S: 250-8BITMIME
    S: 250-SIZE
    S: 250-DSN
    S: 250 HELP
    C: MAIL FROM:<Smith@bar.com>
    S: 250 OK
    C: RCPT TO:<Jones@foo.com>
    S: 250 OK
    C: DATA
    S: 354 Start mail input; end with <CRLF>.<CRLF>
    C: Blah blah blah...
    C: ...etc. etc. etc.
    C: .
    S: 250 OK
    C: QUIT
    S: 221 foo.com Service closing transmission channel

也就是按照EHLO,MAIL FROM, RCTP TO, DATA, . , QUIT的command序列与Server交互.


因此要对这个server的smtp实现做fuzzing test，我们就要为每个command随机产生出各种合法(valid)的数据来与Mail Server交互，这里每次command发出的数据包，我们称之为Request，比如MAIL FROM这个Request中，MAIL FROM:是不变的，但是后面跟的邮件地址是可变的(mutable)。在Sulley中，Request是按照“块（block）”结构的方式来构造的，而每个block又是有primitive，也就是一些很基本的数据类型，比如整数，字符，随机数/串之等等。比如这个Mail From的request，我们可以定义一个block，它由几个 MAIL FROM: < test @ test.com > 这几个部分组成，每个部分可以直接用一个primitive来表示

搞定了这里所有的Request之后，接下来把这些request按照一个合理的顺序组织成一个Session，这样Sulley就可以根据session的定义来与Mail Server来交互。简单的讲，可以把每个Request想象成一个节点，而Session定义了节点之间的有向连接，一个A--->B代表当发完A这个请求且收到响应之后继续发B请求。值得注意的是，可以注册回调函数，这样在发出请求之前或者是收到响应之后进行相应的处理

因此我们的Session定义如下所示, （注: ehlo 和 helo都可以作为起始命令，蓝色节点为回调函数）

一旦把这些定义清楚之后，然后结合测试环境，比如Mail Server的IP地址，端口，我们就可以完成我们的Python script。此处不贴出具体代码，只给出链接。

把Script运行起来之后，直接通过console可以看到测试的进程


也可以直接通过Sulley自带的web端口来查看，端口26000，我在本机的例子

http://127.0.0.1:26000/


如果在测试过程，Mail Server异常退出了，那script也会因为请求timeout而停下来。如果希望测试能继续下去的话，这个时候ProcMon和VMControl就可以发挥作用了，它可以帮助你重启你的目标程序或者rollback虚拟机，参考ReadMe. 如果为了事后分析而需要把测试中的流量记录下来，可以使用NetMon.

等整个测试结束之后，如果一旦发现程序crash了，结合coredump文件，记录的日志/流量，我们可以就可以尝试着重现，定位问题了。

关于Sulley的API的具体使用可以直接参考ReadMe和Source code。个人感觉做好Fuzzing test的重点是对于测试目标的了解，分析的越清楚，对于Requet的定义可以更加精准，这样最后产生出来的随机输入才会更有效。

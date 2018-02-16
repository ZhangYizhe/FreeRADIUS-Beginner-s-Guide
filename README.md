## 欢迎来到《FreeRADIUS初学者的指南》

这是一个 **个人项目**，旨在完成对《FreeRADIUS-Beginner-s-Guide》这本书籍的翻译工作，如果你在浏览过程中有什么建议或意见，欢迎提交！


-------
# 目录
### 储备知识
* [AAA(Authentication, Authorization和Accounting)](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/AAA(Authentication%2C%20Authorization%E5%92%8CAccounting).md)
* [Alice、Bob和Isaac](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Alice、Bob和Isaac.md)
* [NAS（网络接入服务器）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/NAS（网络接入服务器）.md)
* [byte（字节）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/byte（字节）.md)

-------
### Radius

> RADIUS是Remote Access Dial In User Service（远程访问拨入用户服务）的缩写. RADIUS是一个AAA解决方案的一部分

* [RADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS.md)

#### Radius数据包

> 了解一个RADIUS数据包的格式将会大大帮助理解RADIUS协议. 让我们更细致的查看RADIUS数据包. 我们将会看一个简单的authentication请求. 一个客户端发送一个Access-Request包到服务器. 服务器返回一个Access-Accept包来表示成功.

* [RADIUS数据包](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md)
* [Code]()
* [Identifier（鉴定）]()
* [Length]()
* [Authenticator（认证）]()
* [Attributes（属性）]()
* [AVPs]()

#### AVP格式
* [Type](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#type)
* [Length](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#length-1)
* [Value](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#vendor-specific-attributesvsas扩展协议)
* [Vendor-Specific Attributes(VSAs)（扩展协议）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#proxying代理-和-realms领域)
* [Proxying（代理） 和 Realms（领域）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#proxying代理-和-realms领域)
* [RADIUS服务器端](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#radius服务器端)
* [RADIUS客户端](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS数据包.md#radius客户端)

-------
### RADIUS其它
* [RADIUS扩展](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/RADIUS扩展.md)
* [关于FreeRADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/关于FreeRADIUS.md)
* [AAA与RADIUS基础总结](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/AAA与RADIUS基础总结.md)


-------
### 安装FreeRADIUS

> 在Linux服务器上安装FreeRADIUS有两种方法。您可以简单地安装预构建的二进制包，或者从源代码构建和安装FreeRADIUS。本章将向您展示如何同时完成这两项工作。

* [安装FreeRADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/%E5%AE%89%E8%A3%85FreeRADIUS.md)
* [预构建二进制](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#预构建二进制)
* [安装FreeRADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#采取行动的时间--安装freeradius)
* [额外的包](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#额外的包)
* [可用的包](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#可用的包)
* [特殊注意事项](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#特殊注意事项)
* [注意防火墙](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#注意防火墙)
* [使用源码进行安装](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/安装FreeRADIUS.md#使用源码进行安装)

-------
### Authentication
> 这一章对认证进行了放大。授权和核算将在后面的书中进行。身份验证是一个过程，我们确定某人是否属于他或她声称的那个人。最常用的方法是使用唯一的用户名和密码。

* [Authentication（认证）（待建完）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authentication-认证-待建完.md)

-------
### [（用户名和密码的来源）Sources of Usernames and Passwords](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md)
> 在已有的章节中，用户的详细信息被保存在用户文件中。然后，FreeRADIUS使用该文件的内容来验证身份验证过程中的凭证。FreeRADIUS很可能是企业设置的一部分，现有用户已经在其他地方创建了。本章将讨论如何利用现有的用户存储。

* [用户存储](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#用户存储)
* [MySQL作为用户存储](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#mysql作为用户存储)
* [在FreeRADIUS中合并MySQL数据库](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#在freeradius中合并mysql数据库)
* [FreeRADIUS的MySQL安装包](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#freeradius的mysql安装包)
* [准备数据库](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#准备数据库)
* [配置FreeRADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#配置freeradius)
* [连接信息](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#连接信息)
* [包含SQL配置](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#包含sql配置)
* [虚拟服务器](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#虚拟服务器)
* [测试MySQL用户存储](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#测试mysql用户存储)
* [与平面文件相比，SQL的优势](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#与平面文件相比sql的优势)
* [SQL数据库的其他用途](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#sql数据库的其他用途)
* [重复用户](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#重复用户)
* [数据库模式](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#数据库模式)
* [组](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#组)
* [探索群组使用](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#探索群组使用)
* [使用SQL组](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#使用sql组)
* [控制组的使用](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#控制组的使用)
* [Profiles](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/用户名和密码的来源（Sources%20of%20Usernames%20and%20Passwords）.md#profiles)

-------
### [Accounting（核算）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#accounting核算)
> 这一章是关于会计的。

* [Accounting（核算）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md)
* [基础核算](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#基础核算)

#### 从NAS模拟核算
* [模拟文件](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#模拟文件)
* [开始一个会话](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#开始一个会话)
* [结束一个会话](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#结束一个会话)
* [孤立会话](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#孤立会话)
* [独立核算](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#独立核算)

#### NAS:重要的AVPs
* [Acct-Status-Type](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#acct-status-type)
* [Acct-Session-Id](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#acct-session-id)
* [AVPs indicating usage](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#avps-indicating-usage)
* [NAS: included AVPs](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#nas-included-avps)

#### FreeRADIUS: pre-accounting section
* [FreeRADIUS: pre-accounting section](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#freeradius-pre-accounting-section)
* [Realms](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#realms)
* [设置Acct-Type](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#设置acct-type)

#### FreeRADIUS: accounting section
* [FreeRADIUS: accounting section](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#freeradius-accounting-section)
* [最小化独立（orphan）会话](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Accounting（核算）.md#最小化独立orphan会话)


-------
### [Authorization（授权）](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#authorization授权)
> 授权是评估请求中信息的过程。此信息可用于对照从文件、数据库或LDAP目录获得的关于用户的信息进行验证。授权发生在身份验证之前，不涉及密码检查。我们可以使用各种逻辑和比较来确定用户是否被授权连接到网络。我们还可以确定他们使用网络的时间或服务质量。这些都是授权的组成部分，本章将对此进行讨论。

* [实施限制](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#实施限制)
* [在FreeRADIUS授权](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#在freeradius授权)
* [unlang介绍](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#unlang介绍)

#### [使用条件语句](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用条件语句)
* [使用unlang中的if语句](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用unlang中的if语句)
* [使用if语句获取返回代码](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用if语句获取返回代码)
* [使用if语句授权用户](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用if语句授权用户)
* [模块返回码](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#模块返回码)
* [unlang关键词](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#unlang关键词)

#### [使用条件语句的其他测试](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用条件语句的其他测试)
* [检查属性是否存在](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#检查属性是否存在)
* [使用逻辑表达式对用户进行身份验证](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/content/Authorization.md#使用逻辑表达式对用户进行身份验证)



# 安装FreeRADIUS

 在Linux服务器上安装FreeRADIUS有两种方法。您可以简单地安装预构建的二进制包，或者从源代码构建和安装FreeRADIUS。本章将向您展示如何同时完成这两项工作。

在本章中，我们将：

1. 从预构建的二进制包中安装FreeRADIUS。
2. 从源代码构建和安装FreeRADIUS。
3. 调查FreeRADIUS安装的程序。
4. 确保FreeRADIUS正常运行。

### 在开始之前
有各种各样的Linux发行版可供选择。我们将讨论三种流行的发行版本，以尽可能广泛地涵盖受众，避免争执。

> 一场争执通常始于两个相等的热情的gnu/linux支持者之间。问题是，当他们支持不同的gnu/linux发行版时，他们相信他们的发行版优于所有其他可用的发行版或操作系统。

基于他们所使用的包管理系统，大多数Linux发行版都属于两个组中的一个。

一组使用Red Hat包管理器(RPM)，而第二组使用dpkg包管理器。我们选择了两个基于rpm的发行版，CentOS和SUSE，它们在企业中很流行。我们选择了Ubuntu，而不是将Debian作为一个基于dpkg发行的发行版，因为它在初学者中非常流行。由于Ubuntu是由Debian演变而来的，讨论Ubuntu的部分也应该在没有重大变化的情况下应用于Debian。

本章的步骤要求安装下列步骤之一

发行	|版本
|---|---|
CentOS|5.5
SUSE|SLES 11
Ubuntu|10.4

一个具有root访问权限的典型服务器安装将被用作基础。如果您有一个版本的发行版与指定的版本不同，请使用本章作为指导方针。

### 预构建二进制
今天的Linux发行版有很多预装的软件，可以轻松安装。可以使用单个命令从软件存储库中安装FreeRADIUS。这将解决依赖关系，并安装所有必需的包，以呈现一个工作系统。

软件存储库是在Linux上运行的包管理系统使用的。如果你是新包管理系统,为进一步阅读参考这个URL:http://en.wikipedia.org/wiki/Package_management_system

三个发行版的默认安装将包括包含FreeRADIUS包的存储库。如果不是这种情况，请参考软件包管理系统的文档，以确定如何包含包含FreeRADIUS包的存储库。

### 采取行动的时间 —— 安装FreeRADIUS

预先构建的FreeRADIUS包可以通过以下命令分别在每个发行版上安装:

CentOS

`#> yum install freeradius2 freeradius2-utils`
    
SUSE

`#> zypper in freeradius-server freeradius-server-utils freeradiusserver-doc`
    
Ubuntu

` $> sudo apt-get install freeradius`

### 刚才发生了什么?
我们安装了为Linux发行版提供的预构建的FreeRADIUS包，以提供基本的工作半径服务器安装。

####  优势

使用预构建的FreeRADIUS包有以下优点:

* 解决依赖关系是自动处理的。这包括处理未来的安全更新，跟踪所有与我们的包一起安装的可选软件包，以及确保安装了一个依赖包的正确版本。
* Linux分销商的QA测试确保了正确的工作软件。
* 更新由Linux分发器负责。
* 特定于分配的调整已经实现了。

使用预构建的二进制包的一个折衷方案是，在机器上没有最新版本的FreeRADIUS。

### 额外的包

FreeRADIUS是一个功能丰富的软件。每个发行版通过将其分布在多个包中，以不同的方式呈现它的FreeRADIUS。

CentOS和Ubuntu包含一些FreeRADIUS服务器模块作为可选的软件包。这使基本的服务器安装包保持精简。安装可选的服务器模块包也将安装必需的依赖项。这意味着，例如：当您安装freeradius-MySQL包时，所有必需的MySQL库也将作为依赖项安装。

SUSE将他们的包按功能划分。您将发现客户端和服务器各自都有自己的包集。SUSE还为FreeRADIUS提供了实用工具和文档包。

### 可用的包
这一节列出了每个分发版的免费的FreeRADIUS包。推荐使用粗体的名称为基本的FreeRADIUS安装:

#### CentOS

Package name | Short description
---|---
freeradius2|高度可配置的RADIUS服务器
freeradius2-krb5|Kerberos 5对FreeRADIUS的支持
freeradius2-ldap|LDAP对FreeRADIUS的支持
freeradius2-mysql|MySQL支持FreeRADIUS
freeradius2-perl|Perl支持FreeRADIUS
freeradius2-postgresql|PostgreSQL对FreeRADIUS的支持
freeradius2-python|Python支持FreeRADIUS
freeradius2-unixODBC|Unix ODBC支持FreeRADIUS
freeradius2-utils|FreeRADIUS公用事业

#### SUSE

Package name|Short description
---|---
freeradius-client |FreeRADIUS客户端软件
freeradius-client-libs|共享的FreeRADIUS客户端库
freeradius-server|高度可配置的RADIUS服务器
freeradius-server-dialupadmin|FreeRADIUS网络管理
freeradius-server-doc|FreeRADIUS文档
freeradius-server-libs|FreeRADIUS共享库
freeradius-server-utils|FreeRADIUS客户

请注意，由SUSE提供的freeradius-Client包被软件开发人员用来为AAA使用RADIUS，像radtest这样的客户机程序被包含在freeradius-server-utils包中。

#### Ubuntu

Package name|Short description
---|---
freeradius |FreeRADIUS服务器包
freeradius-dbg |为FreeRADIUS包包含独立的调试符号
libfreeradius2 |FreeRADIUS共享库
freeradius-ldap |FreeRADIUS服务器的LDAP模块
freeradius-common |FreeRADIUS的常见文件，包括字典和手册页
freeradius-iodbc|FreeRADIUS服务器的iODBC模块
freeradius-krb5 |FreeRADIUS服务器的Kerberos模块
freeradius-utils |FreeRADIUS客户端实用程序，包括诸如radclient、radtest、smbencrypt、radsniff和radzap等程序
freeradius-postgresql |FreeRADIUS服务器的PostgreSQL模块
freeradius-mysql |FreeRADIUS服务器的MySQL模块
freeradius-dialupadmin |网络管理插件
libfreeradius-dev |FreeRADIUS共享库开发文件

### 特殊注意事项

旧版本的Ubuntu没有将SSL库支持编译到预编译的二进制包中。当在这些版本上安装FreeRADIUS时，如果您需要在EAP扩展上提供SSL支持，您需要自己构建自己的版本。

SUSE还提供了 yast-i 命令来安装软件；而是使用zypper，因为在安装包及其依赖项时，它具有更好的决策能力。

在CentOS的FreeRADIUS包的名称是freeradius2而不是预期的FreeRADIUS。这是因为，与CentOS 5一起出现的FreeRADIUS版本最初是1.1.3。在1之间的配置文件的更改。x和2。然而，FreeRADIUS的x版本是如此之大，以至于需要改变名称。

并非所有的FreeRADIUS模块都有各自的匹配包在Ubuntu或CentOS。没有匹配包的模块只包含在主FreeRADIUS包中。

#### 注意防火墙
CentOS和SUSE在默认情况下安装了一个活跃防火墙。请确保在1812端口和1813年端口对外开放。

#### CentOS
有一个实用程序可以在CentOS上配置防火墙，称为system-configsecuritylevel-tui，它必须以root身份运行。这将启动一个基于指针的程序。选择Customize选项并按Enter。Allow incoming | Other Ports列表应该包括 1812:udp 1813:udp。选择OK返回主屏幕，然后再次选择OK以提交更改。

通过检查以下命令的输出，确认端口现在是打开的:

`#> /sbin/iptables -L -n | grep 181*`

ACCEPT udp -- 0.0.0.0/0 0.0.0.0/0 state NEW udp dpt:1812

ACCEPT udp -- 0.0.0.0/0 0.0.0.0/0 state NEW udp dpt:1813

尽管不推荐您可以使用以下命令禁用防火墙:

`#> /etc/init.d/iptables save`

`#> /etc/init.d/iptables stop`

`#> /sbin/chkconfig iptables off`

要确认防火墙是否被禁用，请使用以下命令:

`#> /sbin/iptables -L -n`

Chain INPUT (policy ACCEPT)

target     prot opt source     destination

Chain FORWARD (policy ACCEPT)

target     prot opt source     destination

Chain OUTPUT (policy ACCEPT)

target     prot opt source     destination

#### SUSE
在SUSE上配置防火墙可能有一个catch-22的情况，因为默认的防火墙是如此的安全，甚至连SSH都不能进入！登录SLES服务器并启动YaST。选择安全性和用户防火墙。在左边选择允许的服务。我建议您将安全的Shell服务器添加到外部区域。单击Advanced按钮，并将1812 1813添加到UDP端口。单击OK。单击Next和Finish，以提交这些更改。

`#> iptables -L -n | grep 181`

ACCEPT     udp -- 0.0.0.0/0     0.0.0.0/0     udp dpt:1812

ACCEPT     udp -- 0.0.0.0/0     0.0.0.0/0     udp dpt:1813

尽管不推荐，您可以以以下方式禁用防火墙:

1. 使用YaST并选择安全和用户防火墙。
2. 在左边选择启动。选择禁用防火墙自动从右侧开始。现在也选择停止防火墙，并按下Enter键来停止当前运行的防火墙。
3. 单击Next和Finish，以提交这些更改。

通过检查以下命令的输出，确认防火墙现在已经禁用了。

`#> iptables -L -n`

Chain INPUT (policy ACCEPT)

target     prot opt source     destination

Chain FORWARD (policy ACCEPT)

target     prot opt source     destination


Chain OUTPUT (policy ACCEPT)

target     prot opt source     destination

#### 使用源码进行安装

从源代码构建软件，过去是配置、制作、安装的同义词。我们将使用发行版的包管理器从源代码构建新的软件包。
如果您已经从预构建的二进制包中安装了FreeRADIUS，那么下面的部分是可选的。

（33页源码安装部分暂不做翻译）



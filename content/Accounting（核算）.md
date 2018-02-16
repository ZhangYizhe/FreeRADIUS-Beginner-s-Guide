# Accounting（核算）

前几章介绍了身份验证以及授权的一些方面。这一章是关于会计的。

在本章中，我们将

* 看看在FreeRADIUS的基础核算是如何运作的
* 看看如何限制用户的会话
* 发现限制用户使用的方法
* 看看核算数据的整理

让我们开始吧。

### 本章要求

您需要一个工作的FreeRADIUS服务器来进行实际的练习;一个干净的安装是首选。

### 基础核算

核算是指跟踪用户使用NAS资源的情况。核算不仅包括以计费方式进行的成本回收。它还可以用于容量规划、生成趋势图，以及在给定时间点对资源使用情况了解更多。在这一章中，我们将了解如何在FreeRADIUS中完成核算工作。

FreeRADIUS是一个AAA级服务器。RADIUS的AAA可以分为两部分。一个组件由授权和身份验证组成，它使用UDP端口1812。第二个组件是核算，并使用UDP端口1813。这两个分量独立地相互作用。不同的监听（listen）部分在radiusd.conf配置文件证实了这一点。以下是核算的监听部分:


```
#   This second "listen" section is for listening on the accounting#   port, too.#listen {    ipaddr = *#   ipv6addr = ::    port = 0    type = acct#   interface = eth0#   clients = per_socket_clients}
```

这个监听部分会导致FreeRADIUS侦听核算请求。要了解更多关于监听部分的信息，请参阅radiusd.conf中的注释。

在监听代码中注意 prot = 0。当端口被指定为0时，FreeRADIUS将从/etc/services文件中读取该端口的值。但是，通过传递–p \<port number>的参数，您可以在启动时覆盖该值，这将迫使FreeRADIUS服务器只监听指定的端口。


/etc/services文件用于将端口号和协议映射到服务名。

``` 
radius          1812/tcpradius          1812/udpradius-acct     1813/tcp    radacct # RadiusAccountingradius-acct     1813/udp    radacct
```
/etc/services文件引用端口1645和1646作为old-radius和old-radacct。

上面的提取表明，FreeRADIUS在默认情况下能够处理核算请求。我们来看看核算是怎么做的。

### 从NAS模拟核算

在第3章，从FreeRADIUS开始，我们介绍了radclient命令。这一节创建了三个文件，可以与radclient一起使用，以模拟NAS通常发送给RADIUS服务器的核算数据包。

### 模拟文件

这三个文件中的AVPS类似于从hostapd程序中发送的AVPs。

> hostapd是一个用于控制wifi网络身份验证的守护进程。它可以配置为与认证一起进行核算处理，并且通常安装在基于Atheros-based的无线接入点的OpenWRT上。你也可以用它来把无线网卡变成Linux上的接入点。

当用户的会话开始时，NAS将通知RADIUS服务器。在会话期间，NAS可能会将会话的更新发送到RADIUS服务器，然后当会话结束时，NAS也会通知RADIUS服务器。

这些核算包内部是Acct-Status-Type AVP，它将反映Start开始、Interim-Update内部更新和Stop停止的会话状态。这对应于我们将要创建的三个文件。

下面的文件命名为4088_06_acct_start.txt，将创建一个会话:


```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Start
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
```

下面的文件命名为 4088_06_acct_interim-update.txt，将更新会话:


```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Interim-Update
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
Acct-Session-Time = 11
Acct-Input-Packets = 15
Acct-Output-Packets = 3
Acct-Input-Octets = 1407
Acct-Output-Octets = 467
```

最后，下面的文件命名为4088_06_acct_stop.txt，将结束会话:


```
Packet-Type=4
Packet-Dst-Port=1813
Acct-Session-Id = "4D2BB8AC-00000098"
Acct-Status-Type = Stop
Acct-Authentic = RADIUS
User-Name = "alice"
NAS-Port = 0
Called-Station-Id = "00-02-6F-AA-AA-AA:My Wireless"
Calling-Station-Id = "00-1C-B3-AA-AA-AA"
NAS-Port-Type = Wireless-802.11
Connect-Info = "CONNECT 48Mbps 802.11b"
Acct-Session-Time = 30
Acct-Input-Packets = 25
Acct-Output-Packets = 7
Acct-Input-Octets = 3407
Acct-Output-Octets = 867
Acct-Terminate-Cause = User-Request
```

通过使用这三个文件和radclient程序，我们现在可以探索FreeRADIUS核算的各个方面。

### 开始一个会话

当NAS从RADIUS服务器接收一个 Access-Accept 包时，NAS尝试将Identifier字段与一个等待的Access-Request相匹配。如果找到匹配，会将NAS配置为核算，NAS将向RADIUS服务器发送一个Code 4 RADIUS(核算请求)。这标志着会话的开始。让我们用radclient来模仿NAS的动作:

1、在调试模式下启动FreeRADIUS。

2、使用radclient命令向FreeRADIUS发送一个会计请求的4088_06_acct_start.txt文件:

`$>radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`

3、观察FreeRADIUS和radclient命令的输出。以下是来自radclient的反馈，表明请求是通过返回Code 5 (Accounting-Response)来获得成功的:

`Received response ID 66, code 5, length = 20`

4、通过发出radwho命令确认alice有一个活动会话。您必须是能够发出这个命令的根用户。

取决于FreeRADIUS是如何编译的，以及您正在使用的发行版，radwho命令可能会返回一个错误。如果是这样的话，下面的部分将会解决这个问题:

5、尽管是root用户，但你会得到以下错误:

`radwho: Error reading /var/log/radius/sradutmp: No such file or directory`

6、sradutmp文件不存在，因为sradutmp模块是在sites-enabled/default虚拟服务器的核算部分中禁用的。激活sradutmp，取消注释以下一行:
    
`#sradutmp`

7、再次在调试模式下重新启动FreeRADIUS。发布radclient和radwho命令。您现在应该看到如下内容:

   
``` 
#radwho
Login     Name     What     TTY     When     From     Location
alice       alice        shell      S0       Sun      16:34     127.0.0.1
```

Alice的活跃会话现在通过radwho反映出来了。接下来，我们将使用停止请求结束这个活动会话。

### 结束一个会话

当用户注销或他们的会话超时时，NAS将向RADIUS服务器发送一个停止请求，以便计算详细信息，以反映在NAS上发生的事件。按照以下步骤来结束一个请求:

1、确保FreeRADIUS服务器仍然在调试模式下运行。

2、使用radclient命令向FreeRADIUS发送一个会计请求的4088_06_acct_stop.txt文件:
    
`$>radclient 127.0.0.1 auto testing123 -f 4088_06_acct_stop.txt`
    
3、确认会话是通过检查radwho的输出来完成的:

``` 
#radwho
Login     Name     What     TTY     When     From     Location
```

### 孤立会话

有时可能会发生NAS挂起。当NAS重置后,会计信息在FreeRADIUS仍然反映了旧状态。您可以使用radzap命令关闭任何打开的核算记录。让我们看看radzap的行动:

1、使用radclient命令向FreeRADIUS发送一个会计请求的4088_06_acct_start.txt文件:

`$>radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt`

2、与radwho命令确认开放会话:

    
```
#radwho
Login     Name     What     TTY     When     From     Location
alice       alice        shell      S0       Sun      16:34     127.0.0.1
```

3、现在，您可以使用下面的命令来从127.0.0.1中杀死所有活动会话。

`radzap -N 127.0.0.1 127.0.0.1 testing123`
   
Radwho将会显示现在没有活动的会话。

如果你看一下FreeRADIUS服务器的反馈，当你发布了radzap，你会看到它发出以下的请求:


```
rad_recv: Accounting-Request packet from host 127.0.0.1 port 43629,id=195, length=38
    Acct-Status-Type = Accounting-Off
    NAS-IP-Address = 127.0.0.1
    Acct-Delay-Time = 0
```

这就使我们结束了基本会计的实践练习。接下来会进行详细的讨论。

### 刚刚发生了什么？

我们使用radclient命令向FreeRADIUS服务器发送了核算请求，这与NAS的操作方式类似。我们还模拟了一个场景，其中一个NAS挂起，将FreeRADIUS的核算数据与NAS的状态同步。让我们来看看一些有趣的点，同时提到我们刚刚完成的练习。

### 独立核算

FreeRADIUS的核算独立于授权和身份验证。它使用一个单独的端口，由客户端发送到服务器的帐户请求数据包组成。服务器响应响应包以响应请求。

核算数据用于衡量网络上的使用情况。NAS可以报告用户连接到网络的时间以及用户的数据使用情况。

核算记录没有反映用户在会话期间访问过的网站的细节。它们只表示时间和数据的使用。

### NAS:包括AVPs

虽然通过NAS发送的AVPs各有不同,一些AVPs应该出现在Accounting-Request很重要。

### Acct-Status-Type

这个属性表示这个Accounting-Request是否标志着用户服务(Start)或结束(Stop)的开始。每个选项都由一个特定的数字表示。当会话开始时，它将被指定为1(开始)。当会话的数据更新时，它将被指定为3(Interim-Update)，当会话结束时，它将被指定为2(停止)。Acct-Status-Type还可以包含诸如 Accounting-On(以7为代表的)和 Accounting-Off(以8表示)来关闭对NAS的所有开放会话。Acct-Status-Type 的值决定了FreeRADIUS对用户的核算数据进行操作的方式。

### Acct-Session-Id

Acct-Session-Id用于唯一标识用户会话。这是结合Acct-Status-Type来记录用户的会话状态。虽然Acct-Status-Type值的变化反映了状态(Start, Interim-Update, or Stop),但在整个会话Acct-Session-Id都是不变的。

### AVPs indicating usage

下表显示了Accounting-Request包内部的AVPs，它反映了用户的使用情况:

AVP | Description
---|---Acct-Session-Time |The duration of the sessionAcct-Input-Octets |Bytes send from the user to the NASAcct-Output-Octets |Bytes send from the NAS out to the user

这三个AVPs表示用户的时间和数据使用情况。它们只出现在Acct-Status-Type = Interim-Update或Acct-Status-Type = Stop时。Acct-Status-Type = Start不能包含它们。

### NAS: included AVPs

包含Accounting-Request的AVPs将取决于Acct-Status-Type的值。当一个会话开始时，不需要发送显示时间和数据使用的AVPs。这些AVPs只包含在后续请求中。

Stop请求通常表示终止原因：

`Acct-Terminate-Cause = User-Request`

包括在内的AVPs也依赖于NAS。有时一个NAS不包括必需的AVPs(Hostapd)和有时互换(Chillispot)周围的输入和输出。您永远不知道NAS将会给服务器带来什么。因为这个最好总是第一个测试,看看什么AVPs被包含,客户可能需要额外的配置以使核算工作达到预期的目的。

> 有时会有一个AVP叫做Acct-Delay-Time。这个AVP的值可以被RADIUS服务器使用，用来在会话调整起始和停止时间时记录详细信息。当NAS很难将Accounting-Request发送到RADIUS服务器，它必须重新发送请求时，通常会出现这种情况。如果Acct-Delay-Time的值很大，你应该调查一下为什么会这样。

### FreeRADIUS: pre-accounting section

当FreeRADIUS收到Accounting-Request时，它首先被传递到preacct部分。这个部分被定义在虚拟服务器的文件中。与大多数FreeRADIUS配置一样，默认情况下可以正常工作，但是如果您想要为用户操作AVP值，那么就可以这样做。preacct部分中的注释表明在本节中可以做什么。

一个有趣的模块是preprocess模块(rlm_preprocess)。当需要时，这个模块将返回请求的完整性。在我们的示例中，它添加了NAS-IP-Address AVP，因为它缺失了。这个AVP是按RFC 2866要求Accounting-Request。

NAS-IP-Address或者NAS-Identifier必须要存在于一个RADIUSAccounting-Request中。

acct_unique条目通过Acct-Unique-Session-ID的值来确保每个请求都有一个全新的标识符。

### Realms

当您将核算请求转发到另一个RADIUS服务器时，preacct部分也非常重要。suffix模块(realm模块的实例)用于标识和触发此类通信的路由。还有IPASS和ntdomain，但是注释掉了。这两个都是realm模块的实例。suffix、IPASS和ntdomain都希望在User-Name AVP中寻找一个独特的模式来确定请求的范围。在第12章，漫游和代理中，将对其他RADIUS服务器的传输进行深入的讨论。

### 设置Acct-Type

您也可以使用preacct部分中的files模块。将使用此模块设置Acct-Type内部AVP。Acct-Type AVP用于通过强制模块的不同实例来处理accounting部分内的帐户流量，从而分离帐户流量。这与Acct-Type内部AVP可以在authorize部分中设置以便在authenticate部分中指定要使用的身份验证方法的原理相同。您可以在以下URL上阅读有关使用此功能的详细信息:

> http://freeradius.org/radiusd/doc/Acct-Type
> （已失效）

### FreeRADIUS: accounting section

Accounting-Request由preacct部分处理后，将传递到accounting部分。这也在虚拟服务器的文件中定义。这是我们激活sradutmp模块的部分。accounting部分负责实际记录核算数据。有多种方法可以做到这一点。默认情况下，它将使用详细信息模块记录为文本文件。但是，我们也可以指定它应该使用SQL模块记录到SQL数据库中。此部分还可用于记录用户的使用情况(daily模块)。然后可以使用该用法来确定授权结果。

我们鼓励您阅读会计部分中的注释，以了解包含的模块的功能。

### 最小化独立（orphan）会话

当NAS挂起时，它无法向RADIUS服务器发送任何请求。重新启动NAS后，应发送Acct-Status-Type = Accounting-On。正常关闭时，应发送Acct-Status-Type = Accounting-Off.。这使核算记录更加健全和可靠。radzap命令模拟NAS的正常关闭，用于关闭orphan会话。


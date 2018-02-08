# RADIUS扩展
在初始的RFC定义了通常的RADIUS和RADIUS accounting, 许多扩展被建议来扩展RADIUS的使用或者提高一些弱点.

也有一个改进的RADIUS协议叫做Diameter(一个文字游戏--两倍好于RADIUS). 然后Diameter的接受非常缓慢, RADIUS仍然保持事实的标准对于可预见的未来. 一个主要的原因可能是事实上许多Diameter的增强已经被许多RADIUS的扩展实现. 例如, RadSec协议通过TCP和TLS传输RADIUS协议. 这个让RADIUS在漫游环境更容易扩展.

尽管有许多扩展, 我们将只研究两个可能被使用的。

##### Dynamic Authorization extension [RFC5176]

这个扩展帮助创建从RADIUS服务器到NAS的反馈回环. 这个实际上交换了客户端和服务器端之间的角色. RADIUS服务器成为NAS的一个客户端.

动态authorization（授权）允许RADIUS服务器通知NAS关于在其上一个用户现有session已经做的改变. 有2个这种扩展的流行应用.

##### Disconnect-Message [DM]

也叫做一个POD(Packet Of Disconnect（断开的数据包）), 这个用来断开一个现有用户的session. RADIUS服务器发送断开连接请求, NAS必须回复无论断开是否成功.

##### Change-of-Authorizaion Message（更改授权消息） [CoA]

这个消息提供来改变一个先有用户session的authorization数据. 例如, 我们现在可以动态改变每个session的带宽限制. 这让我们可以当互联网连接下降时可以增加每个session的带宽. 反之也可以。

MikroTik RouterOS包括这个功能在一些使用RADIUS的服务上。

下表列出包含的RADIUS包的code和名称：

| RADIUS&nbsp;code(decimal) | Packet type | Sent by |
| --- | --- | --- |
| 40 | Disconnect-Request | RADIUS&nbsp;server |
| 41 | Disconnect-ACK | NAS |
| 42 | Disconnect-NAK | NAS |
| 43 | CoA-Request | RADIUS server |
| 44 | CoA-ACK | NAS |
| 45 | CoA-NAK | NAS |



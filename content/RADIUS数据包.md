# RADIUS数据包

了解一个RADIUS数据包的格式将会大大帮助理解RADIUS协议. 让我们更细致的查看RADIUS数据包. 我们将会看一个简单的authentication请求. 一个客户端发送一个Access-Request包到服务器. 服务器返回一个Access-Accept包来表示成功.

显示在这里的RADIUS数据包只是一个UDP数据包的payload（有效载荷）。

屏幕截图是捕捉在RADIUS客户端和服务器端之间的流量得到的. 这里的屏幕截图是一个简单的Authentication（认证）请求发送到一个RADIUS服务器的结果。

下面的截图显示来自RADIUS客户端的Access-Request（访问请求）包。

![radius-request.png](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/images/radius-request.png)

下面的截图显示RADIUS服务器返回给这个请求一个Access-Accept（接受访问）数据包。

![radius-respond.png](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/images/radius-respond.png)

### Code

每一个数据包由一个code来识别. 这个域是一个字节. 这个code的值决定这个包的某些特性和需求. 下面的表格可以被用作一个索引来列出当前RADIUS数据包的定义的code.

| RADIUS code(decimal) | Packet type | Sent by |
| --- | --- | --- |
|1|Access-Request（访问请求）|NAS|
|2|Access-Accept（接受访问）|RADIUS&nbsp;server|
|3|Access-Reject（访问拒绝）|RADIUS&nbsp;server|
|4|Accounting-Request（计费请求）|NAS|
|5|Accounting-Response（计费应答）|RADIUS&nbsp;server|
|11|Access-Challenge（访问问询）|RADIUS&nbsp;server|
|12|Status-Server(Experimental)（状态 - 服务器（实验））||
|13|Status-Client(Experimental)||
|255|Reserved（保留）||

### Identifier（鉴定）


每个包的第二个字节包含一个唯一的标识符. 他是由客户端生成的, 并且用做辅助来匹配请求和回复. RADIUS数据包是通过无连接的UDP传输的. 这样需要RADIUS实现他自己的算法来提交从客户端的重试请求. 当一个客户端重发一个请求给服务器, 数据包的identifier将会保持不变. 服务器将会通过匹配数据包中的identifier回复请求。

### Length


这个是数据包的第3, 第4个byte. 他表示一直到哪里在数据包中是有用的数据. 在这个边界之外的字节被认为是填充和被默默地忽略.

### Authenticator（认证）


这个域占用2个byte, 根据包是来自客户端还是服务器端有不同的格式. 他也依赖包的类型, 例如, Access-Request 或者 Accounting-Request. 如果他是一个请求, 这个域作为一个Request Authenticator（请求身份验证器）. 如果他是一个回复, 这个域作为一个Response Authenticator（响应身份验证器）.

Request Authenticator的值是一个不会重复的随机数. Response Authenticator的值是在回复包的许多域的MD5 哈希值, 带有客户端和服务器端之间的共享密钥(shared secret).

如果请求包含User-Password attribute（属性）, 那么这个attribute的值会被加密. 这个加密的值通常是把共享的密钥(shared secret)结合authenticator生成MD5哈希值, 然后与用户的密码进行异或生成的. 这就是为什么共享密钥(shared secret)要在客户端和服务端保持一致, 为了解密用户的密码.


### Attributes（属性）


RADIUS包的其余部分包含0或者更多attribute, 作为AVP(Attribute Value Pairs（属性值对）). 这些AVPs的结尾是由包的length域指定的.

### AVPs


AVPs是RADIUS协议的苦力. AVPs可以被分类为check或者reply attribute（回复属性）. check attribute（检查属性）是从客户端发到服务器端. reply attribute是从服务器端发到客户端.

attribute 作为信息的载体在客户端和服务器端之间服务. 他们被客户端使用来提供他们自身的信息和连接到他上的用户. 当服务器端回复客户端时他们也被使用。基于在服务器端回复接收到的AVPs，客户端可以使用这个回复来控制用户的连接。

### Conclusion（结论）


RADIUS包是通过UDP传输. code域表示RADIUS包的类型. attribute是用来提供指定的信息用于authentication, authorization 和 accounting. 例如为了authenticate一个用户, User-Name和User-Password AVPs将会和一些其他的attribute一起被包含在Access-Request包中.


### AVP格式

#### Type


AVP的第一个byte是type域. 这个域的数值是和一个attribute名称联系在一起的, 所以我们人类也可以理解. 这些attribute和数值的关系是由IANA(http://www.iana.org)控制的. attribute的名称通常是有描述性的, 例如 User-Name(1), User-Password(2), 或者 NAS-IP-Address(4).

**RADIUS也允许扩展协议. attribute类型26(叫做Vendor-Specific)允许这样做. Vendor-Specific的attribute的值可以反过来包含由一个vendor管理的Vendor Specific Attributes(VSAs).**

#### Length


length域是AVP的第二个byte. 这个与RADIUS包中的length一个含义, 用来表示AVP的长度。 由于length域将会标记AVP的结尾，所以这个方法允许AVPs带有不同大小的值。

#### Value


AVP的value可以在大小上不同. 这个value域可以是0或者更多byte. 这个value域可以包含一下数据类型中的一个: text（文本）, string, address, integer（整数）或者time.

Text和string可以多达253byte. Address, integer和time是4个byte大小.

如果我们取出一个请求包中的NAS-IP-Address AVP, 我们会看到长度是6 byte. 一个byte表示类型, 一个byte表示长度, 然后4个byte表示IP地址, 总共6byte.

#### _Vendor-Specific Attributes(VSAs)（扩展协议）_


VSAs允许vendor（供应商）定义他们自己的attribute. attribute定义的格式基本上是普通的AVPs带有额外的一个vendor域名. VSAs是通过AVP类型26发送. 这表示VSAs是AVPs的一个扩展, 并且包含在AVPs之内.

这让RADIUS非常灵活, 并且允许一个vendor来创建扩展来自定义他们的RADIUS实现. 例如CoovaChilli有一个VSA attribute叫做ChilliSpot-Max-Total-Octets. 当CoovaChilli客户端从RADIUS服务器接收到这个attribute的回复时, 他使用这个值来限制通过captive portal的数据.

NAS将会默默地忽略任何他不支持的VSAs. 一些vendors发布了他们的VSAs, 但是这不是需要的. 其他仅仅列出他们在网站或者文档里. 这个然后可以被用来决定他们设备实现RADIUS的能力.

#### Proxying（代理） 和 Realms（领域）


RADIUS协议允许扩展. Proxying允许一个RADIUS服务器作为一个客户端连接到另一个RADIUS服务器. 这个最终形成一个链条。


在proxying上的讨论也包含Realms. Realms是用来组织用户和形成用户名的一部分. 一个用户名区分于Realm name带有一个指定的分隔符. realm name可以是用户名加上前缀或者后缀. 今天的流行的标准使用域名作为后缀, 并且用@分割, 例如:alice@freeradius.org. 这个然而只是一个约定. realm可以是任何名字, 分隔符也可以是任何符号. windows用户通常用一个前缀指定域名, 用\作为分隔符, 例如: my_domain\alice.

当RADIUS服务器接收到一个请求带有用户名包含一个realm时, 他被设计决定是否处理这个请求或者转发这个请求给另一个RADIUS服务器用来处理这个指定的realm. 这需要第二台RADIUS服务器应该有转发RADIUS服务器作为一个客户端的定义, 并且他们也有一个共享的密钥(shared secret).

#### RADIUS服务器端

RADIUS协议是基于C/S架构. RADIUS服务器将会监听UDP端口**1812**和**1813**. 1812端口是用于**authentication（认证）**. 这将会包含Access-Request, Access-Accept, Access-Reject, 和Access-Challenge包. 1813端口用于**accounting（计费）**. 这个将会包含Accounting-Request和Accounting-Response包.

一个客户端和服务器需要一个共享的密钥(shared secret)为了加密和解密在RADIUS包中的某些域.

#### RADIUS客户端


RADIUS客户端通常提供访问TCP/IP网络. 客户端作为一个RADIUS服务器和一个想要网络权限的用户/设备之间的中间人.


RADIUS的proxying（代理）功能也允许一个RADIUS服务器成为另一台RADIUS服务器的客户端, 将会最终形成一个链条.


来自RADIUS服务器的反馈不仅仅决定是否一个用户允许访问网络(authentication（认证）), 也可以引导客户端来强加某些限制在用户上(authorization（授权）). 限制的例子有在session（会话）上的一个时间限制或者限制连接速度.


# RADIUS

RADIUS是Remote Access Dial In User Service（远程访问拨入用户服务）的缩写. RADIUS是一个AAA解决方案的一部分
    
ISP和网络管理员们应该熟悉RADIUS, 由于他被许多控制访问TCP/IP网络的设备使用. 下面是一些例子:

* 一个带有VPN服务器的防火墙可以使用RADIUS.
* 带有WPA-2企业加密的Wi-Fi AP包含RADIUS.
* 当Alice通过DSL连接一个现有的电信设施时, 电信设备将会使用RADIUS协议来联系Isaac（ ISP(Internet Service Provider)/我们的网络）的RADIUS服务器为了决定她是否可以通过DSL(代理)获得网络访问权限.

RADIUS协议是一个C/S协议, 利用UDP来通信. 使用UDP而不是TCP暗示通信不是严格on state. 在客户端和服务器端的一个典型的数据流有一个来自客户端的单一请求, 紧接着一个来自服务器端的单一回复构成. 这让RADIUS成为一个非常轻量级的协议并且对在低网速环境下的效率有帮助.

在客户端和服务器端之间可以建立成功的通信之前, 每一端必须定义一个shared secret. 这个是用来authenticate（认证）客户端。

一个NAS（网络接入服务器）作为一个RADIUS客户端. 因此当你读到关于一个RADIUS客户端, 他表示一个NAS.

![nas-radius](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/images/nas-radius.png)

RADIUS包有一个指定的格式, 定义在RFC里面. 在一个RADIUS包中, 两个关键的组件是:

* code: 表示包的类型.
* attribute: 携带RADIUS使用的必要数据.


###### RADIUS服务器端

 RADIUS协议是基于C/S架构. RADIUS服务器将会监听UDP端口1812和1813. 
 1812端口是用于authentication（认证）. 这将会包含Access-Request, Access-Accept, Access-Reject, 和Access-Challenge包. 1813端口用于accounting（计费）. 这个将会包含Accounting-Request和Accounting-Response包.
 
一个客户端和服务器需要一个共享的密钥(shared secret)为了加密和解密在RADIUS包中的某些域.

###### RADIUS客户端

RADIUS客户端通常提供访问TCP/IP网络. 客户端作为一个RADIUS服务器和一个想要网络权限的用户/设备之间的中间人.

RADIUS的proxying（代理）功能也允许一个RADIUS服务器成为另一台RADIUS服务器的客户端, 将会最终形成一个链条.

来自RADIUS服务器的反馈不仅仅决定是否一个用户允许访问网络(authentication（认证）), 也可以引导客户端来强加某些限制在用户上(authorization（授权）). 限制的例子有在session（会话）上的一个时间限制或者限制连接速度.


# AAA(Authentication, Authorization和Accounting)

AAA有时也被称为"三A框架"(Triple A Framework). AAA是一个高层次的架构模型, 可以被用各种方法实现.

## Authentication（认证）

Authentication通常是考虑的第一步, 为了获取网络访问和他提供的服务. 这是一个过程用来确认是否Alice提供的credential（凭据）是有效的. 提供credential的最常见的方式是通过用户名和密码. 其他方式像one-time token, certificate, PIN numbers, 或者甚至biometric scanning也可以使用.

在成功authentication之后一个session（会话）被初始化. 这个session持续直到网络连接被终止掉.

## Authorization（授权）

Authorization是一种Isaac（ISP(Internet Service Provider)/我们的网络）控制资源使用的方式. 在Alice已经authenticate了她自己之后, Isaac可以强加某些限制或者授予某些权利. Isaac可以例如检查Alice从哪台设备访问网络和基于这点来做出决定. 他可以限制Alice可以拥有的打开的session的个数, 给她一个预先决定的IP地址, 只允许某些流量通过, 或者甚至执行QoS基于一个SLA(Service Level Agreement, 服务级别协议).

Authorization通常包含逻辑. 如果Alice是学生组的一部分那么在工作时间没有网络访问权限. 如果Bob通过一个captive portal访问网络, 那么一个带宽限制被强加来阻止他超量使用网络连接.

逻辑可以基于许多事情. Authorization决定例如可以基于组成员或者你连接的NAS或者甚至你访问资源是星期几.

## Accounting（会计）

Accounting是一种测量资源使用量的方式. 在Isaac已经确定谁是Alice并且在建立的session上强加合适的控制, 他也可以测量她的使用量. Accounting是测量使用量的不间断的过程.

这允许Isaac来跟踪Alice花费了多少时间和资源在一个建立的session上. 获取accounting数据允许Isaac来对Alice的资源进行计费, 趋势分析, 和行为监视.


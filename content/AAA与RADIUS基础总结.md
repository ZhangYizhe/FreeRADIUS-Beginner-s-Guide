# AAA与RADIUS基础总结


| 名称 | 代表 | 简称 |
| --- | --- | --- |
| AAA | Authentication, Authorization, Accounting | 对于访问和使用的合适控制的必要的三个组件 |
| NAS | Network Access Server | 例如一个控制网络访问设备, 一个VPN服务器, 作为一个RADIUS客户端. |
| AVP | Attribute Value Pair | 在RADIUS包中一个3域的组件来包含一个特殊的域和他的数据 |
| VSA | Vendor-Specific Attributes | 由指定vendor管理的扩展的AVP |



1. AAA是一个安全架构模型。
2. RADIUS是AAA的一个特定的实现。
3. FreeRADIUS是RADIUS的一种实践应用。
4. 因此我们有AAA -> RADIUS -> FreeRADIUS。
5. RADIUS是关于中心控制和NAS vendor支持的事实标准。
6. RADIUS是一个 C/S 协议. 他使用UDP, 监听在1812端口用于authentication（认证）, 在1813端口用于accounting（核算）请求。
7. RADIUS数据包有零个或多个AVP，其中包含了所使用的数据RADIUS。
8. RADIUS数据包有0或者更多的AVPs, 包含用在RADIUS中的数据。
9. FreeRADIUS实现了RADIUS协议以及在RFC中指定的各种扩展。
10. FreeRADIUS是一种非常受欢迎的、使用广泛且非常灵活的RADIUS服务器。


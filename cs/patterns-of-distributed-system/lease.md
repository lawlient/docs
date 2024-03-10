# 租约

> 一定期限内给予持有者特定权力的协议。


**期限**就是租约的根本。

这一特性使得租约可以容忍机器失效和网络分割。

在期限之内，租约其实就是服务器和客户端之间的协议，而这个协议的内容可以五花八门。

- 如果协议内容是服务器确认客户端还存活，那么这个租约的功能就相当于心跳；
- 如果协议内容是服务器保证内容不会被修改，那么这个租约就相当于读锁；
- 如果协议内容是服务器保证内容只能被这个客户端修改，那么这个租约就相当于写锁。


租约这种灵活性和容错性，使其成为了维护分布式系统一致性的有效工具。



iptables的底层是现实netfilter，netfilter作为一个通用的、抽象的框架提供一整套hook函数的管理机制，使得数据包过滤、包处理、等成为可能。netfilter的架构就是在整个网络流程的若干位置放置一些钩子，并在每个钩子上挂载一些处理函数进行处理。

IP层的5个钩子点的位置，对应iptables就是5条内置链。

![](https://img2020.cnblogs.com/blog/974353/202102/974353-20210224162232689-1149812078.jpg)
图1 netfilter在Linux网络中的地位

iptables -> Tables -> Chains -> Rules, Tables有Filter, NAT, Mangle, Raw四种常用内置表，另外加入了一张新表security,用于在数据包上应用SELinux. Chains有INPUT、FORWARD、OUTPUT、PREROUTING、POSTROUTING五种内置链。
数据匹配后，常见的处理动作：
- ACCEPT: 同意数据包通过，并继续执行后续的规则
- DROP: 防火墙丢弃包，不再进行后续处理。应用场景是不让数据源意识到你的系统的存在，可以用来模拟宕机。
- REJECT: 给客户端返回一个connection refused或destination unreachable报文。应用场景是不让数据源访问你的系统，这里没有你要的服务内容。
- SNAT：源地址转换，解决内网用户用同一个公网地址上网的问题。
- MASQUERADE：是SNAT的一种特殊形式，适用于动态的、临时会变的ip上。
- DNAT：目标地址转换。
- QUEUE: 将数据包移交到用户空间，供用户空间的程序处理
- RETURN: 跳出当前链，停止执行当前链中的后续Rules，并返回到调用链(the calling chain)中。
- JUMP：跳转到其他用户自定义的链继续执行。JUMP存在的意义在于：用户自定义链中的规则和系统预定义的5条链中的规则没有区别。由于自定义的链没有与netfilter里的钩子进行绑定，所以它不会自动触发，只能从其他链的规则中跳转过来.
- REDIRECT：在本机做端口映射。
- LOG：在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则，也就是说除了记录以外不对数据包做任何其他操作，仍然让下一条规则去匹配。

如下图2所示是iptables的工作原理，内核就如同紫禁城，不是随随便便就能进去玩耍的，有什么事必须走相关流程，iptables就相当一个御林军接待可以把一些规则保安请求上报皇上(内核)，然后皇上在把这个规则纷发到各个安保点，由各个安保点执行相关规定。
![](https://img2020.cnblogs.com/blog/974353/202102/974353-20210224162249032-1050861208.png)
图2 iptables的工作原理

紫禁城有五个安保大门分别是PREROUTING、INPUT、FORWORD、OUTPUT和POSTROUTING，每个门岗都一个checklist，需要验明正身。如下图所示蓝色的是各个节点需要检查的表项。
![](https://img2020.cnblogs.com/blog/974353/202102/974353-20210224171541485-1192708606.jpg)
图3 一个网络包经过iptables的处理路径

这些检查表项是哪些内容呢？
- **filter表**：用于控制到达某条链上的数据包是继续放行、直接丢弃或拒绝，除了PREROUTING和POSTROUTIN其他节点都有检查；
- **nat表**：用于修改数据包的源和目的地址，分为DNAT和SNAT。对于协议栈来说，刚进来的需要检查DNAT，马上要出去的要检查SNAT(srcIP从私网IP转换为外网地址，对外通信)；
- **mangle表**：用于修改数据包的IP头信息，这是唯一一个在各个节点都需要检查的表；
- **raw表**：iptables是有状态的，即iptables对数据包有连接追踪(connection tracking)机制，而raw是用来去除这种追踪机制的。刚从协议栈外进到协议栈内的都要检查，相当于不能带追踪器进到紫禁城；
- **security表**：最不常用的表（通常说iptables有4张表，Security表是新加入的特性），用于在数据包上应用SELinux.
OUTPUT节点是唯一一个5张表都要查的。

## 查看表
- iptables -nvL (默认是filter表)
- iptables -t mangle -L (查看mangle表，同样可用于nat, raw表)
- iptables -L (列出所有表)
- iptables -L INPUT (只看input链)
- iptables -vL INPUT (显示input链上的数据统计信息)
- iptables -vnL INPUT (比上面多现实IP信息)
- iptables -vnL INPUT --line-numbers (比上面多现实行号)
## 添加规则策略示例
- 配置允许ssh连接(-A 表示在末尾添加一条规则，-I是在链开头的添加规则)
```
iptables -A INPUT -s 10.2.3.4/24 -p tcp --dport 22 -j ACCEPT
```

- 阻止来自某个IP/网段的所有连接
```
iptables -A INPUT -s 10.1.2.3 -j DROP
```

- 封锁端口，要阻止从本地1234端口建立对外连接
```
iptables -A OUTPUT -p tcp --dport 1234 -j DROP
```

- 端口转发，web服务监听在8080端口但希望用户能通过80端口访问到服务
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```
- 禁用PING
```
iptables -A  INPUT -p icmp -j DROP
```

- 删除规则
```
iptables -F
iptables -t nat -F
iptables -t filter -D INPUT 1 (删除第一行的规则，也可以用添加的相应命令)
```

- 保存于恢复
以上方法对iptables规则做出的改变是临时的，机器重启后会丢失。如果想永久保存这些更改，则需要使用如下命令：
```
iptables-save //永久保存规则
iptables-save > iptables.bak //重定向到一个文件
iptables-restore < iptables.bak //恢复备份文件中的规则
```

## snat和dnat
### snat
- iptables-t nat -A POSTROUTING -s 10.8.0.0/255.255.255.0 -o eth0 -j SNAT --to-source192.168.5.3 #该命令表示把所有10.8.0.0网段的目的地是eth0的数据包SNAT成192.168.5.3的ip然后发出去
- iptables-t nat -A POSTROUTING -s 10.8.0.0/255.255.255.0 -o eth0 -j MASQUERADE 
  #MASQUERADE，地址伪装，算是snat中的一种特例，可以实现自动化的snat.MASQUERADE会自动读取eth0现在的ip地址然后做SNAT出去，这样就实现了很好的动态SNAT地址转换。
- iptables -t nat -A POSTROUTING -s 10.1.1.0/24 ! -o br0 -j MASQUERADE # 把10.1.1.0/24网段的不走br0的数据包进行自动snat，一般自动伪装成它走的下一个网卡的地址

## 开关防火墙
- systemctl stop firewalld.service
- systemctl start firewalld.service

# ipvsadm 常用命令
- 查看所有规则
```
ipvsadm -Ln
```


### 参考文档
- 杜军《Kubernetes网络权威指南》电子工业出版社
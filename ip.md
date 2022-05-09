## ip link 网络设备配置命令
man ip link查看系统说明

### 查看命令
```
列出所有的link: ip link list 
查看link详情：  ip -d link show kube-ipvs0, 可以看出该端口是veth型的还是dummy网卡还是正常的网卡
```
#### 查看容器对应主机的端口
1. 到容器中Cat /sys/class/net/eth0/iflink，例如得到12
2. ip link列出来的12号端口就是该容器在主机端的veth-pair端

### ip link 设置SR-IOV端口对
以下命令组合是创建一个容器时，为它创建SR-IOV端口及其它网络资源的命令。该端口interface-name为eth0，它是由vfnic102和vfnic107bond到一起组成的。该ep的命名空间pid为5269
```
ip link set service0 vf 56 trust on
ip link set service0 vf 56 spoofchk off
ip link set service0 vf 56 mac 02:02:0a:f3:05:c8
ip link set service0 vf 56 vlan 2046
ip link set service1 vf 56 trust on
ip link set service1 vf 56 spoofchk off
ip link set service1 vf 56 mac 02:02:0a:f3:05:c8
ip link set service1 vf 56 vlan 2046

ip link set vfnic110 netns 3151832
ip link set vfnic115 netns 3151832
nsenter -t 3151832 -n -F -- ip link set vfnic110 up
nsenter -t 3151832 -n -F -- ip link set vfnic110 address 02:02:0a:f3:05:c8
nsenter -t 3151832 -n -F -- ip link set vfnic110 down
nsenter -t 3151832 -n -F -- ip link set vfnic110 up
nsenter -t 3151832 -n -F -- ip link set vfnic115 address 02:02:0a:f3:05:c8
nsenter -t 3151832 -n -F -- ip link set vfnic115 down
nsenter -t 3151832 -n -F -- ip link set vfnic115 up
nsenter -t 3151832 -n -F -- ip link add eth0 type bond mode balance-xor xmit_hash_policy layer2+3
nsenter -t 3151832 -n -F -- ip link set vfnic110 down
nsenter -t 3151832 -n -F -- ip link set vfnic115 down
nsenter -t 3151832 -n -F -- ip link set vfnic110 master eth0
nsenter -t 3151832 -n -F -- ip link set vfnic115 master eth0
nsenter -t 3151832 -n -F -- ip link set vfnic110 up
nsenter -t 3151832 -n -F -- ip link set vfnic115 up
nsenter -t 3151832 -n -F -- ip address add 10.243.5.200/27 dev eth0
nsenter -t 3151832 -n -F -- ip link set eth0 up
nsenter -t 3151832 -n -F -- ip link add default gw 10.243.5.222 eth0
```

删除这个端口的命令是
```
nsenter -t 3151832 -n -F -- ip link set eth0 down
nsenter -t 3151832 -n -F -- ip address del 10.243.5.200/27 dev eth0
nsenter -t 3151832 -n -F -- ip link del eth0 type bond mode balance-xor xmit_hash_policy layer2+3
nsenter -t 3151832 -n -F -- ip link set vfnic115 down
nsenter -t 3151832 -n -F -- ip link set vfnic115 address 10:00:00:00:00:00
nsenter -t 3151832 -n -F -- ip link set vfnic110 down
nsenter -t 3151832 -n -F -- ip link set vfnic110 address 10:00:00:00:00:00
nsenter -t 3151832 -n -F -- ip link set vfnic110 netns 1 // 把vfnic110从命名空间3242883中移出到root
nsenter -t 3151832 -n -F -- ip link set vfnic115 netns 1
ip link set service1 vf 56 vlan 0
ip link set service1 vf 56 mac 10:00:00:00:00:00
ip link set service1 vf 56 spoofchk on
ip link set service1 vf 56 trust off
ip link set service0 vf 56 vlan 0
ip link set service0 vf 56 mac 10:00:00:00:00:00
ip link set service0 vf 56 spoofchk on
ip link set service0 vf 56 trust off
```

### ip link 设置容器OVS端口对
以下命令组合是创建一个容器时，为它创建OVS端口及其它网络资源的命令。
```
// create port
ip link add vvport467 type veth peer name vport467 txqueuelen 0
// request IP in server
// set link up
ip link set vvport467 up
// Add the OVS side of the port to ovs, vvport467
docker exec -ti 529f90b84e1d ovs-vsctl add-port contivVlanBridge vvport467 
docker exec -ti 529f90b84e1d ovs-vsctl set port vvport467 tag=2046
ip link set vport467 mtu 1500
ip link set vport467 address 02:02:0a:f3:05:c8
// move vport467 to pod namespace
ip link set vport467 netns 1645285
// rename vport467 to newname eth0
nsenter -t 1645285 -n -F -- ip link set dev vport467 name eth0
nsenter -t 1645285 -n -F -- ip address add 10.243.5.200/27 dev eth0
nsenter -t 1645285 -n -F -- ip link set dev eth0 up
nsenter -t 1645285 -n -F -- route add default gw 10.243.5.222 eth0
```

删除OVS端口的命令如下：
```
docker exec -ti $(ovs-pod-id) ovs-vsctl --if-exists del-port contivVlanBridge vvport467 
ip link del vvport467 type veth peer name eth0 txqueuelen 0
```

### ip link 配置IPIP模式
```
 创建ns
ip netns add ns1
ip netns add ns2
 添加虚拟网卡对
ip link add v1 type veth peer name v1_r
ip link add v2 type veth peer name v2_r
ip link set v1 netns ns1
ip link set v2 netns ns2
 添加地址
ip a a 10.10.10.1/24 dev v1_r
ip l s v1_r up
ip a a 10.10.20.1/24 dev v2_r
ip l s v2_r up
ip netns exec ns1 ip a a 10.10.10.2/24 dev v1
ip netns exec ns1 ip l s v1 up
ip netns exec ns2 ip a a 10.10.20.2/24 dev v2
ip netns exec ns2 ip l s v2 up
 添加路由
ip netns exec ns1 route add -net 10.10.20.0 netmask 255.255.255.0 gw 10.10.10.1
ip netns exec ns2 route add -net 10.10.10.0 netmask 255.255.255.0 gw 10.10.20.1
 添加ip转发
echo 1 > /proc/sys/net/ipv4/ip_forward
Kubernetes Network IPIP Mode
 在 ns1 上创建 tun1 和 IPIP tunnel
ip netns exec ns1 ip tunnel add tun1 mode ipip remote 10.10.20.2 local 10.10.10.2
ip netns exec ns1 ip l s tun1 up
ip netns exec ns1 ip a a 10.10.100.10 peer 10.10.200.10 dev tun1
 在 ns2 上创建 tun1 和 IPIP tunnel
ip netns exec ns2 ip tunnel add tun2 mode ipip remote 10.10.10.2 local 10.10.20.2
ip netns exec ns2 ip l s tun2 up
ip netns exec ns2 ip a a 10.10.200.10 peer 10.10.100.10 dev tun2
上面的命令是在 NS1 上创建 tun 设备 tun1，并设置隧道模式为 ipip，然后还需要设置隧道端点，用 remote 和 local 
表示，这是 隧道外层 IP，对应的还有 隧道内层 IP，用 ip addr xx peer xx 配置。
# ip netns exec ns1 ping 10.10.200.10 -c 4
PING 10.10.200.10 (10.10.200.10) 56(84) bytes of data.
64 bytes from 10.10.200.10: icmp_seq=1 ttl=64 time=0.090 ms
64 bytes from 10.10.200.10: icmp_seq=2 ttl=64 time=0.148 ms
64 bytes from 10.10.200.10: icmp_seq=3 ttl=64 time=0.112 ms
64 bytes from 10.10.200.10: icmp_seq=4 ttl=64 time=0.110 ms
--- 10.10.200.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3000ms
rtt min/avg/max/mdev = 0.090/0.115/0.148/0.020 ms
```

## ip route 路由设置
 - ip route  //查看所有路由
route add default gw 10.243.5.222 eth0
route add -net 10.243.39.96/27 gw 10.243.39.126 dev eth1
route add -p 10.243.39.96 mask 255.255.255.224  10.243.39.126

### 策略路由
/etc/iproute2/rt_tables 是Linux系统路由表定义文件，其中0、253、254、255号表是Linux系统维护的表，1-252号表是自定义的表
 - man ip rule // 查看路由策略数据库管理命令
 - ip rule list //查看所有路由策略数据
 - ip route list table 0 //查看0号路由表中的路由信息
策略路由通常用于多个多个网口不同网络平面的连通性，或者不同vlan的路由设置。例如: 主机上有网卡eth0 10.243.5.201和网卡eth1 10.243.39.98，这两个地址分属不同的vlan. eth0的网关是10.243.5.222，eth1的网关是10.243.39.126. 默认路由只有设置一个，如下：
```
default via 10.243.5.222 dev eth0 
10.243.5.192/27 dev eth0  proto kernel  scope link  src 10.243.5.201 
10.243.39.96/27 dev eth1  proto kernel  scope link  src 10.243.39.98
```
这时需要设置策略路由，让10.243.39.98/27网段的数据走路由表16，在路由表16中设置eth1的默认网关：
```
ip route add default via 10.243.39.126 dev eth1 table 16
ip rule add from 10.243.39.98/27 lookup 16
```

## 删除网桥端口
ip a 看到端口br-d70cd48149bc是down的，并且是多余的。可以用如下命令删除：
ip l delete  br-d70cd48149bc

## 查看tuntap设备
ip tuntap list
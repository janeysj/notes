## 权威参考
http://tldp.org/HOWTO/Traffic-Control-HOWTO/

## 上行限速

TC 只能精确地限制出口的流量。例如网卡eth0 192.168.1.1  b8:ca:3a:5e:c9:e0，
   限制该网卡IP地址上行的规则如下：
```
    tc qdisc del dev eth0 root //首先清空队列

    tc qdisc add dev eth0 root handle 1: htb default 10     //默认队列10

	tc class add dev eth0 parent 1: classid 1:1 htb rate 100mbit ceil 100mbit // 总流量为100mbit

	tc class add dev eth0 parent 1:1 classid 1:10 htb rate 100mbit ceil 100mbit prio 1 //10号队列允许最大流量100mbit
	tc class add dev eth0 parent 1:1 classid 1:11 htb rate 50mbit ceil 50mbit prio 1   //1号队列允许最大流量50mbit

	#添加支类规则队列，采用sfq伪随机队列，并且10秒重置一次散列函数。
	tc qdisc add dev ifb0 parent 1:10 handle 10: sfq perturb 10
	tc qdisc add dev ifb0 parent 1:11 handle 11: sfq perturb 10

	tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 192.168.1.1 flowid 1:11// 源地址为192.168.1.1的走11号队列

	#修改分类带宽：
	tc class change dev eth0 parent 1:1 classid 1:11 htb rate 60mbit ceil 60mbit prio 1

	#删除分类：
	tc class del dev eth0 parent 1:1 classid 1:11 htb rate 60mbit ceil 60mbit prio 1

	#删除队列：
	tc qdisc del dev ifb0 parent 1:11 handle 11: sfq perturb 10

	#唯独删除filter和其他不一样，filter不支持change,并且如果使用tc filter del dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 192.168.1.1 flowid 1:11命令会把eth0设备上的所有filter规则都清空。正确的方法是，
	filter parent 1: protocol ip pref 1 u32 fh 800::801 order 2049 key ht 800 bkt 0 flowid :11
    match c0a84bef/ffffffff at 16
    找到分类  :11 对应的所有filter的句柄号，就是 800::801 这种东西
    tc filter delete dev eth0 parent 1: protocol ip prio 1 handle 800::801 u32 就好了。而且这里注意要先删除分类绑定的filter,才能删除class.
```

## 下行限速
- 因为无法控制别人发送多少数据包，因此TC限制入口方向的功能较弱，不支持分类。
- TC入口方向的限速虽然不是很强大，但是限制入口方向的总带宽确实准确的。因此如果不是细分入口流量类型，限制总的入口流量还是可以的，但仅限于简单的drop规则

### 限制ingress总带宽
```
tc qdisc del dev eth0 ingress
tc qdisc add dev eth0 ingress handle ffff:
tc filter add dev eth0 parent ffff: protocol all prio 1 basic police rate 50mbit burst 50mbit mtu 65535 drop
```

### 利用ifb0把ingress转换为egress，限制细分流量
1.  限制eth0下行的方法
    #加载IFB内核模块，创建ifb虚拟设备
	modprobe ifb numifbs=1 (numifbs： ifb数量）
	ip link set dev ifb0 up

	1).创建入队列
	#删除
	tc qdisc del dev eth0 handle ffff: ingress
	#添加
	tc qdisc add dev eth0 handle ffff: ingress


	2).把eth0的入口流量导入ifb0
	tc filter add dev eth0 parent ffff: protocol ip prio 10 u32 match u32 0 0 action mirred egress redirect dev ifb0

	3).TC工具对ifb0设置QoS规则
	tc qdisc add dev ifb0 root handle 1: htb default 20

	tc class add dev ifb0 parent 1: classid 1:1 htb rate 100mbit ceil 100mbit

	tc class add dev ifb0 parent 1:1 classid 1:10 htb rate 80mbit ceil 80mbit prio 1
	tc class add dev ifb0 parent 1:1 classid 1:11 htb rate 5mbit ceil 5mbit prio 1
	tc class add dev ifb0 parent 1:1 classid 1:12 htb rate 30mbit ceil 30mbit prio 1
	tc class add dev ifb0 parent 1:1 classid 1:20 htb rate 14mbit ceil 14mbit prio 1

	#添加支类规则队列，采用sfq伪随机队列，并且10秒重置一次散列函数。
	tc qdisc add dev ifb0 parent 1:10 handle 10: sfq perturb 10
	tc qdisc add dev ifb0 parent 1:11 handle 11: sfq perturb 10
	tc qdisc add dev ifb0 parent 1:12 handle 12: sfq perturb 10
	tc qdisc add dev ifb0 parent 1:20 handle 20: sfq perturb 10

	tc filter add dev ifb0 protocol ip parent 1:0 prio 1 u32 match ip dst 192.168.1.1 flowid 1:12

2. 注意，设置ifb0规则如果让eth0当掉或者插拔网线或者修改网卡参数那么加载在ifb0上的规则将不生效，需要重新从头到尾添加.

3. u32匹配器
   (这些-4 -12 16 20 之类的数字是数据帧中IP报文的偏移量，例如20就是找到IP报文的起始点，向后数20个字节，游标定位在这里，这个之后的数字就是要匹配的目标字节)
   目的地址为mac0:mac1:mac2:mac3:mac4:mac5的流
   tc filter add protocol ip parent 1:0 prio 1 u32 match u16 0x0800 0xffff at -2 match u32 mac_2_5 0xffffffff at -12 match u16 mac_0_1 0xffff at -14 flowid 1:11
   结果为:
   match 00000800/0000ffff at -4   //协议号？
   match 3e43c108/ffffffff at -12  //mac_2_5
   match 0000fa16/0000ffff at -16  //mac_0_1

   tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip dst 192.168.1.1 flowid 1:12
   tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 192.168.1.1 flowid 1:11
   结果为：
   match c0a8bc11/ffffffff at 12 //src ip 是 c0a8bc11/ffffffff
   match c0a8bc11/ffffffff at 16 //dst ip 是 c0a8bc11/ffffffff

   tc filter add dev eth0 parent 1: protocol ip prio 1 u32 flowid 1:11 match ip dport 5201 0xffff
   结果为：
   match 00001451/0000ffff at 20 //目的端口

   tc filter add dev eth0 parent 1: protocol ip prio 1 u32 flowid 1:11 match ip sport 5201 0xffff
   结果为：
   match 14510000/ffff0000 at 20 //源端口


4. 如果用户没有制定burst值得话，设置为速率的0.8 倍可以达到较好的性能

5. 优先级是数值越小越有限，如下例， 先匹配10号流再匹配11号流
```
tc filter add dev cali4312dcf99b3 protocol ip parent 1:0 prio 2 u32 match ip dst 172.24.100.153 flowid 1:11 

tc filter add dev cali4312dcf99b3 protocol ip parent 1:0 prio 1 u32 match ip src 172.24.100.191 flowid 1:10
```
6. 利用ifb0的缺点是对应真实网卡如果down再up起来，ifb0上的规则会自动消失，需要watch到该事件并重新添加规则

## tbf 限速
- tc qdisc change dev calia76646c4473  root tbf rate 2Mbit latency 50ms burst 2000 mpu 64 mtu 150000
- tc qdisc change dev bwpee838893af79 root tbf rate 2Mbit latency 50ms burst 2000 mpu 64 mtu 150000

## 模拟网络故障
使用了linux内核netem模块，首先要确认安装了netem模块 ```lsmod |grep netem```, 如果没有需要安装 kernel-modules-extra， 注意kernel-modules-extra的版本要和linux-kernel版本相同。如果是centos系统，可以通过命令``` yum install kernel-modules-extra-4.18.0-80.11.1.el7.centos.sn8.x86_64```安装该模块。可以通过命令```hostnamectl```查看内核版本信息。

### 模拟网络延迟
 - tc  qdisc  add  dev  eth0  root  netem  delay  100ms  //延迟100毫秒发送
 - tc  qdisc  add  dev  eth0  root  netem  delay  100ms  10ms  //延迟 100ms ± 10ms （90 ~ 110 ms 之间的任意值)发送
 - tc  qdisc  add  dev  eth0  root  netem  delay  100ms  10ms  30%   //大约有 30% 的包会延迟 ± 10ms 发送

### 模拟网络丢包
 - tc  qdisc  add  dev  eth0  root  netem  loss  1%  //随机丢掉 1% 的数据包
 - tc  qdisc  add  dev  eth0  root  netem  loss  1%  30% //随机丢掉 1% 的数据包，成功率为 30%

### 模拟包乱序 
 - tc  qdisc  change  dev  eth0  root  netem  delay  10ms   reorder  25%  50% //有 25% 的数据包(50%相关)会被立即发送，其他的延迟 10 秒
 - tc  qdisc  add  dev  eth0  root  netem  delay  100ms  10ms //新版本中，该命令也会在一定程度上打乱发包的次序

### 模拟包重复
 - tc  qdisc  add  dev  eth0  root  netem  duplicate 1% //随机产生 1% 的重复数据包 

### 模拟包损坏
 - tc  qdisc  add  dev  eth0  root  netem  corrupt  0.2%  //随机产生 0.2% 的损坏的数据包 

### 模拟端口不通
```
tc qdisc del dev eth0 root
tc qdisc add dev eth0 root handle 1: htb
tc class add dev eth0 parent 1: classid 1:10 htb rate 1000Mbps
tc filter add dev eth0 parent 1: protocol ip prio 1 u32 flowid 1:10 match ip dport 80 0xffff
tc filter add dev eth0 parent 1: protocol ip prio 1 u32 flowid 1:10 match ip sport 22 0xffff match ip dst 10.243.5.10/27
# 10 号队列的包全部丢掉
tc qdisc add dev eth0 parent 1:10 handle 10: netem loss 100%
``` 

## 常见问题
1. 限速后前一秒流量较大，后面正常。一般是因为burst值设置较大的。

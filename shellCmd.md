### 脚本第一行 #!/bin/bash
### 解决Linux环境下执行脚本时报错：/bin/bash^M: 坏的解释器: 没有那个文件或目录
  ```
  sed -i 's/\r$//' test.sh
  ```
### sed 
  -i 表示替换原文件
  #### 批量替换多个文件中的字符串
  * 格式: sed -i "s/查找字段/替换字段/g" \`grep 查找字段 -rl 路径\`

  * 例如：替换/home下所有文件中的xxx为ooo
   ```
    sed -i "s/xxx/ooo/g" `grep xxx -rl /home`

    sed -i "s/neutron/jdnet/g" `grep neutron -rl ./`
  ```
  * 单个文件中的字符串替换
  ```
  sed -i "s/111/222/g" 2.txt  将文件1.txt内的文字“111”替换成“222”
  sed -i "s/nvp_log_error(\"lock------------\");//g" vswitch*fw
  注意：符号前要加反斜杠
  ```
  * 在指定位置添加字符
  ```
  sed -i 's/^/HEAD&/g' test.file  在文件的每一行头添加字符HEAD
  sed -i 's/$/&TAIL/g' test.file  在文件的每一行尾添加字符TAIL
  ```
  * 删除所有的空行
  ```
  sed -i '/^\s*$/d' file.txt
  ```
  #### 删除指定行、指定列
    sed -i '1d' netplugin.log     删除二进制log的第一行, d表示delete
  #### 删除包含指定字符的行
    sed -i '/关键字符/d' 文件名    
  #### 删除file的每一行前n个字符n=.的个数
    sed -i 's/^......//' filename

  #### 在文件的第一行添加 Hello World
    sed -i  -e '1i Hello World.' /etc/rc.local    # 第一个i表示替换原文件，第二个i表示insert, 1表示第一行

  #### 打印指定行
    set -n "8p" test.file          打印第8行    

  #### sed命令中使用变量
    sed -i "/${gw}/d" /etc/rc.local    

  #### 输出一行中的数字
    sed 's/[^0-9]//g'    # 意思是把非0-9的字符都删掉，因此就只剩数字了，同理还可以有 sed 's/[^a-z]//g'

### df du
  - df -h  显示目前所有文件系统的可用空间及使用情形
  - du -h --max-depth=1 work/testing 查询文件或文件夹的磁盘使用空间
    ```
    统计总数大小

    du -sh workpath/

    du -sm * | sort -n //统计当前目录大小 并安大小 排序

    du -sk * | sort -n

    du -sk * | grep aimtag //看指定目标aimtag的大小

    du -m | cut -d "/" -f 2 //看第二个/ 字符前的文字

    查看此文件夹有多少文件 /*/*/* 有多少文件

    du workpath/

    du workpath/*/*/* |wc -l

    40752

    解释：

    wc [-lmw]

    参数说明：

    -l :多少行

    -m:多少字符

    -w:多少字
    ```   

### find
  ```
  find api/ -name "*.go" 查找api目录下所有的go文件
  find ./ -name "*.yaml" -or -name "*.yml" //查找当前目录下以yaml或者yml结尾的文件
  find /etc/ | xargs grep paste --color=auto
  find ~ -empty (查找home目录下的所有空文件)
  find /home/* -mtime +3 -delete  查找 /home/目录下所有三天前的文件，并删除。
  ```

### grep
```
// 查找k8s_cluster_cidr值中指定字符串cluster-cidr后面的内容, -o 只展示部分信息
echo $k8s_cluster_cidr|grep -o "cluster-cidr=.*"|awk -F"=" '{print $2}'|awk '{print $1}'
// 查找包含calico或者flannel或者multus的行, -E 表示使用expression正则表达式
grep -E 'calico|flannel|multus'
// 只查找完整单词，不是前缀、中间或者后缀
grep -w China
// 反向查找，排除包含Japan的内容
grep -v Japan
```

### wc
  计算输入的行数、单词数、字符数和字节数，一般和管道一起使用。
  ```
  wc -l file.txt  统计文件的行数
  wc -w file.txt  统计文件的单词数
  wc -c file.txt  统计文件的字符数
  wc -m file.txt  统计文件的字节数

  find api/ -name "*.go"|wc -l   统计api目录下所有go文件的个数
  find api/ -name "*.go"|xargs cat|wc -l  统计api目录下所有go文件的所有行数
  find api/ -name "*.go"|xargs cat|grep -v ^$|wc -l   统计api目录下所有go文件除去空行的所有行数
  ```

### awk 
  模式匹配搜索和处理语言
  #### 操作行
  * 取文件的第4行
    ```
    awk 'NR==4{print}' file.txt 
    awk 'FNR==4' file.txt
    ```
  * 从文件中第四行开始取
    ```
    awk 'FNR>3' file.txt        
    ```
  * 从文件中取前三行
    ```
    awk 'FNR<=3' file.txt
    ```
  #### 操作列
    awk '{print $2}' name.txt   取文件的第二列 
    awk '{print $3 "\t" $5}' name.txt   取文件的第三列和第五列并用tab键分开 
    neuron net-list|awk '{print $2}'   awk支持管道操作
    sid=`echo $line |awk '{split($2,arr,"-");print arr[4]}'` 打印改行内容并把第二列的字符以字符"-"做split操作，split的结果放在数组arr中，并打印数组的第四个元素（注意：第一个元素是arr[1], 不是arr[0]）

    MANAGE_IF="172.18.0.0/16"
    vtepif=`echo $MANAGE_IF |awk '{split($1,arr,".");print arr[1],arr[2]}'|awk '{print $1 "." $2 ".(.*)"}'` # 172.18.(.*)
  实例：逐行处理文件pod.txt和mactable.txt, 根据文件pod中每行的IP为索引，到mactable文件中查找IP对应的mac地址和interface
  ```
  #!/usr/bin/bash
  cat pod.txt | while read line
  do
    #处理每行内容 "$line"
    #第三行是ip
    ip=`echo $line |awk '{print $3}'`
    echo $line |grep -Po '(\d{1,3}\.){3}\d{1,3}' |wc -l 可以计算这一行中有几个IP地址类型的字符串
    ml=`grep -w $ip mactable.txt|awk '{print $3 "\t" $5}'`
    echo $ml
  done
  ```

  #### 字符串替换
  ```
#ippools="10.100.0.0/24,10.110.0.0/24,10.120.0.0/24"
if [[ $# > 0 ]]; then
    # create mimic ippools by parameters
    # read ippools from parameter
    ippools=$1
    # split by ","
    array=(${ippools//,/ })
    for var in ${array[@]}
    do
       echo $var
       # transfer 10.100.0.0/24 to 10.100.0.0-24 etc..
       index=`sed "s/\//-/g" <<< $var`
       # fill parameters to template to create yaml
       sed "s!{{INDEX}}!$index!g; s!{{IPPOOLCIDR}}!$var!g" ippool.yaml.template > ippool$index.yaml
       ./calicoctl create -f ippool$index.yaml
    done
fi
  ```

  #### 配合管道操作
  kubectl get pod -n kube-system | grep voyage-agent | awk '{system("kubectl delete pod "$1" -n kube-system")}'

### join 将两个文件中，指定栏位内容相同的行连接起来
注意指定栏位的内容的排列顺序要一致
```
# cat testip.list 
10.243.129.1
10.243.129.3
10.243.129.4

#cat testmac.list 
10.243.129.1  6c:92:bf:64:2c:1a
10.243.129.2  6c:92:bf:64:2c:1b
10.243.129.3  6c:92:bf:64:2c:1c
10.243.129.4  6c:92:bf:64:2c:1d

# join testip.list testmac.list 
10.243.129.1 6c:92:bf:64:2c:1a
10.243.129.3 6c:92:bf:64:2c:1c
10.243.129.4 6c:92:bf:64:2c:1d
```

### 删除Windows文件“造成”的'^M'字符
```
 cat file | tr -s "\r" "\n" > new_file 
 或
 cat file | tr -d "\r" > new_file
```

### 解决“/bin/bash^M: bad interpreter: No such file or directory”
1. 使用linux命令dos2unix filename，直接把文件转换为unix格式
2. 使用sed命令sed -i "s/\r//" filename  或者 sed -i "s/^M//" filename直接替换结尾符为unix格式
3. vi filename打开文件，执行 : set ff=unix 设置文件为unix，然后执行:wq，保存成unix格式。

### bash shell 脚本写法
#### 赋值，等号前不能有空格
* 基础变量赋值
```
a=1
b="hello, world"
cmd_output=`ls`
c=$b
```

* 用户显式输入赋值，格式read  变量1   变量2
```
echo –n “ please enter your name”  //-n 表示用户输入和提示信息在同一行
read name
echo “your name is $name”
``` 

* 把数字放进字符串
```
bw=200
bw_string="${bw}Kbit"
```

* 使用变量字符串组成另外一个字符串，带‘’的
```
a=1
handle=test$a
body='{"handleid": '${handle}'}'

* 数组定义与遍历
indexs=(0 1 2)
SERVERS=("21.0.12.4" "11.3.103.11" "11.3.58.38")

for i in ${indexs[@]}
do   
    	index=$i
	#script name
	svr=${SERVERS[index]}

	echo "backup ", ${SERVERS[index]}
	echo " " >> $LOGFILE	
	echo "———————————————–" >> $LOGFILE	
done;
```
#### 取值
* 基础取值方法
```
$a(取变量a的值)

$#： 位置参数个数（不包括Shell脚本名）
$*:  位置参数组成的字符串还是传递给脚本或函数的所有参数？
$!:   上一个后台命令对应的进程号
$?:   上一个命令的退出状态，为十进制数字，如果返回为0，则代表执行成功。
$0	当前脚本的文件名
$n	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
$@	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，啥不同呢？
$$:   当前的进程号PID
${@:2} 当脚本有多个参数时，取第二个参数之后的所有参数
${@:3:1} 当脚本有多个参数时，取第3个参数之后的1个参数
```
* 取时间
```
#/bin/bash
DATE=`date '+%Y%m%d-%H%M'`
date -u +'%b-%d-%Y-%H:%M:%S.UTC' // Jan-15-2020-03:31:31.UTC 显示UTC时间
date '+%B-%d-%Y-%H:%M:%S'        // January-15-2020-11:32:12 显示当前系统时间
date '+%B-%d-%Y-%H:%M:%S%N'      // March-26-2020-10:48:30921270866 显示当前系统时间,精确到纳秒
date '+%s%N'                     // 1585190086791674716      显示时间戳，精确到纳秒
date '+%s'                       // 1585190086               显示时间戳，精确到秒
其他格式可以参看man date的帮助文档
```

* 常用shell环境变量，环境变量的名称由大写字母组成
```
HOME: 用户主目录的全路径名，cd $HOME 即可切换到用户的主目录
PATH： 类似于windows下的路径，Shell会在里面依次寻找二进制的可执行文件。
    echo $PATH可以显示当前的PATH
    添加新的PATH方法 $PATH=$PATH:新PATH
TERM:  终端类型 echo $TERM
PS1:   提示符，root用户默认为#，普通用户默认为$
SHELL： Shell解释器的绝对路径
LOGNAME: 登录用户的用户名
UID：  当前用户的UID    echo $UID
```

* 命令行上的位置变量，就是根据出现在命令行上的参数的位置确定的变量
```
$命令  参数1   参数2  参数3
其中  $0  对应  执行的命令名
     $1  对应  参数1
     $2  对应  参数2
     $3  对应  参数3
```
#### 输出值
* echo 
```
echo $fix_ippool
echo "/root/auto-calico-route.sh ${calico_ippool} ${fix_ippool} ${gw}" >> /etc/rc.local //双引号能取变量的值，单引号不行. 把这句话追加到文件最后一行
```
* cat

#### 判断
* if else，注意方括号左右要有一个空格;注意if then else fi语句中，then和else直接要有一句话，不能then后面直接加else.
```
if test-commands; then
    consequence-command;
[elif more-test-commands; then
    more-consequents; ]
[else alternate-consequents; ]
fi


if [ -f /tmp/restart_netplugin ]; then ... fi
-e filename 如果 filename存在，则为真
-d filename 如果 filename为目录，则为真
-f filename 如果 filename为常规文件，则为真
-L filename 如果 filename为符号链接，则为真
-r filename 如果 filename可读，则为真
-w filename 如果 filename可写，则为真
-x filename 如果 filename可执行，则为真
-s filename 如果文件长度不为0，则为真
-h filename 如果文件是软链接，则为真
filename1 -nt filename2 如果 filename1比 filename2新，则为真。
filename1 -ot filename2 如果 filename1比 filename2旧，则为真。
-eq 等于
-ne 不等于
-gt 大于
-ge 大于等于
-lt 小于
-le 小于等于
-z 判断 变量的值，是否为空。若变量的值为空，返回0，为true；反之返回1，为false
-n 判断变量的值，是否为空。若变量的值为空，返回1，为false；反之返回0，为true
```

```
notRunningCount=`kubectl get pods -n kube-system | grep calico-node | grep  -v Running |wc -l`
notReadyCount=`kubectl get pods -n kube-system | grep calico-node | grep   "0/1" |wc -l`
if [[ $notRunningCount != "0" && $notReadyCount != "0" ]]; then
#if [ $notRunningCount -ne "0" -a $notReadyCount -ne "0" ]; then #也可以这样写，这样符号的区别
  echo "calico-node is not ready!"
fi
```

* switch case
```
case variable in
  value1)
         commands ;;
  value2)
        commands ;;
  *)
       commands ;;
esac
```

* 判断字符串中是否包含某个substr
```
实例：字符串中是否包含'https'
if [[ $EtcdEndpoints == "https"* ]];then
       if [[ $EtcdKeyFile == "" || $EtcdKeyFile == 'none' ]];then
	       echo "ERROR: EtcdKeyFile is none when use SSL!"
		   exit 1   
       fi   
 fi
 ```
或者 ``````


#### 循环
* while
```
a=0
while [ "$a" -le 10 ]
#–ge大于等于,–lt小于,–le小于等于,-eq等于,-ne不等于
do
    let a++  #let 算术运算,支持的操作符多而且不容易出错
done
```

* for
```
for name [in words … ]; do commands; done

for (( expr1; expr2; expr3 )) ; do commands; done

for i in (0,1,2,3,4,5,6,7,8,9); do echo $i; done
```

```
#!/bin/bash

for i in {11..55};
do
    ./make-kubelet-pm.sh hollow-node-${i}
    ./make-proxy-pm.sh hollow-node-${i}
    ./make-kubelet-config.sh hollow-node-${i} 10.243.129.124
done

```

实例1：删除残留docker容器, 既已经停掉的容器
```
  for line in $(docker ps -a |  awk -F' ' '{print $1}')
  do 
      docker rm $line
  done
```

实例2：删除neutron port-list 中的每个port, 注意[]左右两边有要“有空格”
```
  for line in $(neutron port-list -c id)
  do
      if [ $line != "|" ]; then 
          neutron port-delete $line
      fi
  done
```

实例3：读取文件中每一行，并用eval执行命令
```
#!/bin/bash
cat ip.txt | while read line
do
    #处理每行内容 "$line"
    #curl   -i -H "Content-Type:application/json"    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"    -g -X GET http://localhost:8081/version
    podname=`echo $line |awk '{print $2}'`
    namespace=`echo $line|awk '{print $1}'`
    ip=`echo $line|awk '{print $3}'`
    hostname=`echo $line|awk '{print $4}'`
    body="'{apiVersion: v1,kind: ipam,\"ip\": ${ip},\"handleid\": $namespace.$podname,\"hostname\": ${hostname}}'"
    curlCMD="curl -i -H \"Content-Type:application/json\" -H \"User-Agent: 9ee46d36266856c7faedb0a499533380\"  -H \"X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe\" -X POST -d ${body}  http://21.0.157.13:8081/ipam/assignip"
    echo $curlCMD
    eval $curlCMD
done
```

实例4：ssh上每一个标签node-network-driver为ovs的节点上执行命令
for i in \`kubectl get nodes -o wide --show-labels|grep node-network-driver=ovs| awk '{print $6;}'\` ; do ssh ${i} docker exec -i $(ssh ${i} docker ps --filter "name=k8s_voyage-openvswitch" -q) ovs-vsctl set-controller contivVlanBridge; done

实例5：在容器名带有Tag的网络命名空间执行命令
```
cat docker-enter-cmd.sh
#!/bin/bash
if [ $# == 0 ]; then
  echo -e "\033[31m Please input docker name tag \033[0m"
  exit 0
fi

dockertag=$1
if [ $# == 1 ]; then
  echo -e "\033[31m Please input command for the tagged docker \033[0m"
  exit 0
fi
dockerid=`docker ps|grep $dockertag|awk 'NR==1{print}'|awk '{print $1}'`
echo -e "\033[31m docker ID: "$dockerid
if [ $dockerid == '' ]; then
  echo " No such pod"
  exit 0
fi
pid=`docker inspect $dockerid|grep -w Pid|sed 's/[^0-9]//g'`
echo " docker Pid: "$pid
echo " exec \"" ${@:2}  "\" inside the docker with " $1 "name tag"
echo -e "------------------\033[0m"
nsenter -t $pid -n -F -- ${@:2}


```

* util
```
until cp $1 $2; do
    #until 语句只有until 之后的条件不满足时才执行
    echo 'Attempt to copy failed. waiting...'
    sleep 5
done
```

#### 死循环
注意如下的while死循环do和done之间一定要有语句，否则会报错
```
#!/bin/bash

i=1
while : 
do
# do nothing, just sleep forever
#  echo $i
  ((i++))
done

```

#### 在linux中直接敲循环命令
```
[root@tt]# for i in {0..3};do echo $i;done
0
1
2
3
[root@tt]# while true; do time curl http://172.18.8.129:8090;sleep 1;done
```

#### 变量运算
```
a=100
let a++
b=`expr $a / 2`(主要空格)
```

#### 求变量的长度
```
pod="helloworld"
${#pod}
```

#### set 标记脚本控制流程
      set -e
      ......
      set +e
      set -e表示从这里开始，如果脚本命令返回值不为0，就立即退出整个脚本，不再执行下面的命令；set +e的意思恰好相反，表示从这里开始如果返回值为非0不退出整个脚本
      set -o pipefail
      set -e有一个例外情况，就是不适用于管道命令。
所谓管道命令，就是多个子命令通过管道运算符(|)组合成为一个大的命令。Bash 会把最后一个子命令的返回值，作为整个命令的返回值。也就是说，只要最后一个子命令不失败，管道命令总是会执行成功，因此它后面命令依然会执行，set -e就失效了。使用set -o pipefail可以检查管道命令中的所有的命令，如果中间有任何一步有错就不再执行
     
      set -x
      ......
      set +x
      set -x表示从这里开始，打印命令本身，并且这些命令前面加+符号，到set +x处停止打印命令。主要用于Debug.

      set -u : 表示如果有未定义的变量就报错退出;如果不设置此项不报错不退出而是直接跳过此行

### split (split a file into pieces)
```
把一个大文BLM.txt切割成每行2000行的较小个文件，前缀是BLM_, 文件后缀是2位数:
split -l 2000 BLM.txt -d -a 2 BLM_
```

### scp 
#### 同时拷贝多个文件
1. scp /opt/kubernetes/{bin,ssl,cfg} root@192.168.1.2:/opt/kubernetes/
2. scp /usr/lib/systemd/system/{kubelet,kube-proxy}.service root@192.168.1.2:/usr/lib/systemd/system
3. 该方法同样适用于mkdir、ls、cat等命令
#### 自动输入密码
* 方法一，rsa秘钥
scp 远程拷文件，建议用搭配 ssh 方法:
1. 在客户机上生成 ssh 需要的 rsa 密钥: ssh-keygen -t rsa
2. 把生成的 id_rsa.pub拷到远程服务器用户的 .ssh 目录下，并更名为authorized_keys
这样一来，scp 拷文件就不需要密码了

* 方法二，sshpass 二进制命令文件，比expect的优点是不依赖环境库  
注意：如果把脚本A拷贝到远端多个节点，再用sshpass远程执行该脚本A，那么脚本A要尽量简单，最好不要有output否则会有意想不到的错误
```
  sshpass -p ${PASSWD[$i]} ssh -o StrictHostKeyChecking=no ${USERNAME[$i]}@${NODE[$i]} /etc/cron.hourly/auto-add-calico-route
  sshpass -p ${PASSWD[$i]} scp -o StrictHostKeyChecking=no -r  rc.local ${USERNAME[$i]}@${NODE[$i]}:/etc/
  sshpass -p ${PASSWD[$i]} ssh -o StrictHostKeyChecking=no ${USERNAME[$i]}@${NODE[$i]} chmod +x /etc/rc.local
  sshpass -p ${PASSWD[$i]} ssh -o StrictHostKeyChecking=no ${USERNAME[$i]}@${NODE[$i]} systemctl enable rc-local
  sshpass -p ${PASSWD[$i]} ssh -o StrictHostKeyChecking=no ${USERNAME[$i]}@${NODE[$i]} systemctl restart rc-local

```

* 方法三，expect 命令交互
```
#!/bin/bash
# You must install expect tool on each node.
# This script will enable rc.local service specially for ubuntu OS by auto-calico-route.sh
# Maybe you should type ctrl+c when shell pause while executing 'systemctl restart rc-local' on remote nodes
hosts=${1:-172.18.8.152,172.18.8.153,172.18.8.154,172.18.8.161,172.18.8.162,172.18.8.163}
usernames=${2:-root,root,root,root,root,root}
passwds=${3:-password,kylin@123,password,password,kylin@123,password}
CALICO_IPPOOL=${4:-10.26.0.0/16}
FIX_IPPOOL=${5:-10.251.0.0/16}
gw=${6:-100.200.0.1/24}

NODE=(${hosts//,/ })
USERNAME=(${usernames//,/ })
PASSWD=(${passwds//,/ })
nodeCount=`echo $NODE|wc -l`
GW=`echo $gw |awk '{split($1,arr,"/");print arr[1]}'`
for ((i=0; i<${#NODE[@]}; i ++))
do
echo "ssh "${USERNAME[$i]}@${NODE[$i]} " on "${NODE[$i]}
/usr/bin/expect<<-EOF
set timeout 60;
spawn  scp -r auto-add-calico-route.sh ${NODE[$i]}:/root/
expect {
"*yes/no" {send "yes\r"; exp_continue}
"*password:" {send "${PASSWD[$i]}\r";}
}
spawn ssh ${USERNAME[$i]}@${NODE[$i]}
expect {
"*yes/no" {send "yes\r"; exp_continue}
"*password:" {send "${PASSWD[$i]}\r";}
}
expect "*#"
send "/root/auto-add-calico-route.sh $CALICO_IPPOOL $FIX_IPPOOL $GW\r"
send "ip route\r"
expect "*#"
send "exit\r"
expect eof
EOF
done;
```
如果不设置timeout的话会自动退出，所以必须设置，或者用default字段可以设置expect超时或退出时的动作。
expect用于自动化地执行linux环境下的命令行交互任务，例如scp、ssh之类需要用户手动输入密码然后确认的任务。有了这个工具，定义在scp过程中可能遇到的情况，然后编写相应的处理语句，就可以自动地完成scp操作了。

下面是拷贝一个脚本到一系列节点，并ssh到这些节点执行该脚本的脚本
```
#!/bin/bash
index=(0 1 2 3 4 5)
NODES=("172.18.8.152" "172.18.8.153" "172.18.8.154" "172.18.8.161" "172.18.8.162" "172.18.8.163")
PASSWD=("password"    "kylin@123"    "password"     "password"     "kylin@123"    "password")
CALICO_IPPOOL="10.26.0.0/16"
FIX_IPPOOL="10.251.0.0/16"
GW="100.200.0.1"
for i in ${index[@]}
do
/usr/bin/expect<<-EOF
set timeout 60;
spawn  scp -r auto-calico-route.sh ${NODES[$i]}:/root/
expect {
"*yes/no" {send "yes\r"; exp_continue}
"*password:" {send "${PASSWD[$i]}\r";} 
}
spawn ssh root@${NODES[$i]}
expect {
"*yes/no" {send "yes\r"; exp_continue}
"*password:" {send "${PASSWD[$i]}\r";} 
}
expect "*#"
send "/root/auto-calico-route.sh $CALICO_IPPOOL $FIX_IPPOOL $GW\r"
send "ip route\r"
expect "*#"
send "exit\r"
expect eof
EOF
done;

```

### sort 自动排列
```
以filename文件中有若干行文本为例：
  sort filename.txt              按照字符串比较结果升序排序
  sort filename.txt |uniq -d     输出重复的行
  sort -u filename.txt           同上排序，在输出行中去除重复行
  sort -r filename.txt           逆序排列
  sort number.txt -o number.txt  排序后写入文件（写入源文件只能用-o,用定向输出原文件将被清空）
  sort -n number.txt             以int数值大小为序，而不是以字符串排序
  sort -n -k 2 -t : facebook.txt 使用冒号作为间隔符，并针对第二列来进行数值升序排序
  sort  -t "." -k1n,1 -k2n,2 -k3n,3 -k4n,4 ip.txt    IP地址排序，每一行都是192.168.1.2格式
```

### uniq  检查及删除重复的行列
该命令一般与 sort 命令结合使用，因为当重复的行并不相邻时，uniq 命令是不起作用，所以要结合sort命令。
```
统计各行在文件中出现的次数：
$ sort testfile1 | uniq -c
   3 Hello 95  
   3 Linux 85 
   3 test 30

在文件中找出重复的行：
$ sort testfile1 | uniq -d
Hello 95  
Linux 85 
test 30     
```

### diff 逐行比较文本
```
diff -B file1.c file2.c   忽略空白行笔记文件file1.c和文件file2.c
diff -E file1.c file2.c   忽略因tab引起的不同
```

### crontab Linux定时工具 // TODO: single crontab
```
crontab -u john -l          查看某个用户的crontab入口
*/10 * * * *                设置一个每十分钟执行一次的计划任务
```

### telnet 192.168.1.2 8081
退出： Ctrl  + ]

### cat 重定向文件
```
cat > file.name <<EOF
>xx
>oo
EOF
结束
```

如果是在脚本中就不需要加左边的>符号


### Ctrl+快捷键
1. Ctrl + r 搜索词 #启动反向搜索并键入命令的某些部分。它将查询历史记录，并向您显示与搜索词匹配的命令。
2. Ctrl + A转到行的开头; Ctrl + E转到结尾

## mobaxterm的使用
该软件可下载portable版本，直接运行
1. 添加命令快捷键：选择菜单Macro输入命令，然后点击stop进行编辑
2. 通过跳板机跳转：编辑session,选择network settings->点击SSH Gateway填写跳板机信息


### beyondCompare4密钥过期解决
 - 删除C:\Users\Lenovo\AppData\Roaming\Scooter Software\Beyond Compare 4下的所有文件，重启Beyond Compare 4即可（注意：用户名下的AppData文件夹有可能会被隐藏起来）
 - 删除D:\Program Files\Beyond Compare 4\BCUnRAR.dll 可以恢复30天试用期 (先试上面的，不行再试这个)
 - 重新输入密钥进行注册
 
## 版本
bionic focal impish jammy 都是ubuntu 版本名称，选择LTS版本，到 https://developer.aliyun.com/mirror/ 找OS镜像
bionic (18.04LTS)
focal (20.04LTS)
hirsute (21.04)
impish (21.10)
jammy (22.04LTS)
### 查看版本
- uname -ra
- uname -v
- lsb_release -a

### 查看系统函数
https://manpages.ubuntu.com/

### 永久修改DNS
编辑/etc/systemd/resolved.conf，它是 /etc/resolved.conf 的源文件。修改完成后执行 systemctl restart systemd-resolved.service  

## 安装源
安装正确的镜像不需要修改源。
如果非要修改，配备 /etc/apt/sources.list 并在该文件最下面添加以下源（注意raring 表示的是版本）：
```
deb http://mirrors.aliyun.com/ubuntu/ raring main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ raring-security main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ raring-updates main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ raring-proposed main restricted universe multiverse  
deb http://mirrors.aliyun.com/ubuntu/ raring-backports main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ raring main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ raring-security main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ raring-updates main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ raring-proposed main restricted universe multiverse  
deb-src http://mirrors.aliyun.com/ubuntu/ raring-backports main restricted universe multiverse  12345678910
```
并添加阿里云的DNS地址到/etc/resolv.conf中
```
#这里用的是阿里云的DNS服务器
nameserver 223.5.5.5  
nameserver 223.6.6.6
```
最后更新 `sudo apt-get update`

## 记一次安装ebpf环境整个过程
 - 1. 到官网 https://ubuntu.com/download/desktop 下载22.04版本镜像写到U盘，查到172.18.8.211主机上重启，按Del键进入boot选择USB然后按esc退出进入安装界面选择ubuntu22.04->选择最小化->填写主机名，带IP标志.  
 - 2. sudo su root进入root, vi /etc/netplan/01-network-manager-all.yaml ,编辑该文件，然后 netplan apply使之生效，查看地址是否正确，是否能ping网关、外网以及域名解析。
 - 3. 更新系统
  > - sudo apt update
  > - sudo apt list --upgradable
  > - sudo apt upgrade -y
 - 4. 开启ssh服务
 - 5. 安装golang
 - 6. 安装ebpf 编译工具 sudo apt install -y git lrzsz make clang libelf-dev llvm libfl-dev libssl-dev libedit-dev zlib1g-dev
 - 7. git 下载tracee代码然后配置GO环境变量，最后make all


## 用户切换
1. 切换到root用户: sudo su root

## IP地址固定
编辑文件/etc/netplan/01-network-manager-all.yaml 
```
cat /etc/netplan/01-network-manager-all.yaml 
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
     ens33:
         addresses: [192.168.31.142/24]
         gateway4: 192.168.31.2
         nameservers:
             addresses: [114.114.114.114, 8.8.8.8]
```
编辑完成后，输入命令 `sudo netplan apply`  
重启网络服务：systemctl restart network-manager 或者 systemctl restart NetworkManager //注意要大写


## SSH服务
1. sudo apt-get install openssh-server
2. sudo service ssh start
3. 允许root ssh登录：
   ```
   修改root密码：sudo passwd root
   修改sshd配置：打开/etc/ssh/sshd_config(注意不是/etc/ssh/ssh_config), 修改 PermitRootLogin yes
   重启ssh服务：service sshd restart或者systemctl restart sshd
   ```

## samba服务
1. 安装： sudo apt-get install -y samba samba-common
2. 修改共享文件夹权限：chmod 777 /root/share -R
3. 修改配置文件/etc/samba/smb.conf后，重启服务 sudo service smbd restart

apt install -y selinux-utils， getenforce 已经是Disabled了

4. 关闭防火墙 sudo ufw disable

5. 在配置文件 /etc/samba/smb.conf 最后添加如下语句，注意每个目录层级的属性都要是777 
```
[ebpf]
   comment = eBPF Files
   path = /root/ebpf
   browseable = yes
   directory mask = 0777
   write list = root
   writeable = yes
   public = yes
```   


打开控制面板→程序和功能→启用或关闭Windows功能。在弹出窗口中勾选 SMB1.0/CIFS 文件共享支持，子项中的后两项勾选。

## Linux常用工具
### net-tools
```
apt --fix-broken install
sudo apt install net-tools
```

## docker
### 安装Docker
 1. 卸载老版本
    $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
    $ sudo rm -rf /var/lib/docker
    $ sudo rm -rf /var/lib/containerd
 2. sudo apt-get install apt-transport-https ca-certificates  curl gnupg lsb-release
 3. 增加Docker官方GPG Key： `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`
 4. 设置稳定版本的仓库, 对于X86_64/amd64机器: 
  ```  
  echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
 5. 安装Docker 引擎: `sudo apt-get update; sudo apt-get install docker-ce docker-ce-cli containerd.io`
 6. 启动Docker: `sudo service docker start`

 ## golang 升级
  ```
  1. 删除老版本
  sudo rm -rf /usr/local/go
  sudo apt-get remove golang
  sudo apt-get remove golang-go
  sudo apt-get autoremove

  2. 手动安装
  sudo tar xfz go1.17.6.linux-amd64.tar.gz -C /usr/local

  3. 添加如下内容到 /etc/profile 文件尾
  export GOROOT=/usr/local/go  
  export PATH=$GOPATH:$GOBIN:$GOROOT/bin:$PATH

  4. 生效环境变量
  source /etc/profile
  ```

## 
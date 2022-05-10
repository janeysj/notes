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


## SSH服务
1. sudo apt-get install openssh-server
2. sudo service ssh start

## samba服务
1. 安装： sudo apt-get install -y samba samba-common
2. 修改共享文件夹权限：chmod 777 /root/share -R
3. 
. 修改配置文件/etc/samba/smb.conf后，重启服务 sudo service smbd restart

apt install -y selinux-utils， getenforce 已经是Disabled了

打开控制面板→程序和功能→启用或关闭Windows功能。在弹出窗口中勾选 SMB1.0/CIFS 文件共享支持，子项中的后两项勾选。仍然不行！！！

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

 ## falco安装
 1. 

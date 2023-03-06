1. centos中yum install samba -y
2. 修改配置文件/etc/samba/smb.conf,fileshare是共享后的文件夹名称。一般只要修改path就可以了
   ```
   [206share]
   comment = 206share 
   path = /home/sj/
   browseable = yes
   directory mask = 0777
   write list = root
   writeable = yes
   public = yes
   ```
3. 共享的文件夹的属性要可读写 chmod 777 file/ -R，并且文件的每一级都要修改属性
4. 启动samba进程 service smb restart 或者 systemctl start smbd
5. /etc/selinux/config 中SELINUX=disabled，必须重启系统才能生效
6. systemctl stop firewalld.service； systemctl disable firewalld.service 或者放开对应端口
7. win10访问不了samba共享文件夹解决方法：在运行窗口输入“gpedit.msc”；在弹出的“本地策略组编辑器里选择”下图红色标记。双击“启用不安全来宾登录”，在弹出的窗口里选择下图红色箭头指向的“已启用”，点“确认”按钮。这样win10就可以访问smb服务器的共享文件夹了。
8. 如果vmware虚拟机已经是nat模式，可以ping通baidu也能ping通windows但是从windows不通虚拟机。也就是vmware ping不通vmware网关。这时需要关闭windows的防火墙并且从起网络连接中的vmware network adapter两张网卡
9. sudo smbpasswd -a sj  // 添加用户名密码，有时候也不需要



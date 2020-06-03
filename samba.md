1. centos中yum install samba
2. 修改配置文件/etc/samba/smb.conf,fileshare是共享后的文件夹名称。一般只要修改path就可以了
3. 共享的文件夹的属性要可读写 chmod 777 file/ -R
4. 启动samba进程systemctl start smbd
5. win10访问不了samba共享文件夹解决方法：在运行窗口输入“gpedit.msc”；在弹出的“本地策略组编辑器里选择”下图红色标记。双击“启用不安全来宾登录”，在弹出的窗口里选择下图红色箭头指向的“已启用”，点“确认”按钮。这样win10就可以访问smb服务器的共享文件夹了。

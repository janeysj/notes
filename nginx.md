## 安装
### 安装准备
getenforce   这个命令可以查看当前是否开启了selinux 如果输出 disabled 或 permissive 那就是关闭了
如果输出 enforcing 那就是开启了 selinux


1、临时关闭selinux
setenforce 0            ##设置SELinux 成为permissive模式
setenforce 1    ##设置SELinux 成为enforcing模式


2、永久关闭selinux,修改/etc/selinux/config 文件将SELINUX=enforcing改为SELINUX=disabled

### 安装步骤
centos7: 配置nginx的Yum源
```
$ vi /etc/yum.repos.d/nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```
$ yum -y install nginx
$ rm -rf /etc/nginx/conf.d/default.conf //也可以不删

## 启动
systemctl start nginx

## 配置
 1. 配置文件： /etc/nginx/nginx.conf; /etc/nginx/conf.d/nginx.conf
 2. 首页文件： /usr/share/nginx/html/index.html

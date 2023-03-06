## centos下安装
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
 1. 配置文件： 
   - /etc/nginx/nginx.conf
   - /etc/nginx/conf.d/nginx.conf

 2. 首页文件： /usr/share/nginx/html/index.html

 3. 如果想做反向代理，并且一分三的转发流量，可以如下配置
 ```
 server {
		listen       20080;
		server_name localhost;

		location / {
			root   html;
			index  index.html index.htm;
			proxy_set_header X-Original-URI $request_uri;
			proxy_set_header Host $host:$server_port;
			proxy_set_header Connection "";
			proxy_http_version 1.1;
			chunked_transfer_encoding on;
			proxy_pass http://10.100.0.4:8080/;

			mirror /mimic1;
			mirror /mimic2;
			mirror_request_body on;
		}

		location /mimic1{
			proxy_pass http://10.100.0.5:8080$request_uri;
			proxy_set_header X-Original-URI $request_uri;
			proxy_set_header Host $host:$server_port;
			proxy_set_header Connection "";
			proxy_http_version 1.1;
			chunked_transfer_encoding on;
		}

		location /mimic2{
			proxy_pass http://10.100.0.6:8080$request_uri;
			proxy_set_header X-Original-URI $request_uri;
			proxy_set_header Host $host:$server_port;
			proxy_set_header Connection "";
			proxy_http_version 1.1;
			chunked_transfer_encoding on;
		}

		error_page   500 502 503 504  /50x.html;
		location = /50x.html {
			root   html;
		}
	}

 ```

## ubuntu下安装
sudo apt-get install nginx
安装完成后自动拉起nginx进程，使用 systemctl status nginx查看进程状态，如果没有起来很有可能是配置文件有问题. 它的配置文件如下：
  - /etc/nginx/nginx.conf
  - /etc/nginx/sites-available/default

如果是防火墙的问题，用如下命令查看：
 sudo ufw app list  
 sudo ufw status

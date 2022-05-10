## 安装
仅需要在控制节点安装一个ansible节点即可，其他节点不需要安装ansible
### 在centos系统安装
```
sudo yum install epel-release -y
sudo yum install ansible -y
```

## 使用
### 编辑配置文件/etc/ansible/hosts
```
[rs]
172.18.8.202 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='redhat@123'
172.18.8.203 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='redhat@123'
172.18.8.129 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
```
如果有连接不上的，手动ssh一下，确保密码正确

### 常用命令
 - ansible rs -m ping
 - ansible rs -a "df -h"
 # 运行脚本命令
 - ansible rs -m shell -a "netstat -ntl | grep 8091"
 # 拷贝文件
 - ansible nodes -m copy -a "src=/usr/sbin/calicoctl dest=/usr/sbin/calicoctl"

### 运行命令时常见错误
 - ```[WARNING]: Platform linux on host 192.168.1.2 is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change this. See https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.``` 错误中提示了说这个节点使用的python解释器是/usr/bin/python，但是ansible使用的可能不是这个，因此手动指定一下就可以了，方法是在/etc/ansible/hosts中指定这个节点的python解释器（有时候需要更新一下python版本）<br>：
 ```
 [nodes]
192.168.1.1 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456'
192.168.1.2 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456' ansible_python_interpreter=/usr/bin/python
192.168.1.3 ansible_ssh_port=22 ansible_ssh_user=root ansible_ssh_pass='123456' 
 ```
 - ```non-zero return code``` 报这个错是执行脚本错误, 当使用shell模块并且返回为空的时候，ansible就会认为shell脚本出错了，rc就返回1.


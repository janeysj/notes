## 使用命令
1. ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/peer.crt \
--key=/etc/kubernetes/pki/etcd/peer.key \
get /registry/namespaces/default

2. 在/root/.bashrc 中最后一句添加
```
alias etcdctl='ETCDCTL_API=3 etcdctl --endpoints https://172.171.19.210:2379  --cacert /opt/etcd/ssl/ca.pem --key /opt/etcd/ssl/server-key.pem --cert /opt/etcd/ssl/server.pem'
```


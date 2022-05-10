 0. 下载二进制calicoctl放到/usr/local/bin/下面
    配置/etc/calico/calicoctl.cfg
    ```
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "https://172.18.8.203:2379"
  kubeconfig: "/root/.kube/config"
  etcdKeyFile: "/etc/kubernetes/pki/apiserver-etcd-client.key"
  etcdCertFile: "/etc/kubernetes/pki/apiserver-etcd-client.crt"
  etcdCACertFile: "/etc/kubernetes/pki/etcd/ca.crt"
    ```
 1. calicoctl node status // BGP建立的状态
 2. calicoctl get nodes   // 获取集群节点
    如果报错十有八九是etcd文件错误
 3. calicoctl get BGPPEER // 客制化的bgppeer
 4. calicoctl get BGPConfiguration //
 5. calicoctl get FelixConfiguration 
 6. calicoctl get ippool
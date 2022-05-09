kubebuilder用来自定义k8s 资源类型，例如GuestBook类型(和deployment、ingress等资源类似)，用只需要简单的命令，和框架化的代码填充即可完成自定资源的定义。kubebuilder还可以控制该资源类型的创建、删除等操作的脚手架工程。这里的控制不是指用户使用kubectl create/delete来控制资源创建和删除的意思，而是用来动态显示资源状态的。例如deployment资源类型，如果是多副本的话，就需要创建多个副本，并感知目前创建了几个副本。如果某个副本异常退出还需要创建新的副本来顶替上。同样，如果副本数被用户调小，则需要删除一些副本。首先，上一张kubebuilder的架构图：
![](https://img2020.cnblogs.com/blog/974353/202111/974353-20211118154424805-339862563.jpg)

从上图可以看出controller是Kubernetes和任何其他operator的核心。控制器的任务是确保任何给定对象的实际状态(包括集群状态和可能的外部状态，如Kubelet的运行容器或云提供商的负载平衡器)与对象中的期望状态相匹配。每个控制器关注一个根类型，但可能与其他类型交互。这个过程称之为reconciling. 在控制器运行时中，为特定类型实现协调的逻辑称为Reconciler.在xxx_controller.go文件中，我们可以看到Reconcile函数，你可以在里面添加客制化代码。

webhook是修改请求和验证请求的，如果通过不了验证一般情况下报错返回。

### kubebuilder安装
到https://github.com/kubernetes-sigs/kubebuilder/releases 页面下载合适的release二进制kubebuilder版本
chmod +x kubebuilder && mv kubebuilder /usr/local/bin/

### 创建API
   mkdir /root/share/hello/src/github.com/world -p
   export GOPATH=/root/share/hello/
   export GOPROXY=https://goproxy.cn
   cd /root/share/hello/src/github.com/world 
   
   kubebuilder init --domain my.domain 
   // 默认namespaced为true
   kubebuilder create api --group net --version v1 --kind DpvsIngress
   // 如果需要修改namespace为cluster，可以使用下面的命令强制更新DpvsIngress API 定义
   kubebuilder create api --group net --version v1 --kind DpvsIngress --namespaced=false --force
   make manifests
   make install #期间报错少了什么组件就安装什么组件
   make run
   
   这个步骤报错及解决方法如下：
   （1） "invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable"
   export  KUBECONFIG=/xxx/xxx
   但是当修改了api/v1下面的代码需要重新make时，可能需要unset该变量否则无法make install
   
   （2）"metrics server failed to listen. You may want to disable the metrics server or use another port if it is due to conflicts  {"error": "error listening on :8080: listen tcp :8080: bind: address already in use"}"
   修改当前目录下的main.go 中 flag.StringVar(&metricsAddr, "metrics-bind-address", ":8090", "The address the metric endpoint binds to.") 语句的监听端口为本机未占用端口
   然后这个main函数就运行起来了，看了似乎是个daemon程序。

### 添加客制化定义
一般在xxx_types.go 中的添加一些spec属性，举例如下所示：
```
type WorldSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Foo is an example field of DpvsIngress. Edit world_types.go to remove/update
	Foo string `json:"foo,omitempty"`

	Selector *metav1.LabelSelector `json:"selector,omitempty"`
	Servers []Servers  `json:"servers"`
}

type Servers struct {
	Servers []VIPInfo `json:"vipInfo,omitempty"`
}

type VIPInfo struct {
	VIP        string   `json:"vip"`
	Database   Databaseinfo   `json:"database"`
}

type Databaseinfo   struct {
	name string `json:"name "`
	ip string `json:"ip "`
}
```
修改后再次执行make install，会在config/crd/bases/目录下生成资源类型YAML文件，并自动更新该资源类型。然后使用如下yaml文件创建一个资源实例：
```
apiVersion: my.domain/v1
kind: World
metadata:
  name: world-sample
spec:
  # TODO(user): Add fields here
  selector:
    matchLabels:
      tag: abc
  servers:
    - vipInfo:
        - vip: "172.0.0.254"
          database:
            name: "zone1"
            ip: "192.168.1.1"
    - vipInfo:
        - vip: "172.0.0.253"
          database:
            name: "zone2"
            ip: "192.168.1.2"
```

### 自定义资源类似的删除
删除自定义资源类型前，先把资源实例删掉。然后再用yaml资源定义文件删除资源类型，如果找不到资源类型文件则可以直接到etcd搜索关键字再删除。

## 参考
- https://book.kubebuilder.io/architecture.html
- https://blog.csdn.net/boling_cavalry/article/details/113922328
## golang 安装与升级
https://golang.google.cn/doc/
### 安装
- i. Download the archive from https://golang.google.cn/dl/. If you are installing Go version 1.2.1 for 64-bit x86 on Linux, the archive you want is called go1.2.1.linux-amd64.tar.gz.
- ii. Extract it into /usr/local, creating a Go tree in /usr/local/go. For example:
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
- iii. 在/etc/profile文件最后一行加上export PATH=$PATH:/usr/local/go/bin
### 版本升级
- i. 备份/usr/local/go
- ii. 安装新版本，并验证版本号
- iii. 删除备份文件

## gRPC
* protoc安装
  1. 到https://github.com/protocolbuffers/protobuf/releases 选择合适的环境文件，例如centos7 golang 使用的是protoc-3.10.1-linux-x86_64.zip
  2. export GOPATH=/usr/local, 执行命令  go get -u github.com/golang/protobuf/protoc-gen-go
  会下载protoc-gen-go二进制文件到$GOPATH/bin下面
  3. 修改GOPATH为你的工作目录，下载源码
  git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
  4. 然后进入$GOPATH/src/google.golang.org/grpc/examples/下面编译一些例子，如果缺少代码依提示解决，通常需要安装如下包
    git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
    git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
    git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
    go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
    git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
    在/etc/hosts中加上 151.101.72.249 github.global.ssl.fastly.net 可以加速下载
  E. 剩下的编写代码层面参考https://www.grpc.io/docs/tutorials/basic/go/	

* 本地设置
  export GOPATH=/root/share/gRPC/
  export PATH=$PATH:$GOPATH/bin

1. 编译pb文件，进入到dir_protoc同一级的目录下
   protoc -I dir_protoc/ dir_protoc/name.proto --go_out=plugins=grpc:dir_protoc/

2. gRPC 定义proto方法时必须有且仅有一个参数，有且仅有一个返回
   server 实现proto中的方法是要根据生成的pb.go文件来写
   如果server的name为ABC， 那么生成的client名字就是NewABCClient

3. 例子
   本机进入到目录/root/share/gRPC/src/google.golang.org/grpc/examples/xxx下
   export GOPATH=/root/share/gRPC/
   go run client/main.go

* server start code
```
    bport := fmt.Sprintf(":%s", port)
	log.Infof("gRPC server start ...")
	lis, err := net.Listen("tcp", bport)
	if err != nil {
		log.Fatalf("failed to listen voyage: %v", err)
	}

	pb.RegisterVoyageServerServer(s, &server{})
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve voyage: %v", err)
	}
```

* client 端调用示例
```
    address := fmt.Sprintf("%s:966", ep.HomingHost)
	dialCtx, dialCancel := context.WithTimeout(context.Background(), 10 * time.Second)
	defer dialCancel()
	conn, err := grpc.DialContext(dialCtx, address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewVoyageAgentClient(conn)

	// Contact the server and print out its response.
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	delEp := pb.Endpoint{
		EndpointID:  ep.EndpointID,
		HomingHost:  ep.HomingHost,		
	}	
	bigerr, err := c.DeleteAgentEP(ctx, &delEp)
	if err != nil {
		log.Errorf("fail to let agent delete ep %s: %v", ep.EpCommonName, err)
	}
	if bigerr != nil {
		log.Errorf("fail to let agent delete ep %s: %q", ep.EpCommonName, bigerr)
	}
	return err
```

## 数据类型
### map
- 定义与初始化<br>
```
type Vertex struct {
    Lat, Long float64
}
var m map[string]Vertex
func main() {
    m = make(map[string]Vertex)
    m["Bell Labs"] = Vertex{
        40.68433, -74.39967,
    }
    fmt.Println(m["Bell Labs"])
}
若顶级类型只是一个类型名，你可以在文法的元素中省略它。
var m = map[string]Vertex{
    "Bell Labs": {40.68433, -74.39967},
    "Google":    {37.42202, -122.08408},
}
```
- 删除元素<br>
```delete(m, key)```
- 获取元素<br>
```elem, ok := m[key]```
如果 key 在 m 中， ok 为 true。否则， ok 为 false，并且 elem 是 map 的元素类型的零值。
- 嵌套使用map<br>
```
package main
import 	"fmt"
func main() {
	m1 := make(map[string]interface{})
	m1["ip"] = "192.168.1.1"
        m1["age"] = 32 // interface类型的map, 支持多种值类型
	m2 := make(map[string]interface{})
	m2["host"] = "AR-21-36"
	m2["state"] = "Running"
	
	m1["attr"] = m2
	
	fmt.Println(m1)
	
	fmt.Println(m1["attr"].(map[string]interface{})["host"])
}
```
- 遍历map<br>
注意：借助关键字range对Go语言的map做遍历访问时，前后两轮遍历访问到的key的顺序是随机化的
```
for k, v := range m {  
    fmt.Printf("k=%v, v=%v\n", k, v)  
}  

也可以
for k := range m {  
    fmt.Printf("k=%v\n", k, v)  
}  
但是这样只能遍历各个量，并取到其中的key
```

### IPv4地址和unit之间的转换
```
func IPString2Unit(ip string) (uint, error) {
	b := net.ParseIP(ip).To4()
	if b == nil {
		return 0, errors.New("invalid ipv4 format")
	}

	return uint(b[3]) | uint(b[2])<<8 | uint(b[1])<<16 | uint(b[0])<<24, nil
}

func Uint2IPString(i uint) (string, error) {
	if i > math.MaxUint32 {
		return "", errors.New("beyond the scope of ipv4")
	}

	ip := make(net.IP, net.IPv4len)
	ip[0] = byte(i >> 24)
	ip[1] = byte(i >> 16)
	ip[2] = byte(i >> 8)
	ip[3] = byte(i)

	return ip.String(), nil
}
```

### 计算函数运行时间
可以在函数开始添加如下代码
```
    start := time.Now()
	defer func() {		
		klog.V(4).Infof("syncProxyRules took %v", time.Since(start))
	}()
```

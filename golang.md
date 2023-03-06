## golang 安装与升级
https://golang.google.cn/doc/
### linux安装
- i. Download the archive from https://golang.google.cn/dl/. If you are installing Go version 1.2.1 for 64-bit x86 on Linux, the archive you want is called go1.2.1.linux-amd64.tar.gz.
- ii. Extract it into /usr/local, creating a Go tree in /usr/local/go. For example:
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
- iii. 在/etc/profile文件最后一行加上export PATH=$PATH:/usr/local/go/bin

### windows安装
Download golangxxx.msi binary file from https://golang.google.cn/dl/ and double click this file to setup.

### 版本升级
- i. 备份/usr/local/go
- ii. 安装新版本，并验证版本号
- iii. 删除备份文件

## 常用框架
- 路由: Gorilla/Mux，standard library
- 测试: built-in testing,testify
- web framework: gin, echo, beego

## 编译与运行
- go run test.go
- go build -gcflags "-N -l" -o test test.go // -N -l关闭编译器代码优化和函数内联，避免断点和单步执行无法准确对应源码行， gdb test 进行单步调试
- go build -v -i  -o test test.go

## gRPC
* protoc安装
  1. 到https://github.com/protocolbuffers/protobuf/releases 选择合适的环境文件，例如centos7 golang 使用的是protoc-3.10.1-linux-x86_64.zip
  2. export GOPATH=/usr/local, 执行命令  go get -u github.com/golang/protobuf/protoc-gen-go
  会下载protoc-gen-go二进制文件到$GOPATH/bin下面, 如果不能下载则到google.golang.org/protobuf@v1.28.0/cmd/protoc-gen-go下面用 go build命令编译，同样编译protoc-gen-go-grpc  
  3. 修改GOPATH为你的工作目录，下载源码
  git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
  4. 然后进入$GOPATH/src/google.golang.org/grpc/examples/下面编译一些例子，如果缺少代码依提示解决，通常需要安装如下包
    git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
    git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
    git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
    go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
    git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
    在/etc/hosts中加上 151.101.72.249 github.global.ssl.fastly.net 可以加速下载
  5. 不一定要把二进制文件下载到$GOPATH/bin下面，可以直接下载到/usr/local/bin下面，依赖文件也不一定下载到上面那些文件夹中,vendor目录下有也行
  E. 剩下的编写代码层面参考https://www.grpc.io/docs/tutorials/basic/go/	

* 本地设置
  export GOPATH=/root/share/gRPC/
  export PATH=$PATH:$GOPATH/bin

1. 编译pb文件，进入到dir_protoc同一级的目录下
   protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    dir_protoc/xxx.proto  
    
   注意，xx.proto文件开头两行要有如下定义：  
   ```
   syntax = 'proto3';

   option go_package = "github.com/xxx/dir_protoc";
   ```

2. gRPC 定义proto方法时必须有且仅有一个参数，有且仅有一个返回
   server 实现proto中的方法是要根据生成的pb.go文件来写
   如果server的name为ABC， 那么生成的client名字就是NewABCClient. 示例如下：
   ```
   // server is used to implement VoyageServer.
	type server struct {
		pb.UnimplementedVoyageServerServer
	}

	func (s *server) CheckServer(ctx context.Context, in *pb.Host) (*pb.CheckServerResponse, error) {
		serverStatus := k8sutils.GetMasterState()
		if serverStatus == k8sutils.MASTER_LEADER {
			return &pb.CheckServerResponse{Status: true}, nil
		}
		return &pb.CheckServerResponse{Status: false}, errors.New("The server is not master or state error.")
	}
   ```

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
### 数组和切片
 - 数组array是固定长度，初试化：
 ```
  var a [10]int
  a[0] = 120
  primes := [6]int{2, 3, 5, 7, 11, 13}
 ```  
 - 切片slice长度不固定，比数组更常用
 ```
  var s1 []int
  var s2 []int = primes[1:4]
 ```
  切片不存值，它类似指针指向一段数组地址，如果改变slice的元素，那么指向的数组的那部分值也会被改变。如果有其他的slice也指向该段数组，那么其他Slice的值也会被改变。
  数组的len和cap值都一样，slice的len是它的取值的个数cap却是指向数组的部分长度，从它指向的元素开始到数组的最后一个元素。
 - copy 要先申请空间
 ```
    c := make([]int, len(s2)) //相当于建立一个新的array并指向它,指定len,默认cap和len相等; c:=make([]int, 3, 5) 指定len为3,cap为5
	copy(c, s2)	
 ```  
 - slice的结构
 ```
 type Slice struct {
    Data   unsafe.Pointer     // Array pointer
    Len   int                 // slice length
    Cap     int               // slice capacity
}
 ```
### map字典（哈希表）
- 定义与初始化<br>
  map必须初始化才能赋值，它是无序的、是非线程安全的。
- key类型必须是可判等的，map、slice和函数不可以是key
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

- map 嵌套
方法一
mainMapA := map[string]map[string]string{}
subMapA := map[string]string{"A_Key_1": "A_SubValue_1", "A_Key_2": "A_SubValue_2"}
mainMapA["MapA"] = subMapA

方法二
mainMapB := make(map[string]map[string]string)
//内部容器必须再次初始化才能使用
subMapB := make(map[string]string)
subMapB["B_Key_1"] = "B_SubValue_1"
subMapB["B_Key_2"] = "B_SubValue_2"
mainMapB["MapB"] = subMapB

- map 判断是否为空
  map没有len()接口，只能用range计算程度。

- map线程安全
  map是非线程安全的，但可以使用带锁的map、syncmap（较少用）
  

### string
 - []string 初始化为 [ "aaaa", "bbb" ]
 - string 和 struct 之间的转换
 ```
 jsonparam, err := json.Marshal(structparam)
 str := string(jsonparam)

 var structparam Person
 json.Unmarshal([]byte(str), &structparam)
 ```
 - string 和 byte 之间的转换
 ```
     // string to []byte
    s1 := "hello"
    b := []byte(s1)
    
    // []byte to string
    s2 := string(b)
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

### 类似枚举型
```
const (
	_ ValueType = iota
	CounterValue
	GaugeValue
	UntypedValue
)

type DeltaType string
// Change type definition
const (
   Added   DeltaType = "Added"
   Updated DeltaType = "Updated"
)
```

### 计算函数运行时间
可以在函数开始添加如下代码
```
    start := time.Now()
	defer func() {		
		klog.V(4).Infof("syncProxyRules took %v", time.Since(start))
	}()
```

### 等待一个random时间
```
r := rand.New(rand.NewSource(time.Now().UnixNano()))
    delaySeconds := r.Intn(100)
    time.Sleep(time.Second * time.Duration(delaySeconds))
```

## golang中func的特殊用法
### 回调函数
函数的参数A的参数是一个参数B，那么A在执行的过程中可以调用B. 如下所示, getPools函数在运行时调用filtered函数就是上面getPools参数中的函数体。
```
func (p poolAccessor) GetEnabledPools(ipVersion int) ([]v3.IPPool, error) {
	return p.getPools(func(pool *v3.IPPool) bool {
		if pool.Spec.Disabled {
			log.Debugf("Skipping disabled IP pool (%s)", pool.Name)
			return false
		}
		if _, cidr, err := net.ParseCIDR(pool.Spec.CIDR); err == nil && cidr.Version() == ipVersion {
			log.Debugf("Adding pool (%s) to the IPPool list", cidr.String())
			return true
		} else {
			log.Debugf("Ignoring IPPool: %s. IP version is different.", pool.Spec.CIDR)
		}
		return false
	})
}

func (p poolAccessor) getPools(filter func(pool *v3.IPPool) bool) ([]v3.IPPool, error) {
	pools, err := p.client.IPPools().List(context.Background(), options.ListOptions{})
	if err != nil {
		return nil, err
	}
	log.Debugf("Got list of all IPPools: %v", pools)
	var filtered []v3.IPPool
	for _, pool := range pools.Items {
		if filter(&pool) {
			filtered = append(filtered, pool)
		}
	}
	return filtered, nil
}
```

### 函数的重命名
```
type AllocatorFactory func(max int, rangeSpec string) (Interface, error)
```
这样就可以用AllocatorFactory类型的变量来表示函数

## interface 接口
interface是golang中实现多态的方法，interface定义了几个接口，实现所有接口的变量都叫做该接口的实现. interface的实现一般都有多个，根据需要的具体类型来调用具体方法。如果interface添加一个新接口，那么他的所有实现都有添加该新接口的实现函数。具体实现接口的调用不是隐式的，看它声明时是那种类型就调用该类型的实现函数。例如，如下：
```
type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rect struct {
	width float64
	height float64
}

type Circle struct {
	radius float64
}

func (r Rect) Area() float64 {
	return r.width * r.height
}

func (r Rect) Perimeter() float64 {
	return 2 * (r.width + r.height)
}

func (c Circle) Area() float64 {
	return math.Pi * c.radius * c.radius
}

func (c Circle) Perimeter() float64 {
	return 2 * math.Pi * c.radius
}

func main() {
	var rect Shape
	rect = Rect{2.0, 3.0}
	s := rect.Area()
	l := rect.Perimeter()
	fmt.Println(s, l)
}
```

## 匿名函数
匿名函数又叫闭包，闭包的参数是引用传递。如下代码，虽然遍历的IP地址应该不一样但是到了func函数里面whiteIP都是遍历到的最后一个值。
```
 for _, whiteIP := range whiteBlackIPs.WhiteIPList {
                // ping whiteIP
                log.Infof("111 test white ip %s", whiteIP)
                go func() {
                    defer wg.Done()
                    log.Infof("222 test white ip %s", whiteIP)
                    canPing, err := pingHost(whiteIP)
                    if err != nil {
                       log.Warningf("----------- Cannot check %s network connection", whiteIP)
                    }
                    if canPing == false {
                       log.Errorf("--------- The white IP %s cannot ping", whiteIP)
                       lock.Lock()
                       alertWhiteIps = append(alertWhiteIps, whiteIP)
                       lock.Unlock()
                    } else {
                       log.Debugf("--------- The white IP %s can ping", whiteIP)
                    }
                }()

            }
```

## 循环
### 死循环
for {}

### 带跳出标准的循环
```
loop:
	for {
		select {
		case <-stopCh:
			return errorStopRequested
		case err := <-errc:
			return err
		case event, ok := <-w.ResultChan():
			if !ok {
				break loop
			}
		default:
				utilruntime.HandleError(fmt.Errorf("%s: unable to understand watch event %#v", r.name, event))
		}
	}
```

## 版本管理工具go mod
### 一般情况下一个工程只需要有一个go.mod文件，如果子文件夹里面也有go.mod文件那么这个子文件夹会到子目录中去寻找目标文件可能会报错
### 使用go mod工具下载依赖文件命令
```
$ //set GOPATH
$ export GO111MODULE=on //go 1.15默认打开
$ export GOPROXY=https://goproxy.cn
$ go mod tidy
$ go mod vendor
```
### 如果没有vendor目录，那么go mod会去GOPATH下找pkg/mod中找文件依赖

## 同步与并发
go race detector工具用于检测是否符合并发要求
### mutex
可以定义为匿名mutex,无需初始化直接使用，一般写在需要保护的变量前面
```
type Counter struct {
	sync.Mutex
	Count uint64
}

var c Counter
c.Lock()
.....
c.Unlock()
```

## heap 优先队列
虽然叫队列，但不是FIFO的队列，而是一个小顶堆。插入push一个元素后，pop出来的是经过处理的最小的堆顶的元素。需要实现pop和push interface.

## 常用第三方库
k8s.io/client-go 库里面有fifo队列, 集合sets, threadSafeMap, 延迟队列delaying_queue, 限速队列rate_limiting_queue等实现代码. 其中限速队列包含一个deplaying_queue和一个限速器Limiter, golang 在 golang.org/x/time/rate 实现了一个通用(令牌)桶限速器。

## 垃圾回收
go源代码/usr/local/go/src/runtime/mgc.go 初始化垃圾回收器gcinit();该文件中还有清理实现函数gcSweep(), /usr/local/go/src/runtime/malloc.go 中的 mallocgc 函数会在分配内存后检查是否需要gc(标志量shouldhelpgc)，如果满足条件就gc: 当申请的大小大于32KB时在heap上申请，需要GC；另外如果超过2分钟没有触发gc,则需要gc;还有其他条件需要gc详见代码。

## 正则表达式
可以参考golang源码包中的regexp文件夹以及http://c.biancheng.net/view/5124.html  
### 匹配字符串
```
1. 表示字符串中至少有一个getsockname和至少一个accept，后面可以再有getsockname和accept也可以没有 花括号表示最少重复的次数. 如下表示的字串必须是连续出现的，四个单词中间不能有别的字符！
re := regexp.MustCompile("(getsockname){1}(accept){1}(getsockname){0}(accept){0}")
str := "getsocknameacceptgetsocknameacceptstatopenstatclose"
if (re.MatchString(str) == true) {
    fmt.Println("Matched")
}

2. 匹配类似openat[3][/home/config.properties]read[6]close[6]这样的字符串，其中包含文件目录名称该名称可以出现.或者- 另外中间的read可以重复出现1-100次。用如下的代码还可以读出中括号里的变量。

    var validID = regexp.MustCompile(`openat\[(?P<fd>\w+)\]\[(?P<filename>\/(\w+\/?\.?\-?)+)\](read\[(?P<fd>\w+)\]){1,100}close\[(?P<fd>\w+)\]`) 
    allstrs := validID.FindAllString("openat[89]openat[23][/usr/local/tomcat/webapps/NTczfs/WEB-INF/lib/struts2-core-2.5.26.jar]read[23]close[23]close[67]statlsrmopenat[11][/etc/sshd.config]read[11]read[11]close[11]", -1)
    for k,str := range allstrs {
        fmt.Println("k: ", k, " str: ", str)
        subs := validID.FindStringSubmatch(str)
        fmt.Println("length is ", len(subs))        
        fmt.Println("subs is ", subs)
        length := len(subs)
        if len(subs) == 6 {
            // subs[0] is the whole matched string, subs[1] is openat fd, subs[2] is openat filename,
            // subs[length-2] is read fd, subs[length-1] is close fd
            if subs[1] == subs[length-2] && subs[1] == subs[length-1] {
                fmt.Println("Found read ", subs[2], " behavior!")
            }
        }
    }

3. 可以给指定格式的字符变换格式，如下所示
func main() {
    content := []byte(`
	# comment line
	option1: value1
	option2: value2

	# another comment line
	path: /home/sjtest
`)

    pattern := regexp.MustCompile(`(?m)(?P<key>\w+):\s+(?P<value>\/(\w+\/?)+)$`)
    // Template to convert "key: value" to "key=value" by
    // referencing the values captured by the regex pattern.
    template := []byte("$key=$value\n")
    result := []byte{}

    for _, submatches := range pattern.FindAllSubmatchIndex(content, -1) {
	// Apply the captured submatches to the template and append the output
	// to the result.
	result = pattern.Expand(result, template, content, submatches)
    }
    fmt.Println(string(result))
}

4. 172.18.开头的网段： 172.18.(.*)
```
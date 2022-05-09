1. GET信息
   curl http://192.168.31.131:8080/api/v1/ 
 或curl -X GET http://192.168.31.131:8080/api/v1/ 

2.  POST信息
   curl -X POST http://192.168.186.128:9091/slice/sj/vdhcpd -d @set.

3. Delete信息
   curl -g -i -X DELETE http://10.8.65.109:9696/v2.0/fw/firewalls/4c6e7a7d-e2e7-4986-8b1d-37ff0aff2c1d.json -H "User-Agent: python-neutronclient" -H "Accept: application/json" -H "X-Auth-Token: c9353c17798e4c7db91d45d6c4c29910"

4. 下载文件
   curl –o local_file_name http://remoter/file/url 
    curl -LO http://remoter/file/url

5. 携带body信息
   curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"reserved_ip": {"group": "rsvtest",   "app": "testconfig",   "ip_address": ["172.24.100.59"]}}' \
    http://localhost:8081/ipam/reserved-ips

curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"reserved_ip" :{"namespace": "default","group":"grouptest","app":"apptest", "ip_address": [{"ip": "172.0.0.1", "podname": "base-nginx-pod123"}]}}' \
    http://localhost:8081/ipam/reserved-ips


    curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"ip": "10.174.37.218"}' \
    http://localhost:8081/ipam/release

curl -i \
        -H "Content-Type:application/json"\
        -H "User-Agent: 9ee46d36266856c7faedb0a499533380" -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
        -X POST -d '{"handleid": "jdpprod.pintuanweb-c2591b82", "ip": "10.174.37.218"}' http://localhost:8081/ipam/release

 curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"app": "pyvisual",  "group": "cpupro",   "ip": "10.172.19.66", "handleid": "smartv.pyvisual-f81f1918", "containerid": "688ddf4dee7f4b433896cda4ed6a4b0fcfc2870d116813482397f2f190135068"}' \
    http://localhost:8081/ipam/reserve-release


// 把申请到的IP放回小池
   curl -i \
    -H "Content-Type:application/json"\
   -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"alloc_reserved_ip": {  "group": "testconfig",   "app": "group3",   "ip_address": "172.24.100.0"}}'\
    http://localhost:8081/ipam/alloc-reserved-ips

// 从大池保持IP到小池
   curl -i \
    -H "Content-Type:application/json"\
   -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"alloc_reserved_ip": {  "group": "testconfig",   "app": "group3"}}'\
    http://localhost:8081/ipam/alloc-reserved-ips

   curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -g -X GET http://localhost:8081/groupippool/gztest-group5

curl   -i -H "Content-Type:application/json"    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"    -g -X GET http://localhost:8081/version

    ///////////////
    curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{apiVersion: v1,kind: ipPool,metadata:  {cidr: "172.24.102.0/24"}}'\
    http://localhost:8081/ippool/
    //////////////

    curl -i \
        -H "Content-Type:application/json"\
        -H "User-Agent: 9ee46d36266856c7faedb0a499533380" -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
        -X POST -d '{apiVersion: v1,kind: ipam,handleid: "testIPAM222",hostname: "node-133",num4: 1, ipv4pools: ["12.0.0.0/8"]}' http://localhost:8082/ipam/autoassign

curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{"app": "apptest",  "group": "grouptest",   "ip": "172.0.0.165", "handleid": "test456", "hostname": "node-133"}' \
    http://localhost:8081/ipam/assign-reserved-ip

    curl -i \
        -H "Content-Type:application/json"\
        -H "User-Agent: 9ee46d36266856c7faedb0a499533380" -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
        -X POST -d '{"ip": "172.76.75.66", "handleid": "testIPAM222"}' http://localhost:8082/ipam/release
        



    curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -X POST -d '{apiVersion: v1,kind: ipam,"ip": "10.183.33.58","handleid": "thaifront.thaijava-f70c6ae0-3557827297-h9zgr","hostname": "A05-R16-I52-101-6002660.JD.LOCAL"}'\
    http://localhost:8081/ipam/assignip

    curl -i \
    -H "Content-Type:application/json"        -H "User-Agent: 9ee46d36266856c7faedb0a499533380"\
    -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"        -X POST -d '{apiVersion: v1,kind: ipam,handleid: "testIPAM222",hostname: "node-133",num4: 1, "ipstoragetype": "bitmap", "namespace": "bigdata", "app": "gztest", "group": "group3",ipv4pools: ["172.40.0.0/26"]}'\
    http://localhost:8081/ipam/preautoassign

 curl -i \
    -H "Content-Type:application/json"\
    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
    -g -X GET http://localhost:7071/ipam/192.168.0.1

    curl -i \
        -H "Content-Type:application/json"\
        -H "User-Agent: 9ee46d36266856c7faedb0a499533380" -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"\
        -X POST -d '{"ip": "10.177.128.116", "mac": "22:13:87:b8:46:c7"}' http://localhost:8083/ipam/update-ipinfo


    scp calico root@10.8.64.180:/opt/cni/bin/


    etcdctl ls /calico -r |grep 100.0

    etcdctl get /calico/ipam/v2/assignment/ipv4/block/172.24.100.0-24

    etcdctl get /calico/ipam/v2/assignment/ipv4/app/testconfig/groupIPPool/group3

    echo "" > /export/log/skynet/skynet-server.log

    echo "" > /var/log/messages

    kubectl get pods -o wide --namespace=gztest

    systemctl restart skynet-server

    systemctl restart kubelet

     kubectl get pods -o wide --all-namespaces

#### containerDNS
```
curl -i GET http://172.22.145.130/domain/
curl -i GET http://172.22.145.130/domain?domain=log22.jd.local
curl -i -X POST http://172.22.145.130/domain/ -d '{"Domain": "log4.jd.local","Username": "tom","Type": "CNAME","Group": "lf"}'
curl -i -X PUT http://172.22.145.130/domain/ -d '{"Id": "2efb0b71-4216-4524-991d-43dab7af28f1","Domain": "123.jd.kepler","Username": "tom","Type": "A","Group": "lf"}'
curl -i -X DELETE http://172.22.145.130/domain/ -d '{"Domain":"123test.jd.kepler"}'

curl -i GET http://172.22.145.130/record?domain=123.jd.kepler
curl -i -X POST http://172.22.145.130/record/ -d '{"Domain": "123.jd.kepler","Type": "A","Zone": "jd.kepler", "Value": "2.3.6.91","ttl": 30,"View_name": "hc","Lb_mode": null,"Lb_weight": null,"Backup_enable": null,"Check_protocol": null,"Check_port": null, "Check_uri": null,"Http_rsp_code": null,     "Http_rsp_value": null,"Http_header": null,"Username": "tom"}'
curl -i -X PUT http://172.22.145.130/record/ -d '{"Domain":"123test.jd.kepler","Id":"7a8d44d2-aba6-4149-97bb-b8169ae2fefe","Value": "3.3.39.8","Ttl":35,"Type":"A"}'
curl -i -X DELETE http://172.22.145.130/record/ -d '{"Domain":"123test.jd.kepler","Id":"389c401b-d159-4dd9-bfbb-efe47248223e","Value": "2.3.6.91"}'
```

#### 批量处理curl脚本
```
#!/bin/bash

cat ip.txt | while read line
do
    #处理每行内容 "$line"
    #curl   -i -H "Content-Type:application/json"    -H "User-Agent: 9ee46d36266856c7faedb0a499533380"  -H "X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe"    -g -X GET http://localhost:8081/version
    podname=`echo $line |awk '{print $2}'`
    namespace=`echo $line|awk '{print $1}'`
    ip=`echo $line|awk '{print $3}'`
    hostname=`echo $line|awk '{print $4}'`
    body="'{apiVersion: v1,kind: ipam,\"ip\": ${ip},\"handleid\": $namespace.$podname,\"hostname\": ${hostname}}'"
    curlCMD="curl -i -H \"Content-Type:application/json\" -H \"User-Agent: 9ee46d36266856c7faedb0a499533380\"  -H \"X-Auth-Token: 8114191fd45f44b0bb5877b7e2564cfe\" -X POST -d ${body}  http://21.0.157.13:8081/ipam/assignip"
    echo $curlCMD
    eval $curlCMD
done
```



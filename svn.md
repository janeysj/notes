### 查看目录文件
1. svn list svn://serverip/branchname/
2. svn list http://svn-server/url
3. svn list http://svn-server/url  --username name --password pwd

### 下载目录文件
1. svn checkout http://svn-server/url/branch/
2. svn co svn://192.168.1.2/branch/neutron
3. svn co  -r 2 svn://192.168.1.2/CNOS/branches/cnos-cetc  /*通过 -r 参数设定checkout出哪个版本*/
4. svn export -r 2 svn://192.168.1.2/CNOS/branches/cnos-cetc  /*通过 -r 参数设定导出出哪个版本，没有.svn文件和原版本没有关系，没有版本信息可能编译不过*/

### 编辑与提交
1. svn stat   查看本地修改
2. svn ci -m "comments"   提交本地修改
3. svn up    获取远端更新
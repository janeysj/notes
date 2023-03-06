## 配置SSH
打开终端, 直接输入命令ssh查看(一般Mac都是默认安装了)帮助

1. 输入命令ssh-keygen -t rsa 指定 rsa 算法生成密钥, 接着连续三个回车键（不需要输入密码）,然后就会生成两个文件 id_rsa 和 id_rsa.pub,而 id_rsa 是密钥,id_rsa.pub 就是公钥.这两文件默 认在~/.ssh下生成, 可在Finder中control+command+G前往（终端不要关闭,下面还要用）

2. 接着添加ssh, 我们先回到github上, 点击设置进入设置页面 点击 SSH and GPG keys ，再点击 New SSH key ，如下图，其中Title不用填，我们只需填Key里的部分（下面会说如何生成Key里所需要的东西）

3. 生成Key里所需要的东西在终端继续输入命令

4. cd ~/.ssh
cat id_rsa.pub 然后在将终端输出的信息复制到Key中，最后直接点击Add SSH key完成SSH配置，这样 你这台设备就有权向GitHub上传代码了
## github.com替代方案
有时候访问github.com太慢，可以通过码云或者github.com.cnpmjs.org来下载代码


## 代码下载，分支切换
git clone git@172.171.3.200:CLab/CNVP.git git clone git@172.171.3.200:CLab/ccf.git git clone git@172.171.3.200:OpenLab/ether.git

git clone git@source.jd.com:app/jcloud-neutron.git

git clone  -c http.proxy="http://172.18.8.7:7890" https://github.com/xxx 设置下载代理

git branch -a 查看分支 git branch 查看当前目录的所有分支

git checkout -b draft remotes/origin/draft 切换到一个新分支 'newfeature'//然后要把本地代码推送到远端git push 本地远端的名字要一样

git checkout release-1.3 切换到分支 'release-1.3'//会默认给本地仓库取一样的名字

git pull 拉下新的分支（not clone），然后才能checkout到此分支

git checkout -- filename 下载指定文件夹

git checkout 123-version 下载指定版本

在使用Tortoise Git时，推送代码提示认证失败，因为修改了密码，但是在Tortoise Git的设置里面没有找到改密码的地方，原来Tortoise Git使用Windows的凭据保存用户名密码，进入控制面板，找到Windows凭据管理删除对应密码再次下载会提示你输入新密码。

## 分支管理
### 分支删除
git branch -D newfeature        确认要删除这个分支
git push origin :oldBranchName  删除远程分支
在github的对应的仓库中到Settings，滑到Setting页面的底布，找到"Danger Zone"的最后一个子项，删除库 //删除仓库，所有关于该库的信息就都删掉了

git config --list 查看config信息 或者 git config -l git remote -v 查看git远端服务器信息

### 分支创建与重命名
可以到github的repository-branches目录下创建分支
git branch -m oldBranchName newBranchName  重命名本地分支并推送重命名分支
git push origin newBranchName 将重命名过的分支提交
git clone https:github.com/... $GOPATH/src/google.golang.org/...  git clone下代码重命名
新建分支一般是在代码托管网页上新建一个分支，注意分支属性要正确。
注意：如果拉分支，先提交新创建的分支，然后再修改代码，否则如果先修改代码，再提交新创建的分支就会是提交的只是分支，修改的代码并没有被提交，还得再次修改提交。

## 代码提交
提交代码要创建两个库，远程库和本地库
1. 先建立远程库
点击登陆后页面上绿色按钮 New repository或者如下图的New repository新建一个远程仓库（remote repository） 点击后会到如下页面，要填三个地方 Repository name 库名（ 最好用英文）
Description 描述（可不填）

Initialize this repository with a README 前面打?

最后点击Create repository生成远程仓库

2. 开始建立本地库，在终端继续输入
cd到目标文件夹。

git init （在本机上想要创建一个新的git仓库）

git add -A

git remote add origin xxxxxxxxx xxxxxx 就是你仓库的地址，具体的地址可以去Github上copy。关联远程仓库。如下图获取仓库地址（选中部分）

git commit -m “firstCommit” 中断会打印所有文件信息

git pull --rebase origin master 更新远程更新到本地：

推送本地更新到远程：

git push origin master（git push -u origin master） 将本地repo于远程的origin的repo合并，第一次用-u，系统要求输入账号密码。

git push （上传add的代码）

去Github上面检查代码，已经上传成功。
一次提交示例：
```
2149 git status -uno
2150 git commit -a -m "Fix vlink del bug and lock sync"
2151 git log
2152 git push
2153 git pull
2154 git push
2155 git config --global push.default simple
2156 git push
```

## 撤销更新
* git checkout filename.c      恢复文件filename.c到本地仓库的最新版本
* git reset --hard <commit_id> 撤销commit致commit_id版本,然后用命令git push -f 推送到远端（一定要谨慎，留有文件备份）

## 查看git log
* git show commit_id//查看某次提交更改内容
* git log
* git log --author=shen.juan
*  git log --pretty=oneline MessageItem.java 查看某个文件的修改历史
* git log -n 1 --stat 想看到最近一次提交所有更改过的文件,对应* 命令
* git log -n 1 -p 想看到最近一次提交所有更改的细节,对应命令
* git shortlog
* git whatchanged --author=shen.juan 查看指定作者提交了哪些* 文件
* git diff
* git diff 文件名 git diff 83f5ca6637280eecbb654eea1b78abef27e1c85c:filename 07d03e76072b0533e147425c8d259eef3cb0b44f:filename

## 文件添加、删除和重命名
* git add dirname
* git rm dirname <br>
  /bin/rm -fr dirname
* git mv file_from file_to
* 忽略文件属性的改变
   git config --add core.filemode false 
  

## 冲突解决方法
1. 如果希望保留生产服务器上所做的改动,仅仅并入新配置项, 处理方法如下:
git stash // 暂存
git pull
git stash pop 然后可以使用git diff -w +文件名 来确认代码自动合并的情况.
2. 如果希望用代码库中的文件完全覆盖本地工作版本. 方法如下,其中commit_id为希望回退到的commit_id git reset --hard <commit_id> git pull 其中git reset是针对版本,如果想针对文件回退本地修改,使用 git checkout HEAD file/to/restore
git checkout -- file.name //撤销本地文件的修改

用rebase合并主干的修改，如果有冲突在此时解决 $ git rebase master

回到主分支 $ git checkout master

合并工作分支的修改，此时不会产生冲突。 $ git merge work

提交到远程主干 $ git push

删除远端分支remotes/origin/123 $ git push origin --delete 123

这样做的好处是，远程主干上的历史永远是线性的。每个人在本地分支解决冲突，不会在主干上产生冲突。 最后用git branch -r -d origin/branch_name或者git push origin :branch-name删除远程分支，用git branch -D branch_name删除本地分支

如果用TortoiseGit工具那么先切换至需要合并到的主分支，merge菜单，弹出的对话框选择需要合入的分支，然后push

修改git配置文件：工程目录下.git/config文件
## fork出来的分支和主master分支间的操作
在web页面上fork出一个project，然后git clone下来。这样不会多出任何分支，代码管理方便。修改完成后提交更改到master需要到页面上点击merger request, 管理员同意后就提交完成。

如果有别人也从master fork一个project并提交了修改，那么使用如下方法把更改同步到本人fork不来的工程

必须配置过 remote，指向上游仓库 。 git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git 2）切换当前工作路径至你的本地工程 3）从上游仓库获取到分支，及相关的提交信息，它们将被保存在本地的 upstream/master 分支 git fetch upstream 4）切换到本地的 master 分支 5）把 upstream/master 分支合并到本地的 master 分支，本地的 master 分支便跟上游仓库保持同步了，并且没有丢失你本地的修改。 git merge upstream/master 提示：同步后的代码仅仅是保存在本地仓库，记得 push 到 Github 哟。
把本地文件上传到github上 一、在github官网上注册并登录自己的账号。 我们都知道github上不是随便都能上传代码的，而是通过一种网络协议————SSH来授权的, SSH是用于计算机之间加密登录的网络协议。目前是每一台Linux电脑的标砖配置。而大多数Git服务器都会选择使用SSH 公钥来进行授权，所以想要在GitHub上提交代码的第一步就是要先添加SSH key 配置。接下来进行第二步。

## commit 相关操作
* git commit --amend 修改commit信息 修改某次提交的commit说明


* 合并多次提交
  1. git rebase -i HEAD~2选择需要忽略的commit, 然后键入wq保存退出；
  2. 如果已经push了多次提交，最后git push -f推送到远端
  注意，如果修改comment失败不能千万不要删除.git/rebase-merge否则将丢失所有修改。而是把文件目录备份一遍重新提交。

## 常见库在github中的目录
* k8s.io/client-go/informers 这种也会托管在github.com/kubernetes/client-go/informers
* golang.org/x/ 这种也会托管在github.com/golang/...

## 代理的设置与清除
```
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

git config --global --unset http.proxy
git config --global --unset https.proxy
```

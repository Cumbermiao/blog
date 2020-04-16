---
title: 'git 高级'
date: 2019-02-19 00:00:00
tags: [git]
published: false
hideInList: false
feature: 
isTop: false
---

### 分支
在进行提交操作时，Git 会保存一个提交对象（commit object）。

git branch 查看当前分支
 git remote show remote-name 查看远程仓库详情，包含分支信息
git branch -m origin_name new_name 修改分支名称
git checkout branch-name 创建/切换到某个分支上
git checkout -b branch-name 创建分支并切换过去

#### 删除分支
```git branch -d branch-name```

一般删除分支前，该分支需要已经合并到其他分支上，因为它包含了还未合并的工作，删除会提示  error: The branch 'branch1' is not fully merged.
如果真的需要删除使用 -D 进行强制删除。

```git push origin --delete branch-name``` 删除远程分支

#### 合并
```git merge branch-name``` 合并分支，冲突的地方需要修改
使用 rebase 命令则是用当前分支的内容覆盖 master的内容，感觉风险很大。首先分支要 git rebase master ,其次切到 master 要 git merge branch1。
合并之后，发现同一个文件 branch1 中的内容覆盖了 master的内容。

### 搭建远程仓库

#### 创建虚拟机
1. 使用 vituralBox 创建一个 Linux 服务器(Ubuntu)，注意网络 主机要能与虚拟机 互通，我网络使用的是 桥接模式，详细区别上网搜。

#### 上传秘钥&免密登录
> 如果本地有多个秘钥，推送时可能会失败，需要手动将秘钥添加到 ssh-agent
```bash
//本地生成密钥对
ssh-keygen -t rsa -C "起个任意的名字" -f "任意名字_rsa"

//ssh-agent 添加秘钥
ssh-agent bash
eval("${ssh-agent -s}")
ssh-add ~/.ssh/id_rsa

cd ~/.ssh/
ssh-copy-id -i id_rsa miao@192.168.26.237

//检查 authorized_keys 是否添加了上传的公钥，没有则手动添加
cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys

//添加之后，可以使用秘钥免密登录
ssh -i id_rsa miao@192.168.26.237

//本机新增 .ssh/config
Host miao     //起个别名
User miao     //远程服务器的用户名
HostName 192.168.26.237       //远程服务器的ip或者域名
IdentityFile ~/.ssh/id_rsa      //生成的私钥的所在目录
Port 22             //远程服务器的端口号，默认是22,这一行也可以不要
后面的几行照抄
Protocol 2
Compression yes
ServerAliveInterval 60
ServerAliveCountMax 20
LogLevel INFO

//配置完之后可以使用别名免密登录
ssh miao

```

#### 服务器创建仓库
```bash
// 使用 git init --bare  创建空仓库
mkdir repo.git && cd repo.git
git init --bare

//修改 repo.git 文件夹的用户权限，权限不够 push 时会报错
chmod repo.git 777 -R

```
#### 本地添加远程仓库，测试push
> 注意本地如果是在分支上开发，需要将分支推到远程仓库上

```bash
mkdir test && cd test
git init
touch readme
vi readme
git add .
git commit -m "init"
git remote add repo git@192.168.26.237:/home/miao/repo.git
git push repo master
```

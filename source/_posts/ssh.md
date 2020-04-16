---
title: ssh命令
date: 2019-06-21 10:28:45
tags:
---

## SSH
SSH 是一种通用的、功能强大的、基于软件的网络安全性解决方案。
术语：
- SSH 泛指SSH协议或者产品名为SSH的产品
- SSH-1 SSH 协议版本1 ， SSH-1 SSH 协议版本2
- SSH1 实现SSH-1协议的软件 ， SSH2 实现SSH-2协议的软件
- ssh 运行安全终端回话和远程命令的客户端程序
- OpenSSHift openBSD项目的产品，同时实现了 SSH-1协议和SSH-2协议。

### ssh 登录远程
使用 user 用户登录到 www.test.com
```
ssh -l user www.test.com
```
也可以使用 user@host 代替 -l 选项
```
ssh user@www.test.com
```

### scp 传输文件
```
scp sourcefile destination
```
例如将远程中的文件拷贝到本地
```
scp user@www.test.com:readme readme
```

### known host
ssh 客户端首次碰到一个新远程主机时需要执行一些额外的工作并显示和下面类似的消息
```bash
ssh -l user www.test.com
Host key not found from the list of known hosts.
Are you sure you want to continue connecting (yes/no)?
```

### 生成秘钥对
```bash
ssh-keygen
```

### ssh 代理
用户每次使用公钥认证运行 ssh 或者 scp 命令都需要重新输入口令， 这种反复输入口令的操作会让人很反感， 为了只需要一次认证就可以让 ssh 和 scp 记住你的身份直到用户退出， ssh 代理就提供了这个功能。

代理是一个程序，他可以把私钥保存在内存中， 并使用其来为 SSH客户端提供认证服务。如果用户在登录回话时启动了一个代理，那么以后的认证都不需要再重新输入密码， 知道用户结束代理为止。这个代理程序就是 ssh-agent 。

将秘钥装入到代理中，可以使用 ```ssh-add id_rsa``` , 查看代理中的所有秘钥 ```ssh-add -l``` , 从代理中卸载一个秘钥 ```ssh-add -d id_rsa``` , 卸载代理中的所有秘钥 ```ssh-add -D``` .

### 代理转发
当我们从本地执行命令 ```scp user@www.test1.com:readme user@www.test2.com:readme``` 复制文件时会失败。
当用户在本地机器运行 scp 命令时， 他连接到 www.test1.com 并间接调用了另一个 scp 命令执行拷贝工作 ， 第二个 scp 命令也会要求秘钥出入口令， 由于远程主机上没有终端回话提示， 导致执行失败。 S
SH代理可以解决以上问题， 第二个 scp 命令只需要查询用户本地的 SSH 代理， 这样就不需要输入口令了。
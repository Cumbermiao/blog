---
title: 'gitlab-ce安装'
date: 2019-02-21 14:28:06
tags: [git]
published: false
hideInList: false
feature: 
isTop: false
---

## VirtualBox 虚拟机（centos7） 安装 gitlab-ce

#### centos7 网络配置
virtualBox 安装 centos 7 步骤省略，注意分配内存时至少 2g , 一般推荐 4g ,不然 gitlab 可能出现 502 无法访问。

网络配置网卡选择桥接网卡，这样 虚拟机和本机 都能 ping 通，虚拟机也能访问外网，但是占用本机所在的网段。如果 yum 下载不了包，公司网络被限制的情况下，改成 网络地址转换NAT 模式，不会占用网段，不会被限制。

初始安装 centos 可能出现 yum 安装包时报错 cannot find a valid baseurl for repo 。
查看网络配置文件 
```
cd  etc/sysconfig/network-scripts 
//查看是否有配置文件 ifcfg-enp0s3 , 我刚开始没有该文件，改了网卡模式后有了。
//修改配置

ONBOOT=yes
BOOTPROTO=dhcp

//重启网络服务
service network restart

//测试网络
ping www.baidu.com

//ip addr 查看ip地址（en0sp3 int）,尝试 本机与虚拟机 互相ping
```

#### 设置yum镜像
yum 网络不好可以设置国内镜像
yum 要先按照 wget 包。
虚拟机网速不稳定，下载会比较慢
```
//备份 repo
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
//下载 （163地址 http://mirrors.163.com/.help/Centos7-Base-163.repo ）
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

yum 安装一些包出现 no package available, 参考 {%  link 地址 https://yq.aliyun.com/articles/548408%}
```
sudo yum install epel-release
```

yum 安装包出现的问题网上搜一下就有

#### 防火墙开启 ssh 和 http 访问
```
sudo yum install -y curl policycoreutils-python openssh-server openssh-clients
sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld

```

#### 安装 postfix ，gitlab 发送通知邮件
```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

#### 安装 gitlab-ce
可以参照 {%  link 地址 https://yq.aliyun.com/articles/240152?spm=5176.10695662.1996646101.searchclickresult.7faa1c28HYeuA0%}
```
//1 直接yum下载
yum install gitlab-ce

//设置 gitlab-ce 镜像 ， 设置之后需要 yum makecache
vim /etc/yum.repos.d/gitlab-ce.repo

[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
Repo_gpgcheck=0
Enabled=1
gpgkey=https://packages.gitlab.com/gpg.key


//2 yum下载没有包，使用 curl 下载 gitlab-ce , 使用 rpm 安装
curl -o gitlab.rmp https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-11.7.5-ce.0.el7.x86_64.rpm 
rpm -ivh gitlab-ce.rpm

//3 修改 gitlab-ce 的 host 为你虚拟机的 ip
vim /etc/gitlab/gitlab.rb

//修改 enternal_url ， 启动
external_url 'http://192.168.26.78'

sudo gitlab-ctl reconfigure
sudo gitlab-ctl start

//常用命令
sudo gitlab-ctl start # 启动所有 gitlab 组件；
sudo gitlab-ctl stop # 停止所有 gitlab 组件；
sudo gitlab-ctl restart # 重启所有 gitlab 组件；
sudo gitlab-ctl status # 查看服务状态；
sudo gitlab-ctl reconfigure # 启动服务；
sudo vim /etc/gitlab/gitlab.rb # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace # 检查gitlab；
sudo gitlab-ctl tail # 查看日志；
```
#### 结尾
到此从本机访问虚拟机 ip 就能打开 gitlab ，首次访问需要设置密码， 用户名 root ，初始密码 5iveL!fe 
注意 gitlab 默认是 80 端口，可以自己修改。


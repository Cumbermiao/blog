---
title: 'git 多账号管理'
date: 2020-04-17 00:00:00
tags: [git]
published: false
hideInList: false
feature: 
isTop: false
---

参考文章：
- [SSH Config 那些你所知道和不知道的事](https://deepzz.com/post/how-to-setup-ssh-config.html)

## 前言

之前研究 git 多账号管理， 每次都不成功， 这次换了笔记本又出现问题了， 这次不搞好绝不罢休。

### 问题

之前问题在于在 ssh config 中配置了密钥， 但是打开 ssh 时 ，并没有自动加上 identity 。
每次拉代码时都需要 `eval \`ssh-agent\`` 启动客户端， 再使用 `ssh-add ~/.ssh/github_rsa` 将所有 identity 加到客户端中。


## 解决步骤

1. .ssh/config 配置

```
# github
Host github
HostName gitub.com
User git
AddKeysToAgent yes
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_rsa
```

2. git clone 克隆仓库
`git clone github:Cumbermiao/blog.git `

最终步骤就如上所示， 但是对于 `AddKeysToAgent` 本应该在 `ssh-agent` 启动时自动添加到 `identity` 中， 但实际查看时并没有 identity 。 
对于 `User` 来说正常时 `git` ， 之前以为时 `git config` 里的 `use.name` ，但实际应该时登录仓库的用户名 `user@github.com`。
对于 git 仓库来说，使用多账户之后 remote url 需要将 `git@github.com` 替换为 `github`。
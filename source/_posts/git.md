---
title: git 基础
date: 2019-02-19
tags:  git
---
本文参照{% link 文章 https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E8%8E%B7%E5%8F%96-Git-%E4%BB%93%E5%BA%93 %}

### git 初始准备

##### Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：
- /etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用带有 --system 选项的 git config 时，它会从此文件读写配置变量.
- ~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户。 可以传递 --global 选项让 Git 读写此文件。
- 当前使用仓库的 Git 目录中的 config 文件（就是 .git/config）：针对该仓库。

##### 初次使用 git 时， 需要设置下面几个信息。

1. 设置用户信息

第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改。
```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```
如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 --global 选项的命令来配置。

2. 检查配置信息

使用 git config --list 查看所有配置 ， 你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：/etc/gitconfig 与 ~/.gitconfig）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。

使用 git config <key> 查看指定 key 的值。

3. 设置编辑器（可选）

### 获取仓库
>有两种取得 Git 项目仓库的方法。 第一种是在本地创建 Git 仓库或者将现有目录下导入所有文件到 Git 中； 第二种是从一个服务器克隆一个现有的 Git 仓库。

#### 本地创建仓库

在本地创建仓库，首先进入项目文件夹， 使用 git init 初始化，初始化会创建 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件。  

如果你文件夹中已经有文件了， 需要使用 git add 命令来实现对指定文件的跟踪，然后执行 git commit 提交。

#### 克隆远程仓库

克隆远程仓库使用 git clone [url] 命令， 注意 git clone 默认配置下远程 Git 仓库中的每一个文件的每一个版本都将被拉取下来。

Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 


### 记录每次更新

仓库中的文件只有两种状态 已追踪 和 未追踪。工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。
{% img [class names] https://git-scm.com/book/en/v2/images/lifecycle.png  [文件变化图] %}

#### 查看文件状态
 git status 会显示当前仓库中 未被追踪的文件（空文件夹不会显示）， 已追踪的文件(add 但是未 commit,暂存状态)。

 只要在 Changes to be committed 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。 你可能会想起之前我们使用 git init 后就运行了 git add (files) 命令，开始跟踪当前目录下的文件。 
 
 git add 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

 Changes not staged for commit ，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 git add 命令。

 git status -s/--short 输出标记：
 - ?? : 新添加的未跟踪文件
 - A : 新添加到暂存区中的文件
 - M : 修改过的文件
 - MM : 左边的 M 表示文件已修改，右边的 M 表示未暂存
 - AM : 类似 MM ， 左边表示新增未跟踪文件， M 表示修改但是未暂存。即 add 之后 有修改但是未再次 add。

#### 查看已暂存和未暂存的修改
 git diff 查看当前未暂存的（未add）的文件内容的修改（与）
 git diff --staged/--cached 查看已经暂存（add）的文件内容的修改

#### 提交暂存内容
 git commit 会进入编辑器书写本次提交说明。
 git commit -m '' 直接在命令行中书写说明，跳过进入编辑器操作。
 git commit -a 自动暂存所有已跟踪的文件进行提交。

#### 删除文件
 如果要删除 git 中某个文件，使用 git rm 暂存区的文件， 改命令还会删除 本地的文件。
 如果要保留本地文件，但是要删除 git 中的该文件使用 git rm --cached , 之后使用 git status 你可以看见本地文件处于未跟踪状态而仓库中该文件已经删除。

#### 移动文件
 使用 git mv from_file to_file 可以对文件重命名 ， 其实这条命令相当于 mv + git rm + git add 三条命令的合集。

#### 查看历史提交
git log 会按提交时间列出所有的更新
```git log -p -n```  -p 显示每次提交的内容差异， -n 表示比较最近 n 次提交
```git log --pretty=format```相关格式参照上面的文章

### 撤销操作

#### 撤销提交
```git commit --amend``` 重新对上一次提交进行提交， 如果上一次提交后没有修改则直接修改 commit 信息，如果上一次后有修改的内容需要先暂存，改命令会提交暂存内容并修改commit 信息。

如果你 add * 暂存了错误的文件并且已经 commit 了， 可以使用 ```git rm``` + ```git commit --amend``` 来删除 git 中的错误文件，因为 rm 删除了暂存区的文件，amend 重新提交会提交暂存区的内容，所以就会覆盖上次 commit。

#### 撤销暂存文件
```git reset HEAD filename ``` 撤销暂存区中的文件

#### 撤销文件修改
```git checkout -- filename``` 撤销文件改动，注意使用该命令无法恢复之前文件的改动

### 远程仓库

#### 添加远程仓库
```git remote add <remote-name> <url>``` 新增远程仓库，remote-name 缩写别名
仓库可能会有多个，一般第一次新增的为origin。添加了远程仓库可以拉取数据。

#### 拉取数据
```git fetch [remote-name]``` 这个命令会访问远程仓库，从中拉取所有你还没有的数据。 执行完成后，你将会拥有那个远程仓库中所有分支的引用，可以随时合并或查看。

#### 推送到远程仓库
```git push [remote-name] [branch-name]```  只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效。

#### 查看远程仓库
```git remote show [remote-name]``` 查看远程仓库信息

#### 远程仓库的移除与重命名
```git remote rename origin_name new_name```  修改仓库的 remote_name
```git remote rm [remote-name]``` 删除远程仓库

### 标签

#### 创建标签
Git 使用两种主要类型的标签：轻量标签（lightweight）与附注标签（annotated）。
轻量标签很像一个不会改变的分支 - 它只是一个特定提交的引用。
附注标签是存储在 Git 数据库中的一个完整对象。 它们是可以被校验的；其中包含打标签者的名字、电子邮件地址、日期时间；还有一个标签信息；并且可以使用 GNU Privacy Guard （GPG）签名与验证。 通常建议创建附注标签，这样你可以拥有以上所有信息。

```git tag tag-name``` 创建轻量标签
```git tag -a tag-name``` 创建轻量标签
```git tag tag-name commit-id``` 可以给之前commit的记录打tag

一般情况下， git push 命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样 - 你可以运行 ```git push origin [tagname]```。

#### 删除标签
```git tag -d tag-name``` 删除标签

一般情况下，我们不应该直接检出标签，```git checkout tag-name``` 检出标签之后，如果进行了一些修改并且提交，标签不会发生变化，但你的新提交将不属于任何分支，并且将无法访问，除非确切的提交哈希。
因此，如果你需要进行更改通常需要创建一个新分支```git checkout tag-name -b ```.

### 设置命令别名
```git config --global alias.co checkout``` 设置 checkout 别名为 co ， ```git co``` 效果与 ```git checkout``` 一样。
对于一些常用的命令，我们可以进行简写，对于一些命令可以进行语义化的书写，如：
- 取消暂存文件 ```git config --global alias.unstage 'reset HEAD --'```

如果要执行外部命令，而不是一个 Git 子命令。 如果是那样的话，可以在命令前面加入 ! 符号。  如果你自己要写一些与 Git 仓库协作的工具的话，那会很有用。 
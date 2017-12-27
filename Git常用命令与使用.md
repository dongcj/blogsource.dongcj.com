---
title: git 常用命令与使用
author: dongcj <ntwk@163.com>
date: 2016/04/17 17:44:33
updated:
tags:
 - git
categories:
  - linux
---

> 参考：http://www.jianshu.com/p/f7b5431418d2

# 常见问题
## 使用 key 方式认证
```bash
# start the ssh-agent in the background
eval "$(ssh-agent -s)"
    Agent pid 59566

ssh-add ~/.ssh/id_rsa

# 测试
ssh git@github.com
```

## Windows 下 tortiseGit 不弹出密码输入
    # 删除以下文件夹中的文件：
    %appdata%\..\Local\Microsoft\Credentials

## 同一网站，多个不同的帐号使用 git key 切换

### 安装时使用 ssh 而不是 plink 
在安装时指定 ssh 

### 取消 global 的 email
    git config --global --unset user.name
    git config --global --unset user.email

### 设置项目级自己的 email
    git config  user.email "xxxx@xx.com"
    git config  user.name "suzie"

### 使用 ssh config
> http://memoryboxes.github.io/blog/2014/12/07/duo-ge-gitzhang-hao-zhi-jian-de-qie-huan/

    vi ~/.ssh/config

```bash
# 在主机 lb.gogs.pro.svi.pub 上的 svicloud 用户
host svicloud.lb.gogs.pro.svi.pub
    hostname lb.gogs.pro.svi.pub
    Port 22
    User svicloud
    IdentityFile ~/.ssh/id_rsa_first

# 在主机 lb.gogs.pro.svi.pub 上的 svicloud 用户
host svicloud.lb.gogs.pro.svi.pub
    hostname lb.gogs.pro.svi.pub
    Port 22
    User svicloud
    IdentityFile ~/.ssh/id_rsa_first
```

## 清除 git 路径中的所有 .git 文件

    $ find . -name ".git" | xargs rm -Rf

## 打包下载 git 文件中目录
    $ git archive --remote=${GIT_REPO} latest www | tar xvf - -C /tmp


# 常用配置
## 配置认证、支持中文

    # 配置用户名和 email
    $ git config --global user.name "dongcj"
    $ git config --global user.email "ntwk@163.com"
    $ git config --global core.autocrlf true

    $ git config --list


## 重新设置 git url 或 新增远程仓库
    
    # 重新设置远程仓库地址
    $ git remote set-url origin git@github.com:dongcj/blog.git

    # 提交到多个远程仓库
    $ git remote add origin git@github.com:dongcj/blog.git



# git 日常用法

```sh
git status                                                # 查看当前版本状态（是否修改）
git ls-files                                              # 列出 git index 包含的文件

git add xyz                                               # 添加 xyz 文件至 index
git add .                                                 # 增加当前子目录下所有更改过的文件至 index

git commit -m 'xxx'                                       # 提交
git commit --amend -m 'xxx'                               # ` 合并 ` 上一次提交（用于反复修改）
git commit -am 'xxx'                                      # 将 add 和 commit 合为一步

git rm xxx                                                # 删除 index 中的文件
git rm -r *                                               # 递归删除
```

## git log
```bash
git log                                                   # 显示提交日志
git log -1                                                # 显示 1 行日志 -n 为 n 行
git log -p test.html                                      # 显示 test.html 的变更日志
git log --stat                                            # 显示提交日志及相关变动文件
git log -p -m                                             # 显示每个文件修改记录
git log v2.0                                              # 显示 v2.0 的日志
git log --pretty=format:'%h %s' --graph                   # 图示提交日志
git reflog                                                # 显示所有提交，包括孤立节点
# 显示最近 20 条提交记录
git --no-pager log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr [%an])%Creset' --abbrev-commit --date=relative  -20
```

> 更多格式化输出 : https://ruby-china.org/topics/939


## git show
```php
git show dfb02e6e4f2f7b573337763e5c0013802e392818         # 显示某个提交修改的详细内容
git show dfb02                                            # 可只用 commitid 的前几位
git show HEAD                                             # 显示 HEAD 提交日志
git show v2.0                                             # 显示 v2.0 的日志及详细内容
git show HEAD^                                            # 显示 HEAD 的父（上一个版本）的提交日志 ^^ 为上两个版本 ^5 为上 5 个版本
git show-branch                                           # 图示当前分支历史
git show-branch --all                                     # 图示所有分支历史
git show HEAD~3
git show -s --pretty=raw 2be7fcb476
git show HEAD@{5}
git show master@{yesterday}                               # 显示 master 分支昨天的状态
```


# git diff
```bash
git diff                                                  # 显示所有未添加至 index 的变更
git diff --cached                                         # 显示所有已添加 index 但还未 commit 的变更
git diff HEAD^                                            # 比较与上一个版本的差异
git diff HEAD -- ./lib                                    # 比较与 HEAD 版本 lib 目录的差异
git diff origin/master..master                            # 比较远程分支 master 上有本地分支 master 上没有的
git diff origin/master..master --stat                     # 只显示差异的文件，不显示具体内容
```


## git branch
```bash
git branch                                                # 显示本地分支
git branch --contains 50089                               # 显示包含提交 50089 的分支
git branch -a                                             # 显示所有分支
git branch -r                                             # 显示所有原创分支
git branch --merged                                       # 显示所有已合并到当前分支的分支
git branch --no-merged                                    # 显示所有未合并到当前分支的分支
git branch -m master master_copy                          # 本地分支改名
git branch -d hotfixes/BJVEP933                           # 删除分支 hotfixes/BJVEP933（本分支修改已合并到其他分支）
git branch -D hotfixes/BJVEP933                           # 强制删除分支 hotfixes/BJVEP933
```

## git checkout
```bash
git checkout abcde file/to/restore                        # 单一文件回退到某个版本
git checkout -b master_copy                               # 从当前分支创建新分支 master_copy 并检出
git checkout -b master master_copy                        # 上面的完整版
git checkout features/performance                         # 检出已存在的 features/performance 分支
git checkout --track hotfixes/BJVEP933                    # 检出远程分支 hotfixes/BJVEP933 并创建本地跟踪分支
git checkout v2.0                                         # 检出版本 v2.0
git checkout -b devel origin/develop                      # 从远程分支 develop 创建新本地分支 devel 并检出
git checkout -- README                                    # 检出 head 版本的 README 文件（可用于修改错误回退）
```

## git fetch
```bash
git fetch                                                 # 获取所有远程分支（不更新本地分支，另需 merge）
git fetch --prune                                         # 获取所有原创分支并清除服务器上已删掉的分支
git fetch --all                                           # 1. 获取所有远程分支
git reset --hard origin/master                            # 2. 先运行 1，再使用远程文件 <b> 强制覆盖 </b> 本地文件
git reset --hard HEAD                                     # 将当前版本重置为 HEAD（通常用于 merge 失败回退）
```


## git tag
```bash
git tag                               # git 列表
git tag v0.1.1  # 打标签
git tag v0.1.1 6224937                # 补打标签
git tag v0.1.2-light                  # 创建轻量标签 - 轻量标签是指向提交对象的引用 ( 暂时没用到 )
git tag -a v0.1.2 -m "0.1.2 版本备注 "  # 附注标签则是仓库中的一个独立对象。建议使用附注标签
git tag -a v0.1.1 9fbc3d0             # 给指定的 commit 打标签（即补打标签）
git checkou [tagname]                 # 切换标签
git tag -d v0.1.2                     # 删除标签
git push origin :refs/tags/v0.9       # 删除远程标签 ( 需要先删除本地标签 )
git push origin v0.1.2                # 提交单一标签
git push origin --tags                # 提交所有标签
git show v1.2.5                       # 查看标签内容


git tag -l | xargs git tag -d         # Delete local tags.
git fetch                             # Fetch remote tags.
git tag -l | xargs -n 1 git push --delete origin    # Delete remote tags.
git tag -l | xargs git tag -d                       # Delete local tasg.

```

## 其它命令
```bash
git merge origin/master                                   # 合并远程 master 分支至当前分支

git cherry-pick ff44785404a8e                             # 合并提交 ff44785404a8e 的修改

git push origin master                                    # 将当前分支 push 到远程 master 分支
git push origin :hotfixes/BJVEP933                        # 删除远程仓库的 hotfixes/BJVEP933 分支
git push --tags                                           # 把所有 tag 推送到远程仓库



git pull origin master                                    # 获取远程分支 master 并 merge 到当前分支

git mv README README2                                     # 重命名文件 README 为 README2

git rebase
```

---
title: 安装源_RedHat,CentOS 备用源
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:16:45
updated:
categories:
  - linux
tags:
  - yum rpmforge
  - 国内源
---

# yum 更新
```bash
yum search fastestmirror
yum install yum-fastestmirror* -y
yum update yum*
yum update kernel*
yum update
```

# 常用源
## rpmforge 源
    # ( 将链接中的 "X" 换为 7/6/5 即可下载不同版本的 repo)
    rpm -Uhv  http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el<X>.rf.x86_64.rpm
    rpm -Uhv http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el<X>.rf.i686.rpm

## epel 源 (elrepo.org)
    yum install epel-release

## elrepo(update kernel)
    rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm

## remi 源
    # ( 将链接中的 "X" 换为 7/6/5 即可下载不同版本的 repo)
    rpm -ivh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

## rpmfusion 源
    # redhat/centos 6
    rpm -Uvh http://download1.rpmfusion.org/free/el/updates/6/x86_64/rpmfusion-free-release-6-1.noarch.rpm
    rpm -Uvh http://download1.rpmfusion.org/free/el/updates/6/i386/rpmfusion-free-release-6-1.noarch.rpm
    rpm -Uvh http://download1.rpmfusion.org/nonfree/el/updates/6/i386/rpmfusion-nonfree-release-6-1.noarch.rpm

   # redhat/centos 7 only have x86_64 packages
    rpm -Uvh http://download1.rpmfusion.org/free/el/updates/7/x86_64/r/rpmfusion-free-release-7-1.noarch.rpm

# 国内镜像站
## CentOS 中国科学技术大学 USTC mirror( 每小时更新一次 )
> 地址：http://centos.ustc.edu.cn/

    cd /etc/yum.repos.d
    mv CentOS-Base.repo  CentOS-Base.repo.save-`date +%F`

### centos 7
```bash
vi USTC.repo
[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### centos 6
```bash
[base]
name=CentOS-$releasever - Base - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.ustc.edu.cn
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/contrib/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
gpgcheck=1
enabled=0
gpgkey=https://mirrors.ustc.edu.cn/centos/RPM-GPG-KEY-CentOS-6
```

## 网易开源镜像站
> 地址：http://mirrors.163.com/.help/centos.html

```bash

cd /etc/yum.repos.d
# centos6
wget http://mirrors.163.com/.help/CentOS6-Base-163.repo

# centos7
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo

# 生成缓存
yum makecache
```

# yum 排除 32 位安装
    # 删除 32 位组件：
    yum remove \*.i\?86

    # 防止 32 位元组件在未来更新时被安装
    # 编辑 /etc/yum.conf 并加入以下一行：
    exclude = *.i?86


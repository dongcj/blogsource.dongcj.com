---
title: GlusterFS 安装
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - storage
tags:
  - gluster
  - dfs
---

> 参考： https://wiki.centos.org/zh/HowTos/GlusterFSonCentOS

# 下载源
    $ wget -P /etc/yum.repos.d     http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/glusterfs-epel.repo

# 安装软件
    yum install glusterfs{,-fuse,-server}
    yum install glusterfs-geo-replication.x86_64

> 如遇到以下错误：
    Transaction check error:
    file /usr/lib/systemd/system/blk-availability.service from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package  lvm2-7:2.02.105-14.el7.x86_64
    file /usr/sbin/blkdeactivate from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64
    file /usr/share/man/man8/blkdeactivate.8.gz from install of device-mapper-7:1.02.107-5.el7_2.2.x86_64 conflicts with file from package lvm2-7:2.02.105-14.el7.x86_64

    # 删除 lvm2
    $ rpm -e lvm2

# 挂载分区及配置防火墙
    # mkdir
    $ mkdir /export/brick1

    # 这里可以修改权限 , 其它程序可以使用， 注意所有服务器上都要修改！！！
    $ chmod 777 /export/brick1

    # firewall
    $ firewall-cmd --zone=public --add-port=24007-24008/tcp --permanent

    $ firewall-cmd --zone=public --add-service=nfs --add-service=samba --add-service=samba-client --permanent
    $ firewall-cmd --zone=public --add-port=111/tcp --add-port=139/tcp --add-port=445/tcp --add-port=965/tcp --add-port=2049/tcp \
    --add-port=38465-38469/tcp --add-port=631/tcp --add-port=111/udp --add-port=963/udp --add-port=49152-49251/tcp  --permanent
    $ firewall-cmd --reload

# 启动 glusterd 服务
    $ service glusterd start

    # if centos7:
    $ systemctl enable glusterd
    $ systemctl start glusterd
    $ systemctl status glusterd

# 配置信任池

    # from server1
    $ gluster peer probe SERVER2

    # from server2
    $ gluster peer probe SERVER1

    # check peer
    $ gluster peer status

# 设置一个 Gluster 卷
    $ gluster volume create gv0 replica 2 HOSTNAME_OF_SERVER1:/export/brick1/gv0 HOSTNAME_OF_SERVER2:/export/brick1/gv0
    $ gluster volume start gv0

    $ gluster volume info
    $ gluster volume status

    # How to remove a volume or peer
    # stop Volume
    $ gluster volume stop gv0

    # delete Volume
    $ gluster volume delete gv0

    # detach 命令：
    $ gluster peer detach NAME_OF_PEERNODE

# Test Gluster volume
    # mount to /mnt
    $ mount -t glusterfs node01.yourdomain.net:/gv0 /mnt

    $ vi /etc/fstab
    gluster1.example.com:/gv0       /mnt/glusterfs  glusterfs   defaults,_netdev  0  0

    # 如果使用 nfs 协议使用
    # GlusterFS NFS 服务器只支持第 3 版的 NFS 沟通协议。
    $ vi /etc/nfsmount.conf
    Defaultvers=3

    $ systemctl restart glusterd.service

    # start force
    $ gluster volume set gv0 nfs.disable off
    $ gluster volume start gv0 force

    $ mount -t nfs PEER_NODE:/gv0 /mnt/glusterfs

# 配置 Quorum()
    ......

> 通过 CIFS 从 Linux ／ Windows 计算机进行访问
> 参考 : https://wiki.centos.org/zh/HowTos/GlusterFSonCentOS


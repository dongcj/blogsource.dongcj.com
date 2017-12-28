---
title: Rfw-RemoteFirewall 远程管理防火墙
author: dongcj <ntwk@163.com>
date: 2017/04/11 11:43:19
updated: 2017/08/11 15:44:39
categories:
  - linux
  - security
tags:
  - rfw
  - RemoteFirewall
---
# install
    > client: curl client (eg: 116.23.45.8)
    > server: rfw servers (default port: 7393)

## install rfw on ALL server(on client & server)
    $ git clone https://github.com/securitykiss-com/rfw.git
    $ cd rfw && python setup.py install

## generate remote crt(on client)
    $ cd /etc/rfw/deploy/
    $ rfwgen  11.11.11.11
    $ rfwgen  22.22.22.22

    # will generate the follow files
    .
    ├── client
    │   └── ca.crt
    ├── offline
    │   └── ca.key
    ├── server_11.11.11.11
    │   ├── server.crt
    │   └── server.key
    └── server_22.22.22.22
      ├── server.crt
      └── server.key

## copy key to server
```bash
# on master server
scp /etc/rfw/deploy/server_11.11.11.11/server.key root@11.11.11.11:/etc/rfw/ssl/
scp /etc/rfw/deploy/server_11.11.11.11/server.crt root@11.11.11.11:/etc/rfw/ssl/
scp /etc/rfw/deploy/server_22.22.22.22/server.key root@22.22.22.22:/etc/rfw/ssl/
scp /etc/rfw/deploy/server_22.22.22.22/server.crt root@22.22.22.22:/etc/rfw/ssl/
```

## edit the server config
    $ vi /etc/rfw/rfw.conf
    outward.server.certfile = /etc/rfw/ssl/server.crt
    outward.server.keyfile = /etc/rfw/ssl/server.key
    auth.username = your_username_here
    auth.password = your_password_here

    $ vi /etc/rfw/white.list
    116.23.45.8


## start rfw(on server)
    $ rfw &

    # 查看 server 的 iptables，只允许 white.list 里的服务器访问
    $ iptables -L -n
    Chain INPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     tcp  --  116.23.45.8       0.0.0.0/0           tcp dpt:7393
    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           tcp dpt:7393

    Chain FORWARD (policy ACCEPT)
    target     prot opt source               destination

    Chain OUTPUT (policy ACCEPT)
    target     prot opt source               destination
    ACCEPT     tcp  --  0.0.0.0/0            116.23.45.8      tcp spt:7393
    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0           tcp spt:7393

# test

## on client
    # block some bad IP for 5 minutes:
    $ curl -i --cacert /home/me/ca.crt --user your_username_here:your_password_here -XPUT https://11.11.11.11:7393/drop/eth/1.2.3.4?expire=5m

    # Check if the rule is present now and not present after 5 minutes:
    $ curl -i --cacert /home/me/ca.crt --user your_username_here:your_password_here https://11.11.11.11:7393/list

## 在 client 上用 web 管理所有服务
    # 未完。。。







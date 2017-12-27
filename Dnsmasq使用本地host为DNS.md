---
title: Dnsmasq 使用本地 hosts 配置提供 DNS 服务
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:16:45
updated:

categories:
  - linux
  - dns

tags:
  - dns dhcp
  - dnsmasq

---

# 使用 dnsmasq 提供外网 dns 服务
    $ apt-get install dnsmasq

## 修改 dnsmasq 主配置文件
    # 将以下文本添加到 /etc/dnsmasq.conf 文件的最后：
    listen-address=127.0.0.1,OUTER_IP      # 这里如果使用 0.0.0.0 会失败
    addn-hosts=/etc/hosts                   # 直接解析本地的 hosts
    resolv-file=/etc/resolv.dnsmasq.conf    # 可以增加本机的 dns 服务 (/etc/resolv.conf 里最多只生效三个 )
    conf-dir=/etc/dnsmasq.d                 # 该目录下的所有文件都会解析


## 生成域名映射
    # 在 /etc/dnsmasq.d/ 目录下新建一个文件，随意起个名字
    vi /etc/dnsmasq.d/dns.conf

    # 指定你要映射的域名，例如 google.com，则将下面贴进 dns.conf 文件
    address="/google.com/172.17.0.4"


## 启动服务
    /etc/init.d/dnsmasq start

## 测试
    dig google.com @OUTER_IP        -->   172.17.0.4



# 本地 hosts 配合 NetworkManager 方式
    $ cat /etc/NetworkManager/dnsmasq.d/hosts.conf
    addn-hosts=/etc/hosts

    $ sudo service networking restart
    $ sudo /etc/init.d/dns-clean restart

    # 修改后要求下面的几个命令输出相同的 IP 地址：
    $ host u1
    $ nslookup u1
    $ getent ahosts u1


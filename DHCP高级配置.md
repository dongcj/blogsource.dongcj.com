---
title: DHCP 高级配置
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - dhcp
  - pxe
---

# 一、DHCP 中继代理配置
---
    # ( 在其它服务上安装 dhcpd 软件 )：

    $ vi /etc/sysconfig/dhcrelay
      # Command line options here
      INTERFACES="eth1 eth2"
      DHCPSERVERS="192.168.1.1"

    # 也可以通过以下命令方式来实现：
    $ dhcrelay -i eth1 -i eth2 192.168.1.1

# 二、DHCP 超网配置
---

    ddns-update-style interim;      /*dhcp 支持的 dns 动态更新方式*/
    ignore client-updates;          /*忽略客户端 DNS 动态更新*/
    shared-network mynet {          /*超网作用域选项，共同部分*/
        option subnet-mask 255.255.255.0;            /*子网掩码*/
        option domain-name "koumm.net";              /*域名*/
        option domain-name-servers 192.168.1.2;      /*dns IP*/
        option broadcast-address 192.168.1.255;      /*广播地址*/
        default-lease-time 86400;                    /*租期 1 天，秒数*/
        max-lease-time 172800;                       /*最长租期 2 天*/

        subnet 192.168.1.0 netmask 255.255.255.0 {   /*1.0 子网段*/
            range 192.168.1.11 192.168.1.100;        /*ip 地址段范围*/
            option routers 192.168.1.1;               /*网关地址*/
            /*绑定 pc1 主机 ip 地址配置*/
            host pc1 {
                hardware ethernet 00:a0:cc:cf:9C:14;
                fixed-address 192.168.1.20;
            }
            /*绑定 pc2 主机 ip 地址配置*/
            host pc2 {
                hardware ethernet 04:20:c1:f8:37:11;
                fixed-address 192.168.1.30;
            }
        }

        subnet 192.168.2.0 netmask 255.255.255.0 {    /*2.0 子网段*/
            range 192.168.2.10 192.168.2.100;         /*ip 地址段范围*/
            option routers 192.168.2.1;                /*网关地址*/
        }

        subnet 192.168.3.0 netmask 255.255.255.0 {    /*3.0 子网段*/
            range 192.168.3.10 192.168.3.100;         /*ip 地址段范围*/
            option routers 192.168.3.1;                /*网关地址*/
        }

    }


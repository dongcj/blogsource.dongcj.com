---
title: DHCP高级配置
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
    # (在其它服务上安装 dhcpd 软件)：

    $ vi /etc/sysconfig/dhcrelay
      # Command line options here
      INTERFACES="eth1 eth2"
      DHCPSERVERS="192.168.1.1"

    # 也可以通过以下命令方式来实现：
    $ dhcrelay -i eth1 -i eth2 192.168.1.1

    # Linux DHCP配置完成后，重新启动DHCP服务。



# 二、DHCP 超网配置
---

    ddns-update-style interim;      /*dhcp支持的dns动态更新方式*/
    ignore client-updates;          /*忽略客户端DNS动态更新*/
    shared-network mynet {          /*超网作用域选项，共同部分*/
        option subnet-mask 255.255.255.0;            /*子网掩码*/
        option domain-name "koumm.net";              /*域名*/
        option domain-name-servers 192.168.1.2;      /*dns IP*/
        option broadcast-address 192.168.1.255;      /*广播地址*/
        default-lease-time 86400;                    /*租期1天，秒数*/
        max-lease-time 172800;                       /*最长租期2天*/

        subnet 192.168.1.0 netmask 255.255.255.0 {   /*1.0子网段*/
            range 192.168.1.11 192.168.1.100;        /*ip地址段范围*/
            option routers 192.168.1.1;               /*网关地址*/
            /*绑定pc1主机ip地址配置*/
            host pc1 {
                hardware ethernet 00:a0:cc:cf:9C:14;
                fixed-address 192.168.1.20;
            }
            /*绑定pc2主机ip地址配置*/
            host pc2 {
                hardware ethernet 04:20:c1:f8:37:11;
                fixed-address 192.168.1.30;
            }
        }

        subnet 192.168.2.0 netmask 255.255.255.0 {    /*2.0子网段*/
            range 192.168.2.10 192.168.2.100;         /*ip地址段范围*/
            option routers 192.168.2.1;                /*网关地址*/
        }

        subnet 192.168.3.0 netmask 255.255.255.0 {    /*3.0子网段*/
            range 192.168.3.10 192.168.3.100;         /*ip地址段范围*/
            option routers 192.168.3.1;                /*网关地址*/
        }

    }
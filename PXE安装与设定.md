---
title: PXE 安装与设定
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - dhcp
  - pxe
  - kickstart
---

# 安装软件

    # 安装的软件：
    dhcp-3.0.5-7.el5.x86_64.rpm
    vsftpd-2.0.5-10.el5.x86_64.rpm( 不须装 )
    xinetd-2.3.14-10.el5.x86_64.rpm
    tftp-server-0.42-3.1.x86_64.rpm
    pykickstart-0.43.3-1.el5.noarch.rpm
    system-config-kickstart-2.6.19.1-1.el5.noarch.rpm
    syslinux

    # 关闭 selinux & iptables

# dhcp 配置
    ddns-update-style interim;
    ignore client-updates;
    allow booting;
    allow bootp;
    class "pxeclients"{
    match if substring(option vendor-class-identifier,0,9) = "PXEClient";
    filename "/pxelinux.0";
    next-server 10.85.138.9;
    }
    subnet 10.85.138.0 netmask 255.255.254.0 {
    option routers 10.85.138.1;
    option subnet-mask 255.255.254.0;
    option domain-name "gwcloud.com";
    option domain-name-servers 10.85.138.1;
    option time-offset -18000; # Eastern Standard Time
    range dynamic-bootp 10.85.138.100 10.85.138.249;
    default-lease-time 21600;
    max-lease-time 43200;
    }


# 增加 tftp 文件
    拷文件至 tftpboot( 注：menu.c32 和 pxelinux.0 要从本机拷贝，否则 TIMEOUT 无效！ )
    $ cp /usr/share/syslinux/menu.c32 /var/lib/tftpboot/
    $ cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/


    $ cp /mnt/isolinux/initrd.img /var/lib/tftpboot/isolinux
    $ cp /mnt/isolinux/vmlinuz /var/lib/tftpboot/isolinux
    $ cp /mnt/isolinux/isolinux.cfg /tftpboot/pxelinux.cfg/default  # default 文件也可以自己修改

# 配置 kickstart
    # 在 /tftpboot/pxelinux.cfg/default 中添加 KS 入口
      label linux
        menu label ^Install or upgrade an existing system
        menu default
        kernel vmlinuz
        append initrd=isolinux/initrd.img ks=ftp://10.85.138.9/pub/ks.cfg

    # 修改 kickstart 自动安装 linux( 见安装文件 )



 > 附录：完整 DHCP 样例

    ddns-update-style interim;
    ddns-updates on;
    ignore client-updates;
    allow booting;  # for pxe only
    allow bootp;    # for pxe only
    authoritative;	# Do not know what is meaning??

    class "pxeclients"{
        match if substring(option vendor-class-identifier,0,9) = "PXEClient";   # for pxe
        # match if substring(option vendor-class-identifier, 0, 4) = "MSFT";    # for normal dhcp
        # option host-name = config-option server.ddns-hostname;    # set the server's hostname, only for linux???
        ddns-hostname = concat("v",
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 1, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 2, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 3, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 4, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 5, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 6, 1))), 2)
        );
        filename "/pxelinux.0";     # for pxe only
        next-server 192.168.88.1;   # for pxe only
    }


    class "MSFT" {
        match if substring(option vendor-class-identifier, 0, 4) = "MSFT";
        option host-name = config-option server.ddns-hostname;
        #ddns-hostname = concat("v", binary-to-ascii(16, 8, "", substring (hardware, 1, 6)));
        #ddns-hostname = pick (option host-name, concat("v", 	# Windows instances will apply for a hostname by default
        ddns-hostname = concat("v",
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 1, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 2, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 3, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 4, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 5, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 6, 1))), 2)
        );
    }


    option host-name = config-option server.ddns-hostname;
    ddns-hostname = pick (option host-name, concat("v",
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 1, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 2, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 3, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 4, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 5, 1))), 2),
            suffix(concat("0", binary-to-ascii(16, 8, "", substring (hardware, 6, 1))), 2)
    ));
    set vendor-string = option vendor-class-identifier;


    subnet 192.168.88.0 netmask 255.255.255.0 {
        option routers 192.168.88.1;
        option subnet-mask 255.255.255.0;
        option domain-name "gwcloud.cn";
        option domain-name-servers 192.168.88.1;
        # option host-name "Server001"      # set the server's hostname, only for linux???
        # option ntp-servers 192.168.88.1   # set the ntp server
        option time-offset -18000;          # Eastern Standard Time
        range dynamic-bootp 192.168.88.100 192.168.88.200;
        default-lease-time 21600;
        max-lease-time 43200;

        # the next is bonding the mac to static ip address
        # host pc1 {
        #    hardware ethernet 00:a0:cc:cf:9C:14;    # host's mac address
        #    fixed-address 192.168.1.30;             # host's ip address
    }











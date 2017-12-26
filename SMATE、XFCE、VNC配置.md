---
title: SMATE、XFCE、VNC配置
author: dongcj <ntwk@163.com>
date: 2016/08/30 17:27:43
updated: 2016/08/30 17:48:08
categories:
  - linux
tags:
  - xorg
  - smate
  - xfce
  - vnc
---


# YUM 安装图像界面及 VNC
( 参考：http://www.ha97.com/4634.html )

    yum install epel-release

    yum grouplist
    yum groupinstall "GNOME Desktop Environment"（CentOS 5.x安装GNOME桌面环境）
    yum groupinstall "X Window System" "Desktop"（CentOS 6.x安装GNOME桌面环境）


    yum groupinstall [ xfce | "MATE Desktop" ]（CentOS安装xfce桌面环境，可选）



    # runlevel 5
    systemctl isolate graphical.target

    # Start the GUI on boot
    systemctl set-default graphical.target


    # 切换 display manager
    # 查看
    ls -l /etc/systemd/system/display-manager.service

    # 切换
    systemctl disable gdm && systemctl enable lightdm

    # 刷新
    systemctl isolate graphical.target


# 安装 vnc server
    # 安装 Xfce 要使用 vncserver，需修改 /root/.vnc/xstartup 里为 exec /bin/sh /etc/xdg/xfce4/xinitrc 即可
    yum install vnc-server vnc* （CentOS 5.x里）
    yum install tigervnc-server tigervnc （CentOS 6.x里）
    /etc/init.d/vncserver restart

    # 关闭具体的vncserver命令:
    vncserver -kill :1

> http://jensd.be/125/linux/rhel/install-mate-or-xfce-on-centos-7
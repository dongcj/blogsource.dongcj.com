---
title: Autossh
author: dongcj <ntwk@163.com>
date: 2016/08/04 16:32:12
updated: 2016/08/04 16:32:45
categories:
  - linux
tag:
  - Linux
  - LinuxCommand
  - autossh
---

> http://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html



# 用 autossh 保证 ssh 隧道稳定性 :
    autossh -M 5122 -N -v -D localhost:8527 root@remote_ssh_server -p remote_ssh_port  



# autossh 代理
    export AUTOSSH_PIDFILE=/var/run/autossh.pid
    export AUTOSSH_POLL=60
    export AUTOSSH_FIRST_POLL=30
    export AUTOSSH_GATETIME=0
    export AUTOSSH_DEBUG=1
    autossh -M 0 -4 -fN -R 8080:127.0.0.1:80 -o "ServerAliveInterval 60" \
        -o "ServerAliveCountMax 3" -o BatchMode=yes \
        -o StrictHostKeyChecking=no USER@HOSTNAME









---
title: SSH_config_proxycommand
author: dongcj <ntwk@163.com>
---

# ProxyCommand
```bash
# 直接跳到远程计算机
ssh -o "ProxyCommand ssh -p 1098 lmx@proxy.machine nc -w 1 %h %p" -p 1098 lmx@target.machine

# 拷贝文件到远程计算机
scp -o "ProxyCommand ssh -p 1098 lmx@proxy.machine nc -w 1 %h %p" -P 1098 -r lmx@target.machine:~/rdsAgent .

# 在远程计算机执行命令
ssh -o "ProxyCommand ssh -p 1098 lmx@proxy.machine nc -w 1 %h %p" -p 1098 lmx@target.machine 'ip a'
```

# ssh config file example

```bash
$ cat ~/.ssh/config
Host *
ForwardAgent yes
ForwardX11 no
KeepAlive yes
ServerAliveInterval 30
CheckHostIP no
ControlMaster auto
ControlPath /tmp/ssh_mux_%h_%p_%r

# client --> 203.195.207.108:46707 --> 118.193.254.66:22
Host hk-jumpserver
user root
IdentityFile /cygdrive/c/Users/root/id_rsa
ProxyCommand ssh -i /cygdrive/c/Users/root/id_rsa root@203.195.207.108 -p 46707 nc 118.193.254.66 22
```






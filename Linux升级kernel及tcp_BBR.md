---
title: Linux 升级 kernel 及 tcp_BBR
author: dongcj <ntwk@163.com>
date: 2016/08/28 17:44:33
updated: 2016/06/07 14:25:00
categories:
  - linux
tags:
  - 升级内核
  - kernel
  - bbr
  - tcp_bbr
---

``` bash
# 这二个是打开 BBR 必需的
# 先查看内核是否大于 4.9，如大于 4.9，直接使用以下命令即可
# 如果不是，则需先按下方升级内核
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

# 保存生效
sysctl -p

# 查看 bbr 是否生效
lsmod | grep bbr
```

# 1. Debian、Ubuntu 升级内核

```bash
# 直接使用 apt-get 安装内核
apt-get update
apt-cache search linux-image-extra

# 安装最新的内核
apt-get install linux-image-extra-<NEWEST_KERNEL_RELEASE>

dpkg -l | grep linux-image

apt-get remove <OLD_KERNEL_RELEASE>

update-grub
reboot

# 查看一下目前的内核版本
# 应该是显示为 <NEWEST_KERNEL_RELEASE>
uname -r

```
<!-- more -->

# 2. RHEL、CentOS 升级内核
```bash
yum update -y
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

# 如果是 el7
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

# 如果是 el6
rpm -Uvh http://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm

# 安装内核
yum --enablerepo=elrepo-kernel install kernel-ml

# 查看当前已安装的内核
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

# 设定默认内核 (centos7)
grub2-set-default 0
reboot

# 设定默认内核 (centos6)
vi /etc/grub.conf
# 修改 default=<ID 号 >

# 设定 /etc/sysctl.conf
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# 生效
sysctl -p
```

# 3. 手动安装
## 3.1. 下载内核二进制包
> 链接：http://mirrors.kernel.org/debian/pool/main/l/linux/

## 3.2. 解压安装
```bash
ar x  linux-image-4.9.0-11-generic_4.9.0-11.12_amd64_5.deb
bzip2 -d data.tar.bz2
tar -xf data.tar
install -m644 boot/vmlinuz-4.9.0-11-generic /boot/vmlinuz-4.9.0-11-generic
cp -Rav lib/modules/4.9.0-11-generic/ /lib/modules/
depmod -a 4.9.0-11-generic
```

## 3.3. 加入引导
```bash
dracut -f -v --hostonly -k '/lib/modules/4.9.0-11-generic'  /boot/initramfs-4.9.0-11-generic 4.9.0-11-generic
# 注 : centos7 和 6 的步骤不同，centos6 是 grub，需要手动自动写 , 但注意：root=UUID= 那里的 uuid 不能修改！！！；

# centos7 是 grub2 可以用下面命令 :
grub2-mkconfig -o /boot/grub2/grub.cfg
```

## 3.4. 修改引导顺序

```bash
# 查看引导内有哪些内核
cat /boot/grub2/grub.cfg |grep menuentry

cat /boot/grub2/grub.cfg |grep menuentry
if [ x"${feature_menuentry_id}" = xy ]; then
  menuentry_id_option="--id"
  menuentry_id_option=""
  export menuentry_id_option
  menuentry 'CentOS Linux (4.9.0-rc8-amd64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-4.9.0-rc8-amd64-advanced-508f0c60-8ce4-48fa-a00e-8db45fa56da8' {
  menuentry 'CentOS Linux (3.10.0-327.36.3.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-327.36.3.el7.x86_64-advanced-508f0c60-8ce4-48fa-a00e-8db45fa56da8' {
  menuentry 'CentOS Linux (0-rescue-d45b6a27fe9641bd8979101342a4f20b) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-d45b6a27fe9641bd8979101342a4f20b-advanced-508f0c60-8ce4-48fa-a00e-8db45fa56da8' {
```

    # 下面命令的内核名称根据系统内部查到的实际名称来替换：
    grub2-set-default 'CentOS Linux (4.9.0-rc8-amd64) 7 (Core)'

    grub2-editenv list

    saved_entry=CentOS Linux (4.9.0-rc8-amd64) 7 (Core)


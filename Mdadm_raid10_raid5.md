---
title: Mdadm_raid10_raid5
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - soft raid
  - mdadm
---

> 建议：
> 如果 raid0，1，raid1+0 可以使用软 raid, raid10 及以上不建议使用软 raid ！


### 每块硬盘分一个区
    # 分区格式为 Linux software raid：

    $ fdisk /dev/sda

      WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
               switch off the mode (command 'c') and change display units to
               sectors (command 'u').

      Command (m for help): n
      Command action
         e   extended
         p   primary partition (1-4)
      p
      Partition number (1-4): 1
      First cylinder (1-91201, default 1):
      Using default value 1
      Last cylinder, +cylinders or +size{K,M,G} (1-91201, default 91201):
      Using default value 91201

      Command (m for help): p

      Disk /dev/sda: 750.2 GB, 750156374016 bytes
      255 heads, 63 sectors/track, 91201 cylinders
      Units = cylinders of 16065 * 512 = 8225280 bytes
      Sector size (logical/physical): 512 bytes / 512 bytes
      I/O size (minimum/optimal): 512 bytes / 512 bytes
      Disk identifier: 0x0005c259

         Device Boot      Start         End      Blocks   Id  System
      /dev/sda1               1       91201   732572001   83  Linux

      Command (m for help): t
      Selected partition 1
      Hex code (type L to list codes): fd
      Changed system type of partition 1 to fd (Linux raid autodetect)

      Command (m for help): w
      The partition table has been altered!

      Calling ioctl() to re-read partition table.
      Syncing disks.


# 更改分区格式
    # 按照上面的 /dev/sda 的分区例子依次给剩下的 5 块硬盘 sdc, sdd, sde, sdf, sdg 分区
    $ fdisk /dev/sdc
    ...
    $ fdisk /dev/sdd
    ...
    $ fdisk /dev/sde
    ...
    $ fdisk /dev/sdf
    ...
    $ fdisk /dev/sdg
    ...


# 创建 RAID
    # 在上面的 6 个相同大小的分区上创建 raid10
    $ mdadm --create /dev/md0 -v --raid-devices=6 --level=raid10 /dev/sda1 /dev/sdc1 /dev/sdd1 /dev/sde1 /dev/sdf1 /dev/sdg1
    mdadm: layout defaults to n2
    mdadm: layout defaults to n2
    mdadm: chunk size defaults to 512K
    mdadm: size set to 732440576K
    mdadm: Defaulting to version 1.2 metadata
    mdadm: array /dev/md0 started.


    # 查看磁盘阵列的初始化过程（build），根据磁盘大小和速度，整个过程大概需要几个小时：
    # watch cat /proc/mdstat
    Every 2.0s: cat /proc/mdstat                                       Tue Feb 11 12:51:25 2014

    Personalities : [raid10]
    md0 : active raid10 sdg1[5] sdf1[4] sde1[3] sdd1[2] sdc1[1] sda1[0]
          2197321728 blocks super 1.2 512K chunks 2 near-copies [6/6] [UUUUUU]
          [>....................]  resync =  0.2% (5826816/2197321728) finish=278.9min speed=13
    0948K/sec

    unused devices:


# 给 md0 设备创建分区和文件系统
    $ fdisk /dev/md0
    $ mkfs.ext4 /dev/md0p1

    $ mkdir /raid10
    $ mount /dev/md0p1 /raid10


# 修改 /etc/fstab 启动时自动挂载
    $ vi /etc/fstab
      ...
      /dev/md0p1 /raid10 ext4 noatime,rw 0 0


> 在上面的 /etc/fstab 文件里使用 /dev/md0p1 设备名不是一个好办法，
> 因为 udev 的缘故，这个设备名常在重启系统后变化，所以最好用 UUID，使用 blkid 命令找到相应分区的 UUID：
    $ blkid
      ...
      /dev/md0p1: UUID="093e0605-1fa2-4279-99b2-746c70b78f1b" TYPE="ext4"


    # 然后修改相应的 fstab，使用 UUID 挂载：
    $ vi /etc/fstab
      ...
      /dev/md0p1 /raid10 ext4 noatime,rw 0 0
      **UUID=093e0605-1fa2-4279-99b2-746c70b78f1b /raid10 ext4 noatime,rw 0 0**


# 查看 RAID 的状态
    $ mdadm --query --detail /dev/md0
    /dev/md0:
            Version : 1.2
      Creation Time : Tue Feb 11 12:50:38 2014
         Raid Level : raid10
         Array Size : 2197321728 (2095.53 GiB 2250.06 GB)
      Used Dev Size : 732440576 (698.51 GiB 750.02 GB)
       Raid Devices : 6
      Total Devices : 6
        Persistence : Superblock is persistent

        Update Time : Tue Feb 11 18:48:10 2014
              State : clean
     Active Devices : 6
    Working Devices : 6
     Failed Devices : 0
      Spare Devices : 0

             Layout : near=2
         Chunk Size : 512K

               Name : local:0  (local to host local)
               UUID : e3044b6c:5ab972ea:8e742b70:3f766a11
             Events : 70

        Number   Major   Minor   RaidDevice State
           0       8        1        0      active sync   /dev/sda1
           1       8       33        1      active sync   /dev/sdc1
           2       8       49        2      active sync   /dev/sdd1
           3       8       65        3      active sync   /dev/sde1
           4       8       81        4      active sync   /dev/sdf1
           5       8       97        5      active sync   /dev/sdg1


# 配置 raid 的配置文件
    $ echo device /dev/sdb1 /dev/sdc1 /dev/sdd1 > /etc/mdadm.conf
    $ mdadm --detail --scan >> /etc/mdadm.conf


# RAID 维护命令
## 删除故障盘
    $ mdadm /dev/md0 -r /dev/sdb1

## 增加新盘
    $ mdadm /dev/md0 -a /dev/sde1

## 停止并移除阵列
    $ mdadm --stop /dev/md99    // 停止
    $ mdadm -As /dev/md0        // 启动
    $ mdadm --remove /dev/md99

## 销毁系统中的阵列
    mdadm --manage /dev/md99 --fail /dev/sd[cde]1
    mdadm --manage /dev/md99 --remove /dev/sd[cde]1
    mdadm --manage /dev/md99 --stop
    mdadm --zero-superblock /dev/sd[cde]1

---
title: Parted,kpartx,losetup,sfdisk,e2fsck,resize2fs
author: dongcj <ntwk@163.com>
date: 2016/08/11 15:43:19
updated: 2016/08/11 15:44:39
categories:
  - linux
tags:
  - parted
  - kpartx
  - losetup
  - sfdisk
  - e2fsck
  - resize2fs
---
# 虚拟机增加系统盘根分区大小方法

    # 此方法适用于分区后连接的空间 , 如果是 swap, 将其删除，并在最末尾建 swap )

    # 先增加实例文件大小，可以使用 dd 命令
    $ dd if=/dev/zero of=/tmp/temp_expand bs=1M count=4096（增加 4 GB）
    $ cat /tmp/temp_expand >>/var/lib/libvirt/images/my_vm.img

    # 挂载文件至系统，以便找出是否有 swap device
    $ kpartx -a -p p /var/lib/libvirt/images/my_vm.img

    # If has swap device, del the swap device
    $ swapDevice=`sfdisk -s -l /dev/loop0 | grep -e 'Linux swap' |awk '{print $1}'`

    # Del swap
    $ parted -s ${loop} rm ${swap_part_num}

    # If does't have, add the new swap device
    $ parted -s /dev/loop1 "unit mb mkpart primary linux-swap -2048MB -0"

    # 或者先只划分空间，再用 mkswap 进行操作
    # 可选的：
    $ mkswap -v1 /dev/mapper/loop1p3

    # 重新刷新新增加的 swap 设备
    $ kpartx -d -p p /var/lib/libvirt/images/my_vm.img
    $ kpartx -a -p p /var/lib/libvirt/images/my_vm.img

    # 将新的 lo 设备分区进行打印
    $ sfdisk -d /dev/loop0 >/tmp/sfdisk_dump

    # 如果有 swap 分区，则 root 分区为 (root start), (swap start - root start)；如果没有 , 则为 (root start),
      new_root=401625,14227239		( 样例，要修改 )
      converted_root_device=`echo "/dev/loop0p1" | sed -e 's/\//\\\\\//g'`
    $ sed -e "s/^${converted_root_device}.*$/$new_root/" /tmp/sfdisk_dump | sfdisk --no-reread --force /dev/loop1

    # 重新刷新新变更的分区
    $ kpartx -d -p p /var/lib/libvirt/images/my_vm.img
    $ kpartx -a -p p /var/lib/libvirt/images/my_vm.img

    # fs resize
    $ e2fsck -pv /dev/mapper/loop0p1
    $ resize2fs -f /dev/mapper/loop0p1

# parted 使用

    # create all 100% space to one part
    $ parted -s /dev/sdb mklabel gpt
    $ parted -a optimal -s /dev/sdb "mkpart primary 0% 100%"

    # create other part
    parted -a optimal -s /dev/sdb "mkpart logical 59G 459G"
    parted -a optimal -s /dev/sdb "mkpart logical 459G 859G"

    # remove part
    parted -s /dev/sdb rm 2
    parted -s /dev/sdb rm 3

# 分区对齐

    $ cat /sys/block/sdb/queue/optimal_io_size
      1048576
    $ cat /sys/block/sdb/queue/minimum_io_size
      262144
    $ cat /sys/block/sdb/alignment_offset
      0
    $ cat /sys/block/sdb/queue/physical_block_size
      512

    #  把 optimal_io_size 的值与 alignment_offset 的值相加，之后除以 physical_block_size 的值。
    # 在此例子中：(1048576 + 0) / 512 = 2048。
    # 这个数值是分区起始的扇区。新的 parted 命令应该写成类似下面这样：

    $ mkpart primary 2048s 100%

    # 检查分区是否对齐

    (parted) align-check optimal 1
      1 aligned


---
title: 安装 PostmarketOS 后提示存储空间不足（low disk space）的解决方法
published: 2023-10-10
description: ""
tags: [Linux, PostmarketOS, Redmi]
category: ""
draft: false
---
第一次写教程，有哪里不对的还请指出。
# 问题
我在红米 2 安装 PostmarketOS 后通知栏提示存储空间不足。

命令行执行`df -h`输出如下：
```BASH
xiaomi-wt88047:~$ df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/dm-1                 2.0G      1.8G     77.0M  96% /
/dev/dm-0               225.7M     48.1M    165.5M  23% /boot
tmpfs                   378.6M      1.7M    376.8M   0% /run
dev                      10.0M         0     10.0M   0% /dev
shm                     946.4M    284.0K    946.1M   0% /dev/shm
none                    946.4M         0    946.4M   0% /lib/firmware/msm-firmware-loader
/dev/mmcblk0p1           64.0M     49.3M     14.6M  77% /lib/firmware/msm-firmware-loader/mnt/modem
/dev/mmcblk0p25          27.5M    156.0K     26.7M   1% /lib/firmware/msm-firmware-loader/mnt/persist
tmpfs                     4.0M         0      4.0M   0% /sys/fs/cgroup
tmpfs                   189.3M     16.0K    189.3M   0% /run/user/10000
/dev/mmcblk1p1           14.8G      1.5G     13.4G  10% /run/media/user/6339-6537   // 这是SD卡
```
可以看到第一行，根分区只分配了2G，并且已经满了。

# 解决方法
来自 https://gitlab.com/postmarketOS/pmaports/-/issues/2235
Adam Thiede @adamthiede 的回答

## 准备工作
推荐：
1. 一台电脑
2. 一个能够连接手机和电脑的USB线

如果实在没有也可以用手机里自带的 Console。
## 0. 有电脑，用SSH连接手机

没电脑可跳过此步，直接用手机自带的命令行 Console 从步骤 1. 开始。

1. 打开手机的 Console
2. 执行`sudo service sshd start`
3. 如果要每次开机都启动，执行`sudo rc-update add sshd`
4. 使用USB连接电脑
5. 打开电脑命令行（如果是 Windows 系统，可使用 cmd 或 PowerShell； Linux 可使用自带的控制台，快捷键一般是`Ctrl + Alt + T`）
6. 电脑命令行输入 `ssh user@172.16.42.1`
7. 提示输入密码，默认是`147147`
8. 如果控制台最后一行显示类似于`xiaomi-wt88047:~$`，则连接成功。

## 1. 使用`lsblk`列出块设备信息
命令行输入`lsblk`，得到以下结果：
```BASH
xiaomi-wt88047:~$ lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk1      179:0    0 14.8G  0 disk 
└─mmcblk1p1  179:1    0 14.8G  0 part /run/media/user/6339-6537
mmcblk0      179:32   0 14.7G  0 disk 
├─mmcblk0p1  179:33   0   64M  0 part /lib/firmware/msm-firmware-loader/mnt/modem
├─mmcblk0p2  179:34   0  512K  0 part 
├─mmcblk0p3  179:35   0  512K  0 part 
├─mmcblk0p4  179:36   0    1M  0 part 
├─mmcblk0p5  179:37   0    1M  0 part 
├─mmcblk0p6  179:38   0  512K  0 part 
├─mmcblk0p7  179:39   0  512K  0 part 
├─mmcblk0p8  179:40   0  512K  0 part 
├─mmcblk0p9  179:41   0  512K  0 part 
├─mmcblk0p10 179:42   0  512K  0 part 
├─mmcblk0p11 179:43   0  512K  0 part 
├─mmcblk0p12 179:44   0    1M  0 part 
├─mmcblk0p13 179:45   0  1.5M  0 part 
├─mmcblk0p14 179:46   0  1.5M  0 part 
├─mmcblk0p15 179:47   0    1M  0 part 
├─mmcblk0p16 179:48   0    1K  0 part 
├─mmcblk0p17 179:49   0    8K  0 part 
├─mmcblk0p18 179:50   0   10M  0 part 
├─mmcblk0p19 179:51   0   32K  0 part 
├─mmcblk0p20 179:52   0  1.5M  0 part 
├─mmcblk0p21 179:53   0   16K  0 part 
├─mmcblk0p22 179:54   0   32M  0 part 
├─mmcblk0p23 179:55   0    1G  0 part 
├─mmcblk0p24 179:56   0  320M  0 part 
├─mmcblk0p25 179:57   0   32M  0 part /lib/firmware/msm-firmware-loader/mnt/persist
├─mmcblk0p26 179:58   0   32M  0 part 
├─mmcblk0p27 179:59   0  512K  0 part 
├─mmcblk0p28 179:60   0   32K  0 part 
├─mmcblk0p29 179:61   0   64M  0 part 
└─mmcblk0p30 179:62   0 12.9G  0 part 
mmcblk0boot0 179:64   0    4M  1 disk 
mmcblk0boot1 179:96   0    4M  1 disk 
zram0        253:0    0  1.4G  0 disk [SWAP]
```
上述 `mmcblk1` 是SD卡， `mmcblk0` 是手机，手机里的 `mmcblk0p30` 很大，有12.9G，是我们接下来要操作的对象。

注：其他手机如一加6，输入`lsblk`会显示类似于`sdkxx`而不是`mmcblk0pxx`，对操作没有影响，找很大的那个分区就对了。

## 2. 使用`fdisk`重新分区
### 进入`fdisk`
控制台输入：`sudo fdisk /dev/（此处填很大的分区）`，对于我的手机来说就是 `sudo fdisk /dev/mmcblk0p30`
输入后就进入了fdisk分区工具，输出如下：
```BASH
xiaomi-wt88047:~$ sudo fdisk /dev/mmcblk0

Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help):
```
### 查看分区情况
输入`p`查看分区情况，输出如下：
```BASH
Command (m for help): p

Disk /dev/mmcblk0p30: 12.93 GiB, 13878935040 bytes, 27107295 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdab5b7a3

Device            Boot  Start     End Sectors  Size Id Type
/dev/mmcblk0p30p1 *      2048  499711  497664  243M 83 Linux
/dev/mmcblk0p30p2      499712 4923391 4423680  2.1G 83 Linux
```
记住第二个分区的`Start`的数值，这里是`499712`。
### 删除分区
输入`d`以进入删除分区的功能，输出如下：
```BASH
Command (m for help): d
Partition number (1,2, default 2):
```
它会提示我们输入要删除的分区号，我要删除第二个分区，输入2，输出如下：
```BASH
Partition number (1,2, default 2): 2

Partition 2 has been deleted.
```
删除成功。
### 新建分区
输入`n`以进入新建分区的功能，输出如下：
```BASH
Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p):
```
它提示我们要新建分区的类型，这里输入`p`，即主分区(primary)。
接下来需要分别输入分区号、扇区起始位置、扇区终止位置，其中扇区起始位置是上面[查看分区情况](#查看分区情况)记下的`Start`数值，为`499712`。
```BASH
Partition number (2-4, default 2): 
First sector (499712-27107294, default 499712): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (499712-27107294, default 27107294):
```
输入完成后输出：
```BASH
Created a new partition 2 of type 'Linux' and of size 12.7 GiB.
Partition #2 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: 
```
此处输入`Y`。

### 写入分区信息并退出
输入`w`以写入分区信息并退出。

## 3. 重启并同步更改到内核
### 重启
控制台输入`sudo reboot`以重启。
### 同步更改到内核
重启后再次[使用SSH连接到电脑](#0-有电脑用ssh连接手机)
控制台输入`resize2fs /dev/dm-1`。
### 查看效果
控制台输入`df -h`
输出：
```BASH
xiaomi-wt88047:~$ df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/dm-1                12.5G      1.8G     10.1G  15% /
（略）
```
成功。
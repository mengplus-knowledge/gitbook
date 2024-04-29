---
title: lvm pv操作手册
categories:
  - 网站运维
tags:
  - lvm
halo:
  site: https://mengplus.top
  name: e1e13df2-669d-435a-9ca0-778f5a87d7d6
  publish: true
---
# lvm pv操作手册

## 简介
物理卷（PV physical volume）
物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
PV是组成VG的基础。

## 命令清单
```bash
$ pv
pvchange   pvck       pvcreate   pvdisplay  pvmove     pvremove   pvresize   pvs        pvscan
```
## 示例环境
- ubuntu22.04 LTS
- 硬盘 `/dev/sdc /dev/sdd`,容量均为1T，未做分区
- lvm版本lvm2
## 前提条件
1. 硬盘数据请做好转移保存
2. 请卸载硬盘，保持硬盘/分区空闲状态
## 查看磁盘状态
```bash
$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 278.5G  0 disk
├─sda1                      8:1    0     1M  0 part
├─sda2                      8:2    0     2G  0 part /boot
└─sda3                      8:3    0 276.5G  0 part
  └─ubuntu--vg-ubuntu--lv 253:2    0   100G  0 lvm  /
sdc                         8:32   0 931.5G  0 disk
sdd                         8:48   0 931.5G  0 disk
```
## pvcreate - 创建一个pv
如果有所选有文件系统，则会给出警告。
```bash
~$ sudo pvcreate /dev/sdc
WARNING: ext4 signature detected on /dev/sdc at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/sdc.
  Physical volume "/dev/sdc" successfully created.
```
## pvs - 查看pv情况
简要的查看情况
```bash
 $ sudo pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sdc   vol-vg    lvm2 a--   931.51g  931.51g
  /dev/sdd   vol-vg    lvm2 a--   931.51g  631.51g
```
详细的查看
```bash
$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdd
  VG Name               
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               2rum5s-qrvr-J72L-s7C7-IPAf-WtWA-hlBKxi
#...

  "/dev/sdc" is a new physical volume of "931.51 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  VG Name
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               GkjTCe-NcLr-isOy-JOsT-2deb-lgBH-IAPzxe
```
## pvscan - 新的pv卷扫描
其它设备中的pv盘插入本机，可以通过pvcan扫描识别到
```bash
$ sudo pvscan
  PV /dev/sdd    VG vol-vg          lvm2 [931.51 GiB / 631.51 GiB free]
  PV /dev/sdc    VG vol-vg          lvm2 [931.51 GiB / 931.51 GiB free]
  PV /dev/sdb1   VG data-vg         lvm2 [<929.46 GiB / <161.46 GiB free]
  PV /dev/sda3   VG ubuntu-vg       lvm2 [276.46 GiB / 176.46 GiB free]
  Total: 4 [<3.00 TiB] / in use: 4 [<3.00 TiB] / in no VG: 0 [0   ]
```
## pvremove - 删除PV
删除PV的前提是从要求从vg组先安全的移除，再执行删除指令，如果pv里有数据怎么办？用pvmove迁移数据
```bash 
pvremove /dev/sdc
```
## pvmove - 迁移pv数据
pvmove命令：该命令可以将一个物理卷上的数据移动到另一个物理卷上，而不需要停止逻辑卷的使用。这种方式可以在不停机的情况下迁移数据，对于需要24/7的系统来说非常有用。
```bash
1、查看当前PV名称及状态
pvcreate /dev/sdb
#pvs
PV         VG     Fmt  Attr PSize  PFree 
/dev/sdc   amfevg lvm2 a--  20.00g 19.00g

2、给VG扩展新磁盘
vgextend  amfevg /dev/sdb

3、将/dev/sdc上的数据在线迁移到/dev/sdb
pvmove /dev/sdc /dev/sdb

4、删除amfevg
vgreduce amfevg /dev/sdc
```
## pvresize – 调整LVM中物理卷的容量大小  

同步磁盘大小
```bash
$ sudo pvresize  /dev/sdc
  Physical volume "/dev/sdc" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```

调整物理卷的容量大小为20GB（需二次确认）  
我使用了 -t 测试命令 不会实际修改PV
```bash
$ sudo pvresize  -t   --setphysicalvolumesize 20G /dev/sdc
  TEST MODE: Metadata will NOT be updated and volumes will not be (de)activated.
/dev/sdc: Requested size 20.00 GiB is less than real size 931.51 GiB. Proceed?  [y/n]: y
  WARNING: /dev/sdc: Pretending size is 41943040 not 1953525168 sectors.
  Physical volume "/dev/sdc" changed
  1 physical volume(s) resized or updated / 0 physical volume(s) not resized
```
## pvck - 用来检测物理卷的LVM元数据的一致性。
默认情况下，物理卷中的前4个扇区保存着LVM卷标。  
```bash
$ pvck -v /dev/sdb1 
Scanning /dev/sdb1 
Found label on /dev/sdb1, sector 1, type=LVM2 001 
Found text metadata area: offset=4096, size=192512 
Found LVM2 metadata record at offset=125952, size=70656, offset2=0 size2=0  
```
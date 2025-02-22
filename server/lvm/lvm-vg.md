---
title: lvm vg操作手册
categories:
  - 网站运维
tags:
  - lvm
halo:
  site: https://mengplus.top
  name: 238d0f18-9a4e-4cf5-b6a1-9b2e5538f574
  publish: true
---

# lvm vg操作手册

## 简介
**Volume Group(VG）** **卷组**，将数个PV进行整合，即形成了VG，在32位的操作系统中，LV的大小与PE的大小有关；在64位的操作系统中，LV几乎没有容量限制。

**提醒**：创建LVM逻辑卷的流程顺序为PV→VG→LV，而删除LVM逻辑卷流程顺序则为LV→VG→PV，因此请在决定删除VG卷组前先确认LV逻辑卷是否已被删除，然后再执行此命令。
## 命令清单
```bash
$ vg
vgcfgbackup    vgck           vgdisplay      vgimport       vgmknodes      vgrename       vgsplit
vgcfgrestore   vgconvert      vgexport       vgimportclone  vgreduce       vgs
vgchange       vgcreate       vgextend       vgmerge        vgremove       vgscan
```

## vgcreate - 创建vg
创建一个名为data_vg的卷组并将sdc sdd物理卷加入其中，PE的大小默认为4M
```bash
[root@localhost ~]# vgcreate vol-vg /dev/sdc /dev/sdd 
```

## vgs查看信息
```bash
$ sudo vgdisplay
  --- Volume group ---
  VG Name               vol-vg
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  8
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <1.82 TiB
  PE Size               4.00 MiB
  Total PE              476934
  Alloc PE / Size       76800 / 300.00 GiB
  Free  PE / Size       400134 / <1.53 TiB
  VG UUID               hlxdcr-DvqQ-S2v4-1gzE-gDos-GMfD-yB0zM7
#简要查看
mengplus@rh1288-v3:/dev/data-vg$ sudo vgs
  VG        #PV #LV #SN Attr   VSize    VFree
  data-vg     1   1   0 wz--n- <929.46g <161.46g
  ubuntu-vg   1   1   0 wz--n-  276.46g  176.46g
  vol-vg      2   1   0 wz--n-   <1.82t   <1.53t
```
## vgextend - 添加pv到vg
先创建了一个vg 额外再添加pv进去，进行如下操作
```bash
$ sudo vgextend vol-vg /dev/sdc
  Volume group "vol-vg" successfully extended
```

## vgreduce  - 移除物理卷PV
是移除未使用的卷组，如果已经被lv则需要先将这个卷组的数据移动(pvmove)到其它分区才能移除这个物理卷
```bash
Remove a PV from a VG.
  vgreduce VG PV ...
        [ COMMON_OPTIONS ]

  Remove all unused PVs from a VG.
  vgreduce -a|--all VG
        [ COMMON_OPTIONS ]

  Remove all missing PVs from a VG.
  vgreduce --removemissing VG
        [    --mirrorsonly ]
        [ COMMON_OPTIONS ]
```
## vgremove - 删除卷组VG
请注意VG中数据将会丢失，删除前请确认数据已经备份
```bash
vgremove VG01
vgremove VG01 Volume group "VG01" successfully removed
```
## vgmerge - 合并卷组VG
根据lvm结构图我们很自热的想到，VG合并与拆分问题
```
vgmerge VG VG
        [ -A|--autobackup y|n ]
        [ -l|--list ]
        [ COMMON_OPTIONS ]

vgmerge -v vg1 vg2  #将卷组vg2合并到卷组vg1
vgmerge -t vg3 vg4  #模拟测试合并卷组操作 (实际不进行操作)
vgmerge -l vg5 vg6  #显示合并的完毕的卷组名
```
## vgsplit -  拆分卷组VG
```bash
Split a VG by specified PVs.
  vgsplit VG VG PV ...
        [ COMMON_OPTIONS ]

  Split a VG by PVs in a specified LV.
  vgsplit -n|--name LV VG VG
        [ COMMON_OPTIONS ]
vgsplit [参数] [源卷组名] [目标卷组名] [物理卷路径]

将卷组vg1拆分为两个，生成新卷组vg2，该卷组成员为/dev/sda3
vgsplit vg1 vg2 /dev/sda3
```
## vgrename - 重命名卷组
重命名卷组/dev/vg1为/dev/vg2

```bash
# vgrename /dev/vg1 /dev/vg2
Volume group "vg1" successfullyrenamed to "vg2"
```
## vgmknodes - 重新加载
使用vgmknodes命令可以重新创建卷组目录和逻辑卷特殊文件。如果卷组中的任何逻辑卷被激活，重新加载其元数据。
```bash
vgmknodes [选项] [卷组名|逻辑卷路径]
```

## vgcfgrestore – 还原卷组描述符区域

---
title: lvm lv使用手册
slug: lvm-lv
categories:
  - 网站运维
tags:
  - lvm
halo:
  site: https://mengplus.top
  name: 8f3691e4-3d6d-48f1-9053-0210c0891959
  publish: true
---
# lvm lv使用手册
## 简介
**LV(logical volume)**：逻辑卷建立在卷组VG基础上，卷组中未分配空间可用于建立新的逻辑卷，逻辑卷建立后可以动态扩展和缩小空间。

> [!NOTE]
>
> 文件系统是建立在逻辑卷lv之上，调整文件系统，需要注意前后顺序
> 创建文件系统过程：lvcreate ->mkfs.ext4
> 扩容文件系统过程：lvextend -> resize2fs
> 缩减文件系统过程：resize2fs ->lvreduce

## 命令清单
```bash
:~$ lv
lvchange     lvcreate     lvextend     lvmconfig    lvmdump      lvmsadc      lvreduce     lvrename     lvs
lvconvert    lvdisplay    lvm          lvmdiskscan  lvmpolld     lvmsar       lvremove     lvresize     lvscan
```
## lvs lvdisplay查看lv列表
````bash
```bash
~$ sudo lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  b-lv      data-vg   -wi-ao---- 768.00g
  ubuntu-lv ubuntu-vg -wi-ao---- 100.00g
  vol-lv    vol-vg    -wi-ao---- 500.00g


#查看详细的信息
~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vol-vg/vol-lv
  LV Name                vol-lv
  VG Name                vol-vg
  LV UUID                S5Dm5E-TOI2-eMV1-npPG-nNLf-ab7r-XSYBJH
  LV Write Access        read/write
  LV Creation host, time rh1288-v3, 2024-04-06 01:45:30 +0800
  LV Status              available
  # open                 1
  LV Size                500.00 GiB
  Current LE             128000
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
````

## lvcreate - 创建逻辑卷lv
类型有很多大部分文档都未全面描述，这里先列举对应翻译
1. Create a linear LV.
   创建线性LV.
   最普通的创建模式，一般我们都是这种方式创建
   ```bash
   lvcreate -L|--size Size[m|UNIT] VG
   	[ -l|--extents Number[PERCENT] ]
   	[    --type linear ]
   	[ COMMON_OPTIONS ]
   	[ PV ... ]
   ```

这里进行一个简单示例，更多操作请看后续的参数介绍
```bash
   #创建示例 在vol-vg卷组中创建一个500G容量名为vol-lv的逻辑卷
   sudo lvcreate -L 500G -n vol-lv  vol-vg
   #创建示例 在vol-vg卷组中创建一个占用20%空闲容量的名为vol-lv的逻辑卷
   sudo lvcreate -L 20%Free  -n vol-lv  vol-vg
```
2. Create a striped LV (infers --type striped).
   创建一个带条LV.
	1. 对于**大量顺序读写操作**，创建striped logical volume可改进数据I/O性能。
	2. -i 指定条带数量，但是不能超过最大的PV
   ```bash
   lvcreate -i|--stripes Number -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -I|--stripesize Size[k|UNIT] ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

3. Create a raid1 or mirror LV (infers --type raid1|mirror).
   创建一个raid1或镜像LV（推断为--type raid1|mirror）。
   ```bash
     Create a raid1 or mirror LV (infers --type raid1|mirror).
     lvcreate -m|--mirrors Number -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -R|--regionsize Size[m|UNIT] ]
           [    --mirrorlog core|disk ]
           [    --minrecoveryrate Size[k|UNIT] ]
           [    --maxrecoveryrate Size[k|UNIT] ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

4. Create a raid LV (a specific raid level must be used, e.g. raid1).
   创建一个raid LV（必须使用特定的raid级别，例如raid1）。
   ```bash
   lvcreate --type raid -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -m|--mirrors Number ]
           [ -i|--stripes Number ]
           [ -I|--stripesize Size[k|UNIT] ]
           [ -R|--regionsize Size[m|UNIT] ]
           [    --minrecoveryrate Size[k|UNIT] ]
           [    --maxrecoveryrate Size[k|UNIT] ]
           [    --raidintegrity y|n ]
           [    --raidintegritymode String ]
           [    --raidintegrityblocksize Number ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

5. Create a raid10 LV.
   创建一个raid10 LV。
   ```bash
     lvcreate -m|--mirrors Number -i|--stripes Number -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -I|--stripesize Size[k|UNIT] ]
           [ -R|--regionsize Size[m|UNIT] ]
           [    --minrecoveryrate Size[k|UNIT] ]
           [    --maxrecoveryrate Size[k|UNIT] ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

6. Create a COW snapshot LV of an origin LV.
   创建源LV的COW快照LV。
   ```bash
     lvcreate -s|--snapshot -L|--size Size[m|UNIT] LV
           [ -l|--extents Number[PERCENT] ]
           [ -i|--stripes Number ]
           [ -I|--stripesize Size[k|UNIT] ]
           [ -c|--chunksize Size[k|UNIT] ]
           [    --type snapshot ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

7. Create a thin pool.
   创建精简池。
   ```bash
     lvcreate --type thin-pool -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -c|--chunksize Size[k|UNIT] ]
           [ -i|--stripes Number ]
           [ -I|--stripesize Size[k|UNIT] ]
           [    --thinpool LV_new ]
           [    --poolmetadatasize Size[m|UNIT] ]
           [    --poolmetadataspare y|n ]
           [    --discards passdown|nopassdown|ignore ]
           [    --errorwhenfull y|n ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

8. Create a cache pool.
   创建缓存池
   ```bash
     lvcreate --type cache-pool -L|--size Size[m|UNIT] VG
           [ -l|--extents Number[PERCENT] ]
           [ -H|--cache ]
           [ -c|--chunksize Size[k|UNIT] ]
           [    --poolmetadatasize Size[m|UNIT] ]
           [    --poolmetadataspare y|n ]
           [    --cachemode writethrough|writeback|passthrough ]
           [    --cachepolicy String ]
           [    --cachesettings String ]
           [    --cachemetadataformat auto|1|2 ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```

9. Create a thin LV in a thin pool (infers --type thin).

   在精简池中创建一个精简LV（推断--类型为精简）。

   ```bash
     lvcreate -V|--virtualsize Size[m|UNIT] --thinpool LV_thinpool VG
           [ -T|--thin ]
           [    --type thin ]
           [    --discards passdown|nopassdown|ignore ]
           [    --errorwhenfull y|n ]
           [ COMMON_OPTIONS ]
   ```

10. Create a thin LV that is a snapshot of an existing thin LV
	 创建一个精简LV，该LV是现有精简LV的快照

    ```bash
      lvcreate -s|--snapshot LV_thin
            [    --type thin ]
            [    --discards passdown|nopassdown|ignore ]
            [    --errorwhenfull y|n ]
            [ COMMON_OPTIONS ]
	```

11. Create a thin LV that is a snapshot of an external origin LV.
    创建一个作为外部源LV快照的精简LV。
    ```bash
      lvcreate --type thin --thinpool LV_thinpool LV
            [ -T|--thin ]
            [ -c|--chunksize Size[k|UNIT] ]
            [    --poolmetadatasize Size[m|UNIT] ]
            [    --poolmetadataspare y|n ]
            [    --discards passdown|nopassdown|ignore ]
            [    --errorwhenfull y|n ]
            [ COMMON_OPTIONS ]
    ```

12. Create a LV that returns VDO when used.=
    创建一个LV，在使用时返回VDO。
    ```bash
      lvcreate --type vdo -L|--size Size[m|UNIT] VG
            [ -l|--extents Number[PERCENT] ]
            [ -V|--virtualsize Size[m|UNIT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --vdo ]
            [    --vdopool LV_new ]
            [    --compression y|n ]
            [    --deduplication y|n ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

13. Create a thin LV, first creating a thin pool for it,  where the new thin pool is named by the --thinpool arg.
    创建一个精简LV，首先为其创建一个精简池，其中新的精简池由--thinpool参数命名。
    ```bash
      lvcreate --type thin -V|--virtualsize Size[m|UNIT] -L|--size Size[m|UNIT] --thinpool LV_new
            [ -l|--extents Number[PERCENT] ]
            [ -T|--thin ]
            [ -c|--chunksize Size[k|UNIT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --poolmetadatasize Size[m|UNIT] ]
            [    --poolmetadataspare y|n ]
            [    --discards passdown|nopassdown|ignore ]
            [    --errorwhenfull y|n ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

14. Create a new LV, then attach the specified cachepool  which converts the new LV to type cache.
    创建一个新的LV，然后附加指定的缓存池其将新的LV转换为类型高速缓存。
    ```bash
      lvcreate --type cache -L|--size Size[m|UNIT] --cachepool LV_cachepool VG
       [ -l|--extents Number[PERCENT] ]
       [ -H|--cache ]
       [ -c|--chunksize Size[k|UNIT] ]
       [ -i|--stripes Number ]
       [ -I|--stripesize Size[k|UNIT] ]
       [    --poolmetadatasize Size[m|UNIT] ]
       [    --poolmetadataspare y|n ]
       [    --cachemode writethrough|writeback|passthrough ]
       [    --cachepolicy String ]
       [    --cachesettings String ]
       [    --cachemetadataformat auto|1|2 ]
       [ COMMON_OPTIONS ]
       [ PV ... ]
    ```

15. Create a new LV, then attach the specified cachevol  which converts the new LV to type cache.
    创建一个新的LV，然后附加指定的cachevol其将新的LV转换为类型高速缓存。
    ```bash
      lvcreate --type cache -L|--size Size[m|UNIT] --cachevol LV VG
            [ -l|--extents Number[PERCENT] ]
            [ -c|--chunksize Size[k|UNIT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --cachemode writethrough|writeback|passthrough ]
            [    --cachepolicy String ]
            [    --cachesettings String ]
            [    --cachemetadataformat auto|1|2 ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

16. Create a new LV, then attach a cachevol created from  the specified cache device, which converts the new LV to type cache.
    创建一个新的LV，然后附加从创建的cachevol指定的缓存设备，用于转换要键入缓存的新LV。
    ```bash

      lvcreate --type cache -L|--size Size[m|UNIT] --cachedevice PV VG
            [ -l|--extents Number[PERCENT] ]
            [ -c|--chunksize Size[k|UNIT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --cachemode writethrough|writeback|passthrough ]
            [    --cachepolicy String ]
            [    --cachesettings String ]
            [    --cachemetadataformat auto|1|2 ]
            [    --cachesize Size[m|UNIT] ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

17. Create a new LV, then attach the specified cachevol  which converts the new LV to type writecache.
    创建一个新的LV，然后附加指定的cachevol其将新LV转换为类型写高速缓存。
    ```bash
      lvcreate --type writecache -L|--size Size[m|UNIT] --cachevol LV VG
            [ -l|--extents Number[PERCENT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --cachesettings String ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

18. Create a new LV, then attach a cachevol created from  the specified cache device, which converts the  new LV to type writecache.
    创建一个新的LV，然后附加从创建的cachevol指定的缓存设备，用于转换要键入writecache的新LV。

    ```bash
      lvcreate --type writecache -L|--size Size[m|UNIT] --cachedevice PV VG
            [ -l|--extents Number[PERCENT] ]
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [    --cachesize Size[m|UNIT] ]
            [    --cachesettings String ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

## 创建文件系统与挂载逻辑卷
创建文件系统不是lvm相关知识，但是考虑到文章连贯性，这里进行简单介绍
### 指令清单
```bash
~$ mkfs
mkfs         mkfs.btrfs   mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.ntfs    mkfs.xfs
mkfs.bfs     mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.vfat
```
分区类型很多，这里不做示例，各有特点根据需要选择合适的文件系统

格式化为ext4类型的操作示例
```bash
   #格式化分区 将vol-lv逻辑卷格式化未ext4文件系统
   sudo mkfs.ext4 /dev/vol-vg/vol-lv
```
### mount挂载逻辑卷

挂载逻辑卷与挂载普通分区一样，将刚创建的逻辑卷挂载在指定目录下
```bash
#创建要被挂载的文件夹
sudo mkdir /mnt/c
#挂载逻辑卷到路径/mnt/c下
sudo mount /dev/vol-vg/vol-lv /mnt/c
```
### fstab 自动挂载逻辑卷
这里放个我系统里的示例，自行理解
```bash
~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-sT3R7UIwMKkO6JPwFyN1wXaFlOEyPdTEqaLkAicWpBXuI4fy9bnSDSF7epBYU8l4 / ext4 defaults 0 1
/dev/data-vg/b-lv /mnt/b ext4 defaults 0 2
#/dev/vol-vg/c-lv  /mnt/c ext4 defaults,nofail  0 2
/dev/vol-vg/vol-lv  /mnt/d ext4 defaults,nofail 0 2
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/84e1b608-ceef-4f48-aece-955bb5441237 /boot ext4 defaults 0 1
/swap.img       none    swap    sw      0       0
```
**注意**：编写fstab完毕后，可以先手动测试下是否生效避免重启后无法开机
   ```bash
#自动挂载硬盘 根据fstab设定挂载
sudo mount -a
   ```

## lvextend扩展lv
扩展lv有几个方式有的扩展容量，不关心用了几个PV，有的是整个PV加入到一个逻辑卷里，
1.    Extend an LV by a specified size.
   将LV扩展到指定的大小。

   ```bash
   lvextend -L|--size [+]Size[m|UNIT] LV
       [ -l|--extents [+]Number[PERCENT] ]
       [ -r|--resizefs ]
       [ -i|--stripes Number ]
       [ -I|--stripesize Size[k|UNIT] ]
       [    --poolmetadatasize [+]Size[m|UNIT] ]
       [ COMMON_OPTIONS ]
       [ PV ... ]
   ```

   示例操作

   ```bash
   #扩展vol容量到600G
   sudo lvextend -L 600G /dev/vol-vg/vol-lv
   #resize2fs命令可以调整ext2\ext3\ext4文件系统的大小，它可以放大或者缩小没有挂载的文件系统的大小。如果文件系统已经挂载，它可以扩大文件系统的大小，前提是内核支持在线调整大小。
   # lvextend -l 100%FREE /dev/VolGroup00/lv_root

   sudo resize2fs /dev/vol-vg/vol-lv
   ```


2.    Extend an LV by specified PV extents.
   将LV延伸指定的PV范围。

   ```
   lvextend LV PV ...
           [ -r|--resizefs ]
           [ -i|--stripes Number ]
           [ -I|--stripesize Size[k|UNIT] ]
           [ COMMON_OPTIONS ]
   ```

3.    Extend a pool metadata SubLV by a specified size.
    按指定的大小扩展池元数据SubLV。

    ```bash
    lvextend --poolmetadatasize [+]Size[m|UNIT] LV_thinpool
            [ -i|--stripes Number ]
            [ -I|--stripesize Size[k|UNIT] ]
            [ COMMON_OPTIONS ]
            [ PV ... ]
    ```

4.  Extend an LV according to a predefined policy.

   根据预定义的策略扩展LV。

   ```
   lvextend --usepolicies LV_snapshot_thinpool
           [ -r|--resizefs ]
           [ COMMON_OPTIONS ]
           [ PV ... ]
   ```



## lvmconfig查看lv的配置信息

谁用的时候再查阅吧，可以用来恢复被误删的分区等信息

## lvreduce 缩减lv容量

lvreduce缩减在用的lv分区如果操作不慎有数据丢失风险，请了解清楚操作流程后再进行操作。

据我了解xfs分区不支持缩减分区，更多操作细节待补充案例

```bash
~$ sudo lvreduce --help
  lvreduce - Reduce the size of a logical volume

  lvreduce -L|--size [-]Size[m|UNIT] LV
        [ -l|--extents [-]Number[PERCENT] ]
        [ -A|--autobackup y|n ]
        [ -f|--force ]
        [ -n|--nofsck ]
        [ -r|--resizefs ]
        [    --noudevsync ]
        [    --reportformat basic|json ]
        [ COMMON_OPTIONS ]

  Common options for lvm:
        [ -d|--debug ]
        [ -h|--help ]
        [ -q|--quiet ]
        [ -v|--verbose ]
        [ -y|--yes ]
        [ -t|--test ]
        [    --commandprofile String ]
        [    --config String ]
        [    --driverloaded y|n ]
        [    --nolocking ]
        [    --lockopt String ]
        [    --longhelp ]
        [    --profile String ]
        [    --version ]

  Use --longhelp to show all options and advanced commands.
```



## lvresize 按指定大小调整lv容量

lvresize本质就是自动选择调用lvexternd 还是lvreduce ，请注意扩展分区容易，不要轻易缩减分区

```bash
 Resize an LV by a specified size.
  lvresize -L|--size [+|-]Size[m|UNIT] LV
        [ -l|--extents [+|-]Number[PERCENT] ]
        [ -r|--resizefs ]
        [    --poolmetadatasize [+]Size[m|UNIT] ]
        [ COMMON_OPTIONS ]
        [ PV ... ]

  Resize an LV by specified PV extents.
  lvresize LV PV ...
        [ -r|--resizefs ]
        [ COMMON_OPTIONS ]

  Resize a pool metadata SubLV by a specified size.
  lvresize --poolmetadatasize [+]Size[m|UNIT] LV_thinpool
        [ COMMON_OPTIONS ]
        [ PV ... ]

  Common options for command:
        [ -A|--autobackup y|n ]
        [ -f|--force ]
        [ -i|--stripes Number ]
        [ -I|--stripesize Size[k|UNIT] ]
        [ -n|--nofsck ]
        [    --alloc contiguous|cling|cling_by_tags|normal|anywhere|inherit ]
        [    --nosync ]
        [    --noudevsync ]
        [    --reportformat basic|json ]
        [    --type linear|striped|snapshot|mirror|raid|thin|cache|vdo|thin-pool|cache-pool|vdo-pool ]

  Common options for lvm:
        [ -d|--debug ]
        [ -h|--help ]
        [ -q|--quiet ]
        [ -v|--verbose ]
        [ -y|--yes ]
        [ -t|--test ]
        [    --commandprofile String ]
        [    --config String ]
        [    --driverloaded y|n ]
        [    --nolocking ]
        [    --lockopt String ]
        [    --longhelp ]
        [    --profile String ]
        [    --version ]

  Use --longhelp to show all options and advanced commands.
```

## lvrename逻辑卷改名

```bash
lvrename - Rename a logical volume

  lvrename VG LV LV_new
        [ COMMON_OPTIONS ]

  lvrename LV LV_new
        [ COMMON_OPTIONS ]

  Common options for command:
        [ -A|--autobackup y|n ]
        [    --noudevsync ]
        [    --reportformat basic|json ]

  Common options for lvm:
        [ -d|--debug ]
        [ -h|--help ]
        [ -q|--quiet ]
        [ -v|--verbose ]
        [ -y|--yes ]
        [ -t|--test ]
        [    --commandprofile String ]
        [    --config String ]
        [    --driverloaded y|n ]
        [    --nolocking ]
        [    --lockopt String ]
        [    --longhelp ]
        [    --profile String ]
        [    --version ]

  Use --longhelp to show all options and advanced commands.
```
 示例
 ```bash
 #将vol-lv逻辑卷名称改为c-lv
 sudo lvrename vol-lv c-lv
 ```

## lvremove 删除逻辑卷

**lvremove命令** 用于删除指定LVM逻辑卷。如果逻辑卷已经使用mount命令加载，则不能使用lvremove命令删除。必须使用umount命令卸载后，逻辑卷方可被删除。

请注意删除逻辑卷将丢失数据

操作示例

```bash
#移除逻辑卷vol-lv
sudo lvremove /dev/vol-vg/vol-lv
```


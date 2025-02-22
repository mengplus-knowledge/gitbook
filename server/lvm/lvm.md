---
title: lvm让Linux磁盘空间的弹性管理
categories:
  - 网站运维
tags:
  - lvm
halo:
  site: https://mengplus.top
  name: 0de667de-f0cf-4004-9597-472c4a6213fb
  publish: true
---
# lvm让Linux磁盘空间的弹性管理

## 什么是LVM？

LVM(Logical Volume Manager)逻辑卷管理是在Linux2.4内核以上实现的磁盘管理技术。它是**Linux环境下对磁盘分区进行管理的一种机制**。现在不仅仅是Linux系统上可以使用LVM这种磁盘管理机制，对于其它的类UNIX操作系统，以及windows操作系统都有类似与LVM这种磁盘管理软件。

## 相关该概念

- **Physical Volume(PV)**

**物理卷**，将实际的磁盘分区（partition）系统识别码（system ID）修改为8e后，在通过pvcreate指令转化为LVM最底层的**物理卷**，作为后续空间管理的基础。

- **Volume Group(VG）**

**卷组**，将数个PV进行整合，即形成了VG，在32位的操作系统中，LV的大小与PE的大小有关；在64位的操作系统中，LV几乎没有容量限制。

- **Physical Extent(PE)**

**物理区块**，他是LVM中的最小存储单元。PE类似于文件系统中的block。

- **Logical Volume(LV)**

**逻辑卷**，由VG划分而来，LV的大小与PE的大小及PE的数量有关，Size（LV）= Count（PE）* Size（PE）

## 结构示意图

![](https://pic1.zhimg.com/80/v2-1a90c2975d253a669a29b9b0b4e35938_720w.webp)

LVM结构示意图

**磁盘划分为PV —> PV组成了VG，同时设置了PE大小 —> 从VG中划分出LV**

## LVM的工作原理？

它就是通过**将底层的物理硬盘抽象的封装起来，然后以逻辑卷的方式呈现给上层应用**。在传统的磁盘管理机制中，我们的上层应用是直接访问文件系统，从而对底层的物理硬盘进行读取，而在LVM中，其通过**对底层的硬盘进行封装**，当我们对底层的物理硬盘进行操作时，其不再是针对于分区进行操作，而是**通过一个叫做逻辑卷的东西来对其进行底层的磁盘管理操作**。比如说我增加一个物理硬盘，这个时候上层的服务是感觉不到的，因为呈现给上层服务的是以逻辑卷的方式。

## LVM的优缺点

优点：

1. 可以在系统运行的状态下动态的扩展文件系统的大小
2. 文件系统跨越多个磁盘，文件系统的大小不受磁盘大小的限制
3. LVM的存储空间可以通过新增磁盘的方式扩容

缺点：
1. 从卷组中移除一个磁盘的时候必须使用reducevg命令
2. 当卷组中有一个磁盘损坏了，整个卷组都会受到影响（由于一份数据可能会存储在不同的磁盘中）
3. 在磁盘创建过程中增加了额外的步骤，所以数据存贮性能会受到影响

## LVM的实战使用

![](https://pic4.zhimg.com/80/v2-80cf48c04f0ac69699bc1539d4a5b1f7_720w.webp)

LVM实现流程示意图

  

- **创建LVM过程**
创建LVM逻辑卷的流程顺序为PV→VG→LV，而删除LVM逻辑卷流程顺序则为LV→VG→PV，因此请在决定删除VG卷组前先确认LV逻辑卷是否已被删除，然后再执行此命令。 

① 通过fdisk修改磁盘分区的SYSTEM ID为8e，将文件系统类型更改为Linux LVM

> ~ fdisk 需要修改的磁盘  
> 选择 t 命令修改  
> 选择分区号，设置系统ID信息

② 创建PV

> ~ pvcreate 需要转化为PV的磁盘分区  
> 其他PV相关指令  
> pvscan：显示当前PV相关信息  
> pvdisplay （+ 磁盘分区路径）：显示详细的（分区）PV信息

③ 创建VG

>~ vgcreate [-s N[mgt]] VG名称 PV名称  
> 可以通过 -s 后面接PE的大小，代为可以是m,g,t  
>   
> 其他VG相关指令  
> vgscan：显示当前VG相关信息  
> vgdisplay：显示目前系统上的VG状态  
> vgextend：在VG内增加额外的PV  
> vgreduce：在VG内一处PV  
> vgchange：设定VG是否启动(active)  
> vgremove：删除一个VG

④ 创建LV

> ~ lvcreate [-L N[mgt]] [-n LV名称] VG名称 （-L 后跟lV容量的大小）  
> ~ lvcreate [-l N] [-n LV名稱] VG名称 （-l 后跟PE的个数）  
>   
> 其他相关命令  
> lvscan：查询系统上的LV  
> lvdisplay：显示系统上的LV详细信息  
> lvextend：增加LV的容量  
> lvreduce：减少LV的容量  
> lvremove：删除一个LV  
> lvresize：对LV进行容量大小的调整

⑤ **格式化新建的LV，否则将无法进行目录挂载**

⑥ 文件目录的挂载

>~ mount LV路径 目录路径

**过程记录：**

创建分区信息如下，注意修改 Id 为 8e

```bash
[root@localhost /]# fdisk /dev/sdb

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): p

Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbb091226

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1         262     2104483+  8e  Linux
```

LVM创建过程

```bash
[root@localhost /]# pvcreate /dev/sdb1       //创建pv
  Physical volume "/dev/sdb1" successfully created
[root@localhost /]# vgcreate pvZHB /dev/sdb1    //创建vg
  Volume group "pvZHB" successfully created
[root@localhost /]# lvcreate -L 2G -n lv_zhb vgZHB    //创建lv
  Logical volume "lv_zhb" created.
[root@localhost /]# mkfs.xfs /dev/vgZHB/lv_zhb      //格式化lv
meta-data=/dev/vgZHB/lv_zhb      isize=256    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost /]# mount /dev/vgZHB/lv_zhb /zhbDir/           //挂载
```

  

- **LV扩容（注意需要先对分区进行格式化，若扩容后再可视化会对原有文件造成影响）**

① 确定VG是否存在多余的容量

> LV是由VG进行划分创建的，若VG无剩余空间则无法对LV进行扩容，所以首先需要新增磁盘，并通过 pvcreate 指令添加PV，vgextend扩展VG空间。  
> **注意：新加磁盘分区后需要进行格式化，否则在最终完成扩容后，扩容空间将无法正常使用。**

② LV 进行扩容

> 在VG剩余空间足够的情况下，只需要通过 lvresize 指令将剩余容量加入到所需要增加的LV装置内即可。  
> ~ lvresize -L +N(M/G/T) 进行扩容的LV路径

③ 若目标目录挂载点的文件系统为xfs，则需要执行如下命令才能最终达到扩容

> ~ xfs_growfs 目标目录

**过程记录**

新建逻辑分区sdb5后，执行如下操作进行扩容

```text
[root@localhost 桌面]# pvcreate /dev/sdb5   //创建pv
  Physical volume "/dev/sdb5" successfully created
[root@localhost 桌面]# vgextend vgZHB /dev/sdb5    //扩展vg
  Volume group "vgZHB" successfully extended
[root@localhost 桌面]# lvresize -L +2G /dev/vgZHB/lv_zhb    //扩展lv
  Size of logical volume vgZHB/lv_zhb changed from 2.00 GiB (512 extents) to 4.00 GiB (1024 extents).
  Logical volume lv_zhb successfully resized.
[root@localhost 桌面]# df -Th                       //查看挂载信息，发现空间无变化
Filesystem           Type     Size  Used Avail Use% Mounted on
/dev/mapper/vg_zhbcentos-lv_root
                     ext4      45G  5.5G   39G  13% /
tmpfs                tmpfs    939M  228K  939M   1% /dev/shm
/dev/sda1            ext4     477M   41M  411M  10% /boot
/dev/sr0             iso9660  2.0G  2.0G     0 100% /media/CentOS-6.10-x86_64-LiveDVD
/dev/mapper/vgZHB-lv_zhb
                     xfs      2.0G   33M  2.0G   2% /zhbDir
[root@localhost /]# xfs_growfs /zhbDir/           //执行xfs_growfhttps://link.zhihu.com/?target=https%3A//blog.51cto.com/13438667/2084924s
meta-data=/dev/mapper/vgZHB-lv_zhb isize=256    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 524288 to 1048576
[root@localhost /]# df -Th                     //查看挂载信息，发现空间已扩容
Filesystem           Type     Size  Used Avail Use% Mounted on
/dev/mapper/vg_zhbcentos-lv_root
                     ext4      45G  5.5G   39G  13% /
tmpfs                tmpfs    939M  228K  939M   1% /dev/shm
/dev/sda1            ext4     477M   41M  411M  10% /boot
/dev/sr0             iso9660  2.0G  2.0G     0 100% /media/CentOS-6.10-x86_64-LiveDVD
/dev/mapper/vgZHB-lv_zhb
                     xfs      4.0G   33M  4.0G   1% /zhbDir
```

  通过了解lvm原理后，接下来正式全方位逐步学习每一步的操作。
  

转载： [LVM——让Linux磁盘空间的弹性管理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/67166867)
参考链接:
1. https://blog.51cto.com/13438667/2084924
2. http://linux.vbird.org/linux_basic/0420quota.php%23lvm_whatis
3. https://www.cnblogs.com/linuxprobe/p/5381538.html

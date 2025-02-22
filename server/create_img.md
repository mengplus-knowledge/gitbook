---
title: 创建虚拟镜像用于测试
slug: create_img
cover: 
categories:
  - 嵌入式开发
tags:
  - linux
halo:
  site: https://mengplus.top
  name: 354724ba-6810-4197-b393-63dc0a5f1b1b
  publish: true
---
# 创建虚拟镜像用于测试
一些测试环境中需要模拟读写硬盘，这里给出一个创建虚拟硬盘用于测试的方案
**当前环境**
1. 鲁班猫2 2h+8G 
2. 外置存储 32G TF 卡
### 1. 创建虚拟磁盘文件

1. 使用dd命令

   不推荐使用dd 它创建的文件占用硬盘空间，也就是说你创建一个空白img给它50G空间它就占用这么大，对于我们测试时浪费的

   ```bash
   dd if=/dev/zero of=/tmp/sda.img bs=1M count=51200
   ```

2. 稀疏文件truncate

   可以任意设置磁盘大小，比如我这里为了模仿设置了500G

   ```bash
   truncate -s 500G sda.img
   ```

   

### 2. 设置虚拟磁盘

接下来，使用`losetup`将这个文件设置为一个块设备：

```bash
bash复制代码
sudo losetup /dev/loop0 /tmp/sda.img
```

### 3. 使用`parted`进行分区

现在，使用`parted`工具对虚拟磁盘进行分区：

```bash
sudo parted /dev/loop0 mklabel msdos
```

然后依次创建各个分区：

```bash
# 创建sda1 (PREBOOT FAT 2G)
sudo parted -a opt /dev/loop0 mkpart primary fat32 1MiB 2049MiB

# 创建sda2 (CEKERNEL FAT 524M)
sudo parted -a opt /dev/loop0 mkpart primary fat32 2049MiB 2673MiB

# 创建sda3 (UnKnown 4.3G)
sudo parted -a opt /dev/loop0 mkpart primary ext4 2673MiB 7003MiB

# 创建sda4 (扩展分区)
sudo parted -a opt /dev/loop0 mkpart extended 7003MiB 100%

# 创建sda5 (Core 524M)
sudo parted -a opt /dev/loop0 mkpart logical ext4 7003MiB 7527MiB

# 创建sda6 (MachineData)
sudo parted -a opt /dev/loop0 mkpart logical ext4 7527MiB 8527MiB

# 创建sda7 (DataModel1)
sudo parted -a opt /dev/loop0 mkpart logical ext4 8527MiB 9527MiB

# 创建sda8 (DataModel2)
sudo parted -a opt /dev/loop0 mkpart logical ext4 9527MiB 10527MiB

# 创建sda9 (CtbData)
sudo parted -a opt /dev/loop0 mkpart logical ext4 10527MiB 11527MiB

# 创建sda10 (Interrupt)
sudo parted -a opt /dev/loop0 mkpart logical ext4 11527MiB 12527MiB

# 创建sda11 (Extensible)
sudo parted -a opt /dev/loop0 mkpart logical ext4 12527MiB 13527MiB

# 创建sda12 (Customer)
sudo parted -a opt /dev/loop0 mkpart logical ext4 13527MiB 100%

    
```

### 4. 格式化分区

接下来，格式化每个分区：

```bash
sudo mkfs.ext4 /dev/loop0p1
sudo mkfs.ext4 /dev/loop0p2
sudo mkfs.ext4 /dev/loop0p3
sudo mkfs.ext4 /dev/loop0p5
sudo mkfs.ext4 /dev/loop0p6
sudo mkfs.ext4 /dev/loop0p7
sudo mkfs.ext4 /dev/loop0p8
sudo mkfs.ext4 /dev/loop0p9
sudo mkfs.ext4 /dev/loop0p10
sudo mkfs.ext4 /dev/loop0p11
sudo mkfs.ext4 /dev/loop0p12
```

### 5. 挂载分区（可选）

如果你需要挂载这些分区，可以使用以下命令：

```bash
sudo mount /dev/loop0p1 /mnt/sda1
sudo mount /dev/loop0p2 /mnt/sda2
sudo mount /dev/loop0p3 /mnt/sda3
sudo mount /dev/loop0p5 /mnt/sda5
sudo mount /dev/loop0p6 /mnt/sda6
sudo mount /dev/loop0p7 /mnt/sda7
sudo mount /dev/loop0p8 /mnt/sda8
sudo mount /dev/loop0p9 /mnt/sda9
sudo mount /dev/loop0p10 /mnt/sda10
sudo mount /dev/loop0p11 /mnt/sda11
sudo mount /dev/loop0p12 /mnt/sda12

    
```

### 6. 清理工作

当你完成测试后，可以卸载分区并删除虚拟磁盘：

```bash
# 卸载所有分区
sudo umount /mnt/sda*

# 删除循环设备映射
sudo losetup -d /dev/loop0

# 删除虚拟磁盘文件
rm /tmp/sda.img
    
```

这样你就成功创建了一个模拟的sda盘，并且按照要求进行了分区和格式化。
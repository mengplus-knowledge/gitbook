---
title: linux 查看与设置卷标
slug: part_label
cover: ""
categories:
  - 嵌入式开发
tags: []
halo:
  site: https://mengplus.top
  name: 84144035-3719-43ed-89ee-0b0affecd613
  publish: true
---
# linux 查看与设置卷标



## 查询卷标

```bash
sudo lsblk  -f
NAME               FSTYPE      LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
loop0
├─loop0p1          ext4        boot  075ad80b-8366-413e-a9cb-4c6993d703de
├─loop0p2          ext4              dd531e90-7e0c-4ca7-afc1-4204df1185ee
├─loop0p3          ext4              dbe80797-920c-4cc5-874f-fe93123f66d8
├─loop0p4
├─loop0p5          ext4              6e8acb79-db56-47d1-81c9-7d302300e97b
├─loop0p6          ext4              37143356-9b1e-4763-8263-d2caacae8375
├─loop0p7          ext4              d8e838f6-9ab9-40db-bd7f-0f63d27c233c
├─loop0p8          ext4              eb37498e-ba0b-4712-99e2-81371adb90fc
├─loop0p9          ext4              11d59fea-7917-4c92-ba98-898e15773ea6
├─loop0p10         ext4              161f2d8c-1448-4ed1-ae32-0dafccbf8cb5
├─loop0p11         ext4              ecf2c2db-dd3b-445d-b52d-412978d00488
└─loop0p12         ext4              3f4307bf-1a9e-4219-b4f3-4b26f6b8d672
mmcblk0
├─mmcblk0p1
├─mmcblk0p2        ext2        boot  900cc8dd-9d83-4486-8cbc-34a5bc7bab1f     58.2M    48% /boot
└─mmcblk0p3        ext4              de22abb4-c71b-4f38-b500-e6e5c49577f7      1.5G    74% /
mmcblk0boot0
mmcblk0boot1
mmcblk1            LVM2_member       ZjLqzT-1Wqg-XyVs-8zFO-DgVR-guMg-iLqe99
└─data--vg-vol--lv ext4              c4dc439d-77a1-4b45-a855-346b345e1fcc     11.8G    19% /mnt/b

```

如果需要指定磁盘

```bash
lsblk -f /dev/sdXn
```

假设你的 NTFS 分区是 `/dev/sda1`，以下是一些具体的命令示例：

```
# 使用 ntfslabel 命令
sudo ntfslabel /dev/sda1

# 使用 blkid 命令
sudo blkid -o value -s LABEL /dev/sda1

# 使用 mount 命令
mount | grep /dev/sda1

# 使用 lsblk 命令
lsblk -f /dev/sda1

    
```

这些命令将帮助你获取 NTFS 卷的名称或标签。

## 设置卷标

在 Linux 系统中，为多个 ext4 分区设置卷名称（也称为卷标）主要有以下几种方法：

1. **使用 e2label 命令**
   - **卸载分区**：首先要确保需要设置卷名称的分区未被挂载。可以使用 `umount` 命令卸载分区，例如，如果要设置 `/dev/sdb1` 分区的卷名称，先执行 `sudo umount /dev/sdb1`。
   - **设置卷名称**：使用 `e2label` 命令来修改卷标名称，语法为 `sudo e2label /dev/sdXn new-name`。其中，`/dev/sdXn` 是你要修改的分区设备名称和分区号，`new-name` 是你期望设置的新名称。比如，将 `/dev/sdb1` 分区的名称设置为 “Data”，可执行 `sudo e2label /dev/sdb1 Data`。
   - **重新挂载分区**：修改完名称后，重新挂载分区以便使设置生效，例如 `sudo mount /dev/sdb1 /mnt/data`。
2. **使用 tune2fs 命令**
   - **卸载分区**：同样要先卸载目标分区，如 `sudo umount /dev/sdb1`。
   - **设置卷名称**：该命令的语法为 `sudo tune2fs /dev/sdXn -L new-name`，这里的参数含义与 `e2label` 命令类似。例如，将 `/dev/sdb2` 分区的名称修改为 “Backup”，执行 `sudo tune2fs /dev/sdb2 -L Backup`。
   - **重新挂载分区**：之后重新挂载分区，如 `sudo mount /dev/sdb1 /mnt/backup`。
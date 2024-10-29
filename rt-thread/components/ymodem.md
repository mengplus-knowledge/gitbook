---
title: Rt-thread Ymodem的详细使用方法
slug: ymodem
categories:
  - rt-thread
tags: []
halo:
  site: https://mengplus.top
  name: fd7dbd8d-856c-4995-8a3d-099e3704ad18
  publish: true
---
# Rt-thread Ymodem的详细使用方法
## 前言
 ymodem是一个非常有好轻量型的文件传输协议，可以方便的应用与嵌入式设备中，通过串口网口等完成文件传输，常应用于OTA升级和配置文件的下发工作。
 xmodem,ymodem,zmodem协议区别:

1. Xmodem：这种古老的传输协议速度较慢，但由于使用了CRC错误侦测方法，传输的准确率可高达99.6%。
2. Ymodem：这是Xmodem的改良版，使用了1024位区段传送，速度比Xmodem要快。
3. Zmodem：采用了串流式（streaming）传输方式，传输速度较快，而且还具有自动改变区段大小和断点续传、快速错误侦测等功能。这是目前最流行的文件传输协议。

 今天这里主要介绍如何在rt-thread中使用ymodem传输文件(rt-thread中只支持了这个协议)。

##　实验环境

1. 主控f103 96kram 256kb flash
2. 操作系统rt-thread V5.10
3. 组件
   1. fal
   2. nfs
   3. ymodem
   4. finsh
4. 上位机 xshell

## 使用演示

1. 接入串口打开 xshell 115200 接入串口，设备上电，看到启动日志

2. 输入help查看支持的命令

   ```bash
   msh> help

   ```



3. 输入mount查看分区挂载情况

   ```bash
   msh> mount

   ```

4. 输入ry ./tmp 将tmp文件夹作为接收文件夹

5. xshell窗口右击 传输 ➡YMODEM 👉用Ymodem发送,进度条走完后即可

6. 使用ls命令查看是否受到指定文件，字节数是否一致

7. 如果是文本文件可以使用`echo ./XXX` 查看文件内容


---
title: Keil v5 C51 MDK包 共存合并方式
categories:
  - 软件分享
tags: []
halo:
  site: https://mengplus.top
  name: 9a8b7ced-1ac8-4054-b0a7-92f6a447cd3b
  publish: true
---
先看效果

<img src="https://mengplus.top/upload/a9e937b2a1b6b6c852122a678f1adacb_70.png" style="zoom: 33%;" /><img src="https://mengplus.top/upload/60626d644301ac12859377be800af6b8_70.png" style="zoom: 33%;" /><img src="https://mengplus.top/upload/60626d644301ac12859377be800af6b8_70.png" style="zoom: 33%;" />

是不是有人想要这样的Keil 既能编程51单片机 又同时能切换到STM32的编程使用，但是 一般情况下 无论51的工程还是STM32的工程都是有keil两个版本需要使用不同的编译器，不能做到双击工程文件来打开，这是想当的麻烦，这里我为大家带来C51版于MDK版Keil并存的安装方式。 我也看了下别人的安装方式 大部分套路一样 修改TOOL.INI这使得看着不那么”正规“还比较麻烦， 这里我为大家带来“正确”的安装方式。

首先参考我的另一篇文章拿到安装包，[https://mengplus.top/archives/keil_MDK_C51_C251](https://mengplus.top/archives/keil_MDK_C51_C251)

根据开发需要选择MDK C51甚至还有C251,最好时都是同时间出来的版本，比如各自的最新版

1. [mdk539](http://www.keil.com/update/whatsnew.asp?p=MDK&v=5.39) 下载地址：[http://www.keil.com/files/eval/MDK539.EXE](http://www.keil.com/files/eval/MDK539.EXE)
   
2. [c51v961](http://www.keil.com/update/whatsnew.asp?p=C51&v=9.61)下载地址:[http://www.keil.com/files/uc51/c51v961.EXE](http://www.keil.com/files/uc51/c51v961.EXE)
   
3. 激活工具[keygen_new(2032).zip](/upload/keygen_new(2032).zip)
   
4. 芯片包下载地址[http://www.keil.com/dd2/Pack/](http://www.keil.com/dd2/Pack/)
   

懒得上图，其实很简单，注意安装顺序就行了

防止出现安装意外 请**清理干净你的旧版本Keil**,默认你有一定的软件安装激活能力(这能力都没有的话 ……)

1. 第一步：先安装C51的包，根据需要选择合适的安装位置，稍后的MDK版也将安装在同一个位置（Tips:可以留在最后一起再激活)
   
2. 第二步：正常安装MDK包，这时你会发现安装路径 用户信息都于刚刚你设置C51包的一样（只要这样显示，说明后面的版本合并会成功），一路next 直到安装成功
   
3. 第三步：激活Keil ,使用管理员权限打开Keil_v5，找到激活证书的地方（File->lic）,如果看到两个待激活的证书，就说明两个版本合并成功，具体 就不说了，于平常激活一样
   
4. 第四步：安装Pack包，根据需要安装Pack，只要装好了MDK版的Keil Pack包就可以安装了，双击pack包然后…… 你就可以正常使用
   

## Pack Installer

mdk自带的包安装与升级工具，但是在国内使用并不友好，如果您有梯子可以在这里直接升级更新，否则请去芯片包下载地址获取最新包手动安装。
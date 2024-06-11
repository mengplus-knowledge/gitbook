---
title: 在SDRAM中运行
slug: onsdram
cover: ""
categories:
  - 嵌入式开发
tags:
  - IAP
halo:
  site: https://mengplus.top
  name: bbb1bce2-27e6-4bb3-941d-db0da73a2922
  publish: true
---
## 前言
SDRAM 中执行程序需要一个特殊处理，即操作MPU配置SDRAM区为代码可执行区，否则默认PC指针跳转过来后立刻报错。
而MPU配置属于内核知识，更多细节需要看内核手册
另外需要注意，CMSIS版本，太老的虽然支持MPU 但是并不完善，推荐用最新的如参考资料1
另外还要注意，代码已经在SDRAM中执行时，不得关闭MPU，否则立刻复位
## 参考资料
1. [ARM-software/CMSIS_6: CMSIS version 6 (successor of CMSIS_5) (github.com)](https://github.com/ARM-software/CMSIS_6)
2. [【经验分享】STM32F429 使用外扩 SDRAM 运行程序的方法 - STM32团队 ST意法半导体中文论坛 (stmicroelectronics.cn)](https://shequ.stmicroelectronics.cn/thread-633444-1-1.html)
3. [MCU怎么在扩展的SDRAM上运行程序？-CSDN博客](https://blog.csdn.net/ybhuangfugui/article/details/90170387)
4. [STM32 MPU 阅读笔记_stm32 mpu 错误-CSDN博客](https://blog.csdn.net/Bin_Watson/article/details/127157732)
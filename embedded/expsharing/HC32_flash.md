---
title: 华大HC32 flash擦除未生效的解决方法
slug: hc32-flash
categories:
  - 嵌入式开发
tags: []
halo:
  site: https://mengplus.top
  name: 0569d814-0347-4552-8e70-397e79198170
  publish: true
cover: ""
---

# 华大HC32 flash擦除未生效的解决方法

## 参考资料
1. https://www.xhsc.com.cn/Productlist/info.aspx?itemid=1780&parent
2. https://blog.csdn.net/weixin_43895102/article/details/119084248
3. https://blog.csdn.net/Oushuwen/article/details/109243294
## 关键信息

在应用手册 《AN_FLASH操作说明_Rev1.1.pdf》可看到如下描述

> 3.5 基于FLASH安全特性的编程方法
> 在实际应用中，对于容量大于32K的MCU，如果需要将FLASH的操作函数、安全功能函数等放置在FLASH的32K安全区，可通过以下便捷的方式实现。
> 说明：
>
> - 本例为说明需要，主要示例将函数“Flash_SectorErase()”放置在安全区“0x400”地址的方法，实际应用当中可以将“示例函数”和“地址”根据自己的需求进行替换。
>
> 3.5.1 基于Keil MDK的编程方法
> 在Keil MDK中，可以简单通过如下方式实现对安全函数的执行地址映射：
> 在目标函数的声明处增加以下代码：
> en_result_t Flash_SectorErase(uint32_t u32SectorAddr) __attribute__((section(".ARM.__at_0x400")));
> 3.5.2 基于IAR的编程方法
> 1、在目标函数定义处增加以下代码：
> en_result_t Flash_SectorErase(uint32_t u32SectorAddr) @".Flash_SectorErase"
> 2、在工程“*.icf”文件中增加以下代码：
> place at address mem:0x00000400 { readonly section .Flash_SectorErase};



要求函数在前**32kb**里面，怎么操作呢

这里给出iar的配置方案 已经测试通过的，直接修改icf文件即可

```yaml
/*###ICF### Section handled by ICF editor, don't touch! ****/
/*-Editor annotation file-*/
/* IcfEditorFile="$TOOLKIT_DIR$\config\ide\IcfEditor\cortex_v1_0.xml" */
/*-Specials-*/
define symbol __ICFEDIT_intvec_start__ = 0x00000000;
/*-Memory Regions-*/
define symbol __ICFEDIT_region_ROM_start__ = 0x00000000;
define symbol __ICFEDIT_region_ROM_end__   = 0x0003FFFF;
define symbol __ICFEDIT_region_RAM_start__ = 0x20000000;
define exported symbol __ICFEDIT_region_RAM_end__   = 0x20007FFF;
/*-Sizes-*/
define symbol __ICFEDIT_size_cstack__ = 0x0400;
define symbol __ICFEDIT_size_heap__   = 0x0100;
/**** End of ICF editor section. ###ICF###*/

define memory mem with size = 4G;
define region ROM_region      = mem:[from __ICFEDIT_region_ROM_start__   to __ICFEDIT_region_ROM_end__];
define region RAM_region      = mem:[from __ICFEDIT_region_RAM_start__   to __ICFEDIT_region_RAM_end__];
define region ROM_safe_region      = mem:[from 0x00000400   to 0x00008000];//这里就是安全区

define block CSTACK    with alignment = 8, size = __ICFEDIT_size_cstack__   { };
define block HEAP      with alignment = 8, size = __ICFEDIT_size_heap__     { };

initialize by copy { readwrite };
do not initialize  { section .noinit };

place at address mem:__ICFEDIT_intvec_start__ { readonly section .intvec };

place in ROM_region   { readonly };
place in RAM_region   { readwrite,  block CSTACK,last block HEAP};
place in ROM_safe_region { readonly object hc32l196_flash.o};//这里flash的函数都放里面完事 ，否则无法正常使用

```


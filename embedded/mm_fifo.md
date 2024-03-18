---
title: mm_fifo微型的环形队列
categories:
  - 开源仓库
tags:
  - fifo
halo:
  site: https://mengplus.top
  name: 03b95f46-b4a9-494f-a28c-7ce04ef78886
  publish: true
time: 2024-03-19
---
# mm_fifo 微型的环形队列
## 简要

FIFO: First in, First out代表先进的数据先出 ，后进的数据后出。

## 仓库地址
1. https://gitee.com/mengplus/mm_fifo.git
## 为什么需要FIFO？

FIFO存储器是系统的缓冲环节，如果没有FIFO存储器，整个系统就不可能正常工作。

FIFO的功能可以概括为

（1）对连续的数据流进行缓存，防止在进机和存储操作时丢失数据；

（2）数据集中起来进行进机和存储，可避免频繁的总线操作，减轻CPU的负担；

（3）允许系统进行DMA操作，提高数据的传输速度。这是至关重要的一点，如果不采用DMA操作，数据传输将达不到传输要求，而且大大增加CPU的负担，无法同时完成数据的存储工作。



微型的环形队列，尽可能兼容多种应用环境,降低环境依赖

## 操作介绍


```C
    typedef struct _MM_FIFO mm_fifo_t;
    /**
     * @brief 初始化环形队列空间
     * @note 为了更好的兼容应用场景，内部不做内存申请
     * @param data_ptr 缓存区地址
     * @param data_size 缓存区大小
     */
    void mm_fifo_init(void *data_ptr, size_t data_size);
    /**
     * @brief 判断是否为空
     * @param self 队列的句柄
     * @return true:队列为空 false 不空
     */
    bool mm_fifo_is_empty(mm_fifo_t *self);
    /**
     * @brief 判断是否为满
     * @param self 队列的句柄
     * @return true:队列为满 false 不满
     */
    bool mm_fifo_is_full(mm_fifo_t *self);
    /**
     * @brief 存入一个数据
     * @param self 队列的句柄
     * @param dat 存入的数据
     * @return true:存入成功 false 失败
     */
    bool mm_fifo_push(mm_fifo_t *self, uint8_t dat);
    /**
     * @brief 取出一个数据
     * @note 未做空判定，用户自行调用
     * @param self 队列的句柄
     * @return 取出的数据
     */
    uint8_t mm_fifo_pop(mm_fifo_t *self);
    /**
     * @brief 取出一个数据
     * @note 取出数据,不从队列中清除
     * @param self 队列的句柄
     * @return 待取出的数据
     */
    uint8_t mm_fifo_pop_peek(mm_fifo_t *self);
    /**
     * @brief 存入多个数据
     * @param self 队列的句柄
     * @param dat 待存入的数据
     * @param data_size 数据数量
     * @return 实际存进去的数据数量
     */
    size_t mm_fifo_push_multi(mm_fifo_t *self, uint8_t *dat, size_t data_size);
    /**
     * @brief 取出多个数据
     * @param self 队列的句柄
     * @param dat 取出数据存放空间
     * @param data_size 要取出的数据
     * @return 实际取出来的数据
     */
    size_t mm_fifo_pop_multi(mm_fifo_t *self, uint8_t *dat, size_t data_size);
    /**
     * @brief 取出多个数据，不从队列中移除
     * @param self 队列的句柄
     * @param dat 取出数据存放空间
     * @param data_size 要取出的数据
     * @return 实际取出来的数据
     */
    size_t mm_fifo_pop_multi_peek(mm_fifo_t *self, uint8_t *dat, size_t data_size);
    /**
     * @brief 未使用空间
     * @param self 队列的句柄
     * @return 队列中元素的数量
     */
    size_t mm_fifo_get_used_space(mm_fifo_t *self);
    /**
     * @brief 使用的空间
     * @param self 队列的句柄
     * @return 队列中空闲的数量
     */
    size_t mm_fifo_get_unused_space(mm_fifo_t *self);

```

## demo

```C

#include <stdio.h>
#include "mm_fifo.h"

uint8_t buff[512];
uint8_t rebuff[128];
int main(int argc, char **argv)
{
    mm_fifo_t *pfifo = mm_fifo_init(buff, sizeof(buff));
    printf("fifo init: %ld\n", sizeof(buff));
    printf("used_space: %ld\n", mm_fifo_get_used_space(pfifo));
    printf("unused_space: %ld\n", mm_fifo_get_unused_space(pfifo));

    for (int i = 0; i < 50; i++)
    { // 装入数据
        mm_fifo_push(pfifo, i);
    }
    printf("used_space: %ld\n", mm_fifo_get_used_space(pfifo));
    printf("unused_space: %ld\n", mm_fifo_get_unused_space(pfifo));

    size_t result = mm_fifo_pop_multi_peek(pfifo, rebuff, sizeof(rebuff));

    printf("pop: %ld\n", result);
    for (size_t i = 0; i < result; i++)
    {
        printf("%d ", rebuff[i]);
    }
    printf("\r\n");

    size_t result = mm_fifo_pop_multi(pfifo, rebuff, sizeof(rebuff));

    printf("pop: %ld\n", result);
    for (size_t i = 0; i < result; i++)
    {
        printf("%d ", rebuff[i]);
    }
    printf("\r\n");
    return 0;
}

```

## demo
```bash
make  -C ./demo/ clean #编译demo
./demo/demo.out        #执行例程
```

## 联系我
使用中发现的问题请提交工单

交流群见QQ 790012859

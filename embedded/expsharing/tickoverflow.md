---
title: 滴答时间溢出问题
categories:
  - 嵌入式开发
tags: []
halo:
  site: https://mengplus.top
  name: 0fb1eb27-117a-4571-92b1-1fd83cd8b2eb
  publish: true
---
# 滴答时间溢出问题
千年虫问题一直困扰着所有程序员，嵌入式资源稀缺，滴答时钟最大计数时uint32_t ms,通过简单计算可知最大计时仅仅49天，那么对于动不动就是几个月不关机超长续航的设备来说，数据溢出后，定时任务又如何运行的呢？

这里我们将分析危害以及给出解决方案。

这里先给出一个我们常用但是有些许问题的例子
```c
static void led_loop(void)
{
    static uint32_t next_tick = 0;

    if(HAL_GetTick() >= next_tick)
    {
        next_tick = HAL_GetTick() + 100;
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
    }
}

int main(void)
{
    // 此处省略一万字

    while (1)
    {
        // other loop
		
        led_loop();
    }
}

```
`HAL_GetTick` 在这里获得了一个uint32_t计数值，最大可计数到**UINT32_MAX**，不考虑溢出情况下他将运行良好，但是在临界溢出时他将发生什么呢？

HAL_GetTick() 等于（UINT32_MAX -10）时，next_tick将得到结果为(UINT32_MAX+90)由于溢出截断得到结果为90，那么条件**if(HAL_GetTick() >= next_tick)** 将持续有效，while速度有多快，这个任务将执行多快，如果你这里是控制的继电器或者其它不能高速开关的设备，将导致设备损坏。

那么这个异常情况会持续多久呢？根据上述进一步分析可知，持续到HAL_GetTick()达到溢出条件。

**那么如何避免这种问题?**

调整判断过程
```c
    if(HAL_GetTick()-next_tick <= UINT32_MAX/2 )
    {
        next_tick = HAL_GetTick() + 100;
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
    }
```
重复上述分析，HAL_GetTick() 等于（UINT32_MAX -10）时，next_tick将得到结果为(UINT32_MAX+90)由于溢出截断得到结果为90，那么条件**if(HAL_GetTick()-next_tick <= UINT32_MAX/2 )** 由于计算结果是一个超大值而不满足条件，当HAL_GetTick()达到溢出条件后，由于next_tick仍大于HAL_GetTick()则仍是一个超大值而不满足条件。

随着HAL_GetTick()数值的逐渐接近next_tick直到相等条件才会满足而执行这个任务，因此解决了随着数据溢出而导致定时任务出错的问题。

## 其它问题
1. 为什么在`if(HAL_GetTick()-next_tick <= UINT32_MAX/2 )` 使用了<= 而不是等于？
	这个是想每隔一段时间翻转下IO口嘛，，所以实际应该是HAL_GetTick()≥next_tick(不考虑溢出)，考虑到溢出，所以将判断条件做了一个变形，即用`HAL_GetTick()-next_tick`的值的范围作为判断条件，HAL_GetTick()-next_tick的值的下限是0，关键在于找出其上限，HAL_GetTick()-next_tick应该<或者<=一个值，那么为什么取值`<= UINT32_MAX/2`是值得考虑的事情
2. 为什么不考虑RTC他可以统计上百年或者uint64_t作为计数？
	首先是效率问题，RTC时间戳也是32位的秒计时，每次判断两个相当与扩展到uint64_t 在32位内核中判断条件效率比较低，高频使用的逻辑不合适，也不够通用，虽然PC中已经普及了64位，时间戳也逐步扩展64位
	但是考虑到我们的硬件环境有的更加低端，很多地方都在使用这个tick，如果它扩展到64位，将增加代码复杂度，所以应考虑解决溢出问题而不是增加计数长度。

## 参考文章
[定时任务的溢出问题_hal gettick()-CSDN博客](https://blog.csdn.net/wenbodong/article/details/110223168)
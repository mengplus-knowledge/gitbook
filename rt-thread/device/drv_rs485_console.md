---
title: rs485接入rt-thread的控制台finSH
slug: drv_rs485_console
categories:
  - 嵌入式开发
tags:
  - rt-thread
halo:
  site: https://mengplus.top
  name: 1f803069-beee-4108-96a7-1ad0ada0f566
  publish: true
---
# rs485接入rt-thread的控制台finSH

## 简介

在使用rt-thread时，控制台是一个非常重要的调试组件方便我们对社保的交互控制，但是有些硬件场景资源有限必须要用rs485作为日志口时，需要将其接入终端.这里给出了一个接入方式。

## 设计思想

在rt-thread中继承了linux设计原则，在rt-thead中一切外设均是设备，·但是rs485不是普通的串口，又增加了一个方向控制引脚，在引入rs485组件驱动起来后又引入了一个新的问题 RS485组件不是标准的rtt设备模型，无法直接接入终端。

因此我们要做的就是将rs485设备做成标准的设备来方便调用接口。

```c
    int rt_hw_rs485_init(void);
    rt_hw_rs485_init();
    rt_console_set_device("rs485_1");
```

虽然网络上已有朋友给出了实现方案，但是他们大都在uart设备基础之上二次改动，甚至改动了核心组件，根本不通用，因此这里给出一个解决方案与大家交流。

下面直接附完整源码，由于高度的分层设计，我这里的代码不涉及硬件操作，几乎没有移植问题

```c
#include <rtthread.h>
#include <rtdevice.h>
#include "board.h"
#include "rs485.h"
struct rt_rs485_device
{
    struct rt_device parent;
    rs485_inst_t *handle;
    const char *name;
    rt_err_t (*init)(rt_device_t dev);
    rt_err_t (*open)(rt_device_t dev, rt_uint16_t oflag);
    rt_err_t (*close)(rt_device_t dev);
    rt_ssize_t (*read)(rt_device_t dev, rt_off_t pos, void *buffer, rt_size_t size);
    rt_ssize_t (*write)(rt_device_t dev, rt_off_t pos, const void *buffer, rt_size_t size);
    rt_err_t (*control)(rt_device_t dev, int cmd, void *args);
};

/* RT-Thread Device Interface */
/*
 * This function initializes rs485 device.
 */
static void rs485_notify(rt_device_t dev)
{
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;
    if (rs485->parent.rx_indicate)
        rs485->parent.rx_indicate(&rs485->parent, 1);
    dev = *(rt_device_t *)rs485->handle;
    if (dev->rx_indicate)
        dev->rx_indicate(dev, 1);
}
static rt_err_t rt_rs485_init(struct rt_device *dev)
{
    rt_err_t result = RT_EOK;
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;
    rs485->handle = rs485_create(rs485->name, 19200, 0, GET_PIN(A, 6), 1);

    return result;
}
static rt_err_t rt_rs485_open(struct rt_device *dev, rt_uint16_t oflag)
{
    rt_err_t ret = RT_EOK;
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485                       = (struct rt_rs485_device *)dev;
    ret                         = rs485_connect(rs485->handle);
    struct rt_device_notify arg = {rs485_notify, &rs485->parent}; /*!<  */
    rt_device_control(*(rt_device_t *)rs485->handle, RT_DEVICE_CTRL_NOTIFY_SET, &arg);

    return ret;
}

static rt_err_t rt_rs485_close(struct rt_device *dev)
{
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;

    return rs485_disconn(rs485->handle);
}

static rt_ssize_t rt_rs485_read(struct rt_device *dev,
                                rt_off_t pos,
                                void *buffer,
                                rt_size_t size)
{
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;
    return rs485_recv(rs485->handle, buffer, size);
}

static rt_ssize_t rt_rs485_write(struct rt_device *dev,
                                 rt_off_t pos,
                                 const void *buffer,
                                 rt_size_t size)
{
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;
    return rs485_send(rs485->handle, (void *)buffer, size);
}


static rt_err_t rt_rs485_control(struct rt_device *dev,
                                 int cmd,
                                 void *args)
{
    rt_err_t ret = RT_EOK;
    struct rt_rs485_device *rs485;

    RT_ASSERT(dev != RT_NULL);
    rs485 = (struct rt_rs485_device *)dev;
    ret   = rs485->control(*(rt_device_t *)rs485->handle, cmd, args);

    return ret;
}
#ifdef RT_USING_DEVICE_OPS
const static struct rt_device_ops rs485_ops =
    {
        rt_rs485_init,
        rt_rs485_open,
        rt_rs485_close,
        rt_rs485_read,
        rt_rs485_write,
        rt_rs485_control};
#endif
/*
 * rs485 register
 */
rt_err_t rt_hw_rs485_register(struct rt_rs485_device *rs485,
                              const char *name, uint8_t flag)
{
    rt_err_t ret;
    rt_device_t device;
    RT_ASSERT(rs485 != RT_NULL);
    device = (rt_device_t) & (rs485->parent);
#ifdef RT_USING_DEVICE_OPS
    device->ops = &serial_ops;
#else
    device->init    = rt_rs485_init;
    device->open    = rt_rs485_open;
    device->close   = rt_rs485_close;
    device->read    = rt_rs485_read;
    device->write   = rt_rs485_write;
    device->control = rt_rs485_control;
#endif
    /* register a character device */
    /* set device type */
    device->type = RT_Device_Class_Char;
    ret          = rt_device_register(device, name, flag);

    return ret;
}
int rt_hw_rs485_init(void)
{
    static struct rt_rs485_device rs485_1;

    /* register UART device */
    rs485_1.name = "lpuart0";
    int result   = rt_hw_rs485_register(&rs485_1,"rs485_1", 0);
    RT_ASSERT(result == RT_EOK);

    return result;
}
//INIT_DRIVER_EARLY_EXPORT(rt_hw_rs485_init); 如果需要自动注册可以启动它 但是不要在系统调度起来之前就将其作为打印口

```

## 需要关注的问题

1. 由于RS485组件用到了系统相关操作，您需要在系统调度起来之后才能调用`rt_console_set_device`，我这里建议在main中调用
2. 明显我为了懒省事调用rs485第三方组件并不是很好的选择，如有需要请根据这个思路重写分serial组件给rs485使用

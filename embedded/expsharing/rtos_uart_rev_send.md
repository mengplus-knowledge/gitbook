---
title: 操作系统下的串口收发推荐方案
slug: rtos_uart_rev_send
categories:
  - 嵌入式开发
tags:
  - uart
halo:
  site: https://mengplus.top
  name: 3f54cf9d-b2a2-4e7d-a861-30ab0b7f4f2e
  publish: true
---
# 操作系统下的串口收发推荐方案

##　前言

收发各自一个fifo。发送时检测不为空，放入fifo等候排队发发送；接收时，收到的所有报文放入一个fifo，主程序周期性检测缓存区以及报文完整性，在实际使用过程中发现有以下几个重大问题。

1. 报文间隔无法设计，收发异步无法及时判定是否正确发送成功；
2. 接收报文无接收完毕信号，无法及时处理回应；
3. 报文发送失败or无回应不能及时重发或者处理；

鉴于以上情况，在rt-thread操作系统下根据需求重新设计，完成了一个响应及时，扩展性强的收发模块。

##　串口收发需求

1. 报文失败重发
2. 报文插队
3. 多实例公用
4. 接收超时
5. 接收断帧
6. 接收缓存
7. 报文校验

## 实现方案

1. 发送缓存区FIFO更换为消息队列，将发送的报文打包成结构体，提高扩展性
2. 接收增加**帧超时**与**字节超时**，方便快速断定一帧数据
3. 为统一入口，为收发各自增加一个EVENT，串口线程独立等待串口事件处理
4. 如果每个发送都应有回应，则在发送完毕后，立刻进入等待回应状态，超时无回应重发

## 详细设计

### 结构体定义

根据上述需求编制代码得到如下结构体定义。在操作系统下使得可读性大大增加，

```c
//消息结构体
typedef struct _MQ_ITEM
{
    uint8_t rev;     /*!< 保留 */
    uint16_t len;    /*!< 数据长度 */
    void *func;      /*!< 消息回应的处理 */
    uint8_t buff[0]; /*!< 有效数据的起始地址 */
} mq_item_t;
//串口事件
enum _THEAD_UART_EVT
{
    UART_EVT_RX_IND   = (1 << 0),
    UART_EVT_TX_IND   = (1 << 1),
    UART_EVT_RX_BREAK = (1 << 2),
};
//串口结构体
typedef struct _UART_MODULE
{
    struct _UART_MODULE *next;
    char name[8];            /*!< device name */
    rt_mutex_t mutex;        /*!< 资源互斥锁 */
    rt_device_t serial;      /*!< uart句柄 */
    uint16_t repeat;         /*!< 串口重复次数 */
    uint16_t timeout;        /*!< ms超时时间 */
    uint8_t byte_tmo;        /*!< ms帧间隔 */
    struct rt_event evt;     /*!< 收发事件 */
    uint8_t mq_buff[4][256]; /*!< 消息队列缓冲区 */
    /* 消息队列控制块 */
    struct rt_messagequeue mq;
    uint8_t mq_temp_tx[256]; /*!< 临时的消息缓存区 */
    void *user_data;         /*!<  用户自定义数据 */
    void (*tick)(void *user_data);
    uart_module_deal_callback deal;
} uart_module_t;
/**
 * @brief 获得uart线程句柄，第一次获取时初始化
 */
uart_module_t *uart_module_get_instance(const char *device_name);


/**
 * @brief 发送报文到本线程队列
 */
void uart_module_mq_send(uart_module_t *ptr, mq_item_t *item_ptr);
/**
 * @brief 发送报文到本线程 阻塞
 */
void uart_module_mq_send_blocking(uart_module_t *ptr, mq_item_t *item_ptr);

rt_ssize_t uart_module_recv(uart_module_t *data_ptr, uint8_t *buf, uint8_t size);
```

### 线程处理过程

实际的收发线程可以如下设计，核心在于`rt_event_recv`等待收发若干事件，

接收事件到达后接收完整一帧报文交给`deal`处理

发送事件到达后从`mq`队列中取出一包报文传输给发送，随即等候回应，无论是否收到报文，均进入deal，其通过判定收到报文长度判断报文无响应。

```c
void uart_entry(void *param)
{
    uart_module_t *uart_ptr = (uart_module_t *)param;
    //dzf_dev_t *dzf_ptr      = (dzf_dev_t *)uart_ptr->user_data;
    while (1)
    {
        rt_uint32_t recved     = 0;
        mq_item_t *item_rx_ptr = (mq_item_t *)mq_temp_rx;
        if (rt_event_recv(&uart_ptr->evt, UART_EVT_RX_IND | UART_EVT_TX_IND, (RT_EVENT_FLAG_OR | RT_EVENT_FLAG_CLEAR), 1000, &recved) == RT_EOK)
        {
            rt_mutex_take(uart_ptr->mutex, RT_WAITING_FOREVER);

            rt_ssize_t recv_len = 0;                                      /*!< 收到的报文长度 */
            uint8_t *buf        = item_rx_ptr->buff;                      /*!< 接收缓存区 公用*/
            uint16_t size       = sizeof(mq_temp_rx) - sizeof(mq_item_t); /*!< 接收缓存区大小 */

            if (recved & UART_EVT_RX_IND)
            {
                recv_len = uart_module_recv(uart_ptr, buf, size);
                if (recv_len)
                { /*!< 收到报文 */

                    LOG_HEX("recv", 16, buf, recv_len);
                }
                item_rx_ptr->len  = recv_len; /*!< 收到的 报文长度 */
                item_rx_ptr->func = NULL;
                if (uart_ptr->deal)
                {
                    uart_ptr->deal(uart_ptr->user_data, item_rx_ptr->func, item_rx_ptr->buff, item_rx_ptr->len);
                }
            }
            if (recved & UART_EVT_TX_IND)
            {
                rt_ssize_t ret;
                do
                {
                    ret = rt_mq_recv(&uart_ptr->mq, item_rx_ptr,
                                     sizeof(mq_temp_rx), 0);
                    if (ret > 0)
                    {
                        uint8_t repreat = uart_ptr->repeat; /*!< 重试次数 */
                        do
                        {
                            rt_ssize_t send_len = rt_device_write(uart_ptr->serial, 0, item_rx_ptr->buff, item_rx_ptr->len);
                            recv_len            = uart_module_recv(uart_ptr, buf, size);
                        } while (recv_len == 0 && repreat && repreat--);

                        item_rx_ptr->len = recv_len; /*!< 收到的 报文长度 */
                        if (recv_len)
                        {                            /*!< 收到报文 */
                            LOG_D("rev_len:%d", item_rx_ptr->len);
                            LOG_HEX("recv", 16, buf, recv_len);
                        }

                        if (uart_ptr->deal)
                        {
                            uart_ptr->deal(uart_ptr->user_data, item_rx_ptr->func, item_rx_ptr->buff, item_rx_ptr->len);
                        }
                    }
                } while (ret > 0);
            }
            rt_mutex_release(uart_ptr->mutex);
        }
    }
}
```

### 发送一帧报文

将报文压入消息队列后，再向线程发送一个event通知线程发送此报文。

```c
void uart_module_mq_send(uart_module_t *data_ptr, mq_item_t *item_ptr)
{
    if (data_ptr == 0 || item_ptr == 0 || item_ptr->len == 0)
    {
        return;
    }
    rt_mutex_take(data_ptr->mutex, RT_WAITING_FOREVER);

    rt_mq_send(&data_ptr->mq, item_ptr, sizeof(mq_item_t) + item_ptr->len);
    rt_event_send(&data_ptr->evt, UART_EVT_TX_IND);
    rt_mutex_release(data_ptr->mutex);
}
```



### 插队发送一个报文

插队报文要求立即发送，这里采用互斥锁来实现，不再交给发送线程，直接在互斥锁处等待上一个报文处理完毕后直接进入发送过程。

```c
void uart_module_mq_send_blocking(uart_module_t *data_ptr, mq_item_t *item_ptr)
{
    if (data_ptr == 0 || item_ptr == 0 || item_ptr->len == 0)
    {
        return;
    }
    rt_mutex_take(data_ptr->mutex, RT_WAITING_FOREVER);

    rt_ssize_t ret;
    uint8_t mq_temp_rx[512];
    mq_item_t *mq_item_rx_ptr = (mq_item_t *)mq_temp_rx;
    uint8_t *buf              = mq_item_rx_ptr->buff;                   /*!< 接收缓存区 公用*/
    uint16_t size             = sizeof(mq_temp_rx) - sizeof(mq_item_t); /*!< 接收缓存区大小 */

    rt_ssize_t recv_len = 0;                                            /*!< 收到的报文长度 */
    ret                 = item_ptr->len;
    if (ret > 0)
    {
        uint8_t repreat      = data_ptr->repeat; /*!< 重试次数 */
        mq_item_rx_ptr->func = item_ptr->func;
        do
        {
            rt_ssize_t send_len = rt_device_write(data_ptr->serial, 0, item_ptr->buff, item_ptr->len);
            recv_len            = uart_module_recv(data_ptr, buf, size);
        } while (recv_len == 0 && repreat && repreat--);

        mq_item_rx_ptr->len = recv_len; /*!< 收到的 报文长度 */
        bool (*dzf_deal)(void *dev_ptr, uint8_t *buff, uint16_t len) = (bool (*)(void *dev_ptr, uint8_t *buff, uint16_t len))mq_item_rx_ptr->func;
        if (dzf_deal)
        {
            dzf_deal(data_ptr->user_data, mq_item_rx_ptr->buff, mq_item_rx_ptr->len);
        }
        else if (data_ptr->deal)
        {
            data_ptr->deal(data_ptr->user_data, mq_item_rx_ptr->func, mq_item_rx_ptr->buff, mq_item_rx_ptr->len);
        }
    }
    rt_mutex_release(data_ptr->mutex);
}
```

### 接收一帧报文

此设计参考rt-thread第三方组件rs485设计而来，通过两个超时事件的设计，可以高效的完成整帧报文的接收设计，避免响应不及时与断帧粘包等问题。

```c
rt_ssize_t uart_module_recv(uart_module_t *data_ptr, uint8_t *buf, uint8_t size)
{
    int recv_len       = 0;
    rt_uint32_t recved = 0;
    while (size)
    {
        int len = rt_device_read(data_ptr->serial, 0, buf + recv_len, size);
        if (len)
        {
            recv_len += len;
            size     -= len;
            continue;
        }
        if (recv_len)
        { /*!< 等字节超时 */
            if (rt_event_recv(&data_ptr->evt, UART_EVT_RX_IND,
                              (RT_EVENT_FLAG_OR | RT_EVENT_FLAG_CLEAR), data_ptr->byte_tmo, &recved)
                != RT_EOK)
            {
                break;
            }
        }
        else
        { /*!< 等报文 */
            if (rt_event_recv(&data_ptr->evt, UART_EVT_RX_IND | UART_EVT_RX_BREAK,
                              (RT_EVENT_FLAG_OR | RT_EVENT_FLAG_CLEAR), data_ptr->timeout, &recved)
                != RT_EOK)
            {
                break;
            }
        }
    }
    return recv_len;
}
```

## 多实例设计

在实际使用过程中往往一次开启多个串口任务，而且大部分的任务在上述过程中几乎都是完全一致的，因此将此处设计成多实例结构，方便后续功能扩展。

为了更好的维护多实例，也可以采用工厂模式进行此处设计。

根据传参，每个实例在第一次请求时创建，并加入链表之中。

```c
/**
 * @brief 多例模式获得设备句柄
 *
 * @return uart_module_t*
 */
uart_module_t *uart_module_get_instance(const char *device_name)
{
    static uart_module_t *data_ptr = NULL;
    uart_module_t *data_cur_ptr    = data_ptr;
    if (device_name == NULL || device_name[0] == '\0')
    {
        return NULL;
    }
    //XXX:need mutex
    while (data_cur_ptr != 0)
    {
        if (rt_strcmp(data_cur_ptr->name, device_name) == 0)
        {
            break;
        }
        else
        {
            data_cur_ptr = data_cur_ptr->next;
        }
    }

    if (data_cur_ptr == NULL)
    {
        //rt_enter_critical();
        data_cur_ptr = rt_calloc(1, sizeof(uart_module_t));
        RT_ASSERT(data_cur_ptr != NULL);
        data_cur_ptr->repeat   = 3;   /*!< 最多重复3次 */
        data_cur_ptr->timeout  = 100; /*!< 超时等待100ms */
        data_cur_ptr->byte_tmo = 10;  /*!< 断帧10ms */
        data_cur_ptr->mutex    = rt_mutex_create("um", RT_IPC_FLAG_PRIO);

        data_cur_ptr->serial = rt_device_find(device_name);
        if (data_cur_ptr->serial == NULL)
        {
            rt_free(data_cur_ptr);
        }
        rt_strcpy(data_cur_ptr->name, device_name);


        RT_ASSERT(data_cur_ptr->serial != NULL);
        /* step2：修改串口配置参数 */
        struct serial_configure config = RT_SERIAL_CONFIG_DEFAULT; /* 初始化配置参数 */
        config.baud_rate               = BAUD_RATE_19200;          // 修改波特率为 115200
        config.data_bits               = DATA_BITS_8;              // 数据位 8
        config.stop_bits               = STOP_BITS_1;              // 停止位 1
        config.bufsz                   = 128;                      // 修改缓冲区 buff size 为 128
        config.parity                  = PARITY_NONE;              // 无奇偶校验位

        /* step3：控制串口设备。通过控制接口传入命令控制字，与控制参数 */
        rt_device_control(data_cur_ptr->serial, RT_DEVICE_CTRL_CONFIG, &config);

        if (rt_device_open(data_cur_ptr->serial, RT_DEVICE_OFLAG_RDWR | RT_DEVICE_FLAG_INT_RX | RT_DEVICE_FLAG_INT_TX) == RT_EOK)
        {
            data_cur_ptr->serial->user_data = data_cur_ptr;
            rt_device_set_rx_indicate(data_cur_ptr->serial, uart_rx_ind);
        }

        rt_event_init(&data_cur_ptr->evt, device_name, RT_IPC_FLAG_PRIO);
        /* 初始化消息队列 */
        rt_err_t result = rt_mq_init(&data_cur_ptr->mq,
                                     device_name,
                                     data_cur_ptr->mq_buff,            /* 内存池指向 msg_pool */
                                     sizeof(data_cur_ptr->mq_buff[0]), /* 每个消息的大小是 256 字节 */
                                     sizeof(data_cur_ptr->mq_buff),    /* 内存池的大小是 msg_pool 的大小 */
                                     RT_IPC_FLAG_PRIO);                /* 如果有多个线程等待，优先级大小的方法分配消息 */

        RT_ASSERT(result == RT_EOK);
        /**追加到链表 */
        data_cur_ptr->next = data_ptr;
        data_ptr           = data_cur_ptr;
    }

    return data_cur_ptr;
}
```

## 报文处理

报文处理根据不同报文应当自定设计，此处给出modbus下的设计样式

```c
bool dzf_deal(dzf_dev_t *dev_ptr, dzf_deal_t func, uint8_t *buff, uint16_t len)
{
    bool res              = 0;
    uint8_t deviceAddress = dev_ptr->config.device_id;
    // 功能码处理函数，返回解析结果或错误码
    // 定义偏移量和长度数组
    uint16_t frameOffsets[10]; // 最多记录 10 帧的起始偏移量
    uint16_t frameLengths[10]; // 最多记录 10 帧的长度
    uint16_t frameCount = 0;   // 正确的报文数量
    uint16_t errorCount = 0;
    uint16_t maxFrames  = 10;
    if (len)
    {
        if (dev_ptr->status.flag_Wavelength_scan_ing == 1)
        {
            //特殊处理	读取激光锁相扫描结果(SCANLOCKINRESULT)
            res = dzf_deal_laser_lock_result(dev_ptr, buff, len);
        }
        else
        {
            // 封装Modbus上下文
            ModbusContext context = {
                .buffer        = buff,
                .length        = len,
                .deviceAddress = deviceAddress,
                .frameOffsets  = frameOffsets,
                .frameLengths  = frameLengths,
                .maxFrames     = maxFrames,
                .frameCount    = &frameCount,
                .errorCount    = &errorCount};

            // 调用报文处理函数
            int result = ProcessModbusRTU(&context);
            for (int i = 0; i < frameCount; i++)
            {
                uint8_t *frame_ptr = (buff + frameOffsets[i]);
                if (func)
                {
                    res = func(dev_ptr, frame_ptr, frameLengths[i]);
                }
                if (res != true)
                {
                    /*!< 未被正确处理 */
                    DZF_LOG_W("id(0x%x) cmd(0x%d) \n", frame_ptr[0], frame_ptr[1]);
                }
            }
            if (res == 0 && (errorCount || frameCount == 0 || frameCount > 1))
            {
                DZF_LOG_W("errorCount(%d) frameCount(%d) \n", errorCount, frameCount);
            }
        }
    }
    else
    {
        if (func)
        {
            res = func(dev_ptr, buff, len);
        }
    }
    if (res)
    {
        if ((dev_ptr->status.state == DZF_OFFLINE) || (dev_ptr->status.state == DZF_NOREADY))
        {
            dev_ptr->status.state = DZF_NORMAL;
        }
    }
    else
    {
        dev_ptr->status.state = DZF_OFFLINE;
    }

    return res;
}
```

##　实际应用效果

当前的设计思想已经在两个产品中进行使用验证，整体效果良好，收发相应及时

## 缺陷与不足

1. 当前的设计依托操作系统，裸机中不方便进行移植使用
2. 缓存区开的较大，资源占用较多
3. 虽然设计思想较为清晰，但是在实现之后仍然感觉过于复杂，模块化程度不高，移植的接口过多

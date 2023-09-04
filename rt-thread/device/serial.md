# 串口

1. [struct rt\_serial\_device](https://github.com/RT-Thread/rt-thread/blob/3602f891211904a27dcbd51e5ba72fefce7326b2/components/drivers/include/drivers/serial.h#L145-L156)

```cpp
struct rt_serial_device
{
    struct rt_device          parent;

    const struct rt_uart_ops *ops;
    struct serial_configure   config;

    void *serial_rx;
    void *serial_tx;

    struct rt_device_notify rx_notify;
};

```

---
title: 结构体赋值操作
slug: struct_how_to_init
categories:
  - 编程设计
tags: []
halo:
  site: https://mengplus.top
  name: 8210cb77-9347-46c6-8218-3fc987806de4
  publish: true
---
# 结构体赋值操作

结构体逐个成员的赋值是让人非常痛苦的事情，降低了可读性，这里给出几个解决方案

```c
#pragma pack(1)
typedef struct MB_READ_REQUEST_0x03
{
    uint8_t id;
    uint8_t cmd;
    uint8_t addr[2];
    uint8_t len[2];
    uint8_t crc[2];
} mb_read_req_03_t;

#pragma pack()
```

1. 初始化赋值

   最常见的全赋值方式，起始位对齐，末尾容易过长or 过短，但是可读性差，容易错位导致错误

   ```c
   mb_read_req_03_t req={DZF_ADDR, 0x03, DEVICEID >> 8, DEVICEID & 0xFF, 0x00, 0x01, 0x9A, 0x36};
   ```

2. 部分初始化赋值(不限制顺序赋值)

   确保每个成员都正确赋值，但是容易忽略未被初始化的成员

   ```c
       mb_read_req_03_t req = {.id      = DZF_ADDR,
                               .cmd     = 0x03,
                               .addr[0] = DEVICEID >> 8,
                               .addr[1] = DEVICEID & 0xFF,
                               .len[0]  = 0x00,
                               .len[1]  = 0x01,
                               .crc[0]  = 0x9A,
                               .crc[1]  = 0x36};
   ```

3. 成员逐个赋值

   代码长，逐个手动指定，不适合成员过多的场景

   ```
       mb_read_req_03_t req;
       req->id      = DZF_ADDR;
       req->cmd     = 0x03;
       req->addr[0] = DEVICEID >> 8;
       req->addr[1] = DEVICEID & 0xFF;
       req->len[0]  = 0;
       req->len[1]  = 1;
       uint16_t crc = ll_crc16_calculator((uint8_t *)req, sizeof(mb_read_req_03_t) - 2);
       req->crc[0]  = crc & 0xFF;
       req->crc[1]  = crc >> 8;
   ```



4. 结构体整体拷贝

   交给系统优化这部分，内部调用了memcpy

   ```c
       mb_read_req_03_t *req = (mb_read_req_03_t *)buf;
       mb_read_req_03_t req_init = {.id      = DZF_ADDR,
                                    .cmd     = 0x03,
                                    .addr[0] = DEVICEID >> 8,
                                    .addr[1] = DEVICEID & 0xFF,
                                    .len[0]  = 0x00,
                                    .len[1]  = 0x01,
                                    .crc[0]  = 0x9A,
                                    .crc[1]  = 0x36};

       *req = req_init;

   //也可以简写
      mb_read_req_03_t *req = (mb_read_req_03_t *)buf;
      *req = (mb_read_req_03_t){DZF_ADDR, 0x03, DEVICEID >> 8, DEVICEID & 0xFF, 0x00, 0x01, 0x9A, 0x36};

   ```

5. 部分成员顺序赋值

   先部分初始化,`.cmd`是一个参考位置，后续的跟着依次赋值，前面未被覆盖到的将不赋值，内部调用了memcpy

   ```c
       mb_read_req_03_t req_init = {.cmd = 0x03,
                                    DEVICEID >> 8,
                                    DEVICEID & 0xFF,
                                    0x00,
                                    0x01};
   ```

## 数组赋值

对于一个数组我们也可以使用类似的操作,使用数组索引地址进行逐个初始化，避免相关修改影响顺序。

```c
typedef struct power_ctl_info
{
    rt_uint8_t lv;       /*!< 电平高低等级 */
    uint8_t delay_100ms; /*!< 延迟启动的时间100ms为一个最小单位 */
} power_ctl_info_t;
typedef struct power_management_INFO
{
    const char *name;
    rt_base_t pin;
    uint8_t lock : 1; /*!< 锁定配置，在本线程中不进行切换 */
    power_ctl_info_t ctl_info[PM_WORKMODE_NUM];
} pm_info_t;

const pm_info_t pinlist[] = {
[HOT1_CTRL_ID] = {"hot1", HOT1_CTRL, 0, .ctl_info = {[PM_CHARGER] = {PIN_LOW, 0}, [PM_STARTUP] = {PIN_HIGH, 10}, [PM_IDLE] = {PIN_HIGH, 0}, [PM_SLEEP] = {PIN_HIGH, 0}, [PM_SHUTDOWN] = {PIN_LOW, 0}, [PM_GET_GAS] = {PIN_HIGH, 0}, [PM_MEASURE] = {PIN_HIGH, 0}}}
}
```


---
title: 字节对齐与协议解析
categories:
  - 嵌入式开发
tags:
  - protocol
halo:
  site: https://mengplus.top
  name: a582ad94-ed88-46b7-9ddc-101a5cbf60c0
  publish: true
---
# 字节对齐与报文解析

```c
#pragma pack(show) //显示当前内存对齐的字节数，编辑器默认8字节对齐

#pragma pack(n) //设置编辑器按照n个字节对齐，n可以取值1,2,4,8,16

#pragma pack(push) //将当前的对齐字节数压入栈顶，不改变对齐字节数

#pragma pack(push,n) //将当前的对齐字节数压入栈顶，并按照n字节对齐

#pragma pack(pop) //弹出栈顶对齐字节数，不改变对齐字节数

#pragma pack(pop,n) //弹出栈顶并直接丢弃，按照n字节对齐
```
## 前言
擅长用字节对齐可以节省脑细胞，提高代码可读性，这里为大家展示下配置字节对齐来高效的解析报文。
## 示例场景
如下报文格式

| 字节                                                     | 类型                                                                   | 功能                                                                 | 说明                                                                                                                                                                                                                                         |
| ------------------------------------------------------ | -------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| CMD                                                    |                                                                      | 0x10                                                               |                                                                                                                                                                                                                                            |
| SCMD                                                   |                                                                      | 0x10                                                               |                                                                                                                                                                                                                                            |
| 2 <br>1 <br>1 <br>4 <br>1 <br>4<br>[4]<br> …           | U16<br>U8 <br>U8 <br>Float <br>U8 <br>Float<br>[Float] <br>…         | Type <br>Chnnl<br>State <br>Value <br>Num <br>AD1 <br>[AD2]<br> …  | 数据 1 的测量状态和测量值： <br>Type：表示数据类型，详见附录 B。 <br>Chnnl：表示数据通道 <br>State：表示数据状态 <br>    b7：数值有效/无效：1-有效，0-无效 <br>    b6~b3：为 0，预留 <br>    b2~b0：数值单位，详见附录 B。 <br>Value：表示数据测量值 <br>Num：电参量个数 <br>AD1：对应实测电参量 1 <br>AD2：对应实测电参量 2 <br>…：对应实测点参量 n |
| [2]<br>[1] <br>[1] <br>[4] <br>[1]<br>[4]<br>[4]<br> … | [U16] <br>[U8] <br>[U8] <br>[Float]<br>[U8]<br>[Float] <br>[Float] … | Type <br>Chnnl <br>State <br>Value <br>Num <br>AD1 <br>[AD2]<br> … | 数据 2 的测量状态和测量值：                                                                                                                                                                                                                            |
| …                                                      | …                                                                    |                                                                    | 数据 n 的测量状态和测量值：                                                                                                                                                                                                                            |

对应报文可以如下设计
```c
#pragma pack(1)
typedef struct _PROTOCOL_0X1010
{
    uint16_t type;
    uint8_t chnnl;
    uint8_t state;
    float value;
    uint8_t num;
    float AD[0];
} protocol_0x1010_t;

#pragma pack()
```
关键信息解释：
1. **#pragma pack(1)** 与 **#pragma pack()** 是成对出现的，否则将影响后续其它结构体的对齐问题，这里进行1字节对齐，牺牲了MCU的效率换来的是可读取，而牺牲的效率我认为比我们手动操作强的多。
2. **AD[0]** 柔性数组，在结构体中可以方便对后续数据的读取工作，AD多个参数便可之间访问，避免类型转换问题。
## 使用示例
```c
        uint32_t i = 0;
        while (i <= pfrm->length)
        {
            protocol_0x1010_ptr data_ptr = (protocol_0x1010_ptr) & (pfrm->text[i]);
            uint8_t sensor_idx = sensor_info_find_idx(data_ptr->type);
            if (sensor_idx < DEV_MAX_ID)
            {
                sensor_value_t *sensor_value_ptr = &m_sensor_module.value[sensor_idx];
                sensor_value_ptr->status.online = (data_ptr->state & 0x80) != 0;
                sensor_value_ptr->tick = tick_cur;
                sensor_value_ptr->meas = data_ptr->value;
                sensor_value_ptr->raw = data_ptr->AD[0];
                i += sizeof(protocol_0x1010_t) + data_ptr->num * sizeof(float);
            }

        }
```

---
title: 粘包的modbus 报文解析
slug: processmodbusrtu
categories:
  - 嵌入式开发
tags:
  - modbus
halo:
  site: https://mengplus.top
  name: a083d7bf-c668-4ebc-a394-82c2acd23fd2
  publish: true
---
# 粘包的modbus 报文解析

## 应用场景

1. 报文有垃圾数据
2. 粘包数据（一包数据有多个报文）

## 解决方案

### 使用效果演示

**实例代码**

```c
 主函数：测试和处理返回的帧
int main() {
    uint8_t modbusData[] = {
        0x00, 0x11, 0x81, 0x03, 0x02, 0x00, 0x00, 0x9A, 0x36,       // 第一帧
        0x81, 0x06, 0x00, 0x01, 0x00, 0x10, 0xC3, 0xB4,             // 第二帧
        0x81, 0x10, 0x00, 0x02, 0x00, 0x02, 0x04, 0x00, 0x11, 0x22, // 第三帧
        0xD3, 0xA5                                                  // CRC
    };
    uint16_t dataLength = sizeof(modbusData);
    uint8_t deviceAddress = 0x81;

    // 定义偏移量和长度数组
    uint16_t frameOffsets[10]; // 最多记录 10 帧的起始偏移量
    uint16_t frameLengths[10]; // 最多记录 10 帧的长度
    uint16_t frameCount = 0;
    uint16_t errorCount = 0;
    uint16_t maxFrames = 10;

    // 封装Modbus上下文
    ModbusContext context = {
        .buffer = modbusData,
        .length = dataLength,
        .deviceAddress = deviceAddress,
        .frameOffsets = frameOffsets,
        .frameLengths = frameLengths,
        .maxFrames = maxFrames,
        .frameCount = &frameCount,
        .errorCount = &errorCount
    };

    // 调用报文处理函数
    int result = ProcessModbusRTU(&context);
    if (result == 0) {
        printf("报文处理成功，共找到 %d 帧\n", frameCount);
        printf("校验失败的帧数：%d\n", errorCount);
        for (int i = 0; i < frameCount; i++) {
            printf("报文 %d 数据：", i + 1);
            for (int j = 0; j < frameLengths[i]; j++) {
                printf("0x%02X ", modbusData[frameOffsets[i] + j]);
            }
            printf("\n");
        }
    } else {
        printf("报文处理失败\n");
    }

    return 0;
}
```

**展示结果**

```bash
报文处理成功，共找到 3 帧
校验失败的帧数：0
报文 1 数据：0x81 0x03 0x02 0x00 0x00 0x9A 0x36
报文 2 数据：0x81 0x06 0x00 0x01 0x00 0x10 0xC3 0xB4
报文 3 数据：0x81 0x10 0x00 0x02 0x00 0x02 0x04 0x00 0x11 0x22 0xD3 0xA5

```

---



## 代码实现

**亮点**

1. 传参使用结构体 效率高
2. 解析的报文通过偏移量记录，节省RAM资源
3. 垃圾过滤
4. 异常记录

### 声明部分

```c
/**
 * @brief CRC-16 计算函数（多项式 0xA001）
 *
 * @param data
 * @param len
 * @return uint16_t
 */
uint16_t ModbusCRC16(const uint8_t *data, uint16_t length);
// 定义Modbus报文处理的上下文结构体
typedef struct
{
    uint8_t *buffer;        // 输入缓冲区
    uint16_t length;        // 输入缓冲区长度
    uint8_t deviceAddress;  // 设备地址
    uint16_t *frameOffsets; // 存储报文偏移量的数组
    uint16_t *frameLengths; // 存储报文长度的数组
    uint16_t maxFrames;     // 最多存储的报文数量
    uint16_t *frameCount;   // 记录处理到的有效报文数
    uint16_t *errorCount;   // 记录校验失败的报文数
} ModbusContext;


/**
 * @brief 报文处理
 *
 * @param context
 * @return int 返回值设计：
 * 0：成功处理所有报文
 * 1：校验失败（某些报文的 CRC 校验失败）
 * 2：功能码不支持（遇到未知的功能码）
 * 3：报文格式错误（如数据长度不符合预期）
 * 4：未完全处理所有数据（例如遇到粘包或报文长度不足）
 */
int ProcessModbusRTU(ModbusContext *context);
```

### 源码

```c

// CRC-16 计算函数（多项式 0xA001）
uint16_t ModbusCRC16(const uint8_t *data, uint16_t length)
{
    uint16_t crc = 0xFFFF;
    for (uint16_t i = 0; i < length; i++)
    {
        crc ^= data[i];
        for (uint8_t j = 0; j < 8; j++)
        {
            if (crc & 0x0001)
            {
                crc >>= 1;
                crc  ^= 0xA001;
            }
            else
            {
                crc >>= 1;
            }
        }
    }
    return crc;
}


// 获取报文实际长度（根据功能码）
uint16_t GetModbusDataLength(uint8_t functionCode, uint8_t *frame)
{
    switch (functionCode)
    {
    case 0x03:               // 读取保持寄存器
        return 3 + frame[2]; // 3字节功能码 + 数据字节数
    case 0x06:               // 写单个寄存器
        return 6;            // 设备地址、功能码、寄存器地址、寄存器值 + CRC
    case 0x10:               // 写多个寄存器
        return 7 + frame[6]; // 7字节固定头 + 数据字节数 + CRC
    default:
        return 0;            // 不支持的功能码
    }
}
// 功能码处理函数
int ProcessModbusRTU(ModbusContext *context)
{
    uint16_t i           = 0;
    *context->frameCount = 0;
    *context->errorCount = 0;

    while (i < context->length)
    {
        // 查找设备地址
        if (context->buffer[i] == context->deviceAddress)
        {
            // 提取功能码
            uint8_t functionCode = context->buffer[i + 1];
            uint16_t dataLength  = GetModbusDataLength(functionCode, &context->buffer[i]);

            // 如果功能码不支持或数据长度不足
            if (dataLength == 0 || i + dataLength + 2 > context->length)
            {
                i++; // 跳过当前字节
                //return 3; // 报文格式错误
                continue;
            }

            // 提取数据帧和CRC
            uint8_t *frame         = &context->buffer[i];
            uint16_t calculatedCRC = ModbusCRC16(frame, dataLength); // 校验包括设备地址和功能码
            uint16_t receivedCRC   = (context->buffer[i + dataLength] | (context->buffer[i + dataLength + 1] << 8));

            // 校验 CRC
            if (calculatedCRC != receivedCRC)
            {
                (*context->errorCount)++; // 记录错误
                i++;                      // 跳过当前字节
                //return 1;                 // 校验失败
                continue;
            }

            // 如果缓冲区已满，不再存储新帧
            if (*context->frameCount >= context->maxFrames)
            {
                return 4; // 未完全处理所有数据
            }

            // 保存帧偏移量和长度
            context->frameOffsets[*context->frameCount] = i;
            context->frameLengths[*context->frameCount] = dataLength + 2; // 数据长度 + CRC（包括设备地址和功能码）
            (*context->frameCount)++;

            // 跳过已处理的帧
            i += dataLength + 2; // 数据长度 + CRC
        }
        else
        {
            i++;
        }
    }
    return 0; // 成功处理所有报文
}

```

## 其它问题

1. 边接收边处理的需求怎么做?是否应该上FIFO缓冲?
   - 本设计的前提条件是整包的报文,如果您收到半包的报文是否意味着后续发送的报文丢失了,报文解析的触发条件这里规划的是帧接收完毕后触发报文解析工作.
   - 如果处理时间比较长,接收处确实应该放个FIFO作为临时缓存区
   - 本设计思想是支持使用FIFO作为数据缓冲的,可以根据需要改动
2. 遇到半包报文怎么处理?
   - 正如上文所说,报文处理逻辑为接收到一包报文后立刻处理,因此如解析过程中遇到半包则应是收发丢失,而不应等待或许会到来的另一半.
3. 是否只适合modbus 其他协议怎么移植使用?
   - 仔细阅读报文处理部分可清楚看到ProcessModbusRTU部分是报文解析与校验部分,它只负责满足modbus协议的校验,因此其他协议同样可以用相同的传参自行实现报文拆分校验部分

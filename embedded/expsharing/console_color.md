---
title: 控制台彩色打印信息
slug: console_color
categories:
  - 嵌入式开发
tags: []
halo:
  site: https://mengplus.top
  name: 57917299-7645-4485-b136-d58b38b3b704
  publish: true
---
>转自：[如何让你的C语言程序打印的log多一点色彩？](https://blog.csdn.net/daocaokafei/article/details/140731825

在平常的调试中，printf字体格式与颜色都是默认一致的。
如果可以根据log信息的重要程度，配以不同的颜色与格式，可以很方便的查找到要点。

## **1、printf字体显示语法说明**

```
printf(“\033[显示方式;字体颜色;背景颜色m 字符串 \033[0m” );
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaKFb2Jtu1wiaUZ0a5345Bj0e1HmZkvWoQz4u9GpjF5BaP2WJb25N4oKmg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

语法说明：

- 第一个**\033[**表示转义序列的开始,设置随后的字体格式 转义序列是以 **ESC** 开头,用 **\033** 完成相同的工作（ESC 的 ASCII 码用十进制表示就是 **27**， **=** 用八进制表示的 **33**）。

- 显示方式：

  0：默认值  1：高亮 、22：非粗体、4：下划线、24：非下划线、5：闪烁、25：非闪烁、7：反显、27：非反显

- 字体颜色

  30: 黑  31: 红 32: 绿 33: 黄 34: 蓝 35: 紫 36: 深绿 37: 白色

- 背景颜色

  40: 黑 41: 红 42: 绿 43: 黄 44: 蓝 45: 紫 46: 深绿 47: 白色

- 红色  'm'：

  表示转义序列的结束

- 结尾处的**\033[0m**是恢复默认值。

其他ANSI控制码：

```bash
    /033[0m 关闭所有属性
    /033[1m 设置高亮度
    /033[4m 下划线
    /033[5m 闪烁
    /033[7m 反显
    /033[8m 消隐
    /033[30m -- /033[37m 设置前景色
    /033[40m -- /033[47m 设置背景色
    /033[nA 光标上移n行
    /033[nB 光标下移n行
    /033[nC 光标右移n行
    /033[nD 光标左移n行
    /033[y;xH设置光标位置
    /033[2J 清屏
    /033[K 清除从光标到行尾的内容
    /033[s 保存光标位置
    /033[u 恢复光标位置
    /033[?25l 隐藏光标
    /033[?25h 显示光标
```

> 注意：其中 显示方式;字体颜色;背景颜色 可以任意组合，"；"隔开即可。使用 ANSI 转义码来设置文本样式和颜色可能会因为不同的终端软件和操作系统而产生不同的效果。同时，这种方式也只适用于在终端上输出，如果需要在 GUI 程序中设置文本颜色等效果，则需要使用相应的 GUI 库提供的接口。

## **2、举例**

```
 printf("\033[1;31mThis text is in red and bold.\033[0m\n");
 printf("\033[0;31mThis text is in red and not bold.\033[0m\n");
```

其中，'1' 表示加粗或高亮，'31' 表示前景色为红色，'\033[' 是转义序列的开始，'m' 是转义序列的结束，'\033[0m' 表示将属性重置为默认值。

运行结果：![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaKJwKSJibYypUgwcS3ica2ibHKe4bukxPX9hYv22t61YicLJFCp2kIfNQzFA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **方式**

```c
 printf("\033[0;36m****一口Linux*****【0;36m】\033[0m\r\n");
 printf("\033[1;36m****一口Linux*****【1;36m】\033[0m\r\n");
 printf("\033[4;36m****一口Linux*****【4;36m】\033[0m\r\n");
 printf("\033[5;36m****一口Linux*****【5;36m】\033[0m\r\n");
 printf("\033[7;36m****一口Linux*****【7;36m】\033[0m\r\n");
 printf("\033[8;36m****一口Linux*****【8;36m】\033[0m\r\n");
 printf("\033[22;36m****一口Linux*****【22;36m】\033[0m\r\n");
 printf("\033[24;36m****一口Linux*****【24;36m】\033[0m\r\n");
 printf("\033[25;36m****一口Linux*****【25;36m】\033[0m\r\n");
 printf("\033[27;36m****一口Linux*****【27;36m】\033[0m\r\n");
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaKicfAGl5EoiaYgOdicRk79LTUuSGxy4PZ4PZtzUibYIibia1rdvXGibHx0B6sg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **色谱**

测试代码[仅打印字体颜色]

```
    printf("\033[30m****一口Linux*****【30】\033[0m\r\n");
    printf("\033[31m****一口Linux*****【31】\033[0m\r\n");
    printf("\033[32m****一口Linux*****【32】\033[0m\r\n");
    printf("\033[33m****一口Linux*****【33】\033[0m\r\n");
    printf("\033[34m****一口Linux*****【34】\033[0m\r\n");
    printf("\033[35m****一口Linux*****【35】\033[0m\r\n");
    printf("\033[36m****一口Linux*****【36】\033[0m\r\n");
    printf("\033[37m****一口Linux*****【37】\033[0m\r\n");

    printf("\033[40m****一口Linux*****【40】\033[0m\r\n");
    printf("\033[41m****一口Linux*****【41】\033[0m\r\n");
    printf("\033[42m****一口Linux*****【42】\033[0m\r\n");
    printf("\033[43m****一口Linux*****【43】\033[0m\r\n");
    printf("\033[44m****一口Linux*****【44】\033[0m\r\n");
    printf("\033[45m****一口Linux*****【45】\033[0m\r\n");
    printf("\033[46m****一口Linux*****【46】\033[0m\r\n");
    printf("\033[47m****一口Linux*****【47】\033[0m\r\n");
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaK99eicsYCVQdBC5Q06Ecf1Rnz78udQqRrzNfH2tXdJVoibHwO0r762FCQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **3、给打印信息封装**

为方便打印字符串为不同颜色，我们可以将一些常用的颜色定义成宏

```
#define HL_TWK_RED_YEL  "\033[1m\033[5;31;43m" //闪烁高亮红字黄底
#define HL_RED_WRT      "\033[1;31;47m"   //高亮红色白底

#define HL_RED          "\033[1;31m"    //高亮红色
#define HL_GRN          "\033[1;32m"    //高亮绿色
#define HL_YEL          "\033[1;33m"    //高亮黄色
#define HL_DGRN          "\033[1;36m"    //高亮深绿

#define PF_CLR  "\033[0m"       //清除
```

将系统提供的printf函数做一个封装：

```
#define myprintf(color, format, args...)        \
    do{           \
            printf(color);             \
            printf(format, ##args);          \
            printf(PF_CLR);             \
    }while(0)
```

比如我们要打印字符串，显示为高亮黄色

```
myprintf(HL_YEL,"%s\n","yikoulinux");
```

## **4.  美化程序的打印log**

假设我们有如下格式的通信信令：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaKOHAIGJiakd6icbm0aJhXLpEaVnoSS9jHWfLFDN6sVicHV7Fibw24UaTSlQ/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)调试通信协议，

我们经常需要将通信的信令以16进制格式全部打印出来，

这些数据看起来非常不直观，

为方便查看log，将几个最重要字段显示出来，

比如msgType、len

```
void dump_frm(char *title,UINT8 *data,int len)
{
 int i=0;

 myprintf(HL_YEL,"%s\n",title);
 for(i=0;i<len;i++)
 {
  if(i==0){
   myprintf(HL_RED,"%02x ",data[i]);
  }else if(i==3 || i==4){
   myprintf(HL_DGRN,"%02x ",data[i]);
  }
  else{
   myprintf(HL_GRN,"%02x ",data[i]);
  }
 }
 putchar('\n');
}
```

将我们的测试针数据，放进去测试一下

```
 UCHAR frm[]={0x12,0x34,0x56,0x00,0x07,0x01,0x02,0x03,0x04,0x05,0x06,0x07};

dump_frm("frm<<<",frm,sizeof(frm));
```

执行结果：![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/icRxcMBeJfc9TF3bYLZmEP9FQicGGtvsaK7jL7YJ486dw49rMzmVPIRGqsteEK9kJbhFSaGv6MfhaiaibdCyurricZA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)可以看到，这种帧格式，看起来会更加直观，

## **5、完整代码**

国际惯例，贴上完整代码，

需要的老铁，直接拷贝带你们的项目里吧

```c
 #include <stdio.h>
#include <string.h>

typedef unsigned char UCHAR;
typedef unsigned char UINT8;
typedef unsigned short UINT16;
#pragma pack(1)
typedef struct protocol_msg_align{
 UINT8 msgType;
 UINT8 data1;
 UINT8 data2;
 UINT16 len;
 char data[100];
}PRO_MSG_ALIGN;
#pragma

#define HL_TWK_RED_YEL  "\033[1m\033[5;31;43m" //闪烁高亮红字黄底
#define HL_RED_WRT      "\033[1;31;47m"   //高亮红色白底

#define HL_RED          "\033[1;31m"    //高亮红色
#define HL_GRN          "\033[1;32m"    //高亮绿色
#define HL_YEL          "\033[1;33m"    //高亮黄色
#define HL_DGRN          "\033[1;36m"    //高亮深绿

#define PF_CLR  "\033[0m"       //清除


#define myprintf(color, format, args...)        \
    do{           \
            printf(color);             \
            printf(format, ##args);          \
            printf(PF_CLR);             \
    }while(0)

void dump_frm(char *title,UINT8 *data,int len)
{
 int i=0;

 myprintf(HL_YEL,"%s\n",title);
 for(i=0;i<len;i++)
 {
  if(i==0){
   myprintf(HL_RED,"%02x ",data[i]);
  }else if(i==3 || i==4){
   myprintf(HL_DGRN,"%02x ",data[i]);
  }
  else{
   myprintf(HL_GRN,"%02x ",data[i]);
  }
 }
 putchar('\n');
}

int main(int args, char *argv[])
{
 UCHAR frm[]={0x12,0x34,0x56,0x00,0x07,0x01,0x02,0x03,0x04,0x05,0x06,0x07};
 dump_frm("frm<<<",frm,sizeof(frm));
#if 0
 printf("\033[1;31mThis text is in red and bold.\033[0m\n");
 printf("\033[0;31mThis text is in red and not bold.\033[0m\n");


 printf("\033[0;36m****一口Linux*****【0;36m】\033[0m\r\n");
 printf("\033[1;36m****一口Linux*****【1;36m】\033[0m\r\n");
 printf("\033[4;36m****一口Linux*****【4;36m】\033[0m\r\n");
 printf("\033[5;36m****一口Linux*****【5;36m】\033[0m\r\n");
 printf("\033[7;36m****一口Linux*****【7;36m】\033[0m\r\n");
 printf("\033[8;36m****一口Linux*****【8;36m】\033[0m\r\n");
 printf("\033[22;36m****一口Linux*****【22;36m】\033[0m\r\n");
 printf("\033[24;36m****一口Linux*****【24;36m】\033[0m\r\n");
 printf("\033[25;36m****一口Linux*****【25;36m】\033[0m\r\n");
 printf("\033[27;36m****一口Linux*****【27;36m】\033[0m\r\n");

    printf("\033[30m****一口Linux*****【30】\033[0m\r\n");
    printf("\033[31m****一口Linux*****【31】\033[0m\r\n");
    printf("\033[32m****一口Linux*****【32】\033[0m\r\n");
    printf("\033[33m****一口Linux*****【33】\033[0m\r\n");
    printf("\033[34m****一口Linux*****【34】\033[0m\r\n");
    printf("\033[35m****一口Linux*****【35】\033[0m\r\n");
    printf("\033[36m****一口Linux*****【36】\033[0m\r\n");
    printf("\033[37m****一口Linux*****【37】\033[0m\r\n");

    printf("\033[40m****一口Linux*****【40】\033[0m\r\n");
    printf("\033[41m****一口Linux*****【41】\033[0m\r\n");
    printf("\033[42m****一口Linux*****【42】\033[0m\r\n");
    printf("\033[43m****一口Linux*****【43】\033[0m\r\n");
    printf("\033[44m****一口Linux*****【44】\033[0m\r\n");
    printf("\033[45m****一口Linux*****【45】\033[0m\r\n");
    printf("\033[46m****一口Linux*****【46】\033[m\r\n");
    printf("\033[47m****一口Linux*****【47】\033[0m\r\n");

 #endif
    return 0;
}
```

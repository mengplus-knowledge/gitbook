## 前言
这里为大家介绍一个轻量型的UI框架的设计原理，面向对象的设计思想，每个页面都独立设计扩展性强，底层切换采用状态机结构，暂时无源码例程，仅做架构交流。

```C


#ifndef __GUI_H_
#define __GUI_H_

#include "CommLCD.h" //LCD屏幕驱动
#include "IncHeads.h"
// #include "core_main.h"
#include "globalData.h"
#include "gltypes.h" //GL数据类型
#include "ohos_init.h"
#include "varDef.h" //参数模块
/*
===============================================================================
==
===============================================================================
*/
/**
 *****************************************************************************
 * @note 此处为了将所有页面句柄在链接阶段放到一起，实现动态长度数组的功能
 * 这样在增删页面时只需要操作*.C文件，无需改动框架部分
 *  @attention
 * 移植时请注意 IAR中 需在链接文件*.lcf 中添加keep {section .zinitvalue.*.init};
 * 来避免页面变量(pagedata)被优化掉
 *
 *****************************************************************************
 **/
// you need add "keep {section .zinitvalue.*.init};" to *.lcf
#ifndef USED_ATTR
#define USED_ATTR __attribute__((used))
#endif

#ifndef MODULE_VALUE_NAME
#define MODULE_VALUE_NAME(name, step) ".zinitvalue." #name #step ".init"
#define MODULE_VALUE_BEGIN(name, step) __section_begin(MODULE_VALUE_NAME(name, step))
#define MODULE_VALUE_END(name, step) __section_end(MODULE_VALUE_NAME(name, step))
#endif

typedef void *InitValue;
#define LAYER_INITVALUE(value, layer, clayer, priority) \
  static GUI_PAGE_T USED_ATTR value __attribute__((section(MODULE_VALUE_NAME(clayer, priority))))

#pragma section = MODULE_VALUE_NAME(GUI.page, 2) // 声明存放页面数据的section
#define STATIC_GUI_PAGE_STRUCT(value) LAYER_INITVALUE(value, GUI_page, GUI.page, 2)
#define GUI_PAGE_BEGIN() (GUI_PAGE_PTR)(MODULE_VALUE_BEGIN(GUI.page, 2)) //__section_begin(MODULE_NAME(name, step))
#define GUI_PAGE_END() (GUI_PAGE_PTR)(MODULE_VALUE_END(GUI.page, 2))     //__section_end(MODULE_NAME(name, step))
/******************************************************************************************************************/
typedef struct
{
  int id;                        // 页面ID
  const char *name;              // 页面名称
  void (*init)(void);            // 创建
  void (*close)(void);           // 关闭
  void (*draw)(uint32 flag);     // 刷新
  void (*deal)(LCD_CMD_T *pcmd); // 响应处理
} GUI_PAGE_T, *GUI_PAGE_PTR;

/* Exported macro ------------------------------------------------------------*/
#define GUI_VERSION "V2.00 20230508"
/*
===============================================================================
== 初始化
===============================================================================
*/
void GUI_Init(void);
void GUI_Handler(void);

void GUI_PagePrevious(); // 上一页
void GUI_PageSet(const GUI_PAGE_T *pg);
void GUI_PageSetPageId(const int pgid);
void GUI_PageSetName(const char *name);
const GUI_PAGE_T *GUI_PageGet(void);
const GUI_PAGE_T *GUI_PageGetForName(const char *name);
void GUI_Refresh(uint32 flag_mask);

#endif

```
源码部分
```C

#include "GUI.h"
#include "CommLCD.h"
#include <string.h>

/*
===============================================================================
==
===============================================================================
*/
struct
{
	const GUI_PAGE_T *pg_last;
	const GUI_PAGE_T *pg_curr;
	const GUI_PAGE_T *pg_next;
	uint32 flg_draw;
} s_varGUI;

/*
===============================================================================
==
===============================================================================
*/
static void GUI_CommandDeal(LCD_CMD_T *pcmd);

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
void GUI_Init(void)
{
	CommLCD_Init(GUI_CommandDeal);

	/*加入工程时 为每个页面登记页面ID 号*/
	GUI_PageSetPageId(0);
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
void GUI_Handler(void)
{
	// 创建
	if (s_varGUI.pg_curr != s_varGUI.pg_next)
	{
		if (s_varGUI.pg_curr != 0 && s_varGUI.pg_curr->close != 0)
			s_varGUI.pg_curr->close();
		s_varGUI.pg_last = s_varGUI.pg_curr;
		s_varGUI.pg_curr = s_varGUI.pg_next;

		if (s_varGUI.pg_curr != 0 && s_varGUI.pg_curr->init != 0)
			s_varGUI.pg_curr->init();

		s_varGUI.flg_draw = 0xffffffff;
	}

	// 刷新
	if (s_varGUI.flg_draw)
	{
		uint32 flag;

		flag = s_varGUI.flg_draw;
		s_varGUI.flg_draw = 0;

		if (s_varGUI.pg_curr != 0 && s_varGUI.pg_curr->draw != 0)
			s_varGUI.pg_curr->draw(flag);
	}
}
void GUI_PagePrevious() // 上一页
{
	if (s_varGUI.pg_last != NULL)
	{
		GUI_PageSet(s_varGUI.pg_last);
		s_varGUI.pg_last = NULL;
	}
}
/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
void GUI_PageSet(const GUI_PAGE_T *pg)
{
	RT_ASSERT(, pg != NULL, return;);
	s_varGUI.pg_next = pg;
}
/*
*********************************************************************************
** 函数名称: 设定页面
** 功    能: 切换到指定ID页面
** 输入参数: 页面ID
** 输出参数: NULL
** 返    回: Void
**********************************************************************************
*/
void GUI_PageSetPageId(const int pgid)
{
	GUI_PAGE_PTR pg = GUI_PAGE_BEGIN();
	for (; pg < GUI_PAGE_END(); pg++)
	{
		if (pg->id == pgid)
		{
			GUI_PageSet(pg);
			break;
		}
	}
}
/*
*********************************************************************************
** 函数名称: 设定页面
** 功    能: 切换到指定name页面
** 输入参数: 页面name
** 输出参数: NULL
** 返    回: Void
**********************************************************************************
*/
void GUI_PageSetName(const char *name)
{
	const GUI_PAGE_T *pg = NULL;
	pg = GUI_PageGetForName(name);
	if (pg != NULL)
		GUI_PageSet(pg);
}
/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
const GUI_PAGE_T *GUI_PageGet(void)
{
	return s_varGUI.pg_curr;
}

const GUI_PAGE_T *GUI_PageGetForName(const char *name)
{
	RT_ASSERT(, name != NULL, return 0);

	GUI_PAGE_PTR pg = GUI_PAGE_BEGIN();
	for (; pg < GUI_PAGE_END(); pg++)
	{
		if (strcmp(name, pg->name) == 0)
		{
			return pg;
		}
	}
	return NULL;
}
/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
void GUI_Refresh(uint32 flag_mask)
{
	s_varGUI.flg_draw |= flag_mask;
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void GUI_CommandDeal(LCD_CMD_T *pcmd)
{
	if (s_varGUI.pg_curr != 0 && s_varGUI.pg_curr->deal != 0)
	{
		RT_ASSERT(, pcmd != 0, return);
		s_varGUI.pg_curr->deal((LCD_CMD_T *)pcmd);
	}
}

```

其中一个页面
```c

/* Includes ------------------------------------------------------------------*/
#include "GUI.h"
#include "hSch.h"
#include "DV_Fm31xx.h"
/* Private typedef -----------------------------------------------------------*/
enum CONTROL_ID
{
    E_PROGRESS = 0x01,
    E_T_ST_LOG = 0x02,
    E_T_VER = 0x04,
};
/* Private macro -------------------------------------------------------------*/

/* Private function prototypes -----------------------------------------------*/
static void init();                // 创建
static void close();               // 关闭
static void draw(uint32 flag);     // 刷新
static void deal(LCD_CMD_T *pcmd); // 响应处理
static void drawCtrl(uint8_t id);  //刷新制定ID控件
/* Private variables ---------------------------------------------------------*/
// 备注：将pageData放入指定section 并在link里指定保留它，
// 这样便能从编译层面将所有的页面句柄放到一起,避免了全局变量指针的使用
STATIC_GUI_PAGE_STRUCT(pageData)  = {
    .id = 0,
    .name = "启动界面",
    .init = init,
    .close = close,
    .draw = draw,
    .deal = deal,
};
static const char *strLog = NULL; //启动提示字符串
static uint8_t progress = 0;
static void page_Refresh()
{
    progress += 1;
    // progress += 5;
    // GUI_Refresh(0x01 << E_PROGRESS);
    // GUI_Refresh(0x01 << E_T_ST_LOG);
    if (progress > 100)
    {
        GUI_PageSetPageId(1);
    }
    if (progress == 20)
    {
        strLog = "外设上电";
    }
    else if (progress == 50)
    {
        strLog = "设备初始化...";
    }

    else if (progress == 60)
    {
        strLog = "参数初始化...";
    }
    else if (progress == 80)
    {
        strLog = "测量初始化...";
    }
    if (progress == 90)
    {
        strLog = "系统启动完成";
    }
    GUI_Refresh(0x01 << E_PROGRESS);
    GUI_Refresh(0x01 << E_T_ST_LOG);
    GUI_Refresh(0x01 << E_T_VER);
}

/* Exported functions --------------------------------------------------------*/
/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
GUI_PAGE_PTR StartPage_PageGet(void)
{
    return &pageData;
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void init() // 创建
{
    LCD_ScreenSet(pageData.id);
    progress = 0;
    strLog = "启动中";
    HSchAddTask(page_Refresh, 7, 100, 1);
    LCD_RtcTimeSet(rtcTime.year % 100, rtcTime.month, rtcTime.date, rtcTime.hour, rtcTime.minute, rtcTime.second);
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void close() // 关闭
{
    HSchDeleteTask(page_Refresh);
    LCD_RtcTimeSet(rtcTime.year % 100, rtcTime.month, rtcTime.date, rtcTime.hour, rtcTime.minute, rtcTime.second);
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void draw(uint32 flag) // 刷新
{
    for (size_t i = 0; i < sizeof(flag) * 8; i++)
    {
        if ((flag & (0x0001 << i)) != 0)
        {
            drawCtrl(i);
        }
    }
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void drawCtrl(uint8_t id)
{
    switch (id)
    {
    case E_PROGRESS:           //更新启动进度
        if (progress < 100)
        {
            LCD_ProcessValueSet(pageData.id, E_PROGRESS, progress);
        }
        break;
    case E_T_ST_LOG:    //更新启动日志
        RT_ASSERT(, strLog != 0, return );
        LCD_TextValueSet(pageData.id, E_T_ST_LOG, strLog);
        break;
    case E_T_VER: // 显示版本号
    {
        char buff[64];
        sprintf(buff, " SV:%s", SOFT_VER);
        LCD_TextValueSet(pageData.id, E_T_VER, buff);
    }
    break;
    default:
        break;
    }
}

/*
*********************************************************************************
** 函数名称:
** 功    能:
** 输入参数:
** 输出参数:
** 返    回:
**********************************************************************************
*/
static void deal(LCD_CMD_T *pcmd) // 响应处理
{
    RT_ASSERT(, pcmd != 0, return );
    RT_ASSERT(, pcmd->ctl_screen != pageData.id, LCD_ScreenSet(pageData.id); return;);
}
```
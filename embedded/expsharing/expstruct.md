---
title: 结构体设计
categories:
  - 编程设计
tags:
  - C语言
halo:
  site: https://mengplus.top
  name: ccaae558-7f8e-4c13-a679-08101c5c48eb
  publish: true
---
# 结构体设计

## 结构体设计原则

1. **成员变量尽量少**：结构体的成员变量应该尽量精简，避免过多的冗余数据，以节省内存空间和提高访问效率。

2. **成员变量尽量按需使用**：只在需要时添加成员变量，避免无意义的变量增加结构体的复杂度。

3. **成员变量同类型合并**：相同类型的成员变量可以合并在一起，减少结构体的大小，提高数据的紧凑性。

4. **成员变量按照4字节对齐**：结构体的成员变量应该按照系统的字节对齐规则进行排列，通常是按照4字节对齐，以提高访问效率。

5. **成员变量命名符合其功能**：成员变量的命名应该清晰明了，符合其所表示的功能，避免使用过于简单或者模糊的命名。

6. **类型命名重于类型注释**：在设计结构体时，应该更注重命名的规范和清晰，而不是过多地依赖于注释来解释结构体的含义。命名应该能够直观地反映出结构体的用途和成员变量的含义。

当设计一个结构体时，需要考虑以下几个方面：

### 1. 数据类型选择

选择合适的数据类型来表示结构体的成员变量，确保能够准确地存储和表示数据，同时尽量节省内存空间。

### 2. 数据成员的功能和关联性

确保结构体中的每个成员变量都有清晰明确的功能，并且彼此之间有一定的关联性，以便于后续的使用和理解。

### 3. 成员变量的顺序

将成员变量按照其功能或者使用频率排序，便于查找和访问。同时，考虑系统的字节对齐规则，确保成员变量的对齐以提高访问效率。

### 4. 结构体的命名

结构体的命名应该简洁明了，能够准确地反映出结构体的用途。避免使用过于晦涩或者含糊的命名，以提高代码的可读性。

### 5. 结构体的扩展性和维护性

考虑到后续可能的需求变化，设计结构体时应该尽量保持灵活性和可扩展性，避免过度耦合和复杂性。

### 6. 成员变量的命名规范

遵循命名规范，例如使用驼峰命名法或下划线命名法，保持一致性和规范性，便于团队协作和代码维护。
## 常见的设计问题
### 1. 过度设计

过度设计指的是在结构体中添加过多的成员变量或者过于复杂的成员变量，导致结构体变得臃肿和难以理解。比如：

```c
struct OverDesignedStruct {
    char name[100];             // 过长的姓名字段
    int age;                    // 年龄
    float height;               // 身高
    float weight;               // 体重
    char address[200];          // 过长的地址字段
    char email[100];            // 过长的邮箱字段
    int phoneNumbers[10];       // 过长的电话号码数组
    // 更多过度设计的成员变量...
};
```

### 2. 不合理的成员变量顺序

如果成员变量的顺序不合理，可能会导致内存空间的浪费，或者访问效率降低。

```c
struct UnorderedStruct {
    int age;            // 年龄
    float height;       // 身高
    char name[50];      // 姓名
    char gender;        // 性别
    // 更多不合理的成员变量顺序...
};
```

### 3. 没有考虑到扩展性和灵活性

设计结构体时如果没有考虑到后续可能的需求变化，可能会导致结构体不易扩展，难以满足新的需求。

```c
struct NoFlexibilityStruct {
    int id;             // 编号
    char name[50];      // 姓名
    float score;        // 分数
    // 没有预留扩展的成员变量...
};
```

### 4. 不清晰的命名和注释

如果结构体的命名不清晰或者没有足够的注释说明，会降低代码的可读性和维护性。

```c
struct UnclearNamedStruct {
    char n[50];     // 不清晰的命名
    int a;          // 不清晰的命名
    float h;        // 不清晰的命名
    float w;        // 不清晰的命名
    char g;         // 不清晰的命名
    // 缺乏注释说明...
};
```

### 5. 缺乏错误处理和数据验证

结构体中的数据缺乏验证和错误处理可能会导致程序运行时出现异常或者不确定的行为。

```c
struct NoErrorHandlingStruct {
    int age;            // 年龄
    float height;       // 身高
    float weight;       // 体重
    char gender;        // 性别
    // 缺乏数据验证...
};
```

避免这些失败的方式，设计一个简洁、清晰和易于维护的结构体是十分重要的。

## 结构体优化示例
### 1. 成员变量合并
```c
VAR_DEF_EXTERN struct
{
	float CH4;	/* 甲烷			*/
	float C2H2; /* 乙炔			*/
	float C2H4; /* 乙烯			*/
	float O2;	/* 氧气			*/
	float CO;	/* 一氧化碳		*/
	float CO2;	/* 二氧化碳		*/
	float C2H6; /* 乙烷		*/
	float H2;	/* 氢气		*/
	float N2;	/* 氮气		*/

	float H2S;
	float C3H6;
	float SO2;
	float NO2;

	// HWX 2022.08.25 ADD.
	float CH4_2; /* 甲烷			*/
	uint8 ch4_2Valid;
} orgData; /* 通道测量值					*/
```
**存在问题**：
1. 没有考虑扩展性，如果有成员增删，需要重新定义结构体。
2. 不同意义的变量放在一起，ch4_2Valid这个变量与前面类型格格不入。

**优化结果**
```c
typedef enum GAS_TYPE
{
	TYPE_CH4,
	TYPE_C2H2,
	TYPE_C2H4,
	TYPE_O2,
	TYPE_CO,
	TYPE_CO2,
	TYPE_C2H6,
	TYPE_H2,
	TYPE_N2,
	TYPE_H2S,
	TYPE_C3H6,
	TYPE_SO2,
	TYPE_NO2,
	TYPE_CH4_2,
	GAS_TYPE_MAX
} GAS_TYPE_E;

VAR_DEF_EXTERN struct
{
	float value[GAS_TYPE_MAX];
	uint8 ch4_2Valid;
} orgData;
```
### 2. 成员变量初始化
```c
#define DEV_MAX_ID			20		// 最大ID号

#define BOARD_ID_CH4		1
#define BOARD_ID_C2H2		2
#define BOARD_ID_C2H4		3
//...
#define BOARD_ID_C3H6		15
#define BOARD_ID_SO2		16
#define BOARD_ID_NO2		17
#define BOARD_ID2_CH4		18

static const struct{
	uint16	type;
	uint8 	addr;
    uint8 	ID;
	uint8	chnnl;
	uint8	*pen;
}calibTypeTab[] = {
	{0, 	199,         199,			 0,	(uint8 *)&flgDis},		//"请选择...",
	{100,	Addr_CH4,    BOARD_ID_CH4,	 0,	&prodParm.enGasCH4},	//"甲烷(CH4)",		1
    //...
	{100,	Addr_CH4_2,  BOARD_ID2_CH4,	 0,	&prodParm.enGasCH4},	//"甲烷(CH4)",		18
};
```
**存在问题**：
1. ID号人工指定序号没有意义，最后还要统计有多少成员，使用**define**是一个失败的设计，应使用enum自动编号功能
2. calibTypeTab数组中ID编号并未发挥应有的功能，既然BOARD_ID_XXX已经顺序编号，则用它作为数组序号更能节省空间

**优化结果**
```c
typedef enum DEV_BOARD_ID
{
	BOARD_ID_CH4,
	BOARD_ID_O2,
//...
	BOARD_ID2_CH4,
	DEV_MAX_ID // 最大ID号
} DEV_BOARD_ID;

static const struct _CALIB_TYPE
{
	uint16_t type;
	uint8_t addr;
	uint8_t chnnl;
} calibTypeTab[] = {
	[BOARD_ID_O2] = {101, 15, 0},	//"氧气(O2)",		4
	[BOARD_ID_H2] = {104, 15, 0},	//"氢气(H2)",		8
	[BOARD_ID_H2S] = {113, 15, 0},	//"H2S",			14
	[BOARD_ID_C3H6] = {117, 15, 0}, //"C3H6",			15
	[BOARD_ID_SO2] = {118, 15, 0},	//"SO2",			16
	[BOARD_ID_NO2] = {111, 15, 0},	//"NO2",			17
	[BOARD_ID2_CH4] = {100, 2, 0},	//"甲烷(CH4)",		18
};
```


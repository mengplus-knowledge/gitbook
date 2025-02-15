---
title: bin文件转换为目标文件导入到工程
slug: add_bin_to_project
cover: ""
categories:
  - 嵌入式开发
tags:
  - C语言
halo:
  site: https://mengplus.top
  name: 49047fe4-df43-4c84-b55c-2ebc84727bb8
  publish: true
---
# bin文件如何转换为目标文件导入到工程

​	在嵌入式开发过程中，往往需要将二进制数据导入到工程之中参与编译，如初始化配置数据、图像、预制参数等，很多操作要么写入使用合并到代码指定段，要么直接存储到指定区域的flash中。这里我提供一个解决方案，将bin文件转换为.o文件，直接添加到工程中参与编译，不指定位置，随它放哪里吧。

## 准备环境

本次实验背景是使用MounRiver Studio 为ch32v003开发一款量产eeprom 24c08的小工具，要求自动填充eeprom指定的数据，当前我拿到了24c08.bin完整的1kb数据用于烧录，因此我需要将这1k数据加载到程序中调用。

1. MounRiver Studio软件
2. gcc软件包位置D:\MounRiver\MounRiver_Studio\toolchain\RISC-V Embedded GCC\bin\riscv-none-embed-XXX

通过询问deeseek获得几个解决方案

1. .bin文件重新生成.h文件，将数据存入数组，这个方式最为简单，缺陷是只适合小文件，通用性欠佳
2. 使用objcopy将bin文件直接转换为.o文件参与编译，这个更具通用性，同时也更加复杂
3. 使用汇编指令`incbin`将其加入工程中
4. 将bin文件拼接到hex再刷入程序中

## 实现方案

### 1.bin文件转为h源码文件

在linux环境中安装xxd工具，执行下方命令即可成功生成源码文件

```c
xxd -i 24c08.bin > 24c08.h
```

### **2.直接合并BIN文件到Hex**

如果你希望将 `24c08.bin` 直接合并到最终的Hex文件中，可以使用工具（如 `srec_cat`）操作：


```bash
srec_cap firmware.hex -Intel 24c08.bin -Binary -offset 0x08010000 -o combined.hex -Intel
```

- `-offset 0x08010000`: 指定二进制数据在Flash中的起始地址。

### 3.使用汇编加载bin文件

`.incbin`可以方便的将外部的二进制文件添加到.s中，使用编译链ar完成转化工作，而且无需额外对bin文件操作，方便批量的bin文件添加

```assembly
.section .rodata.eeprom_data, "a"  # 定义只读段
.align 4
.global eeprom_data_start          # 导出符号
eeprom_data_start:
  .incbin "../applications/system_var/24c08.bin"              # 嵌入二进制文件
.global eeprom_data_end
eeprom_data_end:
  .byte 0                          # 可选：添加结束符

/* 显式定义数据长度符号 */
.global eeprom_data_size
eeprom_data_size:
  .int eeprom_data_end - eeprom_data_start

```

C代码中使用导出的符号

```c
extern const uint32_t eeprom_data_size;
extern const uint8_t eeprom_data_start[];
extern const uint8_t eeprom_data_end[];
```



### 4.bin文件生成object文件

当前操作在win环境下实现

**方法概述**

1. **将BIN文件转换为目标文件**：使用工具（如 `objcopy`）将 `.bin` 文件转换为 `.o` 目标文件。
2. **修改链接脚本**：在链接脚本中为二进制数据分配固定的存储地址。
3. **在代码中通过地址访问数据**：通过符号或绝对地址直接操作二进制数据。

### 1. bin生成.o操作

```bash
d:/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC/bin/riscv-none-embed-objcopy'  -I binary -O elf32-littleriscv  -B riscv:rv32 --rename-section .data=.rodata.24c08    24c08.bin 24c08.o
```



- **关键参数说明**：
  - `-I binary`: 输入文件格式为二进制。
  - `-O elf32-littleriscv `: 输出为RISCV小端格式的ELF文件。
  - `--rename-section .data=.rodata`: 将段名从 `.data` 改为 `.rodata`（避免被初始化到RAM）。
  - `-B riscv:rv32 `:设置输出平台类型



### 2.查看符号表

```bash
$ 'd:/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC/bin/riscv-none-embed-nm'   24c08.o
00000400 R _binary_24c08_bin_end
00000400 A _binary_24c08_bin_size
00000000 R _binary_24c08_bin_start
```



### 查看段信息

查看这里的配置我们可以知道数组转换后我们得到3个参数，分别是数组截至地址、长度、起始地址，这样便满足我们的使用需求

```bash
$ 'd:/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC/bin/riscv-none-embed-objdump'   -h 24c08.o

24c08.o:     file format elf32-littleriscv

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .rodata.24c08 00000400  00000000  00000000  00000034  2**0
                  CONTENTS, ALLOC, LOAD, DATA

```



### 3.  验证目标文件架构信息

通过使用`readelf`可以方便的看到我们生成的目标文件编译信息

```bash
 'd:/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC/bin/riscv-none-embed-readelf.exe'  -h -A  24c08.o
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           RISC-V
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          1268 (bytes into file)
  Flags:                             0x9, RVC, RVE, soft-float ABI
  Size of this header:               52 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           40 (bytes)
  Number of section headers:         5
  Section header string table index: 4
```

### 3.**检查.o文件内容**：

```bash
riscv32-unknown-elf-objdump -s 24c08.o

24c08.o:     file format elf32-littleriscv

Contents of section .rodata.24c08:
 0000 32445550 39334e32 30364200 00000000  2DUP93N206B.....
 0010 00000000 00000000 00000000 00000000  ................
 0020 00000000 00000000 00000000 00000000  ................
 0030 00000000 00000000 00000000 00000000  ................
```

确认 `.rodata.24c08` 段包含二进制数据。

## 在代码中访问二进制数据

如果您未在工程中显式调用它的几个符号使用绝对地址访问，可能导致系统优化掉。

这里给出链接文件改动方案

```json
MEMORY
{
  FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
  RAM (xrw) : ORIGIN = 0x20000000, LENGTH = 128K
}

/* 添加自定义段存放二进制数据 */
SECTIONS
{
  .text : { *(.text*) } > FLASH
  .rodata : {
    *(.rodata*)        /* 原有只读数据 */
    _24c08_data = .;   /* 记录当前地址 */
    KEEP(24c08.o(.rodata))  /* 强制包含24c08.o的.rodata段 */
    . = ALIGN(4);      /* 4字节对齐 */
  } > FLASH

  /* 其他段（如.data、.bss等） */
}
```

**显式调用**，编译链可以自动将其加入到工程中，在工程中进行如下声明，他们是通过`riscv-none-embed-nm`查看到的符号。

在用到的地方如下进行定义即可拿到数据地址信息

```c
extern const uint32_t _binary_24c08_bin_size;// 数据长度（字节）
extern const uint8_t _binary_24c08_bin_start[];// 数据起始地址
extern const uint8_t _binary_24c08_bin_end[];  // 数据结束地址
```

**其它操作**

在链接脚本中为二进制数据 **显式分配固定地址**，避开主程序占用的区域：

```json
SECTIONS
{
  .text : { *(.text*) } > FLASH
  .rodata : {
    *(.rodata*)
  } > FLASH

  /* 将二进制数据固定在Flash的末尾 */
  .bin_section 0x2000F000 : {  /* 假设Flash结束地址为0x20010000 */
    KEEP(24c08.o(.rodata))
  } > FLASH
}
```

## **生成静态库 `.a` 文件**

使用 `ar` 工具（归档工具）将 `24c08.o` 打包为静态库：

```
riscv-none-embed-ar rcs lib24c08.a 24c08.o
```

#### **参数说明**

- `rcs`：
  - `r`：替换或添加文件到库
  - `c`：创建库（如果不存在）
  - `s`：生成索引（加速链接）
- `lib24c08.a`：生成的静态库名称（前缀 `lib` 是标准命名约定）
- `24c08.o`：输入的目标文件

## **常见错误及解决**

#### **错误1：命令未找到**

```bash
riscv32-unknown-elf-objcopy: command not found
```

**原因**：工具链未安装或环境变量未配置。或者使用绝对路径访问比如`'d:/MounRiver/MounRiver_Studio/toolchain/RISC-V Embedded GCC/bin/riscv-none-embed-objcopy'   -O elf32-littleriscv  -B riscv:rv32 --rename-section .data=.rodata.24c08    24c08.bin 24c08.o`

#### **错误2：格式不支持**

```
riscv32-unknown-elf-objcopy: unsupported architecture
```

**原因**：`-O` 参数未指定正确的目标格式。
**解决**：根据架构选择格式：

- 32位小端：`elf32-littleriscv`
- 64位小端：`elf64-littleriscv`

#### **错误3：段名冲突**

```
multiple definition of `_binary_24c08_bin_start`
```

**原因**：未重命名二进制文件的段名。
**解决**：添加 `--rename-section .data=.rodata`。

#### 错误4：objcopy生成的 `.o` 文件缺少 RISC-V 架构特定的Flags 标志

 **使用 `objcopy` 强制设置 Flags**

在转换命令中通过 `--set-flags` 参数手动设置 Flags 为 `0x9`：

```bash
riscv-none-embed-objcopy \
  -I binary \
  -O elf32-littleriscv \
  --rename-section .data=.rodata,alloc,load,readonly,data,contents \
  --set-flags 0x9 \  # 强制设置 Flags 为 0x9（RVC + RVE）
  24c08.bin \
  24c08.o
```

但是我使用的版本不支持这个命令，我通过比对工程生成的.o文件与objcopy生成的差异比对，将一个位写成0x09成功解决


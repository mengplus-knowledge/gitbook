---
title: MSH 宏展开中学习可变参数宏
slug: msh_cmd_export
categories:
  - 编程设计
tags: []
halo:
  site: https://mengplus.top
  name: d1bc37d3-f316-4851-b696-e615c998e55a
  publish: true
---
# MSH 宏展开中学习可变参数宏

**rt-thread版本**:V5.2

rt-thread在近期版本中增加了 子命令的补全功能，让本就复杂的宏更加复杂多变，这里做简单的介绍方便我们使用与学习



## 使用介绍

```c
#ifdef RT_USING_FINSH
#include <finsh.h>
static void reboot(uint8_t argc, char **argv)
{
    rt_hw_cpu_reset();
}
MSH_CMD_EXPORT(reboot, Reboot System);
//MSH_CMD_EXPORT_ALIAS(reboot,reboot, Reboot System)
#endif /* RT_USING_FINSH */
```

1. MSH_CMD_EXPORT介绍

   ```c
   MSH_CMD_EXPORT([函数名], [描述]);
   ```

   使用这个宏默认将函数名作为命令名

2. MSH_CMD_EXPORT_ALIAS介绍

   ```
   MSH_CMD_EXPORT_ALIAS([函数名],[命令的别名], [描述])
   ```

   使用这个宏，将无所谓函数名称，以别名作为函数名

   前者是这个的简化版本，完全可以复用

   接下来说下他的展开过程

## 展开过程

首先我们需要先了解几个宏的展开

```c++
/* MSH_CMD_EXPORT(command, desc) or MSH_CMD_EXPORT(command, desc, opt) */
#define MSH_CMD_EXPORT(...)                                 \
    __MSH_GET_MACRO(__VA_ARGS__, _MSH_FUNCTION_CMD2_OPT,    \
        _MSH_FUNCTION_CMD2)(__VA_ARGS__)
/* #define MSH_CMD_EXPORT_ALIAS(command, alias, desc) or
   #define MSH_CMD_EXPORT_ALIAS(command, alias, desc, opt) */
#define MSH_CMD_EXPORT_ALIAS(...)                                           \
    __MSH_GET_EXPORT_MACRO(__VA_ARGS__, _MSH_FUNCTION_EXPORT_CMD3_OPT,      \
            _MSH_FUNCTION_EXPORT_CMD3)(__VA_ARGS__)


#define __MSH_GET_MACRO(_1, _2, _3, _FUN, ...)  _FUN
#define __MSH_GET_EXPORT_MACRO(_1, _2, _3, _4, _FUN, ...) _FUN

#define _MSH_FUNCTION_CMD2(a0, a1)       \
        MSH_FUNCTION_EXPORT_CMD(a0, a0, a1, 0)

#define _MSH_FUNCTION_CMD2_OPT(a0, a1, a2)       \
        MSH_FUNCTION_EXPORT_CMD(a0, a0, a1, a0##_msh_options)

#define _MSH_FUNCTION_EXPORT_CMD3(a0, a1, a2)       \
        MSH_FUNCTION_EXPORT_CMD(a0, a1, a2, 0)

#define MSH_FUNCTION_EXPORT_CMD(name, cmd, desc, opt)                                  \
                const char __fsym_##cmd##_name[] rt_section(".rodata.name") = #cmd;    \
                const char __fsym_##cmd##_desc[] rt_section(".rodata.name") = #desc;   \
                rt_used const struct finsh_syscall __fsym_##cmd rt_section("FSymTab")= \
                {                           \
                    __fsym_##cmd##_name,    \
                    FINSH_DESC(cmd, desc)   \
                    FINSH_COND(opt)         \
                    (syscall_func)&name     \
                };
```

从宏定义上来看`MSH_CMD_EXPORT`,`MSH_CMD_EXPORT_ALIAS`,这里我用MSH_CMD_EXPORT的展开过程为大家介绍

```C
#define MSH_CMD_EXPORT(command, desc)
#define MSH_CMD_EXPORT(...)   __MSH_GET_MACRO([command, desc], _MSH_FUNCTION_CMD2_OPT, _MSH_FUNCTION_CMD2)([command, desc])
#define __MSH_GET_MACRO([command, desc], _MSH_FUNCTION_CMD2_OPT, _MSH_FUNCTION_CMD2)([command, desc])  _MSH_FUNCTION_CMD2([command, desc])
#define _MSH_FUNCTION_CMD2(command, desc)([command, desc])
```

从`#define MSH_CMD_EXPORT(...)  `可以知道这里用了可变参数宏  `...`里代表你可以输入不定长的参数，在后续均可用`__VA_ARGS__`代替你所输入的所有参数，他与`printf`中的可变参数方式类似。

这里为了便于理解 我将您输入的可变参数设定为 command, desc

1. 输入参数2个 得到宏 , 根据介绍`__VA_ARGS__`功能可知，这里将您的参数直接替换掉即可

   ```c
   #define MSH_CMD_EXPORT(...)                                 \
       __MSH_GET_MACRO(__VA_ARGS__, _MSH_FUNCTION_CMD2_OPT,    \
           _MSH_FUNCTION_CMD2)(__VA_ARGS__)
   #define MSH_CMD_EXPORT(command, desc)                      \
      __MSH_GET_MACRO(command, desc, _MSH_FUNCTION_CMD2_OPT,  \
           _MSH_FUNCTION_CMD2)(command, desc)
   ```

2. 接下来到了最为难以理解的地方 `__MSH_GET_MACRO`如何展开

   ```c
   #define __MSH_GET_MACRO(_1, _2, _3, _FUN, ...)  _FUN
   ```

   这个也是很好理解的，这个宏意思是你传的可变参数有4个以上，前3个不要，用第四个_FUN替代整个`__MSH_GET_MACRO`，

   所以得到

   ```
   #define __MSH_GET_MACRO(command, desc, _MSH_FUNCTION_CMD2_OPT, _MSH_FUNCTION_CMD2)(command, desc) _MSH_FUNCTION_CMD2([command, desc])
   ```

3. 化繁为简

   最后的结果就是拿到`MSH_FUNCTION_EXPORT_CMD`整个宏

   ```c
   #define _MSH_FUNCTION_CMD2(command, desc)   MSH_FUNCTION_EXPORT_CMD(a0, a0, a1, 0)
   ```

   所以绕了一圈我们得到的还是这个结果

   ```c
   #define MSH_CMD_EXPORT(command, desc)      _MSH_FUNCTION_CMD2(command, desc)
   #define MSH_CMD_EXPORT_ALIAS(command,ALIAS, desc)      MSH_FUNCTION_EXPORT_CMD(command,ALIAS, desc,0)
   ```

4. 总结原因

   上文增加的`__MSH_GET_MACRO`目的就是为了 您的第三个参数opt ,同时兼容旧的写作方式再次看下这个宏

   ```c
   #define MSH_CMD_EXPORT(...)                                 \
       __MSH_GET_MACRO(__VA_ARGS__, _MSH_FUNCTION_CMD2_OPT,    \
           _MSH_FUNCTION_CMD2)(__VA_ARGS__)
   ```

   如果您多传了一个参数  __MSH_GET_MACRO 展开后将使用_`MSH_FUNCTION_CMD2_OPT` 而不再是`_MSH_FUNCTION_CMD2`

   那么第三个参数 OPT是什么 它就是参数补全功能所需要的。




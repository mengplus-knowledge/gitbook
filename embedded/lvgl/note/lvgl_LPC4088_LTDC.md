---
title: 整帧刷新lvgl LPC4088 LTDC
slug: lvgl_lpc4088_ltdc
categories:
  - GUI
tags:
  - lvgl
halo:
  site: https://mengplus.top
  name: 3fd53680-6eaa-44be-81e7-bdae74c02a07
  publish: true
cover: ''
---
## 前言
lvgl在LTDC接口硬件中使用整屏刷新，通过切换LTDC的显存区域可以加快刷新效果。
## 硬件场景
LPC4088 ，LVGL8.3.X 无图像硬件加速

## 方案1(旧方案)

使用厂家推荐的接口，手动将刷新区域写入显存区域

**优势：** 布局刷新效率高

**缺点：** 整屏刷新非常慢

**使用场景：** 局部刷新比较多的，无图片，RAM比较紧张场景

```c
void lcd_fill_array(rt_uint16_t x_start, rt_uint16_t y_start, rt_uint16_t x_end, rt_uint16_t y_end, void *pcolor)
{
    uint16_t *color_ptr = (uint16_t *)pcolor;
    uint32_t offset;
    lcd_framebuffer = _rt_framebuffer;
    for (size_t y = y_start; y <= y_end; y++)
    {
        offset         = y * BSP_LCD_WIDTH + x_start;
        uint32_t width = x_end - x_start + 1;
        memcpy(&lcd_framebuffer[offset], color_ptr, width * 2);
        color_ptr += width;
    }
}
```

## 方案2

将lvgl设置为整屏刷新，拿到的`pcolor`即为整帧屏幕，调用LCD设置缓存区接口直接切换即可。

**优势：** 简化底层刷新过程

**缺点：** 暂时不确定是否影响刷新效率

```c
void lcd_fill_screen(rt_uint16_t x_start, rt_uint16_t y_start, rt_uint16_t x_end, rt_uint16_t y_end, void *pcolor)
{
    Chip_LCD_SetUPFrameBuffer(LPC_LCD, pcolor);
}
```



## FAQ

1. lvgl如何设置为整帧刷新?

   通过下方的示例代码可知，需要两个整屏大小的缓存区buff1，buff2,再启用`full_refresh=1`,即可，在刷新屏幕`disp_flush`回调中将每次收到整帧图像，的地址，其实就是两个buf来回切换。

   ```
    static lv_disp_draw_buf_t draw_buf_dsc_3;
       static lv_color_t *buf_3_1; /*A screen sized buffer*/
       static lv_color_t *buf_3_2; /*Another screen sized buffer*/
       buf_3_1 = rt_malloc_align(sizeof(lv_color_t) * MY_DISP_HOR_RES * MY_DISP_VER_RES, 32);
       if (buf_3_1 == RT_NULL)
           return;
       buf_3_2 = rt_malloc_align(sizeof(lv_color_t) * MY_DISP_HOR_RES * MY_DISP_VER_RES, 32);
       if (buf_3_2 == RT_NULL)
           return;

       lv_disp_draw_buf_init(&draw_buf_dsc_3, buf_3_1, buf_3_2,
                             MY_DISP_HOR_RES * MY_DISP_VER_RES); /*Initialize the display buffer*/

       /*-----------------------------------
        * Register the display in LVGL
        *----------------------------------*/

       lv_disp_drv_init(&disp_drv); /*Basic initialization*/

       /*Set up the functions to access to your display*/

       /*Set the resolution of the display*/
       disp_drv.hor_res = MY_DISP_HOR_RES;
       disp_drv.ver_res = MY_DISP_VER_RES;

       /*Used to copy the buffer's content to the display*/
       disp_drv.flush_cb = disp_flush;

       /*Set a display buffer*/
       disp_drv.draw_buf = &draw_buf_dsc_3;

       /*Required for Example 3)*/
       disp_drv.full_refresh = 1;
   ```


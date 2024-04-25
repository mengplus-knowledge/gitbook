---
title: lvgl(tabview):选项卡控件 禁止默认的滑动切换页面
categories:
  - 嵌入式开发
tags:
  - lvgl
  - tableview
halo:
  site: https://mengplus.top
  name: 53b6745b-9d93-4b0b-a9c7-75fe5190c919
  publish: true
---
## 版本说明
1. 版本：`V8.3.9`
2. 测试环境:`rt-thread`
## 示例代码
```c
    obj = lv_tabview_create(comp_parent, LV_DIR_LEFT, 100);
    lv_obj_clear_flag(lv_tabview_get_content(obj), LV_OBJ_FLAG_SCROLLABLE); /// Flags
```
## 详细解释
源码路径:``packages\LVGL-v8.3.9\src\extra\widgets\tabview\lv_tabview.c``
根据lvgl源码可知切换页面的函数是``void lv_tabview_set_act(lv_obj_t * obj, uint32_t id, lv_anim_enable_t anim_en)``,在文件中全文搜索，可以得知多出有调用，分析排查后可知
1. 页面尺寸调整后调用刷新`lv_tabview_event(const lv_obj_class_t * class_p, lv_event_t * e)`
2. 点击标签页后切换`static void btns_value_changed_event_cb(lv_event_t * e)`
3. 滑动结束调用`void cont_scroll_end_event_cb(lv_event_t * e)`
在这几处调用过程中可以明显怀疑第三处是控制滑动调用的功能,进一步追踪查阅源码可看到
```c
static void lv_tabview_constructor(const lv_obj_class_t * class_p, lv_obj_t * obj)
{
    LV_UNUSED(class_p);
    lv_tabview_t * tabview = (lv_tabview_t *)obj;
///...
    lv_obj_t * btnm;
    lv_obj_t * cont;

    btnm = lv_btnmatrix_create(obj);
    cont = lv_obj_create(obj);

    lv_btnmatrix_set_one_checked(btnm, true);
    lv_obj_add_event_cb(btnm, btns_value_changed_event_cb, LV_EVENT_VALUE_CHANGED, NULL);//按键切换页面事件
    lv_obj_add_flag(btnm, LV_OBJ_FLAG_EVENT_BUBBLE);

    lv_obj_add_event_cb(cont, cont_scroll_end_event_cb, LV_EVENT_ALL, NULL);///添加了滑动切换页面的事件
    lv_obj_set_scrollbar_mode(cont, LV_SCROLLBAR_MODE_OFF);//隐藏掉滑动条
//...
    lv_obj_add_flag(cont, LV_OBJ_FLAG_SCROLL_ONE);//一次只滑动一个
    lv_obj_clear_flag(cont, LV_OBJ_FLAG_SCROLL_ON_FOCUS);
}

```
更多细节欢迎评论区交流

---
title: EVGUI 触控简单处理笔记
date: 2020-04-08 22:51:04
updated: 2020-04-08 22:51:04
categories:
- 系统与底层
tags:
- svg
- embedded system
- vector
- gui
- twin
- window system
- stm32
- µGUI
---

这是我的[硕士论文笔记系列](https://blog.joouis.com/2020/evgui-design-review/)第四篇，原文写于 2016 年六月。

<!-- more -->



## Touchscreen driver

- STM32F429 Discovery 板載 STMPE811 觸屏控制器
- ST 官方提供了驅動程式

### stm32f429i_discovery_ioe

```c
typedef struct {
  uint16_t TouchDetected;
  uint16_t X;
  uint16_t Y;
  uint16_t Z;
} TP_STATE;
```

在 uGUI 的範例中，使用方法如下：
1. 定義一個結構體：`static TP_STATE* TP_State`；
2. 初始化觸控晶片：`IOE_Config()`；
3. 在 SysTick 的 ISR 程式中：

   ```c
   TP_State = IOE_TP_GetState();
   if (TP_State->TouchDetected) { 
     if ((TP_State->X > 0) && (TP_State->X < 239 )) 
     {  
        if ((TP_State->Y > 0) && (TP_State->Y < 319 )) 
        {   
           UG_TouchUpdate(TP_State->X,TP_State->Y,TOUCH_STATE_PRESSED);
        }
     }
   } else {
     UG_TouchUpdate(-1,-1,TOUCH_STATE_RELEASED);
   }
   ```

### TWIN

- 在 EVGUI 中去掉了 TWIN 中的 cursor.c 和 file.c 兩個文件，第一個是因爲觸控屏不再需要指標，第二個則是我們需要實作的部分，把 X11 的滑鼠換成觸控屏的輸入
- TWIN 中事件類型的定義：

  ```c
  
  typedef enum _twin_event_kind {
      /* Mouse */
      TwinEventButtonDown	= 0x0001,
      TwinEventButtonUp	= 0x0002,
      TwinEventMotion	= 0x0003,
      TwinEventEnter	= 0x0004,
      TwinEventLeave	= 0x0005,
  
      /* keyboard */
      TwinEventKeyDown	= 0x0101,
      TwinEventKeyUp	= 0x0102,
      TwinEventUcs4	= 0x0103,
  
      /* Focus */
      TwinEventActivate	= 0x0201,
      TwinEventDeactivate	= 0x0202,
  
      /* Joystick */
      TwinEventJoyButton	= 0x0401,
      TwinEventJoyAxis	= 0x0402,
  
      /* Widgets */
      TwinEventPaint	= 0x1001,
      TwinEventQueryGeometry	= 0x1002,
      TwinEventConfigure	= 0x1003,
      TwinEventDestroy	= 0x1004,
  } twin_event_kind_t;
  ```

- TWIN 中事件的定義：

  ```c
  
  typedef struct _twin_event {
      twin_event_kind_t   kind;
      union {
      struct {
         twin_coord_t    x, y;
         twin_coord_t    screen_x, screen_y;
         twin_count_t    button;
      } pointer;
      struct {
         twin_keysym_t   key; 
      } key; 
      struct {
         twin_js_number_t control;
         twin_js_value_t  value;
      } js;
      struct {
         twin_ucs4_t     ucs4;
      } ucs4;
      struct {
         twin_rect_t     extents;
      } configure;
      } u;
  } twin_event_t;
  ```



## 添加觸控功能

### 開啓事件處理的函式注冊

- 要在 fbdev 的 twin_fbdev_create_ext() 加入 handle_events 這個標志（值爲 1）

```c
if (handle_events) {
		twin_set_file(twin_fbdev_read_events, tf->vt_fd, TWIN_READ, tf);
}
```

### 實作事件讀取函式

- File: `backend/system.c`
- 關於輸入來源：
  - X11 通過 Xlib 的 XNextEvent() 獲取
  - Framebuffer 實作了一個 linux_mouse 模塊，讀取 “/dev/input/mice” 的資料
  - 關於 mice 設備的讀取可以參考[這裡](http://stackoverflow.com/questions/11451618/how-do-you-read-the-mouse-button-state-from-dev-input-mice)
- 下一節有說明，我們實作一個單向鏈表作爲輸入

### 處理 Raw Data

- File: `backend/system.c`
- 回顧，在 `SysTick_Handler` 中：
  - `TP_State = IOE_TP_GetState();`
  - `TP_State` 中有 `TouchDetected` `X` `Y` `Z` 四個成員
  - 判斷 `TouchDetected` 是否爲真且 `X` `Y` 在合法範圍內
- 構建 `twin_event_t` 參照 X11，因爲原生 TWIN 的 framebuffer 模式下 input 本身就有問題，且實作了一個 `fb` 的事件讀取和一個滑鼠的事件讀取，這兩個放在一起更合理
- 由於存在 `screen->clicklock`（在 `twin_screen.c` 中，會判斷 `screen->clicklock` 和 `event→kind != TwinEventButtonUp` 同時爲真才進行追蹤），暫時先不實作 `TwinEventMotion` 這個類別
  - 因爲手指移動時一定是 `ButtonDown` 的類別也滿足上述判斷式
  - 這樣可能導致的後果是：採用相對坐標，手指下次從其他地方滑動時，坐標還是之前的位置，出現很大的便宜
- 實作 `TwinEventMotion` 類別
  - 設定一個是否按住的標志（初始值爲 1）爲 `FirstTouch`
  - 每次 `TouchDetected` 爲真時判斷 `FirstTouch` 的值
    - 若 `FirstTouch` 爲真則把這次動作的類別記爲 `TwinEventMotion`，然後再補一個 `TwinEventButtonDown` 的節點，最後將 `FirstTouch` 置爲 0
    - 若 `FirstTouch` 爲假則只單純添加 `TwinEventButtonDown` 的節點
  - 若 `TouchDetected` 爲假時則把 `FirstTouch` 置爲 1
- 輸入節點爲一個單向鏈表，有兩個 global 節點爲 `TP_Head` 和 `TP_Latest`：

  ```c
  typedef struct _FB_event {
    struct _FBEvent *Next;
    twin_event_t event;
  } FB_event;
  ```

### 問題

- [x] ~~藍屏 → 無法正常載入圖形~~
  - GDB 調試發現在 SysTick Hadnler 重復呼叫幾次，就直接會跳到通用異常處理的 ISR 裏
  - 這樣很容易猜測是 SySTick 中斷頻率太高，一次裏面的事情還沒做完，下一次就發生了，所以掛掉
  - 通過 `SysTick_Config()` 把中斷頻率由原來的 1000Hz 改爲 100Hz，果然就好了
- [x] ~~時鍾不走，觸屏沒反應~~
  - `timeout_delay()` 函式有問題
- [ ] 自己寫一個簡單的時間調度處理
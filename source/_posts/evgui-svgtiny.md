---
title: SVG Tiny 与 Libsvgtiny
date: 2020-04-04 22:51:16
updated: 2020-04-04 22:51:16
categories:
- 系统与底层
tags:
- svg
- svg tiny
- libsvgtiny
- embedded system
- vector
- gui
- twin
- window system
- cairo
---

这是我的[硕士论文笔记系列](https://blog.joouis.com/2020/evgui-design-review/)第二篇，原文写于 2016 年五月。

<!-- more -->



## SVG Tiny 1.2 Spec

- The SVG Working Group **does not** provide a normative DTD for SVG Tiny 1.2 but instead provides a normative **RelaxNG** schema.
- How to reference this specification:
  When referencing this specification as a whole or when referencing a chapter or major section, use the undated URI, http://www.w3.org/TR/SVGTiny12/ , where possible. This allows the reference to always refer to the latest version of this specification.

### Scriptable

The combination of scripting and the HTML DOM is often termed "Dynamic HTML" and is widely used for animation, interactivity and presentational effects. Similarly SVG allows the script-based manipulation of the document tree us-ing a subset of the XML DOM and the SVG uDOM.

### Rendering Model

- The painters model
- Types of graphics elements
  - Shapes: lines, curves
  - Text
  - Repalced content
    - Raster images
    - Video
    - Animation
    - Foreign Objects
- Rendering shapes and text
  - fill, stroke: independent, has opacity
  - paint:
    - Solid color
    - Gradients (linear, radial)

### Document Structure

- `svg` element: 
  - version, baseProfile, width, height, viewBox, …
  - This specification corresponds to **baseProfile="tiny"** and **version="1.2"** .
- `g` element:
  - `id`, `xml: id`
  - 在 group 中的元素擁有 group 的屬性
- `defs` element
- `discard` element
- `title` and `desc` element
- `use` element
- `image` element: must reference content that is a raster image format
- `switch` element: five conditional processing attributes
  - requiredExtensions
  - requiredFeatures
  - requiredFonts
  - requiredFormats
  - systemLanguage
- `prefetch` element
- Attributes common to all elements: id, class, role, …

### Animation

- mpath



## Libsvgtiny

### 介紹

- Libsvgtiny is an implementation of [SVG Tiny](http://www.w3.org/TR/SVGMobile/), written in C. It is currently in development for use with [NetSurf](http://www.netsurf-browser.org/) and is intended to be suitable for use in other projects too.
- The overall idea of the library is to take some SVG as input, and return a list of paths and texts which can be rendered easily. The library does not do the actual rendering. For further details, see the [readme](http://source.netsurf-browser.org/libsvgtiny.git/plain/README).

### 支援情況

- SVG Tiny elements **supported**: defs, g, svg, circle, line, path, polygon, polyline, rect, text
- SVG Tiny elements **not yet supported**: desc, metadata, title, use, a, switch, ellipse, image, font, font-face, font-face-name, font-face-src, glyph, hkern, missing-glyph, animate, animateColor, animateMotion, animateTransform, mpath, set, foreignObject
- Additional elements **supported**: linearGradient, stop
- Text support is **incomplete.**
- The style attribute is **supported**.

### 安裝

```shell
git clone git://git.netsurf-browser.org/libsvgtiny.git
```

- 參考 [疑問通訊](https://listmaster.pepperfish.net/pipermail/netsurf-dev-netsurf-browser.org/2013-November/003228.html)
- 直接編譯 Netsurf 更快，參考 [building instruction](http://source.netsurf-browser.org/netsurf.git/plain/Docs/QUICK-START)

### 資料結構

```c
struct svgtiny_shape {
    float *path;
    unsigned int path_length;                                                                                                                     
    char *text;
    float text_x, text_y;
    svgtiny_colour fill;
    svgtiny_colour stroke;
    float stroke_width;
    void *_internal_extensions;  // TODO: if non-NULL, points to an allocated on the heap extension block. (gradients, fonts)
};

struct svgtiny_diagram {
    int width, height;

    struct svgtiny_shape *shape;
    unsigned int shape_count;

    unsigned short error_line;
    const char *error_message;
};
```

### 使用 Cairo

```c
/**
 * Handle an X11 Expose event of the diagram window.
 */
void event_diagram_expose(const XExposeEvent *expose_event)
{
    cairo_surface_t *surface;
    cairo_t *cr;
    cairo_status_t status;
    unsigned int i;

    if (expose_event->count != 0)
        return;

    surface = cairo_xlib_surface_create(display, diagram_window,
            DefaultVisual(display, DefaultScreen(display)),
            diagram->width * scale, diagram->height * scale);
    if (!surface) {
        fprintf(stderr, "cairo_xlib_surface_create failed\n");
        return;
    }

    cr = cairo_create(surface);
    status = cairo_status(cr);
    if (status != CAIRO_STATUS_SUCCESS) {
        fprintf(stderr, "cairo_create failed: %s\n",
                cairo_status_to_string(status));
        cairo_destroy(cr);
        cairo_surface_destroy(surface);
        return;
    }

    cairo_set_source_rgb(cr, 1, 1, 1);
    cairo_paint(cr);

    for (i = 0; i != diagram->shape_count; i++) {
        if (diagram->shape[i].path) {
            render_path(cr, scale, &diagram->shape[i]);

        } else if (diagram->shape[i].text) {
            cairo_set_source_rgb(cr,
                svgtiny_RED(diagram->shape[i].stroke) / 255.0,
                svgtiny_GREEN(diagram->shape[i].stroke) / 255.0,
                svgtiny_BLUE(diagram->shape[i].stroke) / 255.0);
            cairo_move_to(cr,                                                                                                                     
                    scale * diagram->shape[i].text_x,
                    scale * diagram->shape[i].text_y);
            cairo_show_text(cr, diagram->shape[i].text);
        }
    }

    status = cairo_status(cr);
    if (status != CAIRO_STATUS_SUCCESS) {
        fprintf(stderr, "cairo error: %s\n",
                cairo_status_to_string(status));
        cairo_destroy(cr);
        cairo_surface_destroy(surface);
        return;
    }
    
    cairo_destroy(cr);
    cairo_surface_destroy(surface);source
}

/**
 * Render an svgtiny path using cairo.
 */
void render_path(cairo_t *cr, float scale, struct svgtiny_shape *path)
{
    unsigned int j;

    cairo_new_path(cr);
    for (j = 0; j != path->path_length; ) { 
        switch ((int) path->path[j]) {
        case svgtiny_PATH_MOVE:
            cairo_move_to(cr,
                    scale * path->path[j + 1], 
                    scale * path->path[j + 2]);
            j += 3;
            break;
        case svgtiny_PATH_CLOSE:
            cairo_close_path(cr);
            j += 1;
            break;
        case svgtiny_PATH_LINE:
            cairo_line_to(cr,
                    scale * path->path[j + 1], 
                    scale * path->path[j + 2]);
            j += 3;
            break;
        case svgtiny_PATH_BEZIER:
            cairo_curve_to(cr,
                    scale * path->path[j + 1],
                    scale * path->path[j + 2],
                    scale * path->path[j + 3],
                    scale * path->path[j + 4],
                    scale * path->path[j + 5],
                    scale * path->path[j + 6]);
            j += 7;
            break;
        default:
            printf("error ");
            j += 1;
        }
    }
    if (path->fill != svgtiny_TRANSPARENT) {
        cairo_set_source_rgb(cr,
                svgtiny_RED(path->fill) / 255.0,
                svgtiny_GREEN(path->fill) / 255.0,
                svgtiny_BLUE(path->fill) / 255.0);
        cairo_fill_preserve(cr);
    }
    if (path->stroke != svgtiny_TRANSPARENT) {
        cairo_set_source_rgb(cr,
                svgtiny_RED(path->stroke) / 255.0,
                svgtiny_GREEN(path->stroke) / 255.0,
                svgtiny_BLUE(path->stroke) / 255.0);
        cairo_set_line_width(cr, scale * path->stroke_width);
        cairo_stroke_preserve(cr);
    }
}
```



## 討論

### Libsvgtiny 無法編譯，無論是 official 還是 github 版本

- 改用 https://github.com/DavidPhillipOster/libsvgtiny
- `sed -i 's/-lxml2/-lxml2 -lm/g' Makefile && make`


### Libsvgtiny 有的可以成功顯示，有的部分顯示，有的則顯示失敗

（左、中、右圖分別對應 Image Viewer、Libsvgtiny with Cairo、Libsvgtiny with TWIN）

#### 成功顯示

注意最右邊的圖顯示是正確的，因爲圓形和矩形沒有例如 `fill: #000000` 這樣的屬性，至於左邊兩個爲什麼會自動填黑很有趣。

![](https://hsxmua.bn.files.1drv.com/y4mVRHjbTq_FPN7KNhj_jr2kI0RAnFIITSM-Vej1v_8G7k-UM9X2lplZzlzjzQNZhkDvYFvwU_AVbsyaXs9CHOt8bppajump_Kx-0kN09oDRfxIYFqysnIGHYlxwQEswug05hUwk4gBh7zRvcyRaWJR042NWlwLAWsEAmfYORmM3As5J5yjaJFh79LzhYMJDxNvvfaOgp3CjAgIN__H-fwmog?width=1912&height=451&cropmode=none)

#### 部分成功顯示

- 都可顯示，但由於 Libsvgtiny 和 TWIN 的限制致使顯示有缺陷。中間的圖基本和原圖一樣，不過有條不明顯的黑線，可以修改 `libsvgtiny.c` 或刪除這條 path 多餘的 `stroke-width` 屬性即可修正。最右除了 Libsvgtiny 自身的問題外，和 TWIN 也有關聯。

![](https://hcxeua.bn.files.1drv.com/y4ml_N8t3aeOuTjDC1r8xBq_ywDJORW3FQy3j_6sqxWW-SvMfaZ589WXZsvsIxQbv7cxclTzqnNqGI9t3cY3Yu8bRBwjtGzKCrPV2VZQd1nuiNR7mPvsR9kFz0hhXKbg9Dj_gSb7oBAvenPAggRJqanawaZqXU3H0P_9r2S1Y0tV8asD_XAx4DrAegGEF8083p72hGWs6sEaMifn0j9Yywbcg?width=854&height=284&cropmode=none)

- Cairo 可成功顯示但 TWIN 不行（後已修復）

![](https://hsxnua.bn.files.1drv.com/y4m_nOg9IadeovxjnjzDnUwsFGL8LSZLWQa2tG_yg26a2nkyQrGeZN2y2e83EElONJsrvFWQQGg0B5xydmEfMSG15cT149yu44wxsNp8XF-1Z9zFPWsGKolLv4CBHKNAdCtUZA69A1n3VsG47VTY5PxUGP9jeBALlY1CjvyEDRKjc5hcH846g-3KNjIdIDylnP5DAT8ZEj_QLTyhgZcLRfDUg?width=1199&height=547&cropmode=none)

#### 顯示失敗

![](https://hcxiua.bn.files.1drv.com/y4mqJEAnS1bmpULVGsbJ6y2R-Pnia66bHGIlev0EF3WWVxVeoBTEgjV_G7x5jZ-nl6y9uH7DVHZ5MPBu4txZNw3hUfgVbLq8uh91DVhE8Cd_RNEm7nNu3k4a_0us0dvqp9m98PImTMf84FMkmBbvsQElXA1f8JGvpmo1o1JmrvCkM1Pn-BhfNf4WbkXO-8sxmCkwQSTr7cqpqroxSy8UqZ3_Q?width=1890&height=1028&cropmode=none)

- 其實修改 Libsvgtiny 後使用 Cairo 是可以大致顯示的，簡單說就是 Libsvgtiny 的 fill 邏輯出了問題，要麼都塗黑了如上圖所示，要麼如下圖沒把該塗黑的地方塗黑ORZ

![](https://hcxhua.bn.files.1drv.com/y4mMu0upkRKHdD4MJrHjGzXaScyp2AmKT1HDz_gME-UggXO2r2J9mzrTGEQYHr9E4fpy-OcFdE0PNzq0i1bYYDH-QlYP5EF9nu_U8e5LUErWspvAp-tfTVh9gFPhL1Hyi7M7jHUQ15WklJ2Cjbou2k5P2fPpIsbaowHPOHc4vh7toBvWVY85ZQ-wMo8ltWzW3eJTNfATEAU_ikt0Itoshwo3w?width=1637&height=1024&cropmode=none)

![](https://hcxfua.bn.files.1drv.com/y4mC2VoVCOon8Qu4X4YWVASaNWVdhm60A_IOSHMhPnp4xyybH6qgM6l-CmsyPpV6UOsiaCpzv3S8MKJIXL4HtkL0zK5E7yZAiBpQeUFbZpB3PSd7Hr6D1BCO2z4ZlZNYXTqXDfu8cgCJiRlAHqLJkJ2UXbMDk-kSe0gWAdF3NgWXGPgk-MvHNPeIsRWatLJxmDYxa6_vb0FJyIDta-16RWFUQ?width=1641&height=1031&cropmode=none)

- 使用 Netsurf 開啓老虎圖，直接笑出來了…

![](https://hcxgua.bn.files.1drv.com/y4mDaXibkKkZ2DLw13I-H9R_eFWZvjP5M9Wy9jShTnZTLLC1u2TYdXXg_tydEf9dGEhqdmln1m09ye6YlSLJXk-7vWw03JjvBtSHx2RQfwSA6BEIozngeq3myirgFaEdWP6kbF-lfgF-SfUo1BJXK8jo4x0JID6PhT8nx2aoTztA-vrdUBN3u2bH48H1_RZ02BvBR9WmxYXoqnmo7VV0_zUNQ?width=1272&height=1080&cropmode=none)

### 移植到 STM32F429 上的過程中，編譯鏈接又出問題了

```shell
In file included from /usr/include/libxml2/libxml/parser.h:810:0,
                 from /usr/include/libxml2/libxml/globals.h:18,
                 from /usr/include/libxml2/libxml/threads.h:35,
                 from /usr/include/libxml2/libxml/xmlmemory.h:218,
                 from /usr/include/libxml2/libxml/tree.h:1307,
                 from libsvgtiny/src/xml2dom.c:14:
/usr/include/libxml2/libxml/encoding.h:31:26: fatal error: unicode/ucnv.h: No such file or directory
 #include <unicode/ucnv.h>
                          ^
compilation terminated.
Makefile:85: recipe for target 'libsvgtiny/src/xml2dom.o' failed
make: *** [libsvgtiny/src/xml2dom.o] Error 1
```

Jserv replied:

```shell
$ git clone git://git.gnome.org/libxml2
$ cd libxml2
$ ./autogen.sh
$ sh configure \
  --prefix=/tmp \
  --enable-static \
  --disable-shared \
  --enable-ipv6=no \
  --without-c14n \
  --without-catalog \
  --without-debug \
  --without-docbook \
  --without-fexceptions \
  --without-ftp \
  --without-history \
  --without-html \
  --without-http \
  --without-iconv \
  --without-icu \
  --with-iso8859x \
  --without-legacy \
  --with-minimum \
  --without-output \
  --without-pattern \
  --without-push \
  --without-python \
  --without-reader \
  --without-regexps \
  --without-sax1 \
  --with-schemas \
  --with-schematron \
  --without-threads \
  --with-tree \
  --with-valid  \
  --with-writer \
  --with-xinclude \
  --with-xpath \
  --with-xptr \
  --without-modules \
  --without-zlib
```

### libxml2 太龐大，無法放入 ARM 環境

- 替代品
  - [YXML](https://dev.yorhel.nl/yxml)
  - [EZXML](https://github.com/lxfontes/ezxml)

### Updates

#### 2016/06/08

- 修復 Libsvgtiny 問題，顯示 tiger 黑圖（修改 Libsvgtiny 的 fill、stroke 可顯示不完全正確的 tiger 圖）

![](https://hcxmua.bn.files.1drv.com/y4mhLEBOyK8sGD0_VrOOuxH6fEX9gqHmvH4IimCPlaMICaYNvEXcuu-fJ_FEsFS1o9F6wTo4AGgt9eY1Yxiyrdfr87NDvMgYhXV-Ji63oR0CJPuIQwenFqmz9fyZupLOOJUR58_t-TL_Ce9DoJUQUMSmul5kdzUYfgVqHJEoUHg4iFsTY4x-DfRnBJXjfN7G3JLcvpu2Ii7APNssspgUn-oKw?width=1917&height=1080&cropmode=none)

#### 2016/06/11

- twin 接口問題，顯示 sonic 黑圖（使用 cairo 可正常顯示）

![](https://hcxjua.bn.files.1drv.com/y4m-0PTeYG_SCSKGrAJ_UX_W9gvAcMhyGnbX6_tfB-qxeecqbcwS1K_tgrCstPor3U_KHdKuvpa2sNLTcOko5PSEdpGeccRThEk3Wi5XZdmhx1EA_AdBuUQwJ2h7neioFCYsbbYe8FgLWe_NbXWfWgN-f-PsXbdltn8By-_T-Row0TACiVCBzCT-DoaXKanCRUGvaGUhd_A0Oou7CZcB4GIBg?width=1000&height=1037&cropmode=none)

#### 2016/06/12

- 修復 Libsvgtiny 縮放問題，在 STM32F429Discovery 上顯示成功

![](https://hsxlua.bn.files.1drv.com/y4mBIRRiVRqn_SnqOF1CRSGbMzSD_fppWopMwsrAafydPutl2ZOLjFRfmDvURmHjMj02i5u8sIL9lWJiM3yviTrC1JQe8pfKyPnGx-Oh2__re_0FIozZn4Lxq8gmtkmHDa2EhvgBmjgfTOLwu9_-7a0UqRY7hrKmuz4OcmXT0jH0WM4wDuOyOeYT8nYAHBHo8GR60toXQ2XCFOoATxHW7W3iA?width=14440&height=1920&cropmode=none)

![](https://hcxlua.bn.files.1drv.com/y4mxzBp__N-J4lAX9LtXPkZXqkZC4qKoLvBLoFebz8ipnVAFwkIZhHbLFx5xkh5BUimnO3y6jp6uZKv7S5EwS7PN6K3QZyIO88CLoo2UKhIZstZ97IA-nFQeZ8oBk2umKL7_6XDIlpH2nRGTRR-ZlQtfWhqq9qvCjZexTFsWK1bRrZm6xMrKVxPqCLnQlilFZ6UFe3p0B8ho7Lis_v-mDQcmw?width=1440&height=1920&cropmode=none)
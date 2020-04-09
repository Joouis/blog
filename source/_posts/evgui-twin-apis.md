---
title: TWIN API 整理
date: 2020-04-09 22:24:00
updated: 2020-04-09 22:24:00
categories:
- 系统与底层
tags:
- svg
- embedded system
- vector
- gui
- twin
- window system
- api
---

这是我的[硕士论文笔记系列](https://blog.joouis.com/2020/evgui-design-review/)第五篇，原文写于 2016 年四月。

<!-- more -->



### Box

```c
twin_box_create(twin_box_t *parent, twin_box_dir_t dir);

void _twin_box_init(twin_box_t *box, 
                    twin_box_t *parent, 
                    twin_window_t *window, 
                    twin_box_dir_t dir, 
                    twin_dispatch_proc_t dispatch);
                    
static twin_dispatch_result_t _twin_box_query_geometry(twin_box_t *box);

static twin_dispatch_result_t _twin_box_configure(twin_box_t *box)；

twin_dispatch_result_t _twin_box_dispatch(twin_widget_t *widget, twin_event_t *event)；

static twin_widget_t * _twin_box_xy_to_widget(twin_box_t *box, twin_coord_t x, twin_coord_t y)；
```

### Button

```c
twin_button_t * twin_button_create (twin_box_t      *parent,
                        const char       *value,
                        twin_argb32_t    foreground,
                        twin_fixed_t     font_size,
                        twin_style_t     font_style);
                        
static void _twin_button_paint (twin_button_t *button);

static void _twin_button_set_label_offset (twin_button_t *button);

twin_dispatch_result_t _twin_button_dispatch (twin_widget_t *widget, twin_event_t *event);

void _twin_button_init (twin_button_t    *button,
                        twin_box_t       *parent,
                        const char       *value,
                        twin_argb32_t    foreground,
                        twin_fixed_t     font_size,
                        twin_style_t     font_style,
                        twin_dispatch_proc_t dispatch);
```

### Convolve

```c
void twin_path_convolve (twin_path_t *path,
                        twin_path_t *stroke,
                        twin_path_t *pen);
                        
static int _twin_path_leftpoint (twin_path_t   *path,
                        twin_spoint_t *p1,
                        twin_spoint_t *p2);
                        
static int _around_order (twin_spoint_t    *a1,
                        twin_spoint_t    *a2,
                        twin_spoint_t    *b1,
                        twin_spoint_t    *b2);
                        
static void _twin_subpath_convolve (twin_path_t *path,
                        twin_path_t *stroke,
                        twin_path_t *pen);
```

### Cursor

```c
static unsigned int twin_def_cursor_image[] = {};

twin_pixmap_t *twin_get_default_cursor(int *hx, int *hy);

static inline int twin_read_header(fdtype fd, uint32_t *buf, int size);

twin_pixmap_t *twin_load_X_cursor(const char *file, int index, int *hx, int *hy);
```

### Dispatch

```c
void twin_dispatch(void);
```

### Draw

```c
static twin_src_op  comp2[2][4][3] = {};
static twin_src_msk_op  comp3[2][4][4][3] = {};
static twin_src_op  fill[2][3] = {};

static inline int operand_xindex(twin_operand_t *o);

static twin_xform_t *twin_pixmap_init_xform (twin_pixmap_t *pixmap,
                          twin_coord_t left, twin_coord_t width,
                          twin_coord_t src_x, twin_coord_t src_y);
                          
static void twin_pixmap_free_xform (twin_xform_t *xform);

static void twin_pixmap_read_xform_8 (twin_xform_t *xform, twin_coord_t line);

static void twin_pixmap_read_xform_16 (twin_xform_t *xform, twin_coord_t line);

static void twin_pixmap_read_xform_32 (twin_xform_t *xform, twin_coord_t line);

static void twin_pixmap_read_xform (twin_xform_t *xform, twin_coord_t line);

void twin_composite (twin_pixmap_t  *dst,
                          twin_coord_t    dst_x,
                          twin_coord_t    dst_y,
                          twin_operand_t  *src,
                          twin_coord_t    src_x,
                          twin_coord_t    src_y,
                          twin_operand_t  *msk,
                          twin_coord_t    msk_x,
                          twin_coord_t    msk_y,
                          twin_operator_t operator,
                          twin_coord_t    width,
                          twin_coord_t    height);

void twin_premultiply_alpha(twin_pixmap_t *px);

void twin_fill (twin_pixmap_t    *dst,
                          twin_argb32_t    pixel,
                          twin_operator_t  operator,
                          twin_coord_t     left,
                          twin_coord_t     top,
                          twin_coord_t     right,
                          twin_coord_t     bottom);
                          
static void _twin_composite_simple (twin_pixmap_t   *dst,
                          twin_coord_t    dst_x,
                          twin_coord_t    dst_y,
                          twin_operand_t  *src,
                          twin_coord_t    src_x,
                          twin_coord_t    src_y,
                          twin_operand_t  *msk,
                          twin_coord_t    msk_x,
                          twin_coord_t    msk_y,
                          twin_operator_t operator,
                          twin_coord_t    width,
                          twin_coord_t    height);

static void _twin_composite_xform (twin_pixmap_t    *dst,
                          twin_coord_t     dst_x,
                          twin_coord_t     dst_y,
                          twin_operand_t   *src,
                          twin_coord_t    src_x,
                          twin_coord_t    src_y,
                          twin_operand_t  *msk,
                          twin_coord_t    msk_x,
                          twin_coord_t    msk_y,
                          twin_operator_t operator,
                          twin_coord_t    width,
                          twin_coord_t    height);
                          
static twin_argb32_t _twin_apply_alpha(twin_argb32_t v);
```

### File

```c
twin_file_t * twin_set_file (twin_file_proc_t file_proc,
                          int           fd,
                          twin_file_op_t    ops,
                          void          *closure);
                          
void twin_clear_file (twin_file_t *file);

static twin_order_t _twin_file_order (twin_queue_t *a, twin_queue_t *b);

void _twin_run_file (twin_time_t delay);
```

### Fixed

```c
twin_fixed_t twin_fixed_sqrt (twin_fixed_t a);

twin_sfixed_t _twin_sfixed_sqrt (twin_sfixed_t as);
```

### Font

```c
typedef struct _twin_text_info { 
    twin_point_t    scale;
    twin_point_t    pen;
    twin_point_t    margin;
    twin_point_t    reverse_scale;
    twin_bool_t     snap;
    twin_matrix_t   matrix;
    twin_matrix_t   pen_matrix;
    int         n_snap_x;
    twin_fixed_t    snap_x[TWIN_GLYPH_MAX_SNAP_X];
    int         n_snap_y;
    twin_fixed_t    snap_y[TWIN_GLYPH_MAX_SNAP_Y];
} twin_text_info_t;

static void _twin_text_compute_info (twin_path_t    *path,
                            twin_font_t    *font,
                            twin_text_info_t   *info);

static void _twin_text_compute_snap (twin_text_info_t   *info,
                            const signed char  *b);

static twin_path_t * _twin_text_compute_pen (twin_text_info_t *info);

static twin_fixed_t _twin_snap (twin_fixed_t    v,
                            twin_fixed_t    *snap,
                            int         n);
                            
static twin_bool_t twin_find_ucs4_page(twin_font_t *font, uint32_t page);

twin_bool_t twin_has_ucs4 (twin_font_t *font, twin_ucs4_t ucs4);
 
static const signed char * _twin_g_base (twin_font_t *font, twin_ucs4_t ucs4);
 
static twin_fixed_t _twin_glyph_width (twin_text_info_t  *info,
                              const signed char *b);
                              
void twin_text_metrics_ucs4 (twin_path_t        *path,
                              twin_ucs4_t        ucs4,
                              twin_text_metrics_t    *m);

static const signed char *twin_glyph_draw(twin_font_t       *font,
                              const signed char *b);
                              
void twin_path_ucs4 (twin_path_t *path, twin_ucs4_t ucs4);

twin_fixed_t twin_width_ucs4 (twin_path_t *path, twin_ucs4_t ucs4);

static int _twin_utf8_to_ucs4 (const char       *src_orig,
                              twin_ucs4_t      *dst);
                              
void twin_path_utf8 (twin_path_t *path, const char *string);

twin_fixed_t twin_width_utf8 (twin_path_t *path, const char *string);

void twin_text_metrics_utf8 (twin_path_t     *path,
                              const char          *string,
                              twin_text_metrics_t *m);
```

### Geom

```c
twin_dfixed_t _twin_distance_to_point_squared (twin_spoint_t *a, twin_spoint_t *b);

twin_dfixed_t _twin_distance_to_line_squared (twin_spoint_t *p, twin_spoint_t *p1, twin_spoint_t *p2);
```

### Hull

```c
typedef struct twin_slope {
    twin_sfixed_t dx;
    twin_sfixed_t dy;
} twin_slope_t, twin_distance_t;

typedef struct _twin_hull {
    twin_spoint_t point;
    twin_slope_t slope;
    int discard;
} twin_hull_t;

static void _twin_slope_init (twin_slope_t *slope, twin_spoint_t *a, twin_spoint_t *b);

static twin_hull_t * _twin_hull_create (twin_path_t *path, int *nhull);

static int _twin_slope_compare (twin_slope_t *a, twin_slope_t *b);

static int _twin_hull_vertex_compare (const void *av, const void *bv);

static int _twin_hull_prev_valid (twin_hull_t *hull, int maybe_unused num_hull, int index);

static int _twin_hull_next_valid (twin_hull_t *hull, int num_hull, int index);

static void _twin_hull_eliminate_concave (twin_hull_t *hull, int num_hull);

static twin_path_t * _twin_hull_to_path (twin_hull_t *hull, int num_hull);

twin_path_t * twin_path_convex_hull (twin_path_t *path);
```

### Label

```c
static void _twin_label_query_geometry (twin_label_t *label);

static void _twin_label_paint (twin_label_t *label);

twin_dispatch_result_t _twin_label_dispatch (twin_widget_t *widget, twin_event_t *event);

void twin_label_set (twin_label_t    *label,
              const char  *value,
              twin_argb32_t   foreground,
              twin_fixed_t    font_size,
              twin_style_t    font_style);
              
void _twin_label_init (twin_label_t      *label,
              twin_box_t        *parent,
              const char        *value,
              twin_argb32_t     foreground,
              twin_fixed_t      font_size,
              twin_style_t      font_style,
              twin_dispatch_proc_t  dispatch);
              
twin_label_t * twin_label_create (twin_box_t       *parent,
              const char       *value,
              twin_argb32_t    foreground,
              twin_fixed_t     font_size,
              twin_style_t     font_style);
```

### Matrix

```c
void twin_matrix_multiply (twin_matrix_t *result,
              const twin_matrix_t   *a,
              const twin_matrix_t   *b);
              
void twin_matrix_identity (twin_matrix_t *m);

twin_bool_t twin_matrix_is_identity (twin_matrix_t *m);

void twin_matrix_translate (twin_matrix_t *m, twin_fixed_t tx, twin_fixed_t ty);

void twin_matrix_scale (twin_matrix_t *m, twin_fixed_t sx, twin_fixed_t sy);

twin_fixed_t _twin_matrix_determinant (twin_matrix_t *matrix);

twin_point_t _twin_matrix_expand (twin_matrix_t *matrix);

void twin_matrix_rotate (twin_matrix_t *m, twin_angle_t a);

twin_sfixed_t _twin_matrix_x (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);
twin_sfixed_t _twin_matrix_y (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);

twin_fixed_t _twin_matrix_fx (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);
twin_fixed_t _twin_matrix_fy (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);

twin_sfixed_t _twin_matrix_dx (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);
twin_sfixed_t _twin_matrix_dy (twin_matrix_t *m, twin_fixed_t x, twin_fixed_t y);

twin_sfixed_t _twin_matrix_len (twin_matrix_t *m, twin_fixed_t dx, twin_fixed_t dy);
```

### Path

```c
static int _twin_current_subpath_len (twin_path_t *path);

twin_spoint_t _twin_path_current_spoint (twin_path_t *path);

twin_spoint_t _twin_path_subpath_first_spoint (twin_path_t *path);

void _twin_path_sfinish (twin_path_t *path);

void _twin_path_smove (twin_path_t *path, twin_sfixed_t x, twin_sfixed_t y);

void _twin_path_sdraw (twin_path_t *path, twin_sfixed_t x, twin_sfixed_t y);

void twin_path_move (twin_path_t *path, twin_fixed_t x, twin_fixed_t y);

void twin_path_rmove (twin_path_t *path, twin_fixed_t dx, twin_fixed_t dy);

void twin_path_draw (twin_path_t *path, twin_fixed_t x, twin_fixed_t y);

void twin_path_rdraw (twin_path_t *path, twin_fixed_t dx, twin_fixed_t dy);

void twin_path_close (twin_path_t *path);

void twin_path_circle (twin_path_t   *path,
                  twin_fixed_t  x,
                  twin_fixed_t  y,
                  twin_fixed_t radius);
                  
void twin_path_ellipse (twin_path_t *path,
                  twin_fixed_t x,
                  twin_fixed_t y,
                  twin_fixed_t x_radius,
                  twin_fixed_t y_radius);
                  
static twin_fixed_t _twin_matrix_max_radius (twin_matrix_t *m);

void twin_path_arc (twin_path_t  *path,
                  twin_fixed_t x,
                  twin_fixed_t y,
                  twin_fixed_t x_radius,
                  twin_fixed_t y_radius,
                  twin_angle_t start,
                  twin_angle_t extent);
                  
void twin_path_rectangle (twin_path_t    *path,
                  twin_fixed_t   x,
                  twin_fixed_t   y,
                  twin_fixed_t   w,
                  twin_fixed_t   h);
                  
void twin_path_rounded_rectangle (twin_path_t    *path,
                  twin_fixed_t   x,
                  twin_fixed_t   y,
                  twin_fixed_t   w,
                  twin_fixed_t   h,
                  twin_fixed_t   x_radius,
                  twin_fixed_t   y_radius);

void twin_path_lozenge (twin_path_t  *path,
                  twin_fixed_t x,
                  twin_fixed_t y,
                  twin_fixed_t w,
                  twin_fixed_t h);
                  
void twin_path_tab (twin_path_t  *path,
                  twin_fixed_t x,
                  twin_fixed_t y,
                  twin_fixed_t w,
                  twin_fixed_t h,
                  twin_fixed_t x_radius,
                  twin_fixed_t y_radius);
                  
void twin_path_set_matrix (twin_path_t *path, twin_matrix_t matrix);

twin_matrix_t twin_path_current_matrix (twin_path_t *path);

void twin_path_identity (twin_path_t *path);

void twin_path_translate (twin_path_t *path, twin_fixed_t tx, twin_fixed_t ty);

void twin_path_scale (twin_path_t *path, twin_fixed_t sx, twin_fixed_t sy);

void twin_path_rotate (twin_path_t *path, twin_angle_t a);

void twin_path_set_font_size (twin_path_t *path, twin_fixed_t font_size);

twin_fixed_t twin_path_current_font_size (twin_path_t *path);

void twin_path_set_font_style (twin_path_t *path, twin_style_t font_style);

twin_style_t twin_path_current_font_style (twin_path_t *path);

void twin_path_set_cap_style (twin_path_t *path, twin_cap_t cap_style);

twin_cap_t twin_path_current_cap_style (twin_path_t *path);

void twin_path_empty (twin_path_t *path);

void twin_path_bounds (twin_path_t *path, twin_rect_t *rect);

void twin_path_append (twin_path_t *dst, twin_path_t *src);

twin_state_t twin_path_save (twin_path_t *path);

void twin_path_restore (twin_path_t *path, twin_state_t *state);

twin_path_t * twin_path_create (void);

void twin_path_destroy (twin_path_t *path);

void twin_composite_path (twin_pixmap_t  *dst,
                      twin_operand_t *src,
                      twin_coord_t   src_x,
                      twin_coord_t   src_y,
                      twin_path_t    *path,
                      twin_operator_t    operator);
                      
void twin_paint_path (twin_pixmap_t  *dst,
                      twin_argb32_t  argb,
                      twin_path_t    *path);
                      
void twin_composite_stroke (twin_pixmap_t    *dst,
                      twin_operand_t   *src,
                      twin_coord_t src_x,
                      twin_coord_t src_y,
                      twin_path_t  *stroke,
                      twin_fixed_t pen_width,
                      twin_operator_t  operator);
                      
void twin_paint_stroke (twin_pixmap_t    *dst,
                      twin_argb32_t    argb,
                      twin_path_t      *stroke,
                      twin_fixed_t     pen_width);
```

### Pattern

```c
static const struct {
  unsigned int   width;
  unsigned int   height;
  unsigned int   bytes_per_pixel; /* 3:RGB, 4:RGBA */
  char          *comment;
  unsigned char  pixel_data[8 * 8 * 4];
}

twin_pixmap_t * twin_make_pattern (void);
```

### Pixmap

```c
twin_pixmap_t * twin_pixmap_create (twin_format_t   format,
                        twin_coord_t    width,
                        twin_coord_t    height);

twin_pixmap_t * twin_pixmap_create_const (twin_format_t     format,
                        twin_coord_t      width,
                        twin_coord_t      height,
                        twin_coord_t      stride,
                        twin_pointer_t    pixels);
                        
void twin_pixmap_destroy (twin_pixmap_t *pixmap);

void twin_pixmap_show (twin_pixmap_t *pixmap,
                    twin_screen_t *screen,
                    twin_pixmap_t *lower);
                    
void twin_pixmap_hide (twin_pixmap_t *pixmap);

twin_pointer_t twin_pixmap_pointer (twin_pixmap_t *pixmap, twin_coord_t x, twin_coord_t y);

void twin_pixmap_enable_update (twin_pixmap_t *pixmap);

void twin_pixmap_disable_update (twin_pixmap_t *pixmap);

void twin_pixmap_set_origin (twin_pixmap_t *pixmap,
                          twin_coord_t ox, twin_coord_t oy);
                          
void twin_pixmap_offset (twin_pixmap_t *pixmap,   
                          twin_coord_t offx, twin_coord_t offy);
                      
void twin_pixmap_get_origin (twin_pixmap_t *pixmap,
                          twin_coord_t *ox, twin_coord_t *oy);
                          
void twin_pixmap_origin_to_clip (twin_pixmap_t *pixmap);

void twin_pixmap_clip (twin_pixmap_t *pixmap,
                    twin_coord_t  left,   twin_coord_t top,
                    twin_coord_t  right,  twin_coord_t bottom);
                    
void twin_pixmap_set_clip (twin_pixmap_t *pixmap, twin_rect_t clip);

twin_rect_t twin_pixmap_get_clip (twin_pixmap_t *pixmap);

twin_rect_t twin_pixmap_save_clip (twin_pixmap_t *pixmap);

void twin_pixmap_restore_clip (twin_pixmap_t *pixmap, twin_rect_t rect);

void twin_pixmap_reset_clip (twin_pixmap_t *pixmap);

void twin_pixmap_damage (twin_pixmap_t   *pixmap,
                    twin_coord_t    left,   twin_coord_t top,
                    twin_coord_t    right,  twin_coord_t bottom);
                    
static twin_argb32_t _twin_pixmap_fetch (twin_pixmap_t *pixmap, twin_coord_t x, twin_coord_t y);

twin_bool_t twin_pixmap_transparent (twin_pixmap_t *pixmap, twin_coord_t x, twin_coord_t y);

void twin_pixmap_move (twin_pixmap_t *pixmap, twin_coord_t x, twin_coord_t y);

twin_bool_t twin_pixmap_dispatch (twin_pixmap_t *pixmap, twin_event_t *event);
```

### Poly

```c
typedef struct _twin_edge {
    struct _twin_edge   *next;
    twin_sfixed_t   top, bot;
    twin_sfixed_t   x;
    twin_sfixed_t   e;
    twin_sfixed_t   dx, dy;
    twin_sfixed_t   inc_x;
    twin_sfixed_t   step_x;
    int         winding;
} twin_edge_t;

static int _edge_compare_y (const void *a, const void *b);

static void _edge_step_by (twin_edge_t  *edge, twin_sfixed_t dy);

static twin_sfixed_t _twin_sfixed_grid_ceil (twin_sfixed_t f);

static int _twin_edge_build (twin_spoint_t *vertices, int nvertices, twin_edge_t *edges, twin_sfixed_t dx, twin_sfixed_t dy, twin_sfixed_t top_y);

static void _span_fill (twin_pixmap_t   *pixmap,
                  twin_sfixed_t    y,
                  twin_sfixed_t    left,
                  twin_sfixed_t    right);
                  
static void _twin_edge_fill (twin_pixmap_t *pixmap, twin_edge_t *edges, int nedges);

void twin_fill_path (twin_pixmap_t *pixmap, twin_path_t *path,
                twin_coord_t dx, twin_coord_t dy);
```

### Primitive

```c
static twin_argb32_t __inline in_over (twin_argb32_t  dst,
                            twin_argb32_t  src,
                            twin_a8_t  msk);
                            
static twin_argb32_t __inline in (twin_argb32_t   src, twin_a8_t       msk);

static twin_argb32_t __inline over (twin_argb32_t dst, twin_argb32_t src);

static twin_argb32_t __inline rgb16_to_argb32 (twin_rgb16_t v);

static twin_argb32_t __inline a8_to_argb32 (twin_a8_t v);

static twin_rgb16_t __inline argb32_to_rgb16 (twin_argb32_t v);

static twin_a8_t __inline argb32_to_a8 (twin_argb32_t v);

static inline vector unsigned int over_v (vector unsigned char dst,
                                    vector unsigned char src);
                                    
void _twin_vec_argb32_over_argb32 (twin_pointer_t   dst,
                              twin_source_u    src,
                              int          width);
                              
void _twin_vec_argb32_source_argb32 (twin_pointer_t dst,
                              twin_source_u  src,
                              int            width);
```

### Queue

```c
void _twin_queue_insert (twin_queue_t    **head,
                    twin_queue_proc_t   proc,
                    twin_queue_t    *new);
                    
void _twin_queue_remove (twin_queue_t    **head, twin_queue_t    *old);

void _twin_queue_reorder (twin_queue_t   **head,
                    twin_queue_proc_t   proc,
                    twin_queue_t   *elem);
                    
void _twin_queue_delete (twin_queue_t    **head,
                    twin_queue_t    *old);
                    
twin_queue_t * _twin_queue_set_order (twin_queue_t **head);

void _twin_queue_review_order (twin_queue_t  *first);
```

### Screen

```c
twin_screen_t * twin_screen_create (twin_coord_t    width,
                    twin_coord_t    height,
                    twin_put_begin_t    put_begin,
                    twin_put_span_t put_span,
                    void        *closure);
                    
void twin_screen_destroy (twin_screen_t *screen);

void twin_screen_register_damaged (twin_screen_t *screen,
                    void (*damaged) (void *),
                    void *closure);
                    
void twin_screen_enable_update (twin_screen_t *screen);

void twin_screen_disable_update (twin_screen_t *screen);

void twin_screen_damage (twin_screen_t *screen,
                    twin_coord_t left, twin_coord_t top,
                    twin_coord_t right, twin_coord_t bottom);
                    
void twin_screen_resize (twin_screen_t *screen,
                    twin_coord_t width, twin_coord_t height);
                    
twin_bool_t twin_screen_damaged (twin_screen_t *screen);

static void twin_screen_span_pixmap(twin_screen_t maybe_unused *screen,
                    twin_argb32_t *span,
                    twin_pixmap_t *p, twin_coord_t y,
                    twin_coord_t left, twin_coord_t right,
                    twin_src_op op16, twin_src_op op32);
                    
void twin_screen_update (twin_screen_t *screen);

void twin_screen_set_active (twin_screen_t *screen, twin_pixmap_t *pixmap);

twin_pixmap_t * twin_screen_get_active (twin_screen_t *screen);

void twin_screen_set_background (twin_screen_t *screen, twin_pixmap_t *pixmap);

twin_pixmap_t * twin_screen_get_background (twin_screen_t *screen);

static void twin_screen_damage_cursor(twin_screen_t *screen);

void twin_screen_set_cursor (twin_screen_t *screen, twin_pixmap_t *pixmap,
                      twin_fixed_t hotspot_x, twin_fixed_t hotspot_y);
                      
static void twin_screen_update_cursor(twin_screen_t *screen,
                      twin_coord_t x, twin_coord_t y);
                      
static void _twin_adj_mouse_evt(twin_event_t *event, twin_pixmap_t *pixmap);

twin_bool_t twin_screen_dispatch (twin_screen_t *screen, twin_event_t  *event);
```

### Spline

```c
typedef struct _twin_spline {
    twin_spoint_t    a, b, c, d;
} twin_spline_t;

static void _lerp_half (twin_spoint_t *a, twin_spoint_t *b, twin_spoint_t *result);

static void _de_casteljau (twin_spline_t *spline, twin_spline_t *s1, twin_spline_t *s2);

static twin_dfixed_t _twin_spline_error_squared (twin_spline_t *spline);

static void _twin_spline_decompose (twin_path_t *path,
                          twin_spline_t   *spline,
                          twin_dfixed_t   tolerance_squared);
                          
void _twin_path_scurve (twin_path_t      *path,
                  twin_sfixed_t    x1, twin_sfixed_t y1,
                  twin_sfixed_t    x2, twin_sfixed_t y2,
                  twin_sfixed_t    x3, twin_sfixed_t y3);
                  
void twin_path_curve (twin_path_t    *path,
                  twin_fixed_t   x1, twin_fixed_t y1,
                  twin_fixed_t   x2, twin_fixed_t y2,
                  twin_fixed_t   x3, twin_fixed_t y3);
```

### Timeout

```c
static twin_order_t _twin_timeout_order (twin_queue_t *a, twin_queue_t *b);

static void _twin_queue_timeout (twin_timeout_t   *timeout, twin_time_t time);

void _twin_run_timeout (void);

twin_timeout_t * twin_set_timeout (twin_timeout_proc_t   timeout_proc,
                            twin_time_t       delay,
                            void          *closure);
                            
void twin_clear_timeout (twin_timeout_t *timeout);

twin_time_t _twin_timeout_delay (void);

twin_time_t twin_now (void);
```

### Toplevel

```c
twin_dispatch_result_t _twin_toplevel_dispatch (twin_widget_t *widget, 
                                          twin_event_t *event);

static twin_bool_t _twin_toplevel_event (twin_window_t   *window, 
                                    twin_event_t    *event);

static void _twin_toplevel_draw (twin_window_t    *window);

static void _twin_toplevel_destroy (twin_window_t *window);

void _twin_toplevel_init (twin_toplevel_t        *toplevel,
                      twin_dispatch_proc_t   dispatch,
                      twin_window_t      *window,
                      const char         *name);
                      
twin_toplevel_t * twin_toplevel_create (twin_screen_t     *screen,
                                    twin_format_t     format,
                                    twin_window_style_t   style,
                                    twin_coord_t      x,
                                    twin_coord_t      y,
                                    twin_coord_t      width,
                                    twin_coord_t      height,
                                    const char        *name);
                                    
 static twin_bool_t _twin_toplevel_paint (void *closure);
 
 void _twin_toplevel_queue_paint (twin_widget_t *widget);
 
 static twin_bool_t _twin_toplevel_layout (void *closure);
 
 void _twin_toplevel_queue_layout (twin_widget_t *widget);
 
 void twin_toplevel_show (twin_toplevel_t *toplevel);
```

### Trig

```c
static const uint16_t _twin_sin_table[1 << TWIN_LOG2_SIN] = {};

twin_fixed_t twin_sin (twin_angle_t a);

twin_fixed_t twin_cos (twin_angle_t a);

twin_fixed_t twin_tan (twin_angle_t a);
```

### Widget

```c
static twin_path_t * _twin_path_shape (twin_shape_t  shape,
                                  twin_coord_t  left,
                                  twin_coord_t  top,
                                  twin_coord_t  right,
                                  twin_coord_t  bottom,
                                  twin_fixed_t  radius);
                                  
void _twin_widget_paint_shape (twin_widget_t *widget,
                          twin_shape_t  shape,
                          twin_coord_t  left,
                          twin_coord_t  top,
                          twin_coord_t  right,
                          twin_coord_t  bottom,
                          twin_fixed_t  radius);
                          
static void _twin_widget_paint (twin_widget_t *widget);

twin_dispatch_result_t _twin_widget_dispatch (twin_widget_t *widget, 
                                        twin_event_t *event);
                                        
void _twin_widget_init (twin_widget_t    *widget,
                    twin_box_t       *parent,
                    twin_window_t    *window,
                    twin_widget_layout_t preferred,
                    twin_dispatch_proc_t dispatch);
                    
void _twin_widget_queue_paint (twin_widget_t   *widget);

void _twin_widget_queue_layout (twin_widget_t   *widget);

twin_bool_t _twin_widget_contains (twin_widget_t    *widget,
                              twin_coord_t x, twin_coord_t y);
                              
void _twin_widget_bevel (twin_widget_t   *widget,
                    twin_fixed_t    b,
                    twin_bool_t     down);
                    
twin_widget_t * twin_widget_create (twin_box_t      *parent,
                                twin_argb32_t   background,
                                twin_coord_t    width,
                                twin_coord_t    height,
                                twin_stretch_t  stretch_width,
                                twin_stretch_t  stretch_height);
                                
void twin_widget_set (twin_widget_t *widget, twin_argb32_t background);
```

### Window

```c
twin_window_t * twin_window_create (twin_screen_t   *screen,
                              twin_format_t   format,
                              twin_window_style_t style,
                              twin_coord_t    x,
                              twin_coord_t    y,
                              twin_coord_t    width,
                              twin_coord_t    height);
                              
void twin_window_destroy (twin_window_t *window);

void twin_window_show (twin_window_t *window);

void twin_window_hide (twin_window_t *window);

void twin_window_configure (twin_window_t        *window,
                      twin_window_style_t  style,
                      twin_coord_t     x,
                      twin_coord_t     y,
                      twin_coord_t     width,
                      twin_coord_t     height);
                      
void twin_window_style_size (twin_window_style_t style,
                        twin_rect_t     *size);
                        
void twin_window_set_name (twin_window_t *window, const char    *name);

static void twin_window_frame (twin_window_t *window);

void twin_window_draw (twin_window_t *window);

void twin_window_damage (twin_window_t *window,
                      twin_coord_t left, twin_coord_t top,
                      twin_coord_t right, twin_coord_t bottom);
                      
static twin_bool_t _twin_window_repaint (void *closure);

void twin_window_queue_paint (twin_window_t *window);

twin_bool_t twin_window_dispatch (twin_window_t *window, twin_event_t *event);
```

### Work

```c
static twin_order_t _twin_work_order (twin_queue_t *a, twin_queue_t *b);

static void _twin_queue_work (twin_work_t   *work);

void _twin_run_work (void);

twin_work_t * twin_set_work (twin_work_proc_t work_proc,
                        int           priority,
                        void          *closure);
                        
void twin_clear_work (twin_work_t *work);
```


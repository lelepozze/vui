# Vui syntax

Obtained by CODEX from the file `examples/ctrl_gallery.c` syntax and function reference

This guide documents the functions and macros used by [`ctrl_gallery.c`](examples/ctrl_gallery.c). The API descriptions follow its quoted headers: `../vui.h`, `../backends/vui_sdl2.h`, `../backends/vui_opengl.h`, `../backends/vui_stbtruetype.h`, and `../backends/vui_stbtruetype_manager.h`. SDL2, OpenGL, and stb_truetype declarations come from angle-bracket includes and are external dependencies.

## How to read the calls

Most VUI controls use this shape:

```c
function(sibling_id, ...control arguments..., style_array);
```

`vui_sib_id` is a macro that expands to the current source line number, giving a convenient stable sibling ID as long as each control call stays on its own line. Use an explicit `VuiCtrlSibId` when an ID must be stored or shared, as with select and radio controls. Style arguments such as `vui_ss.button_action` are arrays of `VuiCtrlStyle`, typically describing the control's different states.

Text convenience macros take NUL-terminated strings and determine their length with `strlen`. The underlying `_` forms accept an explicit byte length. `VuiBool` uses `vui_true` and `vui_false`. Passing `NULL` for a toggle/check state means the caller does not need to persist that state; selection controls instead share a pointer to the selected sibling ID.

Several APIs use paired `*_start` / `*_end` calls to establish nested UI scope. Their scope macros, such as `vui_scope_ctrl(id, style) { ... }`, call the matching end function automatically when the block exits. The gallery also uses one-line scope macros (`vui_scope_width(200.f)`) that apply to the next statement/block.

## Frame and layout functions

| Syntax | Purpose |
|---|---|
| `vui_frame_start(right_to_left, dt)` | Starts a UI frame. The gallery passes `vui_false` for left-to-right layout and elapsed frame time in milliseconds. |
| `vui_frame_end()` | Completes frame processing after all controls are declared. |
| `vui_window_start(window_id, size)` / `vui_window_end()` | Begins/ends a VUI window. `size` is a `VuiVec2`; the example uses the screen dimensions. |
| `vui_row_layout()` | Places children in a row. |
| `vui_column_layout()` | Places children in a column. |
| `vui_stack_layout()` | Places children in a stack (declared by the core API; not directly called by this example). |
| `vui_scroll_view_start(id, flags, style)` / `vui_scroll_view_end()` | Opens/closes a scrolling content region. The macro supplies null state pointers; `_` form accepts `VuiVec2*` pointers for content offset and resizable size. |
| `vui_scope_ctrl(id, style) { ... }` | Opens a control/container scope and automatically closes it after the block. `NULL` style is used by the gallery for its layout-only grouping controls. |
| `vui_scope_width(width)` / `vui_scope_height(height)` | Sets dimensions for the next control or block. `vui_fill_len` means consume available space; a positive number is a pixel size. |
| `vui_scope_align(align) { ... }` | Sets alignment for the nested statement/block; the example uses `VuiAlign_left_center`. |
| `vui_scope_style_transition_time(ms) { ... }` | Sets the style transition duration for controls in the block. |

`VuiScrollFlags_vhr` combines vertical scrolling, horizontal scrolling, and resize support. Other flags include `VuiScrollFlags_vertical`, `VuiScrollFlags_horizontal`, and the `*_always_show` variants.

## Display controls

| Syntax | Purpose / result |
|---|---|
| `vui_text(id, text, wrap_width, style)` | Displays text. `wrap_width == 0.f` disables wrapping; a positive width wraps text. Newlines in the string create explicit line breaks. |
| `vui_text_(id, text, text_length, wrap_width, style)` | Explicit-length form of `vui_text`; length is in bytes. |
| `vui_image(id, image_id, tint, style)` | Displays an image registered with `vui_image_add`; tint is multiplied with image color. White preserves its appearance. |
| `vui_separator(id, style)` | Draws a separator using the selected style. |
| `vui_spacing(id, width, height)` | Adds a fixed-size blank layout item (available from the core header; not directly called in this file). |

## Buttons and selection controls

Every control below receives a sibling ID and style array. Button-like APIs return a focus state or boolean as described; the gallery mostly demonstrates appearance and ignores those values.

| Syntax | Purpose / result |
|---|---|
| `vui_text_button(id, text, style)` | Text button; returns `VuiFocusState`. |
| `vui_image_button(id, image_id, tint, style)` | Image-only button; returns `VuiFocusState`. |
| `vui_image_text_button(id, image_id, tint, text, style)` | Button with image and text; returns `VuiFocusState`. |
| `vui_button_start(id, style)` / `vui_button_end()` | Low-level begin/end form for a custom button body; start returns focus state. |
| `vui_button(id, style)` | Basic button control without built-in text/image content; returns focus state. |
| `vui_text_toggle_button(id, pressed_ptr, text, style)` | Text toggle button. Updates the optional `VuiBool` state and returns whether it is pressed. |
| `vui_image_toggle_button(id, pressed_ptr, image_id, tint, style)` | Image toggle variant. |
| `vui_image_text_toggle_button(id, pressed_ptr, image_id, tint, text, style)` | Image-and-text toggle variant. |
| `vui_toggle_button_start(id, pressed_ptr, style)` / `vui_toggle_button_end()` | Low-level toggle scope; start returns the pressed state. |
| `vui_text_select_button(id, selected_id_ptr, text, style)` | Text option in a group. Selecting it writes its `id` to the shared `VuiCtrlSibId*`; returns whether selected. |
| `vui_image_select_button(id, selected_id_ptr, image_id, tint, style)` | Image option in a selection group. |
| `vui_image_text_select_button(id, selected_id_ptr, image_id, tint, text, style)` | Image-and-text selection option. |
| `vui_select_button_start(id, selected_id_ptr, style)` / `vui_select_button_end()` | Low-level selection control scope; start returns whether selected. |
| `vui_text_check_box(id, checked_ptr, text, style)` | Text checkbox. Updates optional `VuiBool*`; returns checked state. |
| `vui_image_check_box_(id, checked_ptr, image_id, tint, style)` | Image checkbox form (the header exposes this function with a trailing underscore). |
| `vui_image_text_check_box(id, checked_ptr, image_id, tint, text, style)` | Image-and-text checkbox convenience macro. |
| `vui_check_box(id, checked_ptr, style)` | Checkbox without attached label/image; returns checked state. |
| `vui_check_box_aux_start(id, checked_ptr, style)` / `vui_check_box_aux_end()` | Lower-level auxiliary checkbox scope. |
| `vui_text_radio_button(id, selected_id_ptr, text, style)` | Text radio option in a shared selection group; returns whether selected. |
| `vui_image_radio_button(id, selected_id_ptr, image_id, tint, style)` | Image radio option. |
| `vui_image_text_radio_button(id, selected_id_ptr, image_id, tint, text, style)` | Image-and-text radio option. |
| `vui_radio_button(id, selected_id_ptr, style)` | Radio option without attached text/image. |
| `vui_radio_button_aux_start(id, selected_id_ptr, style)` / `vui_radio_button_aux_end()` | Lower-level auxiliary radio scope. |

For the text convenience macros, the corresponding `name_` function takes `char* text` plus `uint32_t text_length`. The gallery uses `NULL` for toggle and checkbox state where it does not need an external state variable. Its select and radio examples each share one `selected_sib_id` across options.

## Values, progress, and text entry

| Syntax | Purpose / result |
|---|---|
| `vui_slider_uint(id, &value, min, max, style)` | Edits an unsigned 32-bit integer in the inclusive range. Returns true when the slider is released. |
| `vui_slider_sint(id, &value, min, max, style)` | Edits a signed 32-bit integer. |
| `vui_slider_float(id, &value, min, max, style)` | Edits a float. |
| `vui_progress_bar(id, value, min, max, style)` | Displays progress corresponding to `value` within the supplied range; it is not an input control. |
| `vui_text_box(id, buffer, capacity, style)` | Edits a caller-owned character buffer, with capacity in bytes; returns whether the value changed. |
| `vui_input_box_uint(id, &value, style)` | Text input for `uint32_t`; returns whether changed. |
| `vui_input_box_sint(id, &value, style)` | Text input for `int32_t`; returns whether changed. |
| `vui_input_box_float(id, &value, style)` | Text input for `float`; returns whether changed. |
| `vui_text_box_multiline(id, buffer, capacity, flags, style)` | Multiline editable text area with scroll behavior; returns whether changed. The macro supplies null offset/size pointers; `_` form allows those to be passed. |

## Functions implemented by the example

| Function | Explanation |
|---|---|
| `build_ui(void)` | Builds one complete UI frame: starts frame/window/scroll scopes, lays out the gallery sections, creates controls, and closes each scope. The static values inside retain state between calls. |
| `opengl_assert_no_error()` / `_opengl_assert_no_error(file, line)` | Macro captures source location and checks `glGetError()`. The helper maps known OpenGL error codes to a message and asserts on any error; unknown errors also assert. |
| `mat4x4_ortho(out, left, right, bottom, top, znear, zfar)` | Writes a 4×4 orthographic projection matrix into `out`, used to map pixel coordinates to OpenGL clip space. |
| `file_read_all(path, bytes_out)` | Reads a binary file into a `VuiStk(char)` dynamic stack buffer. Returns `0` on success, otherwise an `errno` value (or `ENOMEM` if capacity allocation fails). |
| `opengl_texture_gen(void)` | Creates and returns an OpenGL 2D texture name. |
| `opengl_texture_set_pixels(tex, internal_format, format, width, height, pixels)` | Binds a texture, configures linear filtering and border clamp, uploads pixel data with `glTexImage2D`, then unbinds it. |
| `opengl_compile_and_use_shader(void)` | Compiles the global vertex and fragment GLSL strings, links a program, makes it current, and stores its ID in `app.opengl.program`. Exits if either shader compilation fails. |
| `glyph_texture_pack_and_update(glyph_texture_id, texture_id)` | Packs queued glyphs through the stb_truetype manager, obtains the pixel buffer and square dimensions, then uploads it as a red-channel OpenGL texture. |
| `app_vui_stbtt_get_styled_glyph_texture_id(font_id, line_height, codept)` | Required manager callback: routes printable ASCII codepoints 33–126 to the cached ASCII atlas and all others to the atlas repacked during updates. |
| `app_button_animate_fn(ctrl, dt, interp_ratio, changed_this_frame)` | Custom pre-animation callback for action buttons. Captures the mouse position on state change and draws a circle ripple while transitioning into/out of active state. |
| `app_progress_bar_animate_fn(ctrl, dt, interp_ratio, changed_this_frame)` | Custom post-animation callback that moves a translucent highlight along the progress bar, resetting when it reaches the end. |
| `App_resize(width, height)` | Rebuilds the orthographic matrix for the new drawable dimensions and updates the OpenGL viewport. |
| `App_init(void)` | Initializes SDL and its OpenGL context, VUI's OpenGL shim and shader, textures, font and glyph atlases, VUI setup/styles, sample images, and blending. |
| `App_update(void)` | Calculates frame time, clears the dynamic glyph list, transfers SDL input into VUI, builds the UI, packs dynamic glyphs, and uploads their atlas. |
| `vui_opengl_setup_state(void)` | Implements the OpenGL backend callback: binds the shader, sets the projection uniform, enables vertex attributes, and describes the position/UV/color fields in `VuiVertex`. |
| `App_render(void)` | Gets the render-command data for the main VUI window and sends it to the OpenGL backend. |
| `main(argc, argv)` | Initializes the app, then loops over SDL events, handles resize/quit, updates and renders VUI, and swaps the SDL OpenGL window. |

## Backend and font functions called by the example

These are declared in the quoted backend headers and implemented by the included backend `.c` files:

| Function | Purpose |
|---|---|
| `vui_sdl2_process_event(&event)` | Passes each polled SDL event to VUI's SDL input adapter. |
| `vui_sdl2_frame_start()` | Copies the accumulated SDL input state into VUI; call after event processing and before `vui_frame_start`. |
| `vui_opengl_init()` | Initializes the VUI OpenGL adapter after creating the GL context. |
| `vui_opengl_render(window_render)` | Executes the render commands in `VuiWindowRender`; requires the shader and vertex attribute state to be configured. |
| `vui_init(&setup)` | Initializes the VUI core with a `VuiSetup`; the gallery supplies its text-position callback, one window, allocator default, and default font. |
| `vui_ss` | Core style set used throughout the gallery (`text_header`, `button_action`, `text_box`, and other style arrays). It is a style value, not a function. |
| `vui_image_add(&image)` | Registers a `VuiImage` description and returns its image ID. |
| `vui_window_render(id, scale, pixel_snapping)` | Builds/returns the render data for a window. The example uses scale `1.f` and disables pixel snapping. |
| `vui_ctrl_get(ctrl_id)` | Looks up the progress bar's child control so its rectangle can be used to draw the animated highlight. |
| `vui_mouse_pos()` | Returns current pointer position; used as the origin for the button ripple animation. |
| `vui_render_circle(position, radius, color)` | Adds a circle to the current VUI render output; used by the button ripple callback. |
| `vui_render_rect(&rect, color, radius)` | Adds a rounded rectangle to the render output; used for the progress bar highlight. |
| `VuiRect_width(&rect)` / `VuiRect_height(&rect)` | Inline geometry helpers that return rectangle dimensions. |
| `VuiVec2_add(a, b)` | Adds vector components; used to place the animated highlight rectangle. |
| `vui_assert(condition, format, ...)` | Aborts with a diagnostic if initialization or file loading fails. |
| `vui_stbtt_font_add(font_bytes)` | Registers TrueType font bytes and returns a `VuiFontId`; the bytes must remain alive while the font is registered. |
| `vui_stbtt_font_get(font_id)` | Returns the stb_truetype font info for that font. |
| `vui_stbtt_glyph_texture_add(texture_id, width, margin, oversample_x, oversample_y)` | Creates a managed glyph atlas associated with a VUI texture ID. Zero-valued sizing/oversampling parameters select manager defaults. |
| `vui_stbtt_glyph_texture_add_styled_glyph(atlas_id, font_id, line_height, glyph_index)` | Queues a glyph at the requested rendered line height for atlas packing. |
| `vui_stbtt_glyph_texture_pack(atlas_id)` | Renders queued glyphs into the manager's pixel buffer. |
| `vui_stbtt_glyph_texture_get_pixels_and_wh(atlas_id, &width)` | Returns packed atlas pixels and optionally writes its square dimension. |
| `vui_stbtt_glyph_texture_clear_styled_glyphs(atlas_id)` | Clears the queued glyphs before rebuilding the dynamic atlas for a frame. |
| `vui_stbtt_position_text(args)` | Font backend callback supplied in `VuiSetup`; measures and positions text for the VUI core. |

The stb_truetype manager header also declares font/atlas removal, atlas resize, and the lower-level rendering/query callbacks. Those are library facilities, not functions called in this example.

The source also calls SDL2 and OpenGL functions directly: `SDL_Init`, `SDL_GL_SetAttribute`, `SDL_CreateWindow`, `SDL_GL_CreateContext`, `SDL_GL_SetSwapInterval`, `SDL_GetPerformanceCounter`, `SDL_GetPerformanceFrequency`, `SDL_PollEvent`, and `SDL_GL_SwapWindow`; plus OpenGL texture, shader/program, viewport, blending, uniform, vertex-attribute, and clear operations (`glGenTextures`, `glBindTexture`, `glTexParameteri`, `glTexImage2D`, shader compile/link calls, `glViewport`, `glEnable`, `glBlendFunc`, `glUseProgram`, uniform/attribute calls, `glGetError`, and `glClear*`). Their syntax and semantics belong to SDL2/OpenGL rather than the quoted VUI headers; here they initialize the window/context and upload/render VUI data.

## Common supporting types and macros

- `VuiVec2_init(x, y)` creates a 2D float vector; `VuiRect_init(...)` creates a rectangle; `VuiColor_white` is the white tint used by image controls.
- `VuiImage` describes pixel dimensions, texture ID, and UV rectangle. `vui_image_add(&image)` registers it and returns a `VuiImageId`.
- `vui_window_render(id, scale, pixel_snapping)` returns the render data consumed by the renderer.
- `vui_lerp(to, from, t)` interpolates from `from` to `to`; the example uses it to animate ripple radii.
- `VuiStk(T)` is the core dynamic-stack pointer type. `VuiStk_cap`, `VuiStk_resize_cap`, and `_VuiStk_header(...)->count` are used by `file_read_all` to allocate and set the number of bytes read.
- `vui_assert(condition, format, ...)` aborts with source information when its condition is false.


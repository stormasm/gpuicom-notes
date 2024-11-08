
- [gpui_macros/gpui_macros.rs](https://github.com/zed-industries/zed/blob/main/crates/gpui_macros/src/gpui_macros.rs)
- [gpui_macros/derive_into_element.rs](https://github.com/zed-industries/zed/blob/main/crates/gpui_macros/src/derive_into_element.rs)

```rust
#[derive(IntoElement)]
```

is used to create a Component out of anything that implements the `RenderOnce` trait.

- accordian
- button
- buttongroup
- checkbox
- divider
- drawer
- icon
- indicator
- label
- link
- list/listitem
- modal
- progress
- radio
- resizable/resize_handle
- skeleton
- tab/tab
- tab/tabbar
- titlebar

Along with...

- crates/story/src/list_story -> *CompanyListItem*

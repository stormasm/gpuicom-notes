
```rust
#[derive(IntoElement)]
```

is used to create a Component out of anything that implements the `RenderOnce` trait.

These are the gpui-components.

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

### References

- [gpui_macros/gpui_macros.rs](https://github.com/zed-industries/zed/blob/main/crates/gpui_macros/src/gpui_macros.rs)
- [gpui_macros/derive_into_element.rs](https://github.com/zed-industries/zed/blob/main/crates/gpui_macros/src/derive_into_element.rs)

---

See [element.rs](https://github.com/zed-industries/zed/blob/main/crates/gpui/src/element.rs)
and search for *Component* for more details...

```rust
//! However, most of the time, you won't need to implement your own elements. GPUI provides a number of
//! elements that should cover most common use cases out of the box and it's recommended that you use those
//! to construct `components`, using the [`RenderOnce`] trait and the `#[derive(IntoElement)]` macro.

//! Only implement elements when you need to take manual control of the layout and painting process,
//! such as when using your own custom layout algorithm or rendering a code editor.

//! Part II

/// You can derive [`IntoElement`] on any type that implements this trait.

/// It is used to construct reusable `components` out of plain data. Think of
/// components as a recipe for a certain pattern of elements. RenderOnce allows
/// you to invoke this pattern, without breaking the fluent builder pattern of
/// the element APIs.
pub trait RenderOnce: 'static {
    /// Render this component into an element tree. Note that this method
    /// takes ownership of self, as compared to [`Render::render()`] method
    /// which takes a mutable reference.
    fn render(self, cx: &mut WindowContext) -> impl IntoElement;
}
```


```

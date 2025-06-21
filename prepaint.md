
##### dock.rs

```rust
fn prepaint(
     &mut self,
     _: Option<&gpui::GlobalElementId>,
     _: Option<&gpui::InspectorElementId>,
     _: gpui::Bounds<Pixels>,
     _: &mut Self::RequestLayoutState,
     _window: &mut gpui::Window,
     _cx: &mut App,
 ) -> Self::PrepaintState {
     ()
 }
```

---

Too big for the code to be located here !

- [element.rs](https://github.com/longbridge/gpui-component/blob/main/crates/ui/src/input/element.rs)

---

##### markdown.rs

```rust
fn prepaint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<gpui::Pixels>,
    request_layout: &mut Self::RequestLayoutState,
    window: &mut Window,
    cx: &mut gpui::App,
) -> Self::PrepaintState {
    request_layout.prepaint(window, cx);
}
```

##### panel.rs

```rust
fn prepaint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    _window: &mut Window,
    _cx: &mut App,
) -> Self::PrepaintState {
    ()
}
```

##### scrollbar.rs

Too big for the code to be located here !

- [scrollbar.rs](https://github.com/longbridge/gpui-component/blob/main/crates/ui/src/scrollbar.rs)

##### svg_img.rs

```rust
fn prepaint(
    &mut self,
    global_id: Option<&GlobalElementId>,
    inspector_id: Option<&gpui::InspectorElementId>,
    bounds: Bounds<Pixels>,
    state: &mut Self::RequestLayoutState,
    window: &mut Window,
    cx: &mut App,
) -> Self::PrepaintState {
    let hitbox = self.interactivity.prepaint(
        global_id,
        inspector_id,
        bounds,
        bounds.size,
        window,
        cx,
        |_, _, hitbox, _, _| hitbox,
    );

    (hitbox, state.clone())
}
```

##### switch.rs

```rust
fn prepaint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<gpui::Pixels>,
    element: &mut Self::RequestLayoutState,
    window: &mut Window,
    cx: &mut App,
) {
    element.prepaint(window, cx);
}
```

##### title_bar.rs

```rust
fn prepaint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    _window: &mut Window,
    _cx: &mut App,
) -> Self::PrepaintState {
}
```



##### dock.rs

```rust
fn paint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    _: &mut Self::PrepaintState,
    window: &mut gpui::Window,
    cx: &mut App,
) {
    window.on_mouse_event({
        let view = self.view.clone();
        let resizing = view.read(cx).resizing;
        move |e: &MouseMoveEvent, phase, window, cx| {
            if !resizing {
                return;
            }
            if !phase.bubble() {
                return;
            }

            view.update(cx, |view, cx| view.resize(e.position, window, cx))
        }
    });

    // When any mouse up, stop dragging
    window.on_mouse_event({
        let view = self.view.clone();
        move |_: &MouseUpEvent, phase, window, cx| {
            if phase.bubble() {
                view.update(cx, |view, cx| view.done_resizing(window, cx));
            }
        }
    })
}
}
```

##### element.rs

---

Too big for the code to be located here !

- [element.rs](https://github.com/longbridge/gpui-component/blob/main/crates/ui/src/input/element.rs)

---

##### markdown.rs

```rust
fn paint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<gpui::Pixels>,
    request_layout: &mut Self::RequestLayoutState,
    _: &mut Self::PrepaintState,
    window: &mut Window,
    cx: &mut gpui::App,
) {
    request_layout.paint(window, cx);
}
}
```

##### resizable/panel.rs

```rust
fn paint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    _: &mut Self::PrepaintState,
    window: &mut Window,
    cx: &mut App,
) {
    window.on_mouse_event({
        let state = self.state.clone();
        let axis = self.axis;
        let current_ix = state.read(cx).resizing_panel_ix;
        move |e: &MouseMoveEvent, phase, window, cx| {
            if !phase.bubble() {
                return;
            }
            let Some(ix) = current_ix else { return };

            state.update(cx, |state, cx| {
                let panel = state.panels.get(ix).expect("BUG: invalid panel index");

                match axis {
                    Axis::Horizontal => {
                        state.resize_panel(ix, e.position.x - panel.bounds.left(), window, cx)
                    }
                    Axis::Vertical => {
                        state.resize_panel(ix, e.position.y - panel.bounds.top(), window, cx);
                    }
                }
                cx.notify();
            })
        }
    });

    // When any mouse up, stop dragging
    window.on_mouse_event({
        let state = self.state.clone();
        let current_ix = state.read(cx).resizing_panel_ix;
        move |_: &MouseUpEvent, phase, _, cx| {
            if current_ix.is_none() {
                return;
            }
            if phase.bubble() {
                state.update(cx, |state, cx| state.done_resizing(cx));
            }
        }
    })
}
}
```

---

##### scrollbar.rs

Too big for the code to be located here !

- [scrollbar.rs](https://github.com/longbridge/gpui-component/blob/main/crates/ui/src/scroll/scrollbar.rs)

---

##### svg_img.rs

```rust
fn paint(
    &mut self,
    global_id: Option<&GlobalElementId>,
    inspector_id: Option<&gpui::InspectorElementId>,
    bounds: Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    state: &mut Self::PrepaintState,
    window: &mut Window,
    cx: &mut App,
) {
    let hitbox = state.0.as_ref();
    let Some(image) = state.1.take() else {
        return;
    };
    let size = image.size(0).map(|x| x.0 as f32);

    self.interactivity.paint(
        global_id,
        inspector_id,
        bounds,
        hitbox,
        window,
        cx,
        |_, window, _| {
            // To calculate the ratio of the original image size to the container bounds size.
            // Scale by shortest side (width or height) to get a fit image.
            // And center the image in the container bounds.
            let ratio = if bounds.size.width < bounds.size.height {
                bounds.size.width / size.width
            } else {
                bounds.size.height / size.height
            };

            let ratio = ratio.0.min(1.0);
            let new_size = size.map(|dim| px(dim) * ratio);

            let new_origin = gpui::Point {
                x: bounds.origin.x + px(((bounds.size.width - new_size.width) / 2.).into()),
                y: bounds.origin.y + px(((bounds.size.height - new_size.height) / 2.).into()),
            };

            let img_bounds = Bounds {
                origin: new_origin.map(|origin| origin.floor()),
                size: new_size.map(|size| size.ceil()),
            };

            if let Err(err) = window.paint_image(img_bounds, px(0.).into(), image, 0, false) {
                eprintln!("failed to paint svg image: {:?}", err);
            }
        },
    )
}
}
```

##### switch.rs

```rust
fn paint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    _: gpui::Bounds<gpui::Pixels>,
    element: &mut Self::RequestLayoutState,
    _: &mut Self::PrepaintState,
    window: &mut Window,
    cx: &mut App,
) {
    element.paint(window, cx)
}
}
```

##### title_bar.rs

```rust
#[allow(unused_variables)]
fn paint(
    &mut self,
    _: Option<&gpui::GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    bounds: gpui::Bounds<Pixels>,
    _: &mut Self::RequestLayoutState,
    _: &mut Self::PrepaintState,
    window: &mut Window,
    cx: &mut App,
) {
    use gpui::{MouseButton, MouseMoveEvent, MouseUpEvent};
    window.on_mouse_event(
        move |ev: &MouseMoveEvent, _, window: &mut Window, cx: &mut App| {
            if bounds.contains(&ev.position) && ev.pressed_button == Some(MouseButton::Left) {
                window.start_window_move();
            }
        },
    );

    window.on_mouse_event(
        move |ev: &MouseUpEvent, _, window: &mut Window, cx: &mut App| {
            if bounds.contains(&ev.position) && ev.button == MouseButton::Right {
                window.show_window_menu(ev.position);
            }
        },
    );
}
}
```

---

##### The following components are being reviewed for implementing element, request_layout, paint and prepaint.

```rust
impl Element
```

- chart, plot
- clipboard
- dock
- input
- menu
- popover
- resizable, scroll
- svg_img
- switch
- text
- title_bar
- virtual_list
- webview

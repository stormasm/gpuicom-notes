
##### dock.rs

```rust
fn request_layout(
        &mut self,
        _: Option<&gpui::GlobalElementId>,
        _: Option<&gpui::InspectorElementId>,
        window: &mut gpui::Window,
        cx: &mut App,
    ) -> (gpui::LayoutId, Self::RequestLayoutState) {
        (window.request_layout(Style::default(), None, cx), ())
    }
```

##### element.rs

```rust
fn request_layout(
        &mut self,
        _id: Option<&GlobalElementId>,
        _: Option<&gpui::InspectorElementId>,
        window: &mut Window,
        cx: &mut App,
    ) -> (LayoutId, Self::RequestLayoutState) {
        let input = self.input.read(cx);
        let line_height = window.line_height();

        let mut style = Style::default();
        style.size.width = relative(1.).into();
        if self.input.read(cx).is_multi_line() {
            style.flex_grow = 1.0;
            if let Some(h) = input.mode.height() {
                style.size.height = h.into();
                style.min_size.height = line_height.into();
            } else {
                style.size.height = relative(1.).into();
                style.min_size.height = (input.mode.rows() * line_height).into();
            }
        } else {
            // For single-line inputs, the minimum height should be the line height
            style.size.height = line_height.into();
        };

        (window.request_layout(style, [], cx), ())
    }
```

##### markdown.rs

```rust
fn request_layout(
        &mut self,
        id: Option<&gpui::GlobalElementId>,
        _: Option<&gpui::InspectorElementId>,
        window: &mut Window,
        cx: &mut gpui::App,
    ) -> (gpui::LayoutId, Self::RequestLayoutState) {
        window.with_element_state(id.unwrap(), |state, window| {
            let mut state: MarkdownState = state.unwrap_or_default();
            state.parse_if_needed(self.text.clone(), &self.style, cx);

            let root = state
                .root
                .clone()
                .expect("BUG: root should not None, maybe parse_if_needed issue.");

            let mut el = div()
                .map(|this| match root {
                    Ok(node) => this.child(node.render(None, true, &self.style, window, cx)),
                    Err(err) => this.child(
                        v_flex()
                            .gap_1()
                            .child("Error parsing Markdown")
                            .child(err.to_string()),
                    ),
                })
                .into_any_element();

            let layout_id = el.request_layout(window, cx);

            ((layout_id, el), state)
        })
    }
```

##### panel.rs

```rust
fn request_layout(
        &mut self,
        _: Option<&gpui::GlobalElementId>,
        _: Option<&gpui::InspectorElementId>,
        window: &mut Window,
        cx: &mut App,
    ) -> (gpui::LayoutId, Self::RequestLayoutState) {
        (window.request_layout(Style::default(), None, cx), ())
    }
```

##### scrollbar.rs

```rust
fn request_layout(
        &mut self,
        _: Option<&GlobalElementId>,
        _: Option<&InspectorElementId>,
        window: &mut Window,
        cx: &mut App,
    ) -> (LayoutId, Self::RequestLayoutState) {
        let mut style = Style::default();
        style.position = Position::Absolute;
        style.flex_grow = 1.0;
        style.flex_shrink = 1.0;
        style.size.width = relative(1.).into();
        style.size.height = relative(1.).into();

        (window.request_layout(style, None, cx), ())
    }
```

##### svg_img.rs

```rust
fn request_layout(
        &mut self,
        global_id: Option<&GlobalElementId>,
        inspector_id: Option<&gpui::InspectorElementId>,
        window: &mut Window,
        cx: &mut App,
    ) -> (gpui::LayoutId, Self::RequestLayoutState) {
        let layout_id = self.interactivity.request_layout(
            global_id,
            inspector_id,
            window,
            cx,
            |style, window, cx| window.request_layout(style, None, cx),
        );

        let global_id = global_id.unwrap();
        let source = &self.source;
        let source_hash = hash(source);

        window.with_element_state::<Option<SvgImgState>, _>(global_id, |state, window| {
            match state {
                Some(state) => {
                    // Try to keep the previous image if it's still loading.
                    let mut prev_image = None;
                    if let Some(mut state) = state {
                        prev_image = state.image.clone();
                        if source_hash == state.hash {
                            state.image = state
                                .task
                                .clone()
                                .now_or_never()
                                .transpose()
                                .ok()
                                .flatten()
                                .or(state.image);

                            return ((layout_id, state.image.clone()), Some(state));
                        }
                    }

                    let task = load_svg(source, window, cx);
                    let mut image = task.clone().now_or_never().transpose().ok().flatten();
                    if let Some(new_image) = image.as_ref() {
                        _ = window.drop_image(new_image.clone());
                    } else {
                        image = prev_image;
                    }

                    (
                        (layout_id, image.clone()),
                        Some(SvgImgState {
                            hash: source_hash,
                            image,
                            task,
                        }),
                    )
                }
                None => {
                    let task = load_svg(source, window, cx);
                    (
                        (layout_id, None),
                        Some(SvgImgState {
                            hash: source_hash,
                            image: None,
                            task,
                        }),
                    )
                }
            }
        })
    }
```

##### switch.rs

```rust
fn request_layout(
    &mut self,
    global_id: Option<&GlobalElementId>,
    _: Option<&gpui::InspectorElementId>,
    window: &mut Window,
    cx: &mut App,
) -> (LayoutId, Self::RequestLayoutState) {
    window.with_element_state::<SwitchState, _>(global_id.unwrap(), move |state, window| {
        let state = state.unwrap_or_default();
        let checked = self.checked;
        let on_click = self.on_click.clone();
        let style = self.base.style();

        let (bg, toggle_bg) = match self.checked {
            true => (cx.theme().primary, cx.theme().background),
            false => (cx.theme().switch, cx.theme().background),
        };

        let (bg, toggle_bg) = match self.disabled {
            true => {
                if self.checked {
                    (cx.theme().muted.darken(0.05), toggle_bg.opacity(0.8))
                } else {
                    (cx.theme().muted, toggle_bg.opacity(0.8))
                }
            }
            false => (bg, toggle_bg),
        };

        let (bg_width, bg_height) = match self.size {
            Size::XSmall | Size::Small => (px(28.), px(16.)),
            _ => (px(36.), px(20.)),
        };
        let bar_width = match self.size {
            Size::XSmall | Size::Small => px(12.),
            _ => px(16.),
        };
        let inset = px(2.);
        let radius = if cx.theme().radius >= px(4.) {
            bg_height
        } else {
            cx.theme().radius
        };

        let mut root = div();
        *root.style() = style.clone();

        let mut element = root
            .child(
                h_flex()
                    .id(self.id.clone())
                    .gap_2()
                    .items_start()
                    .when(self.label_side.is_left(), |this| this.flex_row_reverse())
                    .child(
                        // Switch Bar
                        div()
                            .id(self.id.clone())
                            .w(bg_width)
                            .h(bg_height)
                            .rounded(radius)
                            .flex()
                            .items_center()
                            .border(inset)
                            .border_color(cx.theme().transparent)
                            .bg(bg)
                            .when_some(self.tooltip.clone(), |this, tooltip| {
                                this.tooltip(move |window, cx| {
                                    Tooltip::new(tooltip.clone()).build(window, cx)
                                })
                            })
                            .child(
                                // Switch Toggle
                                div()
                                    .rounded(radius)
                                    .bg(toggle_bg)
                                    .shadow_md()
                                    .size(bar_width)
                                    .map(|this| {
                                        let prev_checked = state.prev_checked.clone();
                                        if !self.disabled
                                            && prev_checked
                                                .borrow()
                                                .map_or(false, |prev| prev != checked)
                                        {
                                            let dur = Duration::from_secs_f64(0.15);
                                            cx.spawn(async move |cx| {
                                                cx.background_executor().timer(dur).await;

                                                *prev_checked.borrow_mut() = Some(checked);
                                            })
                                            .detach();
                                            this.with_animation(
                                                ElementId::NamedInteger(
                                                    "move".into(),
                                                    checked as u64,
                                                ),
                                                Animation::new(dur),
                                                move |this, delta| {
                                                    let max_x =
                                                        bg_width - bar_width - inset * 2;
                                                    let x = if checked {
                                                        max_x * delta
                                                    } else {
                                                        max_x - max_x * delta
                                                    };
                                                    this.left(x)
                                                },
                                            )
                                            .into_any_element()
                                        } else {
                                            let max_x = bg_width - bar_width - inset * 2;
                                            let x = if checked { max_x } else { px(0.) };
                                            this.left(x).into_any_element()
                                        }
                                    }),
                            ),
                    )
                    .when_some(self.label.take(), |this, label| {
                        this.child(div().line_height(bg_height).child(label).map(|this| {
                            match self.size {
                                Size::XSmall | Size::Small => this.text_sm(),
                                _ => this.text_base(),
                            }
                        }))
                    })
                    .when_some(
                        on_click
                            .as_ref()
                            .map(|c| c.clone())
                            .filter(|_| !self.disabled),
                        |this, on_click| {
                            let prev_checked = state.prev_checked.clone();
                            this.on_mouse_down(gpui::MouseButton::Left, move |_, window, cx| {
                                cx.stop_propagation();
                                *prev_checked.borrow_mut() = Some(checked);
                                on_click(&!checked, window, cx);
                            })
                        },
                    ),
            )
            .into_any_element();

        ((element.request_layout(window, cx), element), state)
    })
}
```

##### title_bar.rs

```rust
fn request_layout(
        &mut self,
        _: Option<&gpui::GlobalElementId>,
        _: Option<&gpui::InspectorElementId>,
        window: &mut Window,
        cx: &mut App,
    ) -> (gpui::LayoutId, Self::RequestLayoutState) {
        let mut style = Style::default();
        style.flex_grow = 1.0;
        style.flex_shrink = 1.0;
        style.size.width = relative(1.).into();
        style.size.height = relative(1.).into();

        let id = window.request_layout(style, [], cx);
        (id, ())
    }
```


### DockItem & Dock

- *DockArea* is basically a DockItem and a set of Docks
- *DockItem* is the center that gets initialized in `story_workspace.rs` by `init_default_layout`

A *Dock* is basically a

- *TabPanel* inside a *ViewContext new_view*
- *DockArea*
- *DockPlacement*

### TabPanel, PanelView & StoryContainer

- *TabPanel* is a Vec of PanelViews
- *PanelView* is a *StoryContainer* inside a *WindowContext new_view*

The difference between *DockItem* and *Dock* is that *Dock* only has a *TabPanel*
whereas *DockItem* can have both namely *Split* and *Tabs*.


#### DockAreaState is at the top of the Serializable state tree

- DockItemState is for the center.
- DockState is for {left_dock, right_dock, and bottom_dock}

#### dock/mod.rs

- set_left_dock
- set_bottom_dock
- set_right_dock

```rust

/// The main area of the dock.
pub struct DockArea {
    id: SharedString,
    pub(crate) bounds: Bounds<Pixels>,

    /// The center view of the dockarea.
    items: DockItem,

    /// The left dock of the dockarea.
    left_dock: Option<View<Dock>>,
    /// The bottom dock of the dockarea.
    bottom_dock: Option<View<Dock>>,
    /// The right dock of the dockarea.
    right_dock: Option<View<Dock>>,
    /// The top zoom view of the dockarea, if any.
    zoom_view: Option<AnyView>,
}

/// DockItem is a tree structure that represents the layout of the dock.
#[derive(Clone)]
pub enum DockItem {
    Split {
        axis: gpui::Axis,
        items: Vec<DockItem>,
        sizes: Vec<Option<Pixels>>,
        view: View<StackPanel>,
    },
    Tabs {
        items: Vec<Arc<dyn PanelView>>,
        active_ix: usize,
        view: View<TabPanel>,
    },
}

/// The Dock is a fixed container that places at left, bottom, right of the Windows.
///
/// This is unlike Panel, it can't be move or add any other panel.
pub struct Dock {
    pub(super) placement: DockPlacement,
    dock_area: WeakView<DockArea>,
    pub(crate) panel: View<TabPanel>,
    /// The size is means the width or height of the Dock, if the placement is left or right, the size is width, otherwise the size is height.
    pub(super) size: Pixels,
    pub(super) open: bool,
    is_resizing: bool,
}

#[derive(Debug, Clone, PartialEq, Serialize, Deserialize)]
pub enum DockItemInfo {
    #[serde(rename = "stack")]
    Stack {
        sizes: Vec<Pixels>,
        /// The axis of the stack, 0 is horizontal, 1 is vertical
        axis: usize,
    },
    #[serde(rename = "tabs")]
    Tabs { active_index: usize },
    #[serde(rename = "panel")]
    Panel(serde_json::Value),
}
```

### State

```rust
/// Used to serialize and deserialize the DockArea
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct DockAreaState {
    pub center: DockItemState,
    pub left_dock: Option<DockState>,
    pub right_dock: Option<DockState>,
    pub bottom_dock: Option<DockState>,
}

/// Used to serialize and deserialize the DockItem
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct DockItemState {
    pub panel_name: String,
    pub children: Vec<DockItemState>,
    pub info: DockItemInfo,
}

/// Used to serialize and deserialize the Dock
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct DockState {
    panel: DockItemState,
    placement: DockPlacement,
    size: Pixels,
    open: bool,
}

#[derive(Debug, Clone, PartialEq, Serialize, Deserialize)]
pub enum DockItemInfo {
    #[serde(rename = "stack")]
    Stack {
        sizes: Vec<Pixels>,
        /// The axis of the stack, 0 is horizontal, 1 is vertical
        axis: usize,
    },
    #[serde(rename = "tabs")]
    Tabs { active_index: usize },
    #[serde(rename = "panel")]
    Panel(serde_json::Value),
}

#[derive(Debug, Clone, Copy, Serialize, Deserialize, PartialEq, Eq)]
pub enum DockPlacement {
    #[serde(rename = "left")]
    Left,
    #[serde(rename = "bottom")]
    Bottom,
    #[serde(rename = "right")]
    Right,
}
```

### Panel

```rust
pub struct StackPanel {
    pub(super) parent: Option<View<StackPanel>>,
    pub(super) axis: Axis,
    focus_handle: FocusHandle,
    pub(crate) panels: SmallVec<[Arc<dyn PanelView>; 2]>,
    panel_group: View<ResizablePanelGroup>,
}

pub struct TabPanel {
    focus_handle: FocusHandle,
    dock_area: WeakView<DockArea>,
    /// The stock_panel can be None, if is None, that means the panels can't be split or move
    stack_panel: Option<View<StackPanel>>,
    pub(crate) panels: Vec<Arc<dyn PanelView>>,
    pub(crate) active_ix: usize,
    tab_bar_scroll_handle: ScrollHandle,
    is_zoomed: bool,
    /// If this is true, the Panel closeable will follow the active panel's closeable,
    /// otherwise this TabPanel will not able to close
    pub(crate) closeable: bool,
    /// If this is true, the Panel zoomable will follow the active panel's zoomable,
    /// otherwise this TabPanel will not able to zoom
    pub(crate) zoomable: bool,

    /// When drag move, will get the placement of the panel to be split
    will_split_placement: Option<Placement>,
}
```

### Notes

Dock is only referenced in

- crates/app/src/story_workspace.rs
- crates/story/src/lib.rs
- crates/ui/src/dock

Key point here being that *Dock* is not called any where in the ui crate
except *ui/src/dock* itself which is kind of cool :)

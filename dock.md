
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

/// Used to serialize and deserialize the DockItem
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq)]
pub struct DockItemState {
    pub panel_name: String,
    pub children: Vec<DockItemState>,
    pub info: DockItemInfo,
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

pub trait Panel: EventEmitter<PanelEvent> + FocusableView {
    /// The name of the panel used to serialize, deserialize and identify the panel.
    ///
    /// This is used to identify the panel when deserializing the panel.
    /// Once you have defined a panel name, this must not be changed.
    fn panel_name(&self) -> &'static str;

    /// The title of the panel
    fn title(&self, _cx: &WindowContext) -> AnyElement {
        SharedString::from(t!("Dock.Unnamed")).into_any_element()
    }

    /// The theme of the panel title, default is `None`.
    fn title_style(&self, _cx: &WindowContext) -> Option<TitleStyle> {
        None
    }

    /// Whether the panel can be closed, default is `true`.
    fn closeable(&self, _cx: &WindowContext) -> bool {
        true
    }

    /// Return true if the panel is zoomable, default is `false`.
    fn zoomable(&self, _cx: &WindowContext) -> bool {
        true
    }

    /// Return true if the panel is collapsable, default is `false`.
    fn collapsible(&self, _cx: &WindowContext) -> bool {
        false
    }

    /// The addition popup menu of the panel, default is `None`.
    fn popup_menu(&self, this: PopupMenu, _cx: &WindowContext) -> PopupMenu {
        this
    }

    /// Dump the panel, used to serialize the panel.
    fn dump(&self, _cx: &AppContext) -> DockItemState {
        DockItemState::new(self.panel_name())
    }
}
```

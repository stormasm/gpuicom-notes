
- *TabPanel* is a vector of PanelViews

*reset_default_layout* is ONLY called once the first time the developer
fires up the application or when the software version gets updated

### View Hierarchy without layout.json

```rust
hola 120 StoryWorkspace::new
hola 125 DockArea::new
hola 126 StackPanel::new

load layout error: No such file or directory (os error 2)

hola 130 Panel
hola 140 StoryContainer FocusHandle(FocusId(3v1))
hola 148 new_tabs: TabPanel::new
hola 126 StackPanel::new

hola 130 Panel
hola 140 StoryContainer FocusHandle(FocusId(7v1))

hola 130 Panel
hola 140 StoryContainer FocusHandle(FocusId(9v1))

hola 130 Panel
hola 140 StoryContainer FocusHandle(FocusId(11v1))

hola 130 Panel
hola 140 StoryContainer FocusHandle(FocusId(13v1))

hola 149 Dock::new TabPanel::new
hola 150 Dock::new Left
hola 149 Dock::new TabPanel::new
hola 150 Dock::new Bottom
hola 149 Dock::new TabPanel::new
hola 150 Dock::new Right

Save layout...
Save layout...
```

### layout.json is now there

```rust
hola 120 StoryWorkspace::new
hola 125 DockArea::new
hola 126 StackPanel::new

hola 140 StoryContainer FocusHandle(FocusId(3v1))
hola 148 new_tabs: TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(6v1))
hola 148 new_tabs: TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(9v1))
hola 148 new_tabs: TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(12v1))
hola 148 new_tabs: TabPanel::new
hola 148 new_tabs: TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(16v1))
hola 148 new_tabs: TabPanel::new

hola 126 StackPanel::new
load layout success
Save layout...
```

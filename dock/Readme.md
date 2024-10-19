
### Summary

- *DockItem* should be called *DockCenter*
- *DockItemState* should be called *DockCenterState*
- *DockItem* is the center that gets initialized in `story_workspace.rs` by `init_default_layout`

#### DockAreaState is at the top of the Serializable state tree

- DockItemState is for the center :) thus improperly named to DockCenterState
- DockState is for {left_dock, right_dock, and bottom_dock}

### View Hierarchy

*reset_default_layout* is ONLY called once the first time the developer
fires up the application or when the software version gets updated

```rust

hola 120 StoryWorkspace::new
hola 125 DockArea::new
hola 126 StackPanel::new

hola 140 StoryContainer FocusHandle(FocusId(3v1))
hola 141 TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(6v1))
hola 141 TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(9v1))
hola 141 TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(12v1))
hola 141 TabPanel::new
hola 141 TabPanel::new

hola 140 StoryContainer FocusHandle(FocusId(16v1))
hola 141 TabPanel::new

hola 126 StackPanel::new

Save layout...
Save layout...
```

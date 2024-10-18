
### Summary

- *DockItem* should be called *DockCenter*
- *DockItemState* should be called *DockCenterState*
- *DockItem* is the center that gets initialized in `story_workspace.rs` by `init_default_layout`

#### DockAreaState is at the top of the Serializable state tree

- DockItemState is for the center :) thus improperly named to DockCenterState
- DockState is for {left_dock, right_dock, and bottom_dock}

# Storyrift

AI interactive fiction engine where Claude acts as narrator/DM.

**Core principle:** Claude never changes world facts directly. A script owns state.
Claude proposes what happens, the engine validates and applies or rejects, then
Claude narrates what was accepted.

Per turn: player input -> Claude proposes changes -> engine judges -> Claude narrates the accepted result.

Status: prototype, first slice in design.

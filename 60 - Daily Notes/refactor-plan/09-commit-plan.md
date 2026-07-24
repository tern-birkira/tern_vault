# Commit Plan

All commits ≤25 words. Each must compile independently.

## Commit sequence

| # | Message | What |
|---|---------|------|
| 0 | remove ControlState type instead use ControlType across track-label-editor.
| 1 | add TrackLabelRuntimeDataController with visibility, control type, holding | New files, standalone controller using ControlType |
| 2 | add TrackLabelDataController, extract layout and field logic | New files, pulls from EditorController |
| 3 | migrate FieldInterface and FieldListModel from ControlState to ControlType | Remove toControlType converter, drop FlightTypes.h |
| 4 | add TrackLabelIOController, extract load save logic | New files, owns configModel |
| 5 | rewire FileBrowserController to use IOController | Modify existing files |
| 6 | rewire Instantiator to create new controllers and wire signals | Modify Instantiator |
| 7 | delete TrackLabelEditorController | Remove files |
| 8 | wire QML display states header to runtime data controller | QML changes |
| 9 | wire QML control states header to runtime data controller | QML changes |
| 10 | update CMakeLists and verify build | Build fix if needed |

## Rules

- Each commit compiles on its own (may have temporary dead code during transition)
- Commits 1-2 are additive (new files, old code still exists and compiles)
- Commit 3 updates the field layer to use ControlType (old controller temporarily adapts)
- Commits 4-6 switch the wiring (old controller becomes dead code)
- Commit 7 removes the dead code
- Commits 8-9 fix the QML gap
- Commit 10 cleanup pass

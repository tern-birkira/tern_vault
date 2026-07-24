# Controller Refactor — Overview

## Goal

Split monolithic `TrackLabelEditorController` into focused, decoupled controllers
following the flat-peer pattern established by the aeronautical-editor and map-layer-editor.

## Final Architecture

```
TrackLabelEditorInstantiator (owns everything, coordinator)
├── shared_ptr<TrackLabelRuntimeDataController>   → Q_PROPERTY
├── shared_ptr<TrackLabelDataController>          → Q_PROPERTY
├── shared_ptr<TrackLabelIOController>            → Q_PROPERTY
├── shared_ptr<TrackLabelFileBrowserController>   → Q_PROPERTY
│
├── signal wiring:
│   connect(runtimeData::changed → dataController::reevaluateActiveLayoutVisibility)
│   connect(dataController::activeTypeChanged → dataController::reevaluateActiveLayoutVisibility)
└── QML singleton: "ASD.Editor.TrackLabelEditor" / "TrackLabelEditorInstantiator"
```

## Data Flow

```
.properties file ──┐
                   ▼
XML config ──► IOController ──► DataController (model tree)
                   │
                   ▼
runtime import ──► IOController ──► RuntimeDataController
                                          │
                                          ▼ (signal: changed)
                                    DataController.reevaluateActiveLayoutVisibility()
                                          │
                                          ▼
                                    field.setIsVisible()

FileBrowserController (UI) ──triggers──► IOController
```

## Ownership Rule

- **Instantiator owns all** — it creates and holds every `shared_ptr`.
- **Controllers borrow** — they receive copies of `shared_ptr` via constructor injection.
  No raw-pointer, no parent-cast, no singleton access between controllers.

## Deletion

- `TrackLabelEditorController` — deleted entirely after extraction.

## Files affected

- NEW: `controllers/TrackLabelRuntimeDataController.h/.cpp`
- NEW: `controllers/TrackLabelDataController.h/.cpp`
- NEW: `controllers/TrackLabelIOController.h/.cpp`
- MODIFIED: `controllers/TrackLabelFileBrowserController.h/.cpp`
- MODIFIED: `TrackLabelEditorInstantiator.h/.cpp`
- MODIFIED: `CMakeLists.txt`
- MODIFIED: QML — `DisplayStatesHeader.qml`, `ControlStatesHeader.qml`, `Canvas.qml`
- DELETED: `controllers/TrackLabelEditorController.h/.cpp`

## Commit Order

1. Create TrackLabelRuntimeDataController (with ControlType, NOT ControlState)
2. Create TrackLabelDataController (extract from EditorController, use ControlType)
3. Migrate FieldInterface + FieldListModel from ControlState → ControlType
4. Create TrackLabelIOController (extract load/save)
5. Rewire TrackLabelFileBrowserController to use IOController
6. Update TrackLabelEditorInstantiator (new construction + wiring)
7. Delete TrackLabelEditorController
8. Wire QML headers to runtimeDataController
9. Update CMakeLists.txt
10. Verify build

## Additional Proposals (future work, documented)

- `10-runtime-field-data-proposal.md` — how to override dummyData for runtime simulation
- `11-properties-placeholder-proposal.md` — .properties placeholder system design
- `12-controlstate-to-controltype-migration.md` — ControlState removal strategy

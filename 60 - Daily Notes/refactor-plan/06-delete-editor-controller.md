# Delete TrackLabelEditorController

## What

Remove `controllers/TrackLabelEditorController.h` and `controllers/TrackLabelEditorController.cpp`.

## Where did everything go

| Method / Member | New Home |
|----------------|----------|
| `activeType` property + setter | `TrackLabelDataController` |
| `activeVisibility` property + setter | `TrackLabelRuntimeDataController` |
| `activeControlState` property + setter | `TrackLabelRuntimeDataController` |
| `currentLayout` property | `TrackLabelDataController` |
| `extendedLayout` property | `TrackLabelDataController` |
| `column0ActionsLayout` property | `TrackLabelDataController` |
| `selectedField` property | `TrackLabelDataController` |
| `selectField()` | `TrackLabelDataController` |
| `clearSelection()` | `TrackLabelDataController` |
| `addFieldToCurrentLayout()` | `TrackLabelDataController` |
| `syncViewLayoutMappings()` | `TrackLabelDataController` |
| `reevaluateActiveLayoutVisibility()` | `TrackLabelDataController` (public slot) |
| `loadConfiguration()` | `TrackLabelIOController` |
| `saveConfiguration()` | `TrackLabelIOController` |
| `m_layouts` hash | `TrackLabelDataController` |
| `m_activeVisibility` member | `TrackLabelRuntimeDataController` |
| `m_activeControlState` member | `TrackLabelRuntimeDataController` |

## Verification

After deletion, grep the codebase for any remaining references:
```bash
grep -r "TrackLabelEditorController" source/
```
Should return zero hits.

## Implementation Steps

1. Ensure all extraction is complete (steps 01-05 done and building)
2. Delete .h and .cpp
3. Remove from CMakeLists.txt SOURCES
4. Full build verification

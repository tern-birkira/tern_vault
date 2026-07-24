# CMakeLists.txt + Build Verification

## CMakeLists.txt changes

```cmake
SOURCES
    # ... existing models, serialization ...

    controllers/TrackLabelRuntimeDataController.h    # NEW
    controllers/TrackLabelRuntimeDataController.cpp  # NEW
    controllers/TrackLabelDataController.h           # NEW
    controllers/TrackLabelDataController.cpp         # NEW
    controllers/TrackLabelIOController.h             # NEW
    controllers/TrackLabelIOController.cpp           # NEW
    controllers/TrackLabelFileBrowserController.h    # MODIFIED
    controllers/TrackLabelFileBrowserController.cpp  # MODIFIED

    # REMOVED:
    # controllers/TrackLabelEditorController.h
    # controllers/TrackLabelEditorController.cpp
```

## Build Targets

```bash
# Minimal rebuild (library only)
ninja asd.editor.tracklabeleditor

# Full app binary (includes QML + all libs)
ninja track-label-editor
```

## Verification Checklist

- [ ] `ninja asd.editor.tracklabeleditor` compiles clean (no errors)
- [ ] `ninja track-label-editor` links clean (no undefined symbols)
- [ ] `grep -r "TrackLabelEditorController" source/` returns 0 hits
- [ ] No new compiler warnings introduced
- [ ] QML files load without runtime errors (manual check)

## Implementation Steps

1. Add new source files to CMakeLists SOURCES
2. Remove old controller files from SOURCES
3. Run ninja asd.editor.tracklabeleditor — fix compile errors
4. Run ninja track-label-editor — fix link errors
5. Final grep for stale references

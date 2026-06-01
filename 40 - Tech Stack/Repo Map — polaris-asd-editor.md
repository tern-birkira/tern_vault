 t# Repo Map — `polaris-asd-editor`

*Precise map of everything in the repo. Updated: 2026-05*
*Related: [[Summer Project Overview]] | [[Qt Framework]] | [[CMake Build System]]*

---

## What This Repo Is

A Qt6/C++/QML desktop application that provides GUI editors for Polaris ASD configuration files. Currently ships **two editors** in one combined app:

| Editor                       | Status                                        | What It Edits                                                                                                |
| ---------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Aeronautical Data Editor** | Deployed (live in Hungary, coming to Iceland) | Aeronautical data: waypoints, routes, airways, airspaces — loaded from DAFIF/AIXM format, stored as protobuf |
| **Map Layer Editor**         | Prototype (not yet deployed)                  | Map layer visual config: which layers exist, their display style, presets                                    |

The third app target (`polaris-asd-editor`) is the **combined shell** that hosts both inside one window with a shared header.

---

## Top-Level Layout

```
polaris-asd-editor/
├── CMakeLists.txt             ← root build; finds Qt6, tern, links everything
├── CMakePresets.json          ← Debug / Release / Debug+local-tern-framework presets
├── CMakeModules/
│   ├── FindLocalTernLibraries.cmake   ← lets you use local tern-framework/tern-map builds
│   └── CreateBuildSymlink.cmake
├── schemas/
│   └── polaris-editor-path-config.xsd ← XSD for the editor's own path config
├── platform/
│   └── rocky8/Dockerfile      ← Rocky Linux 8 CI build environment
└── source/
    ├── libs/                  ← shared libs used by ALL editors
    ├── apps/                  ← the three app targets + their private libs
    └── tests/                 ← unit tests
```

---

## source/libs — Shared Across All Editors

| Library                  | CMake target  | What it does                                                                                                                                                                        |
| ------------------------ | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `asd.editor.config`      | shared lib    | `ApplicationConfig` loads the Tern DI application context XML; `PathConfig` reads `import-folder-path` attribute from XML                                                           |
| `asd.editor.filebrowser` | shared lib    | Abstract `FileBrowserControllerBase` — Q_PROPERTY-exposed file selection, import/export/rename/delete. Subclass this for each editor's file browser.                                |
| `asd.editor.header`      | QML module    | `Header.qml` top bar (logo, editor tabs, AIXM/Settings/Undo/Redo buttons, clock); `ClockModel` bridges `ITime` → QML; `PolarisBackendClock` implements `ITime` from Polaris backend |
| `asd.editor.map`         | QML module    | `MainMapController` drives the map view; `MapFeatures.qml`                                                                                                                          |
| `asd.editor.styles`      | QML singleton | `Style.qml` — **the design system**: all named colors (`ebonyClay`, `jordyBlue`, etc.), fonts (Lato, Inconsolata). Import in every QML file.                                        |
| `asd.editor.generic`     | QML module    | Generic widgets: `OrionComboBox`, `OrionScrollBar`, `OrionToolTip`, `PopupConfirmation`, `MouseEventCatcher`, `MoveMapAnimation`                                                    |

---

## source/apps/polaris-asd-editor — The Combined App

**Binary:** `polaris-asd-editor`

```
source/apps/polaris-asd-editor/
├── main.cpp          ← entry point, creates tern::Application + Application
├── Application.h/cpp ← initializes both sub-editors, loads QML engine
└── ui/qml/main.qml   ← root window: Header + Loader switching between editors
```

### Startup Sequence

```
main()
  → tern::Application()       # init tern diagnostics
  → Application::initialize()
      → load /usr/share/polaris/config/polaris-asd-editor-application-context.xml  # polaris-ice repo, smc role (not yet created)
      → get MapsConfig from DI container
      → MapLayerEditorInstantiator( mapsConfig )
      → ApplicationConfig()   # loads aeronautical-editor-application-context.xml (polaris-ice, smc role)
      → EnvironmentEditorInstantiator( applicationConfig )
  → Application::run()
      → load main.qml
      → Qt event loop
```

### main.qml Structure

```qml
ApplicationWindow {         // fullscreen
  Header { ... }            // top bar, always visible
  Loader {                  // switches content
    source: EnvironmentEditor.qml  // default
    // OR: MapLayerEditor.qml
  }
}
```

Switching is driven by `Header`'s `environmentEditorSelected` / `mapLayerEditorSelected` signals.

---

## source/apps/map-layer-editor — Map Layer Editor

**Binary:** `map-layer-editor` (standalone) + embedded in `polaris-asd-editor`

### Private Libs

#### `asd.editor.layers` — Core Data Model

| Class | Role |
|-------|------|
| `MapLayerModel` | Qt model for the layer tree (layer list) |
| `MapLayerPreset` / `MapLayerPresetModel` | Named presets — groups of layer visibility settings |
| `MapLayerPresetVariant` | Variant within a preset |
| `MapFactory` | Creates map layer objects |
| `CfgFileWriter` | Writes `.cfg` layer display config files |
| `XmlLayerCollectionWriter` | Writes XML layer collection files |
| `XmlFolderConfigurationWriter` | Writes XML folder-level config |
| `MapLayerDirChildController` | Controls directory-level layer items |
| `undo.commands/` | Full undo/redo: Add, Delete, Icon, EditPresetName, AddPreset, DeletePreset, SinglePresetVariant commands |

#### `asd.editor.layereditor` — Editor Orchestration

| Class | Role |
|-------|------|
| `MapLayerEditorInstantiator` | Registers as QML singleton `MapLayerEditorInstantiator`; creates undo stack; wires all C++ to QML context |
| `MapLayerFileBrowserController` | Extends `FileBrowserControllerBase` for map layer XML files |

#### `asd.editor.inspector` — "Settings" Panel

| Class | Role |
|-------|------|
| `InspectorDataController` | Controls the settings panel; exposes `isOpen` to QML |
| `InspectorDataModel` | Data model for inspector panel |
| `SelectedAreaOrPathModel` | Model for selected geometric area or path |
| `Commands` | Inspector-specific undo commands |

#### `asd.editor.maprenderer` — AIXM Panel

| Class | Role |
|-------|------|
| `MapDataModel` | Data model bridging AIXM data to the map |
| `IconModel` | Icon management for map features |

#### `asd.editor.qt.searchmodel` — Search

| Class | Role |
|-------|------|
| `AbstractFilterModel` | Base for filter proxy models |
| `AIXMSearchFilterProxyModel` | Filters AIXM data in search |

---

## source/apps/aeronautical-editor — Aeronautical Data Editor

**Binary:** `aeronautical-editor` (standalone) + embedded in `polaris-asd-editor`

### Private Libs

#### `asd.editor.dafif` — DAFIF Data Loading

| Class | Role |
|-------|------|
| `DafifAeronauticalDatabaseLoader` | Loads aeronautical DB from DAFIF flat files (ARPT, NAV, WPT, ATS, HOLD, SUAS, TRM, BDRY) |
| `DafifXmlLoader` | XML variant of DAFIF loader |

DAFIF = Defense Aeronautical Flight Information File. Tab-delimited military format containing waypoints, navaids, airways, airports, etc.

#### `asd.editor.environmenteditor` — Editor Core

**Controllers:**

| Class | Role |
|-------|------|
| `EnvironmentEditorInstantiator` | Registers `EnvironmentEditorInstantiator` QML singleton; owns undo stack; loads data |
| `AeronauticalFileBrowserController` | File browser for AIXM/DAFIF import; extends `FileBrowserControllerBase` |
| `InputFieldController` | Controls form input fields |
| `MapController` | Controls the aeronautical map view |
| `MultiDeleteController` | Handles bulk deletion of aeronautical items |
| `NavigationController` | Navigation between data views |
| `WelcomeScreenController` | Controls the initial welcome/load screen |
| `editorCommands` | Editor-level undo commands |

**Models:**

| Class | Role |
|-------|------|
| `TreeModel` / `TreeProxyModel` | Hierarchical tree view of aeronautical data |
| `MapModel` / `MapDataVariant` / `MapFilterProxyModel` | Map display models |
| `InputFieldModel` | Model for form input fields |
| `SearchModel` | Search over aeronautical items |
| `ErrorLogModel` | Error/warning log |
| `MessageListModel` | General message list |
| `RepeatedListModel` | For repeated protobuf fields |
| `SelectItemModel` | Single-item selection model |
| `ReferenceSearchSourceModel` / `...FilterModel` | Reference field search |
| `WaypointProxyModel` | Waypoint-specific filtered model |

**QML Views (`ui/resources/qml/`):**

| Folder | Contents |
|--------|----------|
| `Views/` | Main editor views (likely tree + detail) |
| `InputFields/` | Custom input components incl. `DMSField.qml` (Degrees-Minutes-Seconds) |
| `Map/` | Aeronautical map display |
| `WelcomeScreen/` | Initial data load screen |
| `ErrorLog/` | Error/warning log panel |
| `Components/` | Reusable sub-components |
| `Loading/` | Loading indicators |

**Data format:** Protobuf — all aeronautical data is `tern::aeronautical::protobuf::TopLevelData`. When switching to Map Layer Editor, the aeronautical data is converted via `tern::aeronautical::protobuf::fromProtobuf()`.

---

## External Dependencies (not in this repo)

| Library | What it provides |
|---------|-----------------|
| `tern` | Framework core: `Application`, diagnostics, `ApplicationContext` (DI container), `ClassFactory`, XML init, exceptions |
| `tern.map` | Map rendering engine |
| `tern.map.qtquick` | Qt Quick bindings for tern.map; `MapsConfig`, `MapFeatureModel` |
| `tern.aeronautical.protobuf` | Protobuf definitions for aeronautical data types |
| `tern.protobuf` | Tern protobuf utilities |
| `tern.aeronautical.xerces` | Xerces-based XML parsing for aeronautical data |
| Qt6 | Core, Gui, Xml, Quick, Widgets, ShaderTools, OpenGL, Multimedia |
| Protobuf | Google Protocol Buffers |
| FreeType | Font rendering |
| LibXml2 | XML parsing (used by `asd.editor.dafif`) |

### The Tern Runtime (DI container)

Polaris uses `tern::runtime::ApplicationContext` as a **dependency injection / service locator**. Initialized from an XML file:

```cpp
tern::runtime::ApplicationContext::initialize( "/usr/share/polaris/config/XXX-context.xml" );
auto* config = ApplicationContext::getInstance< MyClass* >( "bean-id" );
```

Classes loaded via DI implement:
- `tern::runtime::IDynamicallyLoadable` — can be loaded by ClassFactory
- `tern::xml::IXmlInitializable` — initialized from XML element
- `tern::xml::schema::XmlSchemaValidatableBase` — validated against an XSD

`PathConfig` is an example of this pattern in the codebase.

**Two XML files are required per app**, both in `polaris-ice/configuration/config/roles/smc/files/polaris/config/`:

| File | Purpose |
|------|---------|
| `*-class-mappings.xml` | Maps `class-mapping-reference` names → C++ type + `.so` library |
| `*-application-context.xml` | Declares bean instances with their config |

**Class-mappings format:**
```xml
<class-mapping-configuration xmlns="http://tern.is/tern/runtime/class-mapping-configuration" ...>
  <class-mapping name="PathConfig"
                 library="libasd.editor.config.so"
                 object-type="asd::editor::config::PathConfig"/>
</class-mapping-configuration>
```

**Application-context format:**
```xml
<instances xmlns="http://tern.is/tern/runtime/application-context"
           xmlns:xi="http://www.w3.org/2003/XInclude"
           application-name="My App" ...>

  <instance id="Editor-paths" class-mapping-reference="PathConfig">
      <paths:path-config xmlns:paths="http://tern.is/polaris-asd-editor/editorpaths"
                         import-folder-path="${aeronautical.editor.import.folder.path}"/>
  </instance>

  <!-- XInclude for composing multiple context files -->
  <xi:include href="/usr/share/polaris/config/other-config.xml"
              xpointer="xpointer(//instances/*)"/>

  <!-- Threaded runnables -->
  <runnable-instance id="ThreadMaintainer" class-mapping-reference="ThreadMaintainer">
      ...
  </runnable-instance>
</instances>
```

- `${property.name}` values come from a companion `.properties` file at runtime
- `id` is the key passed to `ApplicationContext::getInstance<T>("id")`
- See [[Tern Runtime Questions]] for full format reference

---

## Build System

```bash
# Configure (Debug, default)
cmake --preset Debug

# Build
cmake --build build-debug --parallel

# Test
ctest --test-dir build-debug --output-on-failure
```

**Presets:**
| Preset | Dir | Notes |
|--------|-----|-------|
| `Debug` | `build-debug/` | Standard debug build |
| `Release` | `build-release/` | Optimized |
| `Debug - with local tern-framework` | `build-debug-local-tern-framework/` | Uses local tern-framework checkout |
| `Debug - with local tern-map and tern-framework` | `build-debug-local-tern-map/` | Both local |

C++ standard: **C++17** (via `settings-gnu17` CMake include from `tern-cmake-modules`)

---

## Tests

```
source/tests/libs/
├── asd.editor.dafif/
│   ├── DafifAeronauticalDatabaseLoaderTest.h
│   └── DafifXmlLoaderTest.h
├── asd.editor.environmenteditor/controllers/
│   └── MultiDeleteControllerTest.h
├── asd.editor.layers/
│   └── MapLayerPresetModelTest.h
└── asd.editor.map/
    └── MainMapControllerTest.h
```

Run via `ctest`. Tests use the tern-framework test harness (Google Test or similar — confirm on first run).

---

## Git Workflow Observed

Branch naming: `tasks/DEV-XXXX-short-description`
Merge strategy: fast-forward feature branch commit + merge commit into master.
Example: `tasks/DEV-8223-arispaces-in-selected-layer-drawn-below-aixm-data`
Tags mark releases: `1.5.0`, `1.4.2`, `1.4.1`

---

## Where Your New Work Lives

Your editors (Track Label Editor, Flight List Editor) need to be added as new sub-editors. Based on the existing patterns:

### Option A — New standalone app + embedded in combined app
Same as Map Layer Editor and Aeronautical Editor. Add:
- `source/apps/tracklabel-editor/` — new app binary
- `source/apps/tracklabel-editor/libs/asd.editor.tracklabel/` — private libs
- New tab in `polaris-asd-editor`'s `main.qml`

### Option B — Add to existing combined app as new modules
Add new libs under `source/libs/` if the XML import/export logic is truly shared.

### Key patterns to reuse

| Need | Reuse |
|------|-------|
| File open/save dialog | Subclass `FileBrowserControllerBase` |
| Undo/redo | Copy pattern from `asd.editor.layers/undo.commands/` using `QUndoStack` |
| QML singleton registration | Copy `MapLayerEditorInstantiator` / `EnvironmentEditorInstantiator` pattern |
| Color/font | Import `ASD.Editor.Styles` and use `Style.colorScheme.*` |
| Generic widgets | Import `ASD.Editor.Generic` for `OrionComboBox`, `PopupConfirmation`, etc. |
| XML read/write | Study `XmlLayerCollectionWriter` for write; `QDomDocument` for read-modify-write |
| Config loading | Study `PathConfig` as example of Tern DI-loadable class |

---

## Open Questions (Repo-Specific)

- [x] ~~Where is `polaris-asd-editor-application-context.xml`?~~ → In `polaris-ice` at `configuration/config/roles/smc/files/polaris/config/` — but **doesn't exist yet** (combined app not yet deployed). `aeronautical-editor-application-context.xml` is there. See [[Tern Runtime Questions]].
- [x] ~~What is the tern-framework ClassFactory XML format?~~ → See [[Tern Runtime Questions]] Q2 for full format. Two files needed: `*-class-mappings.xml` + `*-application-context.xml`, both in `polaris-ice`.
- [ ] Does the test framework use Google Test or Catch2?
- [x] ~~What `tern.aeronautical.protobuf` message types are used — where are the `.proto` files?~~ → In `tern-framework/tern.aeronautical.protobuf/`. Top-level: `TopLevelData` → `AeronauticalData` (waypoints, routes, airspaces, runways, STARs, SIDs, holding patterns, approaches). See [[Tern Runtime Questions]] Q3.

---

*Related: [[Summer Project Overview]] | [[ASD Config Overview]] | [[Qt Framework]] | [[CMake Build System]] | [[C++ Guidelines]] | [[QML Guidelines]]*

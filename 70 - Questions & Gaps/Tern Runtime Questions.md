# Open Questions — Tern Runtime & Application Context

*Questions that need answers from team members or other repos*

---

## ~~Q1: Where do application context XML files live?~~ ✅ ANSWERED

**Question:**
~~Where are these two files located — which repo, which package?~~
- ~~`/usr/share/polaris/config/polaris-asd-editor-application-context.xml`~~
- ~~`/usr/share/polaris/config/aeronautical-editor-application-context.xml`~~

**Answer:**
Both live in the **`polaris-ice`** repo (the configuration/deployment repo), under `configuration/config/roles/smc/files/polaris/config/`.

| Runtime path | Source in polaris-ice | Status |
|---|---|---|
| `/usr/share/polaris/config/aeronautical-editor-application-context.xml` | `configuration/config/roles/smc/files/polaris/config/aeronautical-editor-application-context.xml` | ✅ Exists |
| `/usr/share/polaris/config/polaris-asd-editor-application-context.xml` | `configuration/config/roles/smc/files/polaris/config/polaris-asd-editor-application-context.xml` | ❌ Does not exist yet — combined app not yet fully deployed |

There is also a companion **class-mappings file** that must exist alongside each context:
- `aeronautical-editor-class-mappings.xml` — maps `class-mapping-reference` names to actual C++ types + `.so` libraries

```xml
<!-- aeronautical-editor-class-mappings.xml -->
<class-mapping name="MapsConfig" library="libtern.map.qtquick.so"
               object-type="tern::map::qtquick::config::MapsConfig"/>
<class-mapping name="PathConfig" library="libasd.editor.config.so"
               object-type="asd::editor::config::PathConfig"/>
```

To add a new bean (e.g., `TrackLabelPathConfig`):
1. Add a `<class-mapping>` entry to the class-mappings XML in `polaris-ice`
2. Add an `<instance>` entry to the application-context XML in `polaris-ice`
3. Implement the C++ class (see `PathConfig` as example)

**Why it matters:**
Both are loaded at startup via `tern::runtime::ApplicationContext::initialize(path)`.
They act as a **dependency injection container** — they declare which C++ classes get instantiated and with what config (like `PathConfig`, `MapsConfig`).

**Context files:**
- [[Repo Map — polaris-asd-editor]] — full map of polaris-asd-editor, section "The Tern Runtime (DI container)" explains the pattern
- [[CMake Build System]] — build setup
- `source/libs/asd.editor.config/PathConfig.h` — example of a class loaded via this DI system

---

## ~~Q2: What does the application context XML format look like?~~ ✅ ANSWERED

**Question:**
~~What is the Tern runtime XML format for declaring beans / instances?~~

**Answer:**
Full format confirmed from live files in `polaris-ice` and `tern-framework`.

**Root element:**
```xml
<instances
  xmlns="http://tern.is/tern/runtime/application-context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xi="http://www.w3.org/2003/XInclude"
  application-name="My App"
  xsi:schemaLocation="http://tern.is/tern/runtime/application-context
                      /usr/share/tern-framework/schemas/tern/runtime/application-context.xsd">
```

**Declaring a bean:**
```xml
<instance id="Editor-paths" class-mapping-reference="PathConfig">
    <paths:path-config xmlns:paths="http://tern.is/polaris-asd-editor/editorpaths"
                       import-folder-path="${aeronautical.editor.import.folder.path}"/>
</instance>
```

- `id` — name used in `ApplicationContext::getInstance<T>("id")`
- `class-mapping-reference` — name looked up in the class-mappings XML to find C++ type + `.so`
- Child XML — type-specific config, uses a custom namespace per bean type
- `${property.name}` — substituted from a `.properties` file at runtime

**Cross-referencing instances:**
```xml
<instance id="foo" class-mapping-reference="SomeClass">
    <cfg:configuration xmlns:cfg="...">
        <cfg:dependency reference-name="Editor-paths" reference-type="instance"/>
    </cfg:configuration>
</instance>
```

**Threads (runnable beans):**
```xml
<runnable-instance id="ThreadMaintainer" class-mapping-reference="ThreadMaintainer">
    <tm:configuration xmlns:tm="http://tern.is/tern/threading/thread-maintainer" ...>
        <tm:runnable id="..." reference-name="..." eternal="true">
            ...
        </tm:runnable>
    </tm:configuration>
</runnable-instance>
```

**Composition via XInclude:**
```xml
<xi:include href="/usr/share/polaris/config/other-config.xml"
            xpointer="xpointer(//instances/*)"/>
```

**Real example** (`aeronautical-editor-application-context.xml` in polaris-ice):
```xml
<instances xmlns:xsi="..." xmlns:xi="..."
           xmlns="http://tern.is/tern/runtime/application-context"
           application-name="Polaris ASD Editor"
           xsi:schemaLocation="..."
           xmlns:map="http://tern.is/tern-map/polarismaps"
           xmlns:paths="http://tern.is/polaris-asd-editor/editorpaths">

    <instance id="polaris-asd-maps" class-mapping-reference="MapsConfig">
        <map:configuration>
            <map:main-map map-center-latitude="${aeronautical.editor.map.main.center.lat}" .../>
            <map:map-layers map-layer-path="${aeronautical.editor.map.layer.path}" .../>
        </map:configuration>
    </instance>

    <instance id="Editor-paths" class-mapping-reference="PathConfig">
        <paths:path-config import-folder-path="${aeronautical.editor.import.folder.path}"/>
    </instance>
</instances>
```

---

## ~~Q3: Where are the `.proto` files for `tern.aeronautical.protobuf`?~~ ✅ ANSWERED

**Question:**
~~Where are the protobuf schema files defining `tern::aeronautical::protobuf::TopLevelData` and related messages?~~

**Answer:**
All proto files are in the **`tern-framework`** repo at `tern-framework/tern.aeronautical.protobuf/`:

| File | Key messages |
|---|---|
| `aeronautical-editor.proto` | `TopLevelData`, `AeronauticalData`, `Waypoints` — the top-level container |
| `airspace.proto` | `Airspace` |
| `approach.proto` | `Approach` |
| `holdingPattern.proto` | `HoldingPattern` |
| `leg.proto` | `Leg` |
| `radioCommunicationChannel.proto` | `RadioCommunicationChannel` |
| `route.proto` | `Route` |
| `runway.proto` | `Runway` |
| `standardProcedures.proto` | `StandardInstrumentArrival` (STAR), `StandardInstrumentDeparture` (SID) |

`aeronautical-editor.proto` imports `aeronautical.proto` (base types: `AirportHeliport`, `Navaid`, `DesignatedPoint`) — that file is from the `tern.protobuf` external package, not in local repos.

**`TopLevelData` structure:**
```
TopLevelData
└── AeronauticalData
    ├── Waypoints (airports, navaids, designated points)
    ├── Route[]
    ├── Airspace[]
    ├── Runway[]
    ├── STAR[]
    ├── SID[]
    ├── HoldingPattern[]
    └── Approach[]
```

The library's purpose (from its README): allows reading aeronautical data from JSON files representing protobuf messages. The proto files are **internal to this library only** — if transport between components is ever needed, they'd move to `tern-proto`.

---

*All questions answered via code inspection. [[Repo Map — polaris-asd-editor]] updated.*

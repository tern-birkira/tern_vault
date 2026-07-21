# Editor Architecture — Polaris ASD

This document analyses the internal architecture of the existing **Flight Record (SFR) Editor**
(`source/libs/asd.flight.editor`) as a reference model for future editors, specifically the
planned **Track Label Editor** and **Flight List Editor**.

The class names used in diagrams and descriptions are **generic** — they describe the role
each component plays, not the exact class name in the SFR editor codebase.

---

## 1. Layered Architecture Overview

An editor in Polaris ASD is composed of six distinct layers, assembled by a single
**Instantiator** that wires them together and exposes them to the QML engine.

```mermaid
graph LR
    A["① Configuration\nXML → Layout Structs"]
    B["② Field Model\nPer-field data &\nvalidation"]
    C["③ Editor Record\nOne open tab:\naggregates all fields"]
    D["④ Qt Models\nExpose data to\nQML via roles"]
    E["⑤ Command Dispatcher\nDiff fields →\nbackend commands"]
    F["⑥ QML UI\nRender & bind\nto Qt models"]

    A -->|"layout\nstructs"| D
    B -->|"owned\nfields"| C
    C -->|"rows in\nlist model"| D
    D -->|"read by"| F
    F -->|"user action\ntriggers"| D
    D -->|"apply/create\ncalled on"| E
    E -->|"sends commands\nto"| G["Backend\n(PBL)"]
    G -->|"update events\nback to"| D
```

| #   | Layer                  | One-line job                                                                         |
| --- | ---------------------- | ------------------------------------------------------------------------------------ |
| 1   | **Configuration**      | Parse XML into layout structs describing which fields appear and how                 |
| 2   | **Field Model**        | Store, validate, and track changes for a single piece of data                        |
| 3   | **Editor Record**      | Group all fields belonging to one open editor tab                                    |
| 4   | **Qt Models**          | Expose records and layout to QML as `QAbstractListModel` rows                        |
| 5   | **Command Dispatcher** | Compare edited fields to the last-known backend state and send only the changed ones |
| 6   | **QML UI**             | Render the window; bind inputs to Qt model roles                                     |

---

## 2. Diagram A — Configuration: From XML to Layout

The editor layout is **entirely data-driven**. No hard-coded field positions exist in C++ or QML.
An XML file (validated against an XSD schema) is parsed into plain C++ structs that describe
every line, field, and button in the editor.

```mermaid
graph TD
    XSD["XSD Schema\nDefines the grammar\nof the config file"]
    XML["Editor Config XML\nSite-specific file\ne.g. sfrEditorConfig.xml"]
    CFG["EditorConfig\nLoads & validates XML\nagainst the schema"]

    FCFG["FieldConfig\nParses field\ndefinitions"]
    HCFG["HeaderConfig\nParses header\nbutton rows"]
    ECFG["ExtraConfig\nParses auxiliary\noptions e.g. equipment"]

    LS["LineStruct\nWidth + label\nfor one row"]
    FS["FieldStruct\nType · label · width\ndrop-down values\nplaceholder"]
    HS["HeaderButtonStruct\nButton type · width\nalign-right flag"]
    ES["ExtraItemStruct\nCheckbox name · group\ncapability code"]

    XSD -->|"validates"| XML
    XML -->|"parsed by"| CFG
    CFG --> FCFG
    CFG --> HCFG
    CFG --> ECFG
    FCFG -->|"produces"| LS
    LS -->|"contains"| FS
    HCFG -->|"produces"| HS
    ECFG -->|"produces"| ES
```

> **Key insight:** Structs carry no Qt or QML dependency. They can be constructed and
> inspected in plain unit tests without a QApplication.

---

## 3. Diagram B — Field Model: Tracking What the User Typed

The `asd.editor` library provides a **reusable, editor-agnostic** set of field types.
Every field knows its current value, its original value, and whether it is valid.

```mermaid
classDiagram
    class IEditorField {
        <<interface>>
        +invalid() bool
        +modified() bool
        +revertChanges()
    }

    class EditorField {
        -originalText : QString
        -text : QString
        -maxLength : int
        -invalid : bool
        -errorMessage : QString
        -warning : bool
        -warningMessage : QString
        +setText(text)
        +reset(text)
        +clear()
        +validateText() pair~bool,QString~
    }

    class ValidatedTextField {
        Validates text against\na regular expression
    }

    class BooleanField {
        Wraps a true/false toggle
    }

    class ScheduleField {
        Parses and validates\ntime/date schedules
    }

    class LookupField {
        Validates against a\nknown database\ne.g. aircraft type ICAO codes
    }

    class ComplexListField {
        <<also QAbstractListModel>>
        Each element is individually\nvalidated and tracked\ne.g. a multi-waypoint route
    }

    class CheckboxGroupField {
        <<also QAbstractListModel>>
        A group of boolean toggles\ne.g. equipment capability codes
    }

    class ExpandableTextField {
        Long free-text with\na pop-out dialogue
    }

    class MessageListField {
        <<also QAbstractListModel>>
        Read-only log of\npast backend messages
    }

    IEditorField <|-- EditorField
    EditorField <|-- ValidatedTextField
    EditorField <|-- BooleanField
    EditorField <|-- ScheduleField
    EditorField <|-- LookupField
    IEditorField <|-- ComplexListField
    IEditorField <|-- CheckboxGroupField
    IEditorField <|-- ExpandableTextField
    IEditorField <|-- MessageListField
```

> **Note:** Fields that contain a list of items (route waypoints, equipment checkboxes,
> message history) additionally inherit `QAbstractListModel` so QML can iterate them directly.

---

## 4. Diagram C — Editor Record & Qt Models: From Fields to QML

An **Editor Record** groups all fields for one open tab. The **Qt Models** sit above the
records and translate them into the role-based API that QML consumes.

```mermaid
graph TD
    subgraph Record["Editor Record — one per open tab"]
        ER["EditorRecord\n(QObject)\nOwns every field\nComputes aggregate modified/invalid\nSupports revertChanges()"]
        F1["SimpleField\ncallsign, level …"]
        F2["ValidatedTextField\nSSR code, speed …"]
        F3["ComplexListField\nroute waypoints"]
        F4["CheckboxGroupField\nequipment capabilities"]
        F5["ExpandableTextField\nfree text, other info"]

        ER --> F1
        ER --> F2
        ER --> F3
        ER --> F4
        ER --> F5
    end

    subgraph QtModels["Qt Models — expose to QML"]
        EM["EditorModel\nQAbstractListModel\nOne row per open tab\nAlso listens to backend events\nDrives tab selection & visibility"]
        LLM["LayoutLineModel\nQAbstractListModel\nOne row per display line\nDriven by config structs"]
        LFM["LayoutFieldModel\nQAbstractListModel\nOne row per field in a line\nCarries label, width, input type"]
        HM["HeaderModel\nQAbstractListModel\nButton rows for the header strip"]
        RM["RecentlyOpenedModel\nOrdered list of\nrecently opened records"]
        SM["SearchFilterProxyModel\nFilters EditorModel\nfor the search panel"]

        ER -->|"rows"| EM
        EM --> RM
        EM --> SM
        LLM -->|"contains"| LFM
    end

    EM -->|"Q_PROPERTY\nexposed via Instantiator"| QML["QML Engine"]
    LLM -->|"Q_PROPERTY\nexposed via Instantiator"| QML
    HM -->|"Q_PROPERTY\nexposed via Instantiator"| QML
```

> `EditorModel` uses **roles** (integer constants, one per field/flag) so QML can bind
> directly to e.g. `model.callsign` or `model.isModified` without extra controllers.

---

## 5. Diagram D — Command Dispatcher: Turning Edits into Backend Commands

When the controller presses **Apply**, the dispatcher compares the edited record against
the last-known backend snapshot and emits exactly the commands needed — one per changed
field group.

```mermaid
sequenceDiagram
    actor Controller as ATCO Controller
    participant QML as QML UI
    participant EditorModel as EditorModel
    participant Record as EditorRecord
    participant Storage as DataStorage
    participant Dispatcher as CommandDispatcher
    participant Backend as Backend (PBL)

    Controller->>QML: presses Apply
    QML->>EditorModel: applyChanges()
    EditorModel->>Storage: getLatestRecord(id)
    Storage-->>EditorModel: sourceRecord (last known state)
    EditorModel->>Dispatcher: sendEditCommand(editedRecord, sourceRecord)

    loop for each changed field group
        Dispatcher->>Dispatcher: diff editedRecord vs sourceRecord
        Dispatcher->>Backend: send AtcoCommand (e.g. UpdateCallsign)
    end

    Backend-->>EditorModel: onUpdateEvent(updatedRecord)
    EditorModel->>Record: reset fields to new backend values
```

The dispatcher also handles **create** (new record), **convert** (minimal → full), and
**cancel coordination** flows. Each is a separate method on the `ICommandDispatcher` interface
so it can be mocked in tests.

---

## 6. Diagram E — QML UI: Component Hierarchy

The QML layer is a **thin presentation tier**. It contains no business logic — it only
binds to the Qt models exposed by the Instantiator singleton.

```mermaid
graph TD
    ROOT["EditorWindow\nRoot item — manages\nvisibility, drag, focus\nworkspace registration"]

    HEADER["HeaderStrip\nTab bar + drag handle\nClose / open tab buttons"]

    BODY["EditorBody\nMain edit area\nDynamically sized"]
    ACTIONS["ActionBar\nApply · Revert · Create\nClose · Convert buttons"]
    FIELDS["FieldGrid\nRepeater over LayoutLineModel\n→ LayoutFieldModel\nBuilds the form dynamically"]

    DIALOGUES["DialogueManager\nFloating sub-windows\npositioned beside the editor"]
    SEARCH["SearchPanel\nFilters open records\nby callsign / flight id"]

    subgraph Components["Reusable Atomic Inputs"]
        TI["TextInput\nSingle-line with\nvalidation highlight"]
        DD["DropDown\nEnum selector"]
        OV["OverflowInput\nText that opens a\npop-out dialogue"]
        RO["ReadOnlyField\nDisplay only"]
        CB["CheckboxInput\nBoolean toggle"]
    end

    subgraph Dialogues["Sub-Dialogues (pop-out)"]
        RD["ListDialogue\ne.g. Route waypoints"]
        ED["CheckboxGroupDialogue\ne.g. Equipment capabilities"]
        SD["FreeTextDialogue\nSupplementary / other info"]
        MD["MessageHistoryDialogue\nRead-only log"]
    end

    ROOT --> HEADER
    ROOT --> BODY
    ROOT --> SEARCH
    ROOT --> DIALOGUES
    BODY --> ACTIONS
    BODY --> FIELDS
    FIELDS --> TI
    FIELDS --> DD
    FIELDS --> OV
    FIELDS --> RO
    FIELDS --> CB
    DIALOGUES --> RD
    DIALOGUES --> ED
    DIALOGUES --> SD
    DIALOGUES --> MD
```

---

## 7. Layer-by-Layer Explanation

### 7.1 Configuration

All editor layout is **data-driven via XML**, validated against XSD schemas in `schemas/`.
The config class inherits three tern-framework interfaces:

- `IDynamicallyLoadable` — can be loaded/reloaded at runtime
- `IXmlInitializable` — parsed from an XML element
- `XmlSchemaValidatableBase` — validated against the XSD before use

It produces plain C++ structs (line, field, header button, extra-item) that describe the
visual layout (widths, labels, field types, drop-down values) with **no Qt or QML
dependency**. This makes the config layer independently testable.

### 7.2 Domain / Field Model (`asd.editor`)

`asd.editor` is a **reusable base library** independent of any specific editor's data
concepts.

- **`IEditorField`** — pure interface: `invalid()`, `modified()`, `revertChanges()`
- **`EditorField`** — concrete base (`Q_GADGET`): stores current and original text, exposes
  `typeHint`, `maxLength`, `errorMessage`, `warningMessage`. Validation is overridable via
  `validateText()`.
- **Specialisations** — `ValidatedTextField` (regex), `BooleanField`, `ScheduleField`,
  `LookupField`, and complex list fields (`ComplexListField`, `CheckboxGroupField`,
  `ExpandableTextField`, `MessageListField`).
- Complex fields subclass **both** `QAbstractListModel` *and* `IEditorField`, giving them
  independent list-model behaviour for their QML delegates.

### 7.3 Editor Record

The Editor Record is a `QObject` that **owns one field (or specialisation) per editable data
point** on a single tab. It is constructed either empty (new record) or populated from a
backend data object. It provides:

- Aggregate `modified()` / `invalid()` computed across all contained fields
- `revertChanges()` delegated to every contained field
- The single source of truth for "what the controller has typed so far"

### 7.4 Qt Models

| Model                      | Role                                                                                                                                                                                            |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **EditorModel**            | Top-level `QAbstractListModel`; one row = one open tab (Editor Record). Also registers itself as a listener for backend event handler interfaces. Drives tab visibility, selection, and search. |
| **LayoutLineModel**        | List of display lines; each row references a `LayoutFieldModel`.                                                                                                                                |
| **LayoutFieldModel**       | List of field metadata (type, label, width, input context, drop-down values) for one line. Used by QML repeaters to build the form layout dynamically.                                          |
| **HeaderModel**            | List of header button rows.                                                                                                                                                                     |
| **RecentlyOpenedModel**    | Ordered list of recently opened record IDs.                                                                                                                                                     |
| **SearchFilterProxyModel** | Proxy on `EditorModel` for the search panel.                                                                                                                                                    |

`EditorModel` roles (integer constants, one per field/flag) cover every editable field *and*
dialogue-visibility flags, allowing QML to bind directly without additional controllers.

### 7.5 Command Dispatcher

`ICommandDispatcher` defines the contract for translating editor state to backend operations.
The concrete implementation:

1. Compares the edited Record against the latest backend snapshot (from `IDataStorage`)
2. Builds zero or more typed command objects — **one per changed field group**
3. Sends them through the backend interface (`IPolarisBackEnd`)

Each comparison is encapsulated in a private `create*Command()` method, making the
dispatcher easy to unit-test and extend independently. A `CommandFieldConstructor` helper
converts free-text field values into strongly-typed domain values expected by each command.

### 7.6 QML UI

The QML layer is a thin presentation tier that binds to Qt models via the Instantiator
(registered as a QML singleton):

| Component | Purpose |
|-----------|---------|
| `EditorWindow` | Root item; manages visibility, drag, focus, workspace registration |
| `HeaderStrip` | Tab bar + drag handle |
| `EditorBody` | Main edit area; dynamically sized to its content |
| `ActionBar` | Apply / Revert / Create / Close buttons |
| `FieldGrid` | Repeater over `LayoutLineModel` → `LayoutFieldModel`; builds form dynamically |
| `DialogueManager` | Manages floating sub-dialogues positioned beside the editor |
| `SearchPanel` | Filters open tabs by identifier |
| `EditorComponents/` | Reusable atomic inputs: text, dropdown, overflow, read-only, checkbox |
| `EditorDialogues/` | Full sub-dialogues for complex fields: list, checkbox group, free text, message log |

### 7.7 Instantiator

The Instantiator is the **composition root**. It is constructed once at application startup
and:

- Owns all shared instances via `std::shared_ptr`
- Takes all external dependencies as constructor parameters (backend, config, event
  processors, command dispatcher)
- Exposes `EditorModel`, `LayoutLineModel`, `HeaderModel`, and any helper controllers as
  `Q_PROPERTY` constants so QML can access them via the singleton name

---

## 8. Key Design Patterns

| Pattern                                  | Where used                                                                                                                                                      |
| ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Interface-based dependency injection** | All external systems accessed through interfaces (`IDataStorage`, `ICommandDispatcher`). Makes unit testing straightforward with mocks.                         |
| **Data-driven UI layout**                | Config structs drive both `LayoutLineModel`/`LayoutFieldModel` and the QML repeaters; adding a new field requires only XML + enum changes, no C++ or QML edits. |
| **Command pattern**                      | Each field edit becomes a discrete typed command — atomic, independently testable, and easy to extend.                                                          |
| **Event handler fan-out**                | `EditorModel` registers for multiple backend event handler interfaces; updates arrive as typed callbacks, not generic signals.                                  |
| **Composition root (Instantiator)**      | Single class owns the object graph, keeping construction order explicit and preventing circular dependencies.                                                   |
| **Role-based list models**               | `QAbstractListModel` with custom integer roles lets QML bind to any field property without extra controllers or adapters.                                       |

---

## 9. Guidance for the Planned Editors

### Track Label Editor (`asd.track.label.editor`)

Likely needs:
- A **Track Label Record** owning fields for label line content, offset position, etc.
- An **`ITrackLabelCommandDispatcher`** dispatching label offset/content commands
- A simple **`TrackLabelEditorModel`** (single-row or per-track list model)
- A **config** describing which label fields are editable per site
- A **`TrackLabelInstantiator`** as the composition root

The `asd.editor` field library (`IEditorField`, `EditorField`, `ValidatedTextField`) should be
reused directly.

### Flight List Editor (`asd.flight.list.editor`)

Likely needs:
- A **List Row Record** per editable row — either a lightweight specialisation reusing
  `EditorField`, or the full flight record with a smaller active field set
- A **`FlightListEditorModel`** that wraps the existing `FlightDataModel` and adds per-row
  edit state (modified, invalid, selected)
- Column layout driven by config (the same `LayoutLineModel` / `LayoutFieldModel` pattern)
- Commands dispatched via the existing `IAtcoCommandDispatcher`
- A **`FlightListInstantiator`** registering the model with the QML engine

In both cases the **same six-layer structure** applies:

```
Config → Field Model → Record → Qt Model → Command Dispatcher → QML
```

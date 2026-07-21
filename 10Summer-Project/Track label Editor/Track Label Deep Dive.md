# Track Label — Deep Dive Findings

*Session: 2026-05-22 | Covers: config structure, state/visibility, safety nets, rendering pipeline, editor architecture*

---

## 1. Single Config File

`polaris-asd-tracklabel-config.xml` is the **only** file for track labels.
All states (Correlated, Uncorrelated, FlightPlanTrack, Ground) live as sibling `<tracklabel>` blocks inside it.
No multiple-files-per-state pattern. Editor operates on one file.

---

## 2. Two Levels of State-Driven Display

**Level 1 — Track type (macro):** C++ selects the entire `<tracklabel tracklabel-type="...">` block based on how the track is classified. Completely different line/field sets per type.

**Level 2 — Per-field visibility (micro):** Within each block, fields show/hide via:

| Mechanism       | Attribute                                  | Logic                                               |
| --------------- | ------------------------------------------ | --------------------------------------------------- |
| Control state   | `when-unselected-for-control-states`       | Whitelist — show unselected only if in these states |
| Flight property | `show-if flight-property / property-value` | AND condition on a flight data value                |
| Focus           | `only-show-on-focus`                       | Hidden unless track hovered/selected                |
| Holding         | `visible-in-holding`                       | Default false — hidden in holding pattern           |
| User toggle     | `toggleable`                               | Controller can show/hide via middle mouse           |
|                 |                                            |                                                     |

No field has positional coordinates — position = XML document order only.

---

## 3. Safety Nets — Separate, Not Positionally Linked

Three config files involved, three separate concerns:

| Config                              | Controls                                                                  |
| ----------------------------------- | ------------------------------------------------------------------------- |
| `tracklabel-config.xml`             | Field structure and conditional visibility                                |
| `polaris-safety-nets` config        | Alert text strings (STCA, APW…) + which SNS types to check for inhibition |
| `polaris-field-color-mapper` config | Field colors based on runtime conditions (rvsmWarning, controlState…)     |

Safety net alert text (STCA/APW overlay) position is **hardcoded** in `CriticalDataTrackLabelField.qml` — anchored `bottom: left` of the label. Not configurable via any XML.

**`snsInhibitedSsrDot`** is the one bridge: safety-nets config defines which SNS types to watch → `TargetDataModel` checks the track's Mode 3/A SSR code against the inhibited list → boolean role → QML renders a 5×5 dot. The tracklabel config only controls *where on the label* this dot field is placed (like any other field).

---

## 4. XML Order = Render Position

QML is a pure renderer. No positional logic lives in QML at all.

```
XML document order
  → childNodes() iterator (preserves order)
  → C++ vector push_back (lines[], fields[])
  → Qt model row index
  → QML Repeater index
  → ColumnLayout (lines top→bottom) + RowLayout (fields left→right)
```

XSD `xs:sequence` only validates — it enforces element order during schema validation, not at render time. The actual render order is purely the vector order, which is the XML document order.

**To move a field:** reorder within `<tracklabel-line>` (left/right) or move to another line (up/down). No QML changes needed.

---

## 5. Editor Runtime Architecture

Polaris ASD uses read-only models. The editor needs the **same 3-layer stack but mutable**:

```
EditorTrackLabelModel      (QAbstractListModel — rows = lines)
  └─ EditorLineModel       (QAbstractListModel — rows = fields)
       └─ EditorFieldModel (QObject — Q_PROPERTY with NOTIFY for each attr)
```

**Runtime flow:**
1. **Load:** parse XML → build mutable model in memory
2. **Edit:** QML calls `Q_INVOKABLE` C++ methods → model mutates → Qt signals fire → QML auto-updates
3. **Save:** walk model in vector order → serialize back to XML (preserve `${property}` placeholders)

**Key Qt signals the model must emit:**

| Op | Signal | QML effect |
|---|---|---|
| Change attr | `dataChanged(i,i,roles)` | Single field re-renders |
| Add field | `beginInsertRows` / `endInsertRows` | New field appears |
| Remove field | `beginRemoveRows` / `endRemoveRows` | Field disappears |
| Reorder | `beginMoveRows` / `endMoveRows` | Fields shift |

**Critical:** `TrackLabelField` in Polaris has no setters and no `NOTIFY` signals — **cannot be reused** in the editor. Write mutable counterparts with full `Q_PROPERTY(... WRITE ... NOTIFY ...)`.

Start from: `~/tern_vault/Field_gen_from_XSD_scheme/qt-wasm-exploration/apps/schema-editor-generator/src/parsers/SchemaTypes.h`

---

*Related: [[Track Label Editor]] | [[XSD Field Generation]] | [[Summer Project Overview]]*

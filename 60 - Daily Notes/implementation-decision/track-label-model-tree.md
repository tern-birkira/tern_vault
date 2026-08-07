# Track Label Editor — Model Tree Refactor

  

## Overview

  

Refactor the track-label-editor so that its C++ model classes are a **strict 1:1 structural mirror** of the XSD schema (`tracklabel.xsd`, `tracklabel-types.xsd`, `polaris-shared-types.xsd`). The compiler becomes the structural schema validator: if it builds, the data shape is guaranteed valid. A generic reflection-based serialization engine replaces per-editor translation code.

  

### Guiding Principles

  

1. **Every XSD complex type → one C++ class.** Container elements derive from `QAbstractListModel`; leaf/data elements derive from `QObject`.

2. **Every XSD `xs:simpleType` (restriction/enumeration) → one C++ `enum class`** with Q_ENUM, loaded from the type definitions so the compiler and UI enforce allowed values.

3. **XSD cardinality and choice rules are enforced in model mutation methods** (insertRows, add/remove helpers), not by runtime XML validation.

4. **A single reflection-based serialization engine** (the "Model Tree Walker") handles all import/export using `Q_PROPERTY` metadata + an `IXmlSerializable` interface.

5. **XML attributes vs. app-only properties** are separated via a whitelist contract (`xmlAttributeWhitelist()`), preventing runtime UI state from leaking into saved XML files.

  

---

  

## Current State vs. Target State

  

### Classes that exist today

  

| Current Class | Maps to XSD Type | Status |

|---|---|---|

| `TrackLabelModel` | `TrackLabelGenericType` (partially) | Does not distinguish `TrackLabelType` vs `ExtendedTrackLabelType`. No `tracklabel-type` attribute. |

| `FieldListModel` | `TrackLabelLineType` (partially) | Does not model the `xs:choice` between `field`, `field18`, `equipment-field`. Treats all children as generic `FieldInterface`. |

| `FieldInterface` | `TrackLabelGenericFieldType` | ✅ Implements IXmlSerializable. Owns child elements (context-menu-items, visibility, edit). `xmlTagName()` pure virtual. |

| `FieldText` | Sub-object of `TrackLabelGenericFieldType` | ✅ Implements IXmlSerializable. Whitelist: prefix, placeholder. `fieldType` removed. |

| `FieldAppearance` | Sub-object of `TrackLabelGenericFieldType` | ✅ Implements IXmlSerializable. Whitelist: toggleable, blinking, onlyShowOnFocus, visibleInHolding. |

| `FieldLayout` | Sub-object of `TrackLabelGenericFieldType` | ✅ Implements IXmlSerializable. Whitelist: fontAdjustment, fixedWidthInCharacters, leftMargin, bottomMargin. |

| `TrackLabelFieldModel` | `TrackLabelFieldType` | ✅ Created. xmlTagName="field", whitelist={"fieldName"}. |

| `TrackLabelEditorController` | No schema mapping (app-only) | Fine as-is; orchestrates label types. |

  

### Classes that are missing entirely

  

| XSD Type | Needed C++ Class |

|---|---|

| `TrackLabelListType` | `TrackLabelConfigModel` — root container, owns `tracklabel[]`, `extended-tracklabel?`, `column-0-actions?`. Has attributes: `path`, `radio-callsign-filepath`, `has-external-transfer`, `has-force-act`, `has-flight-rule-change`. |

| `TrackLabelType` | Rename/specialize current `TrackLabelModel` — extends `TrackLabelGenericType`, adds `tracklabel-type` (required, type `TrackLabelTypeVariant`). |

| `ExtendedTrackLabelType` | `ExtendedTrackLabelModel` — extends `TrackLabelGenericType`, adds `tracklabel-type` (default `"Extended"`, type `ExtendedTypeVariant`). |

| `TrackLabelFieldType` | `TrackLabelFieldModel` — extends `TrackLabelGenericFieldType`, adds required `field-name` (type `FieldType`). |

| `TrackLabelField18Type` | `TrackLabelField18Model` — extends `TrackLabelGenericFieldType`, owns `field18-item[]`. |

| `TrackLabelEquipmentFieldType` | `TrackLabelEquipmentFieldModel` — extends `TrackLabelGenericFieldType`, owns `field-equipment-item[]` (minOccurs=1). |

| `ContextMenuItem` | `ContextMenuItemModel` — leaf QObject. Attributes: `context-menu` (required, `ContextMenuType`), `mouse`, `type`, `menu-position`, `searchable`. |

| `FieldVisibilityRulesType` | `FieldVisibilityModel` — leaf QObject. Owns exclusive child: `show-if` XOR `hide-if` (xs:choice, max 1). Attribute: `when-unselected-for-control-states`. |

| `VisibilityRuleType` | `VisibilityRuleModel` — leaf QObject. Attributes: `flight-property` (optional, `FieldType`), `property-value` (required, space-separated list). |

| `FieldEditSettings` | `FieldEditSettingsModel` — leaf QObject. Attribute: `on-mouse` (required, `MouseButtonType`). |

| `Field18Item` | `Field18ItemModel` — leaf QObject. Attribute: `type` (`Field18Type`). |

| `EquipmentItem` | `EquipmentItemModel` — leaf QObject. Attribute: `type` (`EquipmentType`). |

| `Column0ActionsType` | `Column0ActionsModel` — container, owns `action[]` (minOccurs=1). |

| `ActionType` | `ActionModel` — leaf QObject. Attribute: `type` (required, `ActionTypeVariant`). |

  

---

  

## Task 1 — Schema-Derived Enum Types ✅

  

Port every `xs:simpleType` enumeration from the XSD into C++ `enum class` values with `Q_ENUM_NS`. Store them in `asd.editor.commontypes`. This gives the compiler and QML dropdowns the exact allowed values.

  

### Subtask 1.1 — New enums from `tracklabel-types.xsd`

  

Create or update `tracklabel_types.h` to contain:

  

| XSD Type | C++ Enum | Values |

|---|---|---|

| `TrackLabelTypeVariant` | `TrackLabelTypeVariant` | `Correlated`, `Uncorrelated`, `FlightPlanTrack`, `Ground` |

| `ExtendedTypeVariant` | `ExtendedTypeVariant` | `Extended` |

| `FieldType` | `FieldType` | All 60+ values from the schema (e.g. `aircraftType`, `callsign`, `clearedFlightLevel`, …) |

| `Field18Type` | `Field18Type` | `PBN`, `COM`, `DAT`, `SUR`, … (22 values) |

| `EquipmentType` | `EquipmentType` | `communicationAndNavigationCapabilities`, … (10 values) |

| `ContextMenuType` | `ContextMenuType` | `CallsignMenu.Correlated`, … (16 values) — use underscores for dots in C++ |

| `MouseButtonType` | `MouseButtonType` | `left`, `middle`, `right` |

| `MousePressType` | `MousePressType` | `press`, `hold` |

| `MenuPosition` | `MenuPosition` | `left`, `right` |

| `ActionTypeVariant` | `ActionTypeVariant` | `ManualOutboundCoordinationTimeoutAction`, `RequestInIndicator`, `RequestOutIndicator`, `TransferAction`, `VFRIndicator` |

  

Each enum must have:

- A `Q_ENUM_NS` registration so QML can use it.

- A `toString()` and `fromString()` conversion pair.

- A `QStringList allValues()` static helper that returns the allowed values for populating UI combo boxes.

  

### Subtask 1.2 — ControlType enum alignment with XSD (`polaris-shared-types.xsd`)

  

The XSD `ControlType` defines: `NotSet`, `NonConcerned`, `Concerned`, `Intruder`, `TransferInInitiated`, `RequestInInitiated`, `Assumed`, `TransferOutInitiated`, `RequestOutInitiated`, `Redundant`, `Completed`, `All`.

  

The current C++ `ControlState` enum has `Unknown` (internal ASD concept) and `ControlStateCount` (sentinel). The editor must think only in schema terms.

  

**Action:** Create a separate `ControlType` enum in `tracklabel_types.h` (not modify `flight_types.h`, which serves ASD internals) with exactly the XSD values:

  

| C++ Enum Value | XSD String |

|---|---|

| `NotSet` | `"NotSet"` |

| `NonConcerned` | `"NonConcerned"` |

| `Concerned` | `"Concerned"` |

| `Intruder` | `"Intruder"` |

| `TransferInInitiated` | `"TransferInInitiated"` |

| `RequestInInitiated` | `"RequestInInitiated"` |

| `Assumed` | `"Assumed"` |

| `TransferOutInitiated` | `"TransferOutInitiated"` |

| `RequestOutInitiated` | `"RequestOutInitiated"` |

| `Redundant` | `"Redundant"` |

| `Completed` | `"Completed"` |

| `All` | `"All"` |

  

Include `toString`, `fromString`, `allValues()` as with other enums.

  

### Subtask 1.3 — Schema-local enum in `tracklabel.xsd`

  

| XSD Type | C++ Enum | Values |

|---|---|---|

| `FontSizeDelta` | `FontSizeDelta` | `-4`, `-2`, `-1`, `0`, `+1`, `+2`, `+4` — use an `int`-backed enum or a constrained int type with a static validator. |

  

### Subtask 1.4 — Merge existing `LabelType` with `TrackLabelTypeVariant`

  

The current `commontypes::tracklabel::LabelType` duplicates `TrackLabelTypeVariant`. Replace it with the schema-derived enum to eliminate the gap.

  

---

  

## Task 2 — Schema-Mirroring Model Classes

  

Create the full class hierarchy that mirrors the XSD 1:1. Each class implements the `IXmlSerializable` interface.

  

### Subtask 2.1 — Define the `IXmlSerializable` interface ✅

  

```cpp

class IXmlSerializable

{

public:

virtual ~IXmlSerializable() = default;

virtual const char* xmlTagName() const { return nullptr; }

virtual QStringList xmlAttributeWhitelist() const = 0;

virtual QList<QObject*> xmlChildren() const { return {}; }

virtual QList<QObject*> serializationExtensions() const { return {}; }

};

```

  

- `xmlTagName()`: The XML element name (e.g. `"field"`, `"visibility"`). Returns `nullptr` by default — extension objects that are not XML elements themselves (e.g. FieldText, FieldAppearance, FieldLayout) inherit this default.

- `xmlAttributeWhitelist()`: Only these `Q_PROPERTY` names get serialized to XML attributes. All others (app-only state like `isVisible`, `isHovered`, `position`, `width`, `height`, `displayName`) are excluded. **Extension sub-objects also implement this interface** so the serializer can call `xmlAttributeWhitelist()` on them directly.

- `xmlChildren()`: Returns child model objects in document order for tree walking. Each container casts its owned children (e.g. `QVector<TrackLabelLineModel*>`) to `QList<QObject*>` before returning.

- `serializationExtensions()`: Returns sub-component QObjects whose properties should be flattened into the same XML element (for the FieldText/FieldAppearance/FieldLayout split). Each extension implements `IXmlSerializable` so the serializer discovers their attributes via `xmlAttributeWhitelist()`.

  

### Subtask 2.2 — Leaf element models (QObject-based) ✅

  

Create each of the following as `QObject` + `IXmlSerializable`:

  

#### `ContextMenuItemModel`

- **Tag:** `context-menu-item`

- **XML attributes (Q_PROPERTY + whitelisted):**

- `contextMenu` → `context-menu` (type: `ContextMenuType`, required)

- `mouse` → `mouse` (type: `MouseButtonType`, default: `left`)

- `type` → `type` (type: `MousePressType`, default: `press`)

- `menuPosition` → `menu-position` (type: `MenuPosition`, default: `left`)

- `searchable` → `searchable` (type: `bool`, default: `false`)

  

#### `VisibilityRuleModel`

- **Tag:** `show-if` or `hide-if` (set at construction; stored as member, not property)

- **XML attributes:**

- `flightProperty` → `flight-property` (type: `FieldType`, optional)

- `propertyValue` → `property-value` (type: `QString` space-separated list, required)

  

#### `FieldVisibilityModel`

- **Tag:** `visibility`

- **XML attributes:**

- `whenUnselectedForControlStates` → `when-unselected-for-control-states` (type: `ControlTypeList` — `QVector<ControlType>` with `Q_DECLARE_METATYPE`, space-separated when serialized)

- **Children (xs:choice, minOccurs=0, maxOccurs=1 — exclusive):**

- `m_showIf` (`VisibilityRuleModel*`, nullable) XOR `m_hideIf` (`VisibilityRuleModel*`, nullable) — stored as `std::variant<std::monostate, ShowIf, HideIf>` so exclusivity is compile-time enforced.

- **Enforcement:** Setting one variant alternative automatically replaces the other. `clearRule()` resets to `std::monostate`.

- **Serialization rules for `when-unselected-for-control-states`:**

- If `NotSet` → the `visibility` element must NOT be generated at all (serializer skips it).

- If `All` → expand to all concrete control types (`NonConcerned Concerned Intruder TransferInInitiated RequestInInitiated Assumed TransferOutInitiated RequestOutInitiated Redundant Completed`) when writing the attribute.

- Otherwise → write the selected subset as a space-separated list.

- **Parent dependency rule:** The `visibility` child element is only valid on a field when `only-show-on-focus` is `false`. If `only-show-on-focus` is `true`, the model must not allow creating/attaching a `FieldVisibilityModel`. The `GenericFieldModel::setOnlyShowOnFocus(true)` setter must clear any existing `m_visibility` child.

  

#### `FieldEditSettingsModel`

- **Tag:** `edit`

- **XML attributes:**

- `onMouse` → `on-mouse` (type: `MouseButtonType`, required)

  

#### `Field18ItemModel`

- **Tag:** `field18-item`

- **XML attributes:**

- `type` → `type` (type: `Field18Type`)

  

#### `EquipmentItemModel`

- **Tag:** `field-equipment-item`

- **XML attributes:**

- `type` → `type` (type: `EquipmentType`)

  

#### `ActionModel`

- **Tag:** `action`

- **XML attributes:**

- `type` → `type` (type: `ActionTypeVariant`, required)

  

### Subtask 2.3 — Field hierarchy (mirrors `TrackLabelGenericFieldType` inheritance) ✅ (partial — FieldInterface + TrackLabelFieldModel done)

  

The XSD uses `xs:extension` inheritance for fields. Mirror this in C++:

  

#### `FieldInterface` (abstract base, QObject + IXmlSerializable) ✅

**Class name kept as `FieldInterface`** (not renamed to `GenericFieldModel`).

Maps to `TrackLabelGenericFieldType`. Holds all generic attributes split across three sub-objects for code readability:

  

- **`FieldText`** (QObject + IXmlSerializable) — `xmlAttributeWhitelist()` returns `{"prefix", "placeholder"}`

- `fieldName` removed from FieldText — it now lives on `TrackLabelFieldModel` as a typed `FieldType` enum property.

- `displayName` is app-only (not in whitelist).

- `fieldType` has been removed (redundant — field-name values are typed by the `FieldType` enum).

- **`FieldAppearance`** (QObject + IXmlSerializable) — `xmlAttributeWhitelist()` returns `{"toggleable", "blinking", "onlyShowOnFocus", "visibleInHolding"}`

- **`FieldLayout`** (QObject + IXmlSerializable) — `xmlAttributeWhitelist()` returns `{"fontAdjustment", "fixedWidthInCharacters", "leftMargin", "bottomMargin"}`

- `position`, `width`, `height` are app-only (not in whitelist).

  

`FieldInterface::xmlAttributeWhitelist()` returns `{}` (empty) — all generic attributes are discovered by the serializer on the extension sub-objects via `serializationExtensions()`. Subclasses override to add their own attributes (e.g. `{"fieldName"}`).

  

`FieldInterface::xmlTagName()` is pure virtual — subclasses provide `"field"`, `"field18"`, or `"equipment-field"`.

  

`FieldInterface::fieldKind()` is pure virtual — subclasses return `FieldKind::Field`, `FieldKind::Field18`, or `FieldKind::EquipmentField`. Used by FieldListModel's `FieldKindRole` for QML delegate selection.

  

Owns child elements from `TrackLabelGenericFieldType`:

- `m_contextMenuItems` — `QVector<ContextMenuItemModel*>` (minOccurs=0, maxOccurs=unbounded)

- `m_visibility` — `std::unique_ptr<FieldVisibilityModel>` (minOccurs=0, maxOccurs=1)

- `m_editSettings` — `std::unique_ptr<FieldEditSettingsModel>` (minOccurs=0, maxOccurs=1)

  

**Q_PROPERTYs are separated with comments** into `// --- App-only properties ---` and sub-objects have `// --- XML attributes ---` sections.

  

**App-only properties** (not whitelisted):

- `isVisible` (bool) — runtime display state

- `displayName` (QString) — human-readable label derived from field-name

- `position` (QPointF) — layout position computed by the view

- `width`, `height` (int) — computed dimensions

  

#### `TrackLabelFieldModel` (extends `FieldInterface`) ✅

- **Tag:** `field`

- **Q_PROPERTY:** `fieldName` of type `FieldType` enum (required attribute)

- **`xmlAttributeWhitelist()`:** returns `{"fieldName"}` — serialized as `field-name` XML attribute

- **Enforcement:** `fieldName` is a `FieldType` enum; only valid enum values can be set.

  

#### `TrackLabelField18Model` (extends `FieldInterface`)

- **Tag:** `field18`

- **Additional children:**

- `m_field18Items` — `QVector<Field18ItemModel*>` (minOccurs=0, maxOccurs=unbounded)

  

#### `TrackLabelEquipmentFieldModel` (extends `FieldInterface`)

- **Tag:** `equipment-field`

- **Additional children:**

- `m_equipmentItems` — `QVector<EquipmentItemModel*>` (minOccurs=1, maxOccurs=unbounded)

- **Enforcement:** `removeRows()` refuses if it would leave 0 items.

  

### Subtask 2.4 — Container models (QAbstractListModel-based)

  

#### `FieldListModel` (kept, not renamed — maps to `TrackLabelLineType`)

- **Tag:** `tracklabel-line`

- **Children (xs:choice, minOccurs=0, maxOccurs=unbounded):**

- Stores a `QVector<FieldInterface*>` where each item is one of:

- `TrackLabelFieldModel`

- `TrackLabelField18Model`

- `TrackLabelEquipmentFieldModel`

- The `xs:choice` with `maxOccurs="unbounded"` means any mix of these three types in any order and quantity is valid.

- **Roles (current implementation):**

- `FieldObjectRole` → `FieldInterface*` (the object pointer; QML accesses all properties directly)

- `FieldKindRole` → `FieldInterface::FieldKind` enum (which subclass variant, for delegate selection)

- `DisplayNameRole` → app-only human-readable name

- `PrefixRole` → from FieldText (generic attribute)

- `PlaceholderRole` → from FieldText (generic attribute)

- `ToggleableRole` → from FieldAppearance (generic attribute)

- `OnlyShowOnFocusRole` → from FieldAppearance (generic attribute)

- `FontAdjustmentRole` → from FieldLayout (generic attribute)

- `FixedWidthInCharactersRole` → from FieldLayout (generic attribute)

- `LeftMarginRole` → from FieldLayout (generic attribute)

- `BottomMarginRole` → from FieldLayout (generic attribute)

- `VisibleInHoldingRole` → from FieldAppearance (generic attribute)

- `IsVisibleRole` → runtime display state (app-only)

- **Removed roles:**

- `FieldNameRole` — only exists on `TrackLabelFieldModel`; QML accesses via `fieldObject.fieldName`

- `PositionRole`, `WidthRole`, `HeightRole` — app-only computed layout; QML accesses via `fieldObject.fieldLayout.position` etc.

  

> **Suggested future simplification:** Reduce to only `FieldObjectRole` + `FieldKindRole`. All per-property roles become redundant once QML delegates bind directly to the QObject pointer's properties. This removes all role-based `data()`/`setData()` boilerplate and lets property change signals drive updates naturally.

  

#### `TrackLabelModel` (mirrors `TrackLabelGenericType`)

- **Tag:** `tracklabel` or `extended-tracklabel` (set by subclass)

- **Children:**

- `m_lines` — `QVector<TrackLabelLineModel*>` (maxOccurs=unbounded)

- Remains a `QAbstractListModel` exposing lines as rows.

  

#### `TrackLabelVariantModel` (mirrors `TrackLabelType`, extends `TrackLabelModel`)

- **Additional XML attributes:**

- `tracklabelType` → `tracklabel-type` (type: `TrackLabelTypeVariant`, required)

  

#### `ExtendedTrackLabelModel` (mirrors `ExtendedTrackLabelType`, extends `TrackLabelModel`)

- **Additional XML attributes:**

- `tracklabelType` → `tracklabel-type` (type: `ExtendedTypeVariant`, default: `Extended`)

  

#### `Column0ActionsModel` (mirrors `Column0ActionsType`)

- **Tag:** `column-0-actions`

- **Children:**

- `m_actions` — `QVector<ActionModel*>` (minOccurs=1, maxOccurs=unbounded)

- **Enforcement:** `removeRows()` refuses if it would leave 0 items.

  

#### `TrackLabelConfigModel` (mirrors `TrackLabelListType` — the root)

- **Tag:** `tracklabel-configuration`

- **XML attributes:**

- `path` (string, required)

- `radioCallsignFilepath` → `radio-callsign-filepath` (string, optional)

- `hasExternalTransfer` → `has-external-transfer` (bool, default: `true`)

- `hasForceAct` → `has-force-act` (bool, default: `true`)

- `hasFlightRuleChange` → `has-flight-rule-change` (bool, default: `false`)

- **Children:**

- `m_tracklabels` — `QVector<TrackLabelVariantModel*>` (maxOccurs=unbounded)

- `m_extendedTracklabel` — `ExtendedTrackLabelModel*` (minOccurs=0, maxOccurs=1, nullable)

- `m_column0Actions` — `Column0ActionsModel*` (minOccurs=0, maxOccurs=1, nullable)

- **Enforcement:**

- Only 0 or 1 `extended-tracklabel`. Setter replaces; no add method for multiples.

- Only 0 or 1 `column-0-actions`. Same pattern.

  

---

  

## Task 3 — XSD and application rule enforcements in Models

  

Schema rules that C++ types alone can't prevent must be enforced in model mutation methods.

  

### Subtask 3.1 — `FieldVisibilityModel`: exclusive choice (`show-if` XOR `hide-if`) ✅

  

The schema defines:

```xml

<xs:choice minOccurs="0"> <!-- maxOccurs defaults to 1 -->

<xs:element name="show-if" type="VisibilityRuleType"/>

<xs:element name="hide-if" type="VisibilityRuleType"/>

</xs:choice>

```

  

**Implementation:** A `std::variant<std::monostate, ShowIf, HideIf>` where `ShowIf` and `HideIf` are tagged structs holding a `std::unique_ptr<VisibilityRuleModel>`. `std::monostate` = no rule. The variant makes exclusivity compile-time enforced — there is physically one slot.

```cpp

struct ShowIf { std::unique_ptr<VisibilityRuleModel> rule; };

struct HideIf { std::unique_ptr<VisibilityRuleModel> rule; };

using VisibilityRule = std::variant<std::monostate, ShowIf, HideIf>;

```

`setShowIf()` and `setHideIf()` replace the variant (unique_ptr auto-destructs the old rule). `xmlChildren()` visits the variant to return 0 or 1 items.

  

### Subtask 3.2 — `TrackLabelConfigModel`: singleton children

  

- `extended-tracklabel` maxOccurs=1 (default) with minOccurs=0 → nullable single pointer, not a list.

- `column-0-actions` same pattern.

  

### Subtask 3.3 — `Column0ActionsModel` and `TrackLabelEquipmentFieldModel`: minOccurs=1

  

These require at least 1 child. `removeRows()` must guard:

```cpp

bool removeRows(int pos, int count, const QModelIndex& parent) override {

if (rowCount() - count < 1) return false;

// ... proceed

}

```

  

### Subtask 3.4 — `TrackLabelFieldModel`: required attribute

  

`field-name` has `use="required"`. The constructor must take a `FieldType` parameter; no default. The model is invalid without it.

  

### Subtask 3.5 — `FontSizeDelta` constrained values

  

`font-adjustment` only accepts `{-4, -2, -1, 0, +1, +2, +4}`. The setter must validate:

```cpp

void setFontAdjustment(int value) {

static const QSet<int> allowed = {-4, -2, -1, 0, 1, 2, 4};

if (!allowed.contains(value)) return;

// ...

}

```

  
  

### Subtask 3.6 — `FieldVisibilityModel` depends on `only-show-on-focus`

  

The `<visibility>` element is only meaningful when the parent field has `only-show-on-focus="false"`. If a field is set to only-show-on-focus, its visibility is governed entirely by focus state, making conditional visibility rules invalid.

  

**Implementation in `FieldInterface`:**

```cpp

void setOnlyShowOnFocus(bool value) {

m_onlyShowOnFocus = value;

if (value) {

m_visibility.reset(); // smart pointer auto-destructs

}

emit onlyShowOnFocusChanged();

}

  

void setVisibility(std::unique_ptr<FieldVisibilityModel> model) {

if (m_fieldAppearance->onlyShowOnFocus()) return; // Reject: not applicable

// ... take ownership

}

```

  

### Subtask 3.7 — `ControlType::NotSet` and `ControlType::All` serialization semantics

  

- `NotSet` means "no visibility constraint" → the serializer must skip the entire `<visibility>` element.

- `All` means "visible for all control states" → expand to all concrete values when writing `when-unselected-for-control-states`.

- On import: if the attribute contains all concrete values, collapse to `All` internally.

  
  

---

  

## Task 4 — Generic Reflection Serialization Engine

  

A single engine that walks any `IXmlSerializable` model tree to produce/consume XML, reusable across all future editors, if they structure models as xsd tree.

  

### Subtask 4.1 — `XmlModelSerializer` class

  

```

XmlModelSerializer

├── serialize(IXmlSerializable* root) → tern::xml::XmlDocument

└── deserialize(tern::xml::XmlElement&, IXmlSerializable* root)

```

  

**Serialize algorithm:**

1. Create element with `root->xmlTagName()`.

2. Get `xmlAttributeMap()` from root; for each `{propertyName, xmlAttrName}` pair, resolve property on root's QMetaObject, read value, write as XML attribute using `xmlAttrName`.

3. Loop `serializationExtensions()`; for each extension, cast to `IXmlSerializable*`, call its `xmlAttributeMap()`, resolve each property on that extension's QMetaObject, and write as attribute on the same element.

4. Loop `xmlChildren()`; for each child, recurse from step 1 and `appendChild()`.

  

**Deserialize algorithm — detailed:**

  

The XML file has an outer envelope (`<instances>` → `<instance>` → `<tlabel:configuration>`) that is not part of the model tree. The deserializer receives the first model-relevant element (e.g. `<tracklabel-configuration>`) and a pre-constructed root model object to populate.

  

#### Subtask 4.1.1 — Entry point: `deserialize(XmlElement&, IXmlSerializable* root)`

  

The root model object is pre-constructed by the caller (not created by the factory). This is because the root type is known at the call site (e.g. `TrackLabelConfigModel`). The entry point delegates to the recursive worker:

  

```

deserialize(element, root):

1. Verify element.localName() matches root->xmlTagName(). Log warning and return on mismatch.

2. Call populateAttributes(element, root).

3. Call populateChildren(element, root).

```

  

#### Subtask 4.1.2 — `populateAttributes(XmlElement&, QObject* target)`

  

Reads XML attributes from the element and writes them to Q_PROPERTYs on the target object and its serialization extensions.

  

```

populateAttributes(element, target):

1. Build an attribute lookup map (xmlAttrName → {QObject* owner, QMetaProperty prop}):

a. For the target itself: cast to IXmlSerializable*, get xmlAttributeMap().

For each {propertyName, xmlAttrName} pair, find the QMetaProperty on target->metaObject().

Store mapping: xmlAttrName → (QObject* owner, QMetaProperty prop).

b. For each extension in target's serializationExtensions():

Cast to IXmlSerializable*, get its xmlAttributeMap().

For each {propertyName, xmlAttrName}, find QMetaProperty on the extension's metaObject().

Store mapping: xmlAttrName → (QObject* extensionOwner, QMetaProperty prop).

  

2. Iterate all attributes on the XmlElement:

a. Get the attribute name (local name, no namespace prefix).

b. Skip namespace declarations (xmlns:*).

c. Look up the attribute name in the map.

d. If not found: log warning ("unknown attribute 'X' on element 'Y'"), skip.

e. If found: convert the string value to the property's type:

- QString: use directly.

- bool: "true"/"false" string comparison.

- int: QString::toInt().

- Q_ENUM types: use QMetaEnum::keyToValue() via the property's QMetaType.

The enum converter must handle XSD enum values (e.g. "CallsignMenu.Correlated").

f. Write: prop.write(owner, convertedValue).

```

  

#### Subtask 4.1.3 — `populateChildren(XmlElement&, QObject* parent)`

  

Iterates child elements, creates model objects via the factory, populates them recursively, and attaches them to the parent.

  

```

populateChildren(element, parent):

1. Iterate child nodes of element (firstChild → nextSibling loop, filter for Element nodeType).

2. For each child XmlElement:

a. Get localName (strip namespace prefix).

b. Look up localName in the tag→factory registry.

c. If not found: log warning ("unknown element 'X' inside 'Y'"), skip.

d. If found: call factory(parent) to create the child QObject.

e. Cast child QObject to IXmlSerializable*.

f. Call populateAttributes(childElement, childObject).

g. Call populateChildren(childElement, childObject) — recurse.

h. Attach child to parent using the appropriate add/set method (see 4.1.4).

```

  

#### Subtask 4.1.4 — Child acceptance registry (attaching children to parents)

  

The parent model knows how to accept children through its public API. The serializer must map tag names to the correct mutator method. The chosen approach is a **child acceptance registry** declared alongside the factory — all wiring in one place, models stay clean, serializer stays generic.

  

**Option C — Child acceptance registry (chosen):**

  

A separate registration map associates (parentType, childTag) → attachment lambda. The serializer looks up the rule and calls it. Models never know about the serializer; the serializer never knows about specific models.

  

```cpp

// Registry declaration

class XmlModelRegistry {

public:

using Factory = std::function<QObject*(QObject* parent)>;

using ChildRule = std::function<void(QObject* parent, QObject* child)>;

  

void addFactory(const QString& tag, Factory factory);

  

template<typename ParentType>

void addChildRule(const QString& childTag, std::function<void(ParentType*, QObject*)> rule);

  

QObject* create(const QString& tag, QObject* parent) const;

bool attachChild(QObject* parent, QObject* child, const QString& childTag) const;

};

```

  

```cpp

// Registration site (single file, e.g. TracklabelRegistry.cpp)

  

// Factories

registry.addFactory("tracklabel", [](QObject* p) { return new TrackLabelVariantModel(p); });

registry.addFactory("field", [](QObject* p) { return new TrackLabelFieldModel(p); });

// ... (see full list in subtask 4.2)

  

// Child rules — TrackLabelConfigModel accepts:

registry.addChildRule<TrackLabelConfigModel>("tracklabel",

[](TrackLabelConfigModel* p, QObject* c) {

p->addTracklabel( qobject_cast<TrackLabelVariantModel*>(c) );

});

registry.addChildRule<TrackLabelConfigModel>("extended-tracklabel",

[](TrackLabelConfigModel* p, QObject* c) {

p->setExtendedTracklabel(

std::unique_ptr<ExtendedTrackLabelModel>(qobject_cast<ExtendedTrackLabelModel*>(c)) );

});

registry.addChildRule<TrackLabelConfigModel>("column-0-actions",

[](TrackLabelConfigModel* p, QObject* c) {

p->setColumn0Actions(

std::unique_ptr<Column0ActionsModel>(qobject_cast<Column0ActionsModel*>(c)) );

});

  

// Child rules — TrackLabelModel (base for Variant/Extended) accepts:

registry.addChildRule<TrackLabelModel>("tracklabel-line",

[](TrackLabelModel* p, QObject* c) {

// TrackLabelModel owns FieldListModel rows

p->appendRow( qobject_cast<FieldListModel*>(c) );

});

  

// Child rules — FieldListModel accepts any FieldInterface subclass:

registry.addChildRule<FieldListModel>("field",

[](FieldListModel* p, QObject* c) {

p->insertField( p->fieldCount(), qobject_cast<FieldInterface*>(c) );

});

registry.addChildRule<FieldListModel>("field18",

[](FieldListModel* p, QObject* c) {

p->insertField( p->fieldCount(), qobject_cast<FieldInterface*>(c) );

});

registry.addChildRule<FieldListModel>("equipment-field",

[](FieldListModel* p, QObject* c) {

p->insertField( p->fieldCount(), qobject_cast<FieldInterface*>(c) );

});

  

// Child rules — FieldInterface accepts child elements:

registry.addChildRule<FieldInterface>("context-menu-item",

[](FieldInterface* p, QObject* c) {

p->addContextMenuItem( qobject_cast<ContextMenuItemModel*>(c) );

});

registry.addChildRule<FieldInterface>("visibility",

[](FieldInterface* p, QObject* c) {

p->setVisibility(

std::unique_ptr<FieldVisibilityModel>(qobject_cast<FieldVisibilityModel*>(c)) );

});

registry.addChildRule<FieldInterface>("edit",

[](FieldInterface* p, QObject* c) {

p->setEditSettings(

std::unique_ptr<FieldEditSettingsModel>(qobject_cast<FieldEditSettingsModel*>(c)) );

});

  

// Child rules — FieldVisibilityModel accepts show-if / hide-if:

registry.addChildRule<FieldVisibilityModel>("show-if",

[](FieldVisibilityModel* p, QObject* c) {

p->setShowIf( std::unique_ptr<VisibilityRuleModel>(qobject_cast<VisibilityRuleModel*>(c)) );

});

registry.addChildRule<FieldVisibilityModel>("hide-if",

[](FieldVisibilityModel* p, QObject* c) {

p->setHideIf( std::unique_ptr<VisibilityRuleModel>(qobject_cast<VisibilityRuleModel*>(c)) );

});

  

// Child rules — Column0ActionsModel:

registry.addChildRule<Column0ActionsModel>("action",

[](Column0ActionsModel* p, QObject* c) {

p->addAction( qobject_cast<ActionModel*>(c) );

});

```

  

**The serializer's `populateChildren` then becomes:**

```cpp

void populateChildren(XmlElement& element, QObject* parent, XmlModelRegistry& registry) {

for (auto child = element.firstChild(); ...; child = child.nextSibling()) {

QString tag = stripNamespace(child.localName());

QObject* childObj = registry.create(tag, parent);

if (!childObj) { logWarning(...); continue; }

  

populateAttributes(child, childObj);

populateChildren(child, childObj, registry); // recurse

  

if (!registry.attachChild(parent, childObj, tag)) {

logError("no child rule for '" + tag + "' under " + parent->metaObject()->className());

delete childObj;

}

}

}

```

  

**Why Option C:**

  

| Concern | Option A (convention) | Option B (acceptChild virtual) | Option C (child registry) |

|---|---|---|---|

| Models stay clean | ✅ | ❌ virtual on every model | ✅ |

| Serializer stays generic | ❌ infers method names | ✅ | ✅ |

| Resilient to schema changes | ❌ method renames break it | ✅ | ✅ one line update |

| All wiring visible in one place | ❌ scattered | ❌ scattered across models | ✅ single registry file |

  

#### Subtask 4.1.5 — Enum value conversion

  

Q_ENUM properties require special handling during deserialization. The serializer must:

  

1. Detect that a QMetaProperty's type is a registered Q_ENUM (via `QMetaType::flags() & QMetaType::IsEnumeration`, or check `QMetaEnum` existence on the enclosing metaObject).

2. Use `QMetaEnum::keyToValue()` to convert the XML string to the enum int value.

3. Handle XSD enum values that contain dots (e.g. `"CallsignMenu.Correlated"`) — the C++ enum key uses underscores (`CallsignMenu_Correlated`). The converter must try both the raw string and a dot-to-underscore variant.

4. Wrap the int in a `QVariant` via `QVariant::fromValue()` with the correct metatype, then `prop.write()`.

  

#### Subtask 4.1.6 — Full deserialization walk example

  

Given this XML:

```xml

<tracklabel-configuration path="/usr/share/polaris/.polaris-asd/" has-external-transfer="false">

<tracklabel tracklabel-type="Uncorrelated">

<tracklabel-line>

<field field-name="callsign" font-adjustment="+4" visible-in-holding="true">

<context-menu-item context-menu="CallsignMenu.Correlated" mouse="right"/>

<visibility when-unselected-for-control-states="Assumed Concerned">

<show-if flight-property="flightRule" property-value="IFR VFR"/>

</visibility>

</field>

</tracklabel-line>

</tracklabel>

<column-0-actions>

<action type="TransferAction"/>

</column-0-actions>

</tracklabel-configuration>

```

  

The walk proceeds:

```

1. deserialize(<tracklabel-configuration>, TrackLabelConfigModel*)

├── populateAttributes → path="/usr/share/polaris/.polaris-asd/", hasExternalTransfer=false

└── populateChildren:

├── <tracklabel> → factory creates TrackLabelVariantModel

│ ├── populateAttributes → tracklabelType=Uncorrelated

│ ├── populateChildren:

│ │ └── <tracklabel-line> → factory creates FieldListModel

│ │ └── populateChildren:

│ │ └── <field> → factory creates TrackLabelFieldModel

│ │ ├── populateAttributes:

│ │ │ ├── target whitelist: ["fieldName"] → fieldName=callsign (FieldType enum)

│ │ │ └── extensions:

│ │ │ ├── FieldText whitelist: ["prefix","placeholder"] → (not present, skip)

│ │ │ ├── FieldAppearance whitelist: ["toggleable","blinking","onlyShowOnFocus","visibleInHolding"]

│ │ │ │ → visibleInHolding=true

│ │ │ └── FieldLayout whitelist: ["fontAdjustment","fixedWidthInCharacters","leftMargin","bottomMargin"]

│ │ │ → fontAdjustment=+4

│ │ └── populateChildren:

│ │ ├── <context-menu-item> → factory creates ContextMenuItemModel

│ │ │ └── populateAttributes → contextMenu=CallsignMenu.Correlated, mouse=right

│ │ └── <visibility> → factory creates FieldVisibilityModel

│ │ ├── populateAttributes → whenUnselectedForControlStates="Assumed Concerned"

│ │ └── populateChildren:

│ │ └── <show-if> → factory creates VisibilityRuleModel

│ │ └── populateAttributes → flightProperty=flightRule, propertyValue="IFR VFR"

│ └── registry.attachChild(parent, TrackLabelVariantModel*, "tracklabel") → addTracklabel()

└── <column-0-actions> → factory creates Column0ActionsModel

├── populateChildren:

│ └── <action> → factory creates ActionModel

│ └── populateAttributes → type=TransferAction

└── registry.attachChild(parent, Column0ActionsModel*, "column-0-actions") → setColumn0Actions()

```

  

#### Subtask 4.1.7 — Error handling strategy

  

| Error | Behavior |

|---|---|

| Unknown element tag (not in factory) | Log warning, skip element and its subtree |

| Unknown attribute (not in any whitelist) | Log warning, skip attribute |

| Attribute value fails type conversion | Log warning with element+attribute+value, skip attribute (use default) |

| Required attribute missing (e.g. `field-name` on `<field>`) | Log error, still create object (model is in invalid state) |

| No child rule found in registry | Log error ("no child rule for 'X' under parent 'Y'"), delete child |

  

All errors are collected in a `QStringList` returned by `deserialize()` so the caller can display them. Deserialization is best-effort — it loads as much as possible and reports problems, rather than aborting on the first error.

  

### Subtask 4.2 — Tag→Factory Registry and Child Rules

  

Combined `XmlModelRegistry` — a `QHash<QString, Factory>` for tag→constructor and a map of `(parentMetaObject, childTag) → ChildRule` for attachment. See subtask 4.1.4 for the full registry design and all registrations.

  

Note: `show-if` and `hide-if` both create `VisibilityRuleModel` — the tag name determines which child rule is invoked on `FieldVisibilityModel`. The child rule for `"show-if"` calls `setShowIf()` and the rule for `"hide-if"` calls `setHideIf()`. This works naturally with the registry approach since each tag has its own lambda.

  

### Subtask 4.3 — Explicit property-to-attribute mapping (`xmlAttributeMap`)

  

Replace `xmlAttributeWhitelist() → QStringList` with `xmlAttributeMap() → QVector<std::pair<QString, QString>>` on `IXmlSerializable`. Each pair is `{propertyName, xmlAttributeName}` — no convention-based camelToKebab conversion.

  

```cpp

// IXmlSerializable:

using XmlAttributeMapping = QVector<std::pair<QString, QString>>;

virtual XmlAttributeMapping xmlAttributeMap() const = 0;

```

  

Example implementations:

```cpp

// TrackLabelFieldModel:

XmlAttributeMapping xmlAttributeMap() const override {

return { {"fieldName", "field-name"} };

}

  

// FieldText:

XmlAttributeMapping xmlAttributeMap() const override {

return { {"prefix", "prefix"}, {"placeholder", "placeholder"} };

}

  

// FieldAppearance:

XmlAttributeMapping xmlAttributeMap() const override {

return { {"toggleable", "toggleable"}, {"blinking", "blinking"},

{"onlyShowOnFocus", "only-show-on-focus"}, {"visibleInHolding", "visible-in-holding"} };

}

  

// FieldLayout:

XmlAttributeMapping xmlAttributeMap() const override {

return { {"fontAdjustment", "font-adjustment"}, {"fixedWidthInCharacters", "fixed-width-in-characters"},

{"leftMargin", "left-margin"}, {"bottomMargin", "bottom-margin"} };

}

  

// TrackLabelConfigModel:

XmlAttributeMapping xmlAttributeMap() const override {

return { {"path", "path"}, {"radioCallsignFilepath", "radio-callsign-filepath"},

{"hasExternalTransfer", "has-external-transfer"}, {"hasForceAct", "has-force-act"},

{"hasFlightRuleChange", "has-flight-rule-change"} };

}

```

  

The serializer uses the map in both directions:

- **Serialize:** read property by `propertyName`, write as `xmlAttributeName`

- **Deserialize:** match incoming XML attribute against `xmlAttributeName`, write to `propertyName`

  

This replaces the old `camelToKebab` / `kebabToCamel` utilities entirely.

  

### Subtask 4.4 — Pre-save validation gate

  

After serialization, call `SchemaValidator::validate()` on the final document. On failure, abort save and surface the error. This catches any edge case the model enforcement might miss.

  

### Subtask 4.5 — `asd.editor.xmlengine` redesign

  

The xmlengine library is **stateless and domain-agnostic**. It knows about `IXmlSerializable` and `XmlModelRegistry` (interfaces), not about tracklabel models specifically. Any future editor (sector config, etc.) reuses the same engine with its own registry.

  

#### What to keep

  

| Class | Role |

|---|---|

| `SchemaValidator` | Pre-load XSD validation. No changes needed. |

| `XmlEngineException.h` | Custom exceptions — keep. |

  

#### What to delete

  

- `SelectiveImporter` — replaced by `XmlModelSerializer::importFromFile()` (one-shot).

- `SelectiveExporter` — replaced by `XmlModelSerializer::exportToFile()` (from-scratch).

- `transformToGeneric()` — GenericNode is gone.

- `transformToNative()` — replaced by XmlModelSerializer::serialize.

- `Xml2Handle.h` — no raw libxml2 needed (no xmlReplaceNode).

- All `GenericTree.h` / `ConfigurationEnvelope` / `xmlinterface` references.

  

#### What to add

  

**`XmlModelSerializer`** — stateless, lives in xmlengine. Two methods:

  

```cpp

class XmlModelSerializer

{

public:

/** Serialize: walks IXmlSerializable tree → builds tern::xml DOM subtree.

* Returns the root XmlElement of the serialized subtree. */

static tern::xml::XmlElement serialize(

QObject* root,

tern::xml::XmlDocument& doc );

  

/** Deserialize: walks tern::xml DOM → populates pre-constructed model tree.

* Returns a list of warnings/errors encountered. */

static QStringList deserialize(

tern::xml::XmlElement& element,

QObject* root,

const XmlModelRegistry& registry );

};

```

  

**`XmlModelRegistry`** — lives in the **editor-specific lib** (e.g. `asd.editor.tracklabeleditor`), not in xmlengine. The engine takes it by const-reference. See subtask 4.1.4 for the full registry design.

  

#### Refactored flow

  

**Load (one-shot, document discarded after deserialization):**

```

1. Load XmlDocument from file.

2. Optionally validate via SchemaValidator.

3. XPath-select <instance id="polaris-asd-tracklabel">.

4. Navigate to the config element: instance → child "configuration" → child "tracklabel-configuration".

5. XmlModelSerializer::deserialize(configElement, configModel, tracklabelRegistry)

→ populates the pre-constructed TrackLabelConfigModel

→ returns QStringList of warnings.

6. XmlDocument goes out of scope — DOM memory freed. Model tree is the source of truth.

```

  

**Save (create from scratch, write to new file):**

```

1. Create a fresh XmlDocument.

2. Build the envelope:

<instances xmlns:tlabel="http://tern.is/polaris-asd/tracklabel">

<instance id="polaris-asd-tracklabel" class-mapping-reference="TrackLabelConfig">

<tlabel:configuration>

3. XmlModelSerializer::serialize(configModel, doc)

→ builds XmlElement subtree from model tree (root = <tlabel:tracklabel-configuration>)

4. Append serialized subtree as child of <tlabel:configuration>.

5. Optionally SchemaValidator::validate(doc) — pre-save gate.

6. doc.save(destinationPath, true) — formatted output.

```

  

The envelope metadata (namespace URI, prefix, instance id, class-mapping-reference) is passed to the exporter as parameters or a small config struct.

  

#### Importer — one-shot deserializer

  

```cpp

/** One-shot: loads file, finds instance, deserializes into model, discards document.

* Returns warnings. */

static QStringList importFromFile(

const std::string& xmlFilePath,

const std::string& instanceId,

QObject* root,

const XmlModelRegistry& registry,

const SchemaValidator* validator = nullptr );

```

  

#### Exporter — from-scratch serializer

  

```cpp

/** Envelope metadata for creating the <instances>/<instance> wrapper. */

struct XmlEnvelopeConfig {

std::string namespaceUri; // e.g. "http://tern.is/polaris-asd/tracklabel"

std::string namespacePrefix; // e.g. "tlabel"

std::string instanceId; // e.g. "polaris-asd-tracklabel"

std::string classMappingRef; // e.g. "TrackLabelConfig"

std::string configElementName; // e.g. "configuration"

};

  

/** Creates a fresh document, wraps in envelope, serializes model, saves. */

static void exportToFile(

QObject* root,

const XmlEnvelopeConfig& envelope,

const std::string& destinationPath,

const SchemaValidator* validator = nullptr );

```

  

#### XmlModelSerializer::serialize algorithm (detailed)

  

```

serialize(root, doc):

1. Cast root to IXmlSerializable*. Get xmlTagName().

xmlTagName() already returns the XML tag name (e.g. "tracklabel-configuration").

XmlElement elem = doc.createElement(root->xmlTagName())

  

2. Write attributes from root's xmlAttributeMap():

For each {propertyName, xmlAttrName} pair:

QMetaProperty prop = find propertyName on root->metaObject()

QVariant value = prop.read(root)

std::string xmlValue = variantToString(value) // handles bool, int, enum, string

elem.setAttribute(xmlAttrName, xmlValue)

  

3. Write attributes from serializationExtensions():

For each extension in root->serializationExtensions():

Cast to IXmlSerializable*, get its xmlAttributeMap()

For each {propertyName, xmlAttrName}:

same as step 2, but read from extension's QMetaObject

  

4. Recurse into xmlChildren():

For each child in root->xmlChildren():

XmlElement childElem = serialize(child, doc) // recurse

elem.appendChild(childElem)

  

5. Return elem.

```

  

#### File layout after refactor

  

```

asd.editor.xmlengine/

├── CMakeLists.txt

├── SchemaValidator.h/cpp ← kept, no changes

├── XmlModelSerializer.h/cpp ← NEW: generic serialize/deserialize

├── XmlModelRegistry.h ← NEW: interface (factory + child rules types)

├── XmlEngineException.h ← kept

└── XmlEngine.h ← updated module doc comment

  

asd.editor.tracklabeleditor/

└── serialization/

└── TracklabelRegistry.h/cpp ← NEW: editor-specific factory + child rules

```

  

---

  

## Task 5 — Refactor Existing Code

  

### Subtask 5.1 — Create `TrackLabelConfigModel` as root

  

This replaces the hash map in `TrackLabelEditorController`. Instead of `QHash<LabelType, TrackLabelModel*>`, the controller owns a single `TrackLabelConfigModel` whose children are the actual `TrackLabelVariantModel` instances keyed by their `tracklabel-type` attribute.

  

### Subtask 5.2 — Update `TrackLabelEditorController`

  

- Replace `QHash<LabelType, TrackLabelModel*> m_layouts` with `TrackLabelConfigModel* m_config`.

- `currentLayout()` returns the `TrackLabelVariantModel*` whose `tracklabelType` matches `m_activeType`.

- Add save/load methods that use `XmlModelSerializer`.

  

### Subtask 5.3 — Update QML bindings

  

All QML files referencing old role names or model types must be updated. The `FieldObjectRole` now returns `FieldInterface*`, and QML should use `fieldObject.fieldName`, `fieldObject.prefix`, etc. directly.

  

---

  

## Task 6 — Schema Value Loading for UI Dropdowns

  

### Subtask 6.1 — Enum value providers

  

For each schema-derived enum, provide a static `QStringList allValues()` that QML ComboBox models can bind to:

  

```cpp

namespace tracklabel_types {

QStringList allFieldTypes(); // 60+ values from FieldType

QStringList allContextMenuTypes(); // 16 values

QStringList allField18Types(); // 22 values

QStringList allEquipmentTypes(); // 10 values

QStringList allFontSizeDeltas(); // 7 values

// ...

}

```

  

### Subtask 6.2 — Register as QML singletons or context properties

  

Expose these lists so QML combo boxes can do:

```qml

ComboBox {

model: TrackLabelTypes.allFieldTypes()

}

```

  

---

  

## Task 7 — Testing

  

### Subtask 7.1 — Round-trip serialization tests

  

For each model class, write a test that:

1. Constructs the model with known values.

2. Serializes to XML via `XmlModelSerializer`.

3. Validates the XML against the XSD.

4. Deserializes back into a fresh model.

5. Asserts all values match.

  

### Subtask 7.2 — Cardinality enforcement tests

  

- Test that `FieldVisibilityModel` rejects setting both `showIf` and `hideIf`.

- Test that `Column0ActionsModel::removeRows()` refuses to remove the last item.

- Test that `TrackLabelEquipmentFieldModel::removeRows()` refuses to remove the last item.

- Test that `TrackLabelConfigModel` refuses a second `extended-tracklabel`.

- Test that `GenericFieldModel::setFontAdjustment()` rejects invalid deltas.

- Test that `TrackLabelFieldModel` requires a valid `FieldType` for `field-name`.

- Test that setting `only-show-on-focus=true` on `FieldInterface` clears any existing visibility child.

  

### Subtask 7.3 — Enum completeness tests

  

For each enum, write a test that parses the XSD file, extracts enumeration values, and asserts they match the C++ enum's `allValues()` list. This catches schema-vs-code drift at test time.

  

---

  

## Task 8 — File & Directory Structure

  

Target layout after refactoring:

  

```

source/apps/track-label-editor/libs/

├── asd.editor.commontypes/

│ ├── app_types.h/cpp

│ ├── flight_types.h/cpp // ASD internal types (ControlState etc.)

│ ├── tracklabel_types.h/cpp // ✅ All XSD enums + ControlTypeList

│ ├── shared_types.h/cpp // ✅ XSD stringList type (StringList)

│ └── IXmlSerializable.h // ✅ Serialization interface (xmlTagName defaults nullptr)

│

├── asd.editor.xmlengine/ // Refactored

│ ├── SchemaValidator.h/cpp // ✅ Pre-load XSD validation

│ ├── XmlModelSerializer.h/cpp // Generic serialize/deserialize + import/export

│ ├── XmlModelRegistry.h // Interface: factory + child rules types

│ └── XmlEngineException.h // ✅ Custom exceptions

│

├── asd.editor.tracklabelfield/ // Refactored

│ ├── FieldInterface.h/cpp // ✅ Abstract base (TrackLabelGenericFieldType)

│ ├── TrackLabelFieldModel.h/cpp // ✅ Concrete: <field field-name="...">

│ ├── TrackLabelField18Model.h/cpp // Not yet created

│ ├── TrackLabelEquipmentFieldModel.h/cpp // Not yet created

│ ├── ContextMenuItemModel.h/cpp // ✅

│ ├── FieldVisibilityModel.h/cpp // ✅ (variant + unique_ptr)

│ ├── VisibilityRuleModel.h/cpp // ✅

│ ├── FieldEditSettingsModel.h/cpp // ✅

│ ├── Field18ItemModel.h/cpp // ✅

│ ├── EquipmentItemModel.h/cpp // ✅

│ ├── ActionModel.h/cpp // ✅

│ ├── FieldText.h/cpp // ✅ Sub-object (IXmlSerializable, prefix+placeholder)

│ ├── FieldAppearance.h/cpp // ✅ Sub-object (IXmlSerializable, toggleable+blinking+...)

│ └── FieldLayout.h/cpp // ✅ Sub-object (IXmlSerializable, fontAdjustment+margins)

│

├── asd.editor.tracklabeleditor/

│ ├── models/

│ │ ├── TrackLabelConfigModel.h/cpp // New: root (TrackLabelListType)

│ │ ├── TrackLabelModel.h/cpp // Refactored: base (TrackLabelGenericType)

│ │ ├── TrackLabelVariantModel.h/cpp // New: (TrackLabelType)

│ │ ├── ExtendedTrackLabelModel.h/cpp // New: (ExtendedTrackLabelType)

│ │ ├── FieldListModel.h/cpp // Kept (maps to TrackLabelLineType)

│ │ └── Column0ActionsModel.h/cpp // New

│ └── controllers/

│ └── TrackLabelEditorController.h/cpp // Updated

```

  

---

  

## Execution Order

  

1. **Task 1** (Enum Types) — no dependencies, unlocks everything else.

2. **Task 2.1** (IXmlSerializable interface) — needed before any model class.

3. **Task 2.2** (Leaf models) — independent of each other, can be done in parallel.

4. **Task 2.3** (Field hierarchy) — depends on leaf models (ContextMenuItemModel, etc.).

5. **Task 2.4** (Container models) — depends on field hierarchy.

6. **Task 3** (Cardinality enforcement) — integrated into Task 2, listed separately for review.

7. **Task 4.3** (xmlAttributeMap on IXmlSerializable) — update interface + all model classes.

8. **Task 4.5** (xmlengine redesign) — delete old importer/exporter, clean up.

9. **Task 4.2** (XmlModelRegistry) — factory + child rules, editor-specific.

10. **Task 4.1** (XmlModelSerializer) — serialize + deserialize engine.

11. **Task 4.4** (Pre-save validation) — wire SchemaValidator into export flow.

12. **Task 5** (Refactor existing code) — depends on Tasks 2+3+4.

13. **Task 6** (Schema value loading for UI) — depends on Task 1.

14. **Task 7** (Testing) — final validation pass.
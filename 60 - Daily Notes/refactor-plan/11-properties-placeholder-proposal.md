# Properties Placeholder System — Proposal

## Context

The polaris-asd runtime uses a `.properties` file system for configuration placeholders:
```properties
pasd.tar.tracklabel.unselected.relevant.cs=Concerned TransferInInitiated RequestInInitiated Assumed TransferOutInitiated RequestOutInitiated
```

The XML config references these as `${pasd.tar.tracklabel.unselected.relevant.cs}` in attribute values.
At runtime, `PropertyPlaceholderReplacer` (from `tern::runtime`) resolves them before parsing.

The editor needs to support this same system for:
1. Editing existing placeholder-backed values
2. Creating new placeholder groupings
3. Round-tripping placeholders (load XML with `${...}` → edit → save XML with `${...}` intact)

## Application Startup

```json
"args": [
    "-P",
    "/usr/share/polaris/config/tracklabel-configuration.properties,/usr/share/polaris/configuration.properties,tracklabel-editable.properties",
    "-C",
    "/usr/share/polaris/config/tracklabel-editor-class-mappings.xml"
]
```

The `-P` flag provides comma-separated `.properties` files.
`tracklabel-editable.properties` is the NEW editor-specific file that users can modify.

## The Core Problem

The XSD validator cannot validate placeholder strings (`${...}`). So:
1. Raw XML with placeholders → XSD validation FAILS
2. Resolved XML (placeholders replaced) → XSD validation PASSES
3. But we LOSE the placeholder names after resolution

We need both: validated model tree AND placeholder names preserved per attribute.

## Proposed Solution: Dual-Pass Deserialization

### Load Flow

```
1. Read raw XML (contains ${placeholder} strings)
2. Create a resolved copy: PropertyPlaceholderReplacer resolves all ${...}
3. Validate the resolved copy against XSD → fail = abort
4. Deserialize resolved copy into model tree (normal path, all values are real)
5. Walk resolved model tree alongside raw XML: for each attribute,
   if raw XML value matches placeholder regex /\$\{[^}]+\}/,
   attach PlaceholderMetadata to that model property
```

### PlaceholderMetadata

```cpp
struct PlaceholderMetadata {
    QString placeholderKey;   // e.g. "pasd.tar.tracklabel.unselected.relevant.cs"
    QString resolvedValue;    // e.g. "Concerned TransferInInitiated ..."
};
```

Attached to a Q_PROPERTY value. When the user edits the value:
- If editing the resolved value directly → placeholder is detached (becomes raw value)
- If editing via placeholder UI → updates the `.properties` mapping

### Save Flow

```
1. For each attribute in the model tree:
   - If PlaceholderMetadata exists → serialize as ${placeholderKey}
   - If no metadata → serialize the raw Q_PROPERTY value
2. Write XML
3. Write updated tracklabel-editable.properties (new/modified entries)
```

## System Requirement: Context-Aware Placeholder Binding

A placeholder key maps to a **specific model class + property**, not just "any property of that type."

### Example

```
Tree:
├── Instance_1 (ModelA, has ControlTypeList font_size)
├── Instance_2 (ModelA, has ControlTypeList font_size)
├── Instance_3 (ModelB, has ControlTypeList font_size)
└── Instance_4 (ModelC, has QString name)
```

Runtime input: `my_control_states : Concerned Assumed`

Enforcement:
- ✅ Instance_1 (ModelA) updates — key `my_control_states` is REGISTERED for ModelA
- ✅ Instance_2 (ModelA) updates — same registration
- ❌ Instance_3 (ModelB) ignores — key is NOT registered for ModelB
- ❌ Instance_4 (ModelC) ignores — complete type/key mismatch

### Implementation: Class-Property Registry

```cpp
/// Maps a placeholder key to the class+property it targets
struct PlaceholderBinding {
    QString placeholderKey;       // "pasd.tar.tracklabel.unselected.relevant.cs"
    const QMetaObject* targetClass;  // &FieldVisibilityModel::staticMetaObject
    QString targetProperty;      // "whenUnselectedForControlStates"
};

class PlaceholderRegistry {
    QHash<QString, PlaceholderBinding> m_bindings;

    /// Returns true if this key is allowed to modify this class+property
    bool canApply(const QString& key, const QMetaObject* classType, const QString& property) const;
};
```

The registry is built during the dual-pass deserialization (step 5) — wherever we find
a placeholder in the raw XML, we record WHICH model class and property it was on.

### How per-placeholder changes work

When user edits a placeholder value (e.g., adds "NonConcerned" to the control states list):
1. UI triggers: `IOController.updatePlaceholderValue("pasd.tar...cs", "Concerned NonConcerned ...")`
2. IOController updates `m_placeholderMap["pasd.tar...cs"]`
3. IOController iterates all model instances:
   - For each instance, check `PlaceholderRegistry::canApply(key, instance->metaObject(), property)`
   - If allowed → update that property with the new resolved value
   - If blocked → skip
4. Trigger visibility re-evaluation

## Where This Lives

```
TrackLabelIOController
├── owns: PlaceholderRegistry
├── owns: QHash<QString, QString> m_placeholderMap  (key → resolved value)
├── owns: unique_ptr<TrackLabelConfigModel> m_configModel
│
├── loadConfiguration():
│   - reads raw XML
│   - resolves placeholders (using m_placeholderMap from .properties files)
│   - validates resolved copy
│   - deserializes into model tree
│   - builds PlaceholderRegistry from raw XML vs. resolved tree
│
├── saveConfiguration():
│   - serializes model tree
│   - re-inserts ${...} for attributes with PlaceholderMetadata
│   - writes XML + updated .properties
│
├── loadProperties(filePath):
│   - parses .properties file into m_placeholderMap
│
├── updatePlaceholderValue(key, newValue):
│   - updates m_placeholderMap
│   - applies to matching model instances (via registry)
│   - triggers re-evaluation
```

## Verification: PropertyPlaceholderReplacer compatibility

Need to verify (branch `tasks/DEV-8825`):
- Does `PropertyPlaceholderReplacer` from `tern::runtime` accept the `-P` file list format?
- Does it support loading our new `tracklabel-editable.properties` from that list?
- Does it handle multiple files with same-key overrides (last wins)?

## Phasing

| Phase | What |
|-------|------|
| Now (this refactor) | IOController created with load/save. No placeholder support yet. |
| Phase 2 | Add `loadProperties()`, `m_placeholderMap`, resolve before validate. |
| Phase 3 | Dual-pass deserialization, PlaceholderRegistry, PlaceholderMetadata on properties. |
| Phase 4 | UI for editing placeholder values, `updatePlaceholderValue()`. |
| Phase 5 | Save flow with placeholder reinsertion + .properties export. |

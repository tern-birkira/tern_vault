# Runtime Field Data (dummyData) — Proposal

## Context

`FieldInterface::dummyData()` is a `QString` Q_PROPERTY that serves as:
1. **Preview text** — shown in the track label view as sample field content
2. **show-if/hide-if evaluation** — the visibility engine uses `dummyData()` as the field's
   current value when evaluating conditions (via `isConditionFulfilled(m_dummyData, lookup)`)

`dummyData` is **NOT serialized** — it's purely app-side runtime data. It already has a
`setDummyData()` setter. It is not part of the XML config at all.

The cross-field lookup (`FlightPropertyLookup`) resolves to another field's `dummyData()`
by FieldType, scanning all fields in the current layout via `TrackLabelModel::findFieldDummyData()`.

## Problem

`dummyData` is currently declared as `Q_PROPERTY(QString dummyData READ dummyData CONSTANT)`.
The CONSTANT means QML never re-reads it after binding. For runtime simulation (Snorri's test module),
we need to be able to change field values externally and have the UI + visibility update.

## Solution: Make dummyData a read/write NOTIFY property

Change from:
```cpp
Q_PROPERTY(QString dummyData READ dummyData CONSTANT)
```

To:
```cpp
Q_PROPERTY(QString dummyData READ dummyData WRITE setDummyData NOTIFY dummyDataChanged)
```

That's it. The setter already exists. Just add the signal and wire it.

### Changes to FieldInterface

```cpp
// FieldInterface.h
Q_PROPERTY(QString dummyData READ dummyData WRITE setDummyData NOTIFY dummyDataChanged)

signals:
    void visibilityChanged();
    void dummyDataChanged();  // NEW

// FieldInterface.cpp
void FieldInterface::setDummyData(const QString& dummyData)
{
    if (m_dummyData != dummyData)
    {
        m_dummyData = dummyData;
        emit dummyDataChanged();  // NEW
    }
}
```

### How runtime import sets field values

```
IOController.importRuntimeValues(source)
  → parses key-value pairs (FieldType → value string)
  → calls runtimeDataController->setFieldValueOverrides(map)

RuntimeDataController emits changed()
  → DataController::reevaluateActiveLayoutVisibility()
    → iterates all fields in layout
    → for each field: if override exists for its FieldType, call field->setDummyData(override)
    → then call field->evaluateVisibility(...)
    → QML updates automatically via dummyDataChanged signal
```

### Override map lives on RuntimeDataController

```cpp
// TrackLabelRuntimeDataController (future addition)
Q_PROPERTY(QVariantMap fieldValueOverrides READ fieldValueOverrides WRITE setFieldValueOverrides NOTIFY changed)
```

Key = FieldType (as int), Value = QString.
When set, `changed()` fires → DataController iterates fields, pushes values via `setDummyData()`,
then re-evaluates visibility.

When cleared (empty map), DataController can restore defaults from a built-in lookup
(`commontypes::tracklabel::fieldToDummyData()` already exists for this).

### FieldListModel DummyDataRole

Already uses `field->dummyData()` — no change needed. The `setData` path for
`DummyDataRole` already calls `field->setDummyData(value.toString())`.
With the NOTIFY signal, QML delegates will auto-update.

## Summary

| What | Change |
|------|--------|
| `FieldInterface.h` | `CONSTANT` → `WRITE setDummyData NOTIFY dummyDataChanged` |
| `FieldInterface.h` | Add `void dummyDataChanged()` signal |
| `FieldInterface.cpp` | `setDummyData` emits `dummyDataChanged()` |
| `RuntimeDataController` (future) | Add `fieldValueOverrides` QVariantMap |
| `DataController::reevaluate` (future) | Push overrides into fields via `setDummyData` |

Minimal change. No new abstractions. The setter already exists.

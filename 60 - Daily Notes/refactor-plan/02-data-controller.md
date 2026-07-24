# TrackLabelDataController

## Responsibility

Owns the tracklabel editing model. Manages active type, layouts, field selection,
field mutation, and visibility evaluation. Reads runtime state from RuntimeDataController
to perform visibility calculations.

## Namespace

`asd::editor::tracklabeleditor::controllers`

## File Location

`controllers/TrackLabelDataController.h`
`controllers/TrackLabelDataController.cpp`

## Interface

```cpp
class TrackLabelDataController : public QObject
{
Q_OBJECT
Q_PROPERTY(commontypes::tracklabel::TrackLabelTypeVariant activeType
           READ activeType WRITE setActiveType NOTIFY activeTypeChanged)
Q_PROPERTY(models::TrackLabelModel* currentLayout READ currentLayout NOTIFY currentLayoutChanged)
Q_PROPERTY(models::ExtendedTrackLabelModel* extendedLayout READ extendedLayout NOTIFY extendedLayoutChanged)
Q_PROPERTY(models::Column0ActionsModel* column0ActionsLayout READ column0ActionsLayout NOTIFY column0ActionsLayoutChanged)
Q_PROPERTY(tracklabelfield::interface::FieldInterface* selectedField READ selectedField NOTIFY selectedFieldChanged)

public:
    explicit TrackLabelDataController(
        std::shared_ptr<TrackLabelRuntimeDataController> runtimeData,
        QObject* parent = nullptr);

    // --- Getters ---
    commontypes::tracklabel::TrackLabelTypeVariant activeType() const;
    models::TrackLabelModel* currentLayout() const;
    models::ExtendedTrackLabelModel* extendedLayout() const;
    models::Column0ActionsModel* column0ActionsLayout() const;
    tracklabelfield::interface::FieldInterface* selectedField() const;

    // --- Setters ---
    void setActiveType(commontypes::tracklabel::TrackLabelTypeVariant type);

    // --- Model operations ---
    void syncViewLayoutMappings(models::TrackLabelConfigModel* config);
    Q_INVOKABLE void selectField(int rowIndex, int colIndex);
    Q_INVOKABLE void clearSelection();

public slots:
    void addFieldToCurrentLayout(int rowIndex, const models::FieldData& field);
    void reevaluateActiveLayoutVisibility();  // parameterless — reads from m_runtimeData

signals:
    void activeTypeChanged();
    void currentLayoutChanged();
    void extendedLayoutChanged();
    void column0ActionsLayoutChanged();
    void selectedFieldChanged();

private:
    std::shared_ptr<TrackLabelRuntimeDataController> m_runtimeData;

    commontypes::tracklabel::TrackLabelTypeVariant m_activeType = commontypes::tracklabel::TrackLabelTypeVariant::Uncorrelated;
    QHash<commontypes::tracklabel::TrackLabelTypeVariant, models::TrackLabelModel*> m_layouts;
    models::ExtendedTrackLabelModel* m_extendedLayout = nullptr;
    models::Column0ActionsModel* m_column0ActionsLayout = nullptr;
    tracklabelfield::interface::FieldInterface* m_selectedField = nullptr;
};
```

## Key Design Decisions

### Parameterless `reevaluateActiveLayoutVisibility()`
Reads from `m_runtimeData->activeVisibility()`, `m_runtimeData->activeControlType()`, and
`m_runtimeData->activeVisibleInHolding()` directly.
This enables a clean `connect(runtimeData, &changed, this, &reevaluate)` — no signal parameters needed.

### Uses `ControlType` not `ControlState`
The runtime data controller provides `ControlType` (XSD-native enum from `TrackLabelTypes.h`).
`evaluateVisibility` on FieldInterface will be updated to accept `ControlType` directly,
eliminating the `toControlType()` converter and all `FlightTypes.h` dependencies.

### `activeVisibleInHolding` passed to evaluateVisibility
When true, fields with `visibleInHolding=true` bypass show-on-focus and control-state rules.
When false (default), only the field's own `visibleInHolding` attribute matters (always-show bypass).

### shared_ptr to RuntimeDataController
Follows project convention (InspectorDataController stores shared_ptr<MapLayerModel>).
Lifetime guaranteed by Instantiator owning both.

### Extracts directly from current TrackLabelEditorController
- `setActiveType` → same logic (emit, emit currentLayoutChanged, clearSelection, reevaluate)
- `currentLayout()` → same (hash lookup)
- `selectField` / `clearSelection` → same
- `addFieldToCurrentLayout` → same
- `syncViewLayoutMappings` → same (but no longer calls load-related signals)
- `reevaluateActiveLayoutVisibility` → same body, but reads from m_runtimeData instead of local members

## Dependencies

- `shared_ptr<TrackLabelRuntimeDataController>` (read-only, injected)
- `asd.editor.tracklabelfield` (FieldInterface, FieldVisibilityModel)
- `asd.editor.commontypes/TrackLabelTypes.h` (ControlType, VisibilityState, FieldType, etc.)
- **NO dependency on FlightTypes.h**
- Models: TrackLabelModel, FieldListModel, ExtendedTrackLabelModel, Column0ActionsModel, TrackLabelConfigModel, TrackLabelVariantModel

## Implementation Steps

1. Create header — extract declarations from TrackLabelEditorController
2. Create cpp — move method bodies, replace `m_activeVisibility` reads with `m_runtimeData->activeVisibility()`
3. reevaluateActiveLayoutVisibility becomes a public slot (was private)
4. setActiveType calls reevaluateActiveLayoutVisibility at end (same as before)
5. syncViewLayoutMappings calls reevaluateActiveLayoutVisibility at end (same as before)

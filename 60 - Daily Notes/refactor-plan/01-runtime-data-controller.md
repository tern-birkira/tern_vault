# TrackLabelRuntimeDataController

## Responsibility

Pure state container for simulated runtime conditions.
These are the "knobs" that affect field visibility from the runtime perspective.
Fully standalone — no dependency on any other controller.

## Namespace

`asd::editor::tracklabeleditor::controllers`

## File Location

`controllers/TrackLabelRuntimeDataController.h`
`controllers/TrackLabelRuntimeDataController.cpp`

## Interface

```cpp
class TrackLabelRuntimeDataController : public QObject
{
Q_OBJECT
Q_PROPERTY(commontypes::tracklabel::VisibilityState activeVisibility
           READ activeVisibility WRITE setActiveVisibility NOTIFY changed)
Q_PROPERTY(commontypes::tracklabel::ControlType activeControlType
           READ activeControlType WRITE setActiveControlType NOTIFY changed)
Q_PROPERTY(bool activeVisibleInHolding
           READ activeVisibleInHolding WRITE setActiveVisibleInHolding NOTIFY changed)

public:
    explicit TrackLabelRuntimeDataController(QObject* parent = nullptr);

    commontypes::tracklabel::VisibilityState activeVisibility() const;
    void setActiveVisibility(commontypes::tracklabel::VisibilityState state);

    commontypes::tracklabel::ControlType activeControlType() const;
    void setActiveControlType(commontypes::tracklabel::ControlType type);

    bool activeVisibleInHolding() const;
    void setActiveVisibleInHolding(bool holding);

signals:
    void changed();  // Emitted on ANY property change

private:
    commontypes::tracklabel::VisibilityState m_activeVisibility = commontypes::tracklabel::VisibilityState::Normal;
    commontypes::tracklabel::ControlType m_activeControlType = commontypes::tracklabel::ControlType::NotSet;
    bool m_activeVisibleInHolding = false;
};
```

## Design Notes

- **Uses `ControlType` (from TrackLabelTypes.h), NOT `ControlState` (from FlightTypes.h).**
  `ControlType` is the XSD-native enum. `ControlState` was a duplicate with different sentinels.
  Using `ControlType` directly eliminates the `toControlType()` converter entirely.
- Single `changed()` signal instead of per-property signals.
  Reason: the downstream consumer (DataController::reevaluateActiveLayoutVisibility) always
  needs a full re-evaluation regardless of which property changed. One signal = one connect.
- Setters still guard against no-op (same value → no emit).
- QML can write directly: `Instantiator.runtimeDataController.activeVisibility = TrackLabelTypes.Hover`
- `activeVisibleInHolding` simulates "is the aircraft currently in holding?" — when true,
  fields with `visible-in-holding=true` bypass the only-show-on-focus and control-state rules.

## Future Extensions (not implemented now)

- `Q_PROPERTY(QVariantMap fieldValueOverrides)` — override map for show-if/hide-if field values
  (see `10-runtime-field-data-proposal.md`)
- These will be populated by `IOController.importRuntimeValues()` when that feature lands.

## Dependencies

- `asd.editor.commontypes/TrackLabelTypes.h` (VisibilityState, ControlType enums)
- **NO dependency on FlightTypes.h** — that file may be removed in the future.
- No other controller dependencies.

## Implementation Steps

1. Create header with class declaration, Q_PROPERTYs, signals
2. Create cpp with constructor + getter/setter implementations
3. Each setter: guard same-value, set member, emit changed()

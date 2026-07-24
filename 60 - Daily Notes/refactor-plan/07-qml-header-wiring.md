# Wire QML Headers to RuntimeDataController

## Problem (pre-existing gap)

`DisplayStatesHeader.qml` and `ControlStatesHeader.qml` use local QML properties
(`property string activeView: "Normal"`, `property string activeControlState: "None"`)
that NEVER reach C++. Tab clicks only change local visual state — visibility logic
in C++ never fires from user interaction with these headers.

## Fix

Bind these QML components to `TrackLabelEditorInstantiator.runtimeDataController`.

## DisplayStatesHeader.qml

Currently:
```qml
property string activeView: "Normal"

TabButton { onClicked: displayStatesHeaderId.activeView = "Normal" }
TabButton { onClicked: displayStatesHeaderId.activeView = "Hover" }
TabButton { onClicked: displayStatesHeaderId.activeView = "Active" }
```

After:
```qml
property var runtimeController: TrackLabelEditorInstantiator.runtimeDataController

// Map string → enum. Could be done via a helper or directly.
TabButton {
    onClicked: runtimeController.activeVisibility = TrackLabelTypes.Normal
}
TabButton {
    onClicked: runtimeController.activeVisibility = TrackLabelTypes.Hover
}
TabButton {
    onClicked: runtimeController.activeVisibility = TrackLabelTypes.Completed
}
```

The `isActive` binding reads from the controller property:
```qml
isActive: runtimeController.activeVisibility === TrackLabelTypes.Normal
```

Note: "Active" in the UI maps to `VisibilityState::Completed` in the enum.
Verify this mapping against TrackLabelTypes.h.

## ControlStatesHeader.qml

Currently:
```qml
property string activeControlState: "None"

TabButton { onClicked: controlStatesHeaderId.activeControlState = "No Control Relationship" }
TabButton { onClicked: controlStatesHeaderId.activeControlState = "Active/Notable Presence" }
TabButton { onClicked: controlStatesHeaderId.activeControlState = "Transfer- and Request" }
```

After:
```qml
property var runtimeController: TrackLabelEditorInstantiator.runtimeDataController

TabButton {
    onClicked: runtimeController.activeControlState = FlightTypes.Unconcerned  // or appropriate enum
}
// etc.
```

String-to-enum mapping needs careful alignment with `commontypes::flight::ControlState`.
The current categories are groups, not individual states. May need:
- A Q_INVOKABLE helper on RuntimeDataController that accepts the group string
- Or: expose a `controlStateCategory` property that maps internally

## Mapping: UI Button Groups → ControlState enum

| Button text | ControlState values |
|-------------|-------------------|
| "No Control Relationship" | `Unconcerned` |
| "Active/Notable Presence" | `Assumed`, `Concerned` |
| "Transfer- and Request" | `TransferInInitiated`, `TransferOutInitiated`, `RequestInInitiated`, `RequestOutInitiated` |

This is a UI grouping problem — each button represents a CATEGORY of states.
The simplest approach: add a `Q_INVOKABLE void setControlStateCategory(const QString& category)`
on RuntimeDataController that maps category string → a representative ControlState value
(or the first value in the group).

Alternatively: expose a `controlStateCategory` string Q_PROPERTY that the header binds to,
and RuntimeDataController internally maps it to the appropriate enum for visibility evaluation.

## Canvas.qml

Currently passes `trackLabelEditorController` to TrackLabelView:
```qml
trackLabelController: TrackLabelEditorInstantiator.trackLabelEditorController
```

After:
```qml
trackLabelController: TrackLabelEditorInstantiator.dataController
```

## Implementation Steps

1. Identify exact enum mapping for display states
2. Identify control state grouping strategy
3. Update DisplayStatesHeader.qml bindings
4. Update ControlStatesHeader.qml bindings
5. Update Canvas.qml (trackLabelController → dataController)
6. Update Editor.qml (clearSelection reference)
7. Update TrackLabelView.qml if needed

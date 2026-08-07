  

# Track label field visibility rules

  

Fact-check and corrected reference for how a `field` (`TrackLabelFieldType`) decides

whether it is shown, based on `schemas/tracklabel.xsd` and the runtime behaviour in

`polaris-asd` (`source/libs/asd.track.label/qml/TrackLabelField.qml` and

`StaticTrackLabelField.qml` / `FreeTextField.qml` / `FreeTextFieldReadOnly.qml`).

  

## Fact-check: "connect activeVisibilityChanged to reevaluateActiveLayoutVisibility as a slot"

  

**False / unnecessary.** `TrackLabelEditorController::setActiveVisibility()` (and

`setActiveType()`, `setActiveControlState()`, `syncViewLayoutMappings()`) already call

`reevaluateActiveLayoutVisibility()` **synchronously and directly**, right after emitting

the `*Changed` signal (`TrackLabelEditorController.cpp:54-63`). Wiring

`activeVisibilityChanged` to the same slot via `connect()` would run the reevaluation

**twice per change** for no benefit.

  

The actual bug: `FieldListModel::evalVisibility()` (`FieldListModel.cpp:418-425`) is an

empty stub ("To be implemented"), and `FieldInterface::evaluateVisibility()`

(`FieldInterface.cpp:195-212`) only handles `only-show-on-focus`. The signal/slot wiring

was never the missing piece — the evaluation logic itself was never written.

  

## Corrected layered rules

  

The original four-layer description mixes up **precedence** and **conditions**. The real

precedence, taken from `StaticTrackLabelField.qml:31`

(`showField: isVisibilityConditionFulfilled && ( visibleWhenSelected || visibleWithoutSelection )`)

and `TrackLabelField.qml:81-93` (`shouldBeVisibleWithoutSelection`), is:

  

### Layer 0 — `show-if` / `hide-if` (gates *everything*, evaluated first)

  

Contrary to the original write-up, this is not a "fourth, independent" layer — it is

checked **before and above** focus/selection, and a failed condition hides the field even

while selected or hovered. Source: `isVisibilityConditionFulfilled` is `&&`-ed with the

selected/unselected branches, not applied only to one of them.

  

- `flight-property` optional. If absent, the XSD says (`tracklabel.xsd:227`) evaluate

against "the value of the track label field containing the condition" — i.e. the

field's own value.

- If present, look up the *other* field of that `FieldType` in the same tracklabel and use

its value.

- `show-if`: condition fulfilled (may proceed to layers 1-3) iff the value is in

`property-value`.

- `hide-if`: condition fulfilled iff the value is **not** in `property-value`.

- No `visibility` element, or a `visibility` element with neither `show-if` nor

`hide-if` → always fulfilled.

  

### Layer 1 — UI focus (selected/hovered vs. Normal)

  

If the track label is focused (selected, or — in the editor's 3-state simplification —

`Hover`/`Completed`), the field is shown (subject to layer 0), full stop. `only-show-on-focus`,

`visible-in-holding` and the control-state list only matter **while unfocused (Normal)**.

  

Correction: in the live app only `trackLabelSelected` forces visibility

(`visibleWhenSelected` in `StaticTrackLabelField.qml:28`) — hover alone does not.

The editor's `VisibilityState::Hover` has no 1:1 runtime counterpart; treating it like

`Completed`/selected is a reasonable editor-only preview simplification, not a bug.

  

### Layer 2 — `visible-in-holding` (only reached when unfocused)

  

Correction: this is checked **before** `only-show-on-focus`, not after it, and it is

**not** "show while holding, per the holding flag". Per `TrackLabelField.qml:84`

(`else if ( model.visibleInHolding ) return true;`, checked before the `visibleOnFocus`

line at `:86`) it unconditionally shows the field while unfocused — bypassing both

`only-show-on-focus` and the control-state check — regardless of whether the flight is

actually in holding.

  

`asd-editor` had no "established in holding" runtime concept at first (no toggle, no

field) — only `activeVisibility` and `activeControlType` existed. This gap is now closed:

`TrackLabelRuntimeDataController::activeVisibleInHolding` (toggled via the "In Holding"

button beneath the control-state header) supplies the runtime half of this condition —

see Layer 5 below for where it applies.

  

### Layer 3 — `only-show-on-focus` (only reached when unfocused and not visible-in-holding)

  

`true` → hidden while unfocused. `false` → continue to layer 4.

  

### Layer 4 — layout correlation

  

Runtime: `else if ( !trackModel.correlatedFlightPlan ) return true;` — fields in an

uncorrelated flight's label always show while unfocused, bypassing the control-state list.

In the editor this maps directly and losslessly to the active layout's

`tracklabel-type`: `TrackLabelTypeVariant::Uncorrelated` ≡ `!correlatedFlightPlan`

(the editor already edits Correlated/Uncorrelated/FlightPlanTrack/Ground as *separate*

layouts, so no per-flight flag is needed).

  

### Layer 5 — `when-unselected-for-control-states`, gated by holding

  

Runtime: `return !trackModel.isEstablishedInHolding && controlStateComparisonHelperId.matchFound;`

Only reached for a correlated layout, unfocused, not `visible-in-holding`-overridden field.

Two conditions both have to hold:

- the simulated flight must **not** be established in holding

(`!activeVisibleInHolding` — the editor's own runtime toggle, see Layer 2), and

- the control-state list must be empty ("if not set the field is shown by default" —

`tracklabel.xsd:214`) or contain the active control state.

  

## Net algorithm (editor)

  

```

isVisible =

conditionFulfilled(show-if/hide-if)

&& ( focused(activeVisibility)

|| ( visibleInHolding

|| ( !onlyShowOnFocus

&& ( isUncorrelatedLayout

|| ( !activeVisibleInHolding

&& ( controlStates.isEmpty()

|| controlStates.contains(activeControlType) ) ) ) ) ) )

```

  

Implemented in `FieldInterface::evaluateVisibility()` (`interface/FieldInterface.cpp`),

called per-field from `FieldListModel::evalVisibility()`, driven by

`TrackLabelDataController::reevaluateActiveLayoutVisibility()` whenever

`TrackLabelRuntimeDataController::changed` fires (wired in

`TrackLabelEditorInstantiator`'s constructor).

  

## Known editor-only gaps (documented ceilings)

  

- `show-if`/`hide-if` cross-field lookup uses the *other* field's `dummyData` (the only

"value" a static config editor has) as a stand-in for live flight data.
  
# Extended track label rules

  

Ground truth from `polaris-asd`, for the open question: does "Completed" (the

editor's selected/expanded visibility state) make sense for every

`TrackLabelTypeVariant`, and how does the extended label fit in?

  

## The XSD models one global extended label, not one per variant

  

`tracklabel.xsd:18-29`: `tracklabel-configuration` has `tracklabel` (one per

variant, `maxOccurs="unbounded"`) plus a single optional `extended-tracklabel`

(`minOccurs="0"`, **not** repeated per variant). There is no per-field

`extended` attribute anywhere in the schema — `ExtendedTrackLabelType` (xsd:86-96)

is just another `TrackLabelGenericType` (a list of `tracklabel-line`s), the

same shape as a normal `tracklabel`.

  

## Extended fields are merged into Correlated and FlightPlanTrack only

  

`TrackLabelConfig::extractExtendedTrackLabel()`

(`source/libs/asd.track.label/config/TrackLabelConfig.cpp:147-195`):

  

- Finds the `Correlated` and `FlightPlanTrack` label configs by type

(`:154-161`). If neither exists, the extended block is dropped (`:163-166`).

- For each extended-tracklabel line `i`, appends its fields to line `i` of

**both** the Correlated and FlightPlanTrack configs (growing their line list

with empty lines if needed, `:172-179`), tagging every merged field

`field.m_extendedField = true` (`:183`).

- Uncorrelated and Ground never receive these fields — the loop only ever

touches `correlatedLabel`/`flightPlanLabel` iterators.

  

So "extended" fields are not a distinct label — they are extra fields spliced

onto the tail of each line of the Correlated and FlightPlanTrack labels,

identically, from one shared XML source block.

  

## `canExtend` is a static type check, independent of config content

  

`TrackLabelController::canExtend()` (`TrackLabelController.cpp:126-129`):

  

```cpp

return ( trackLabelType == TrackLabelEnums::TrackLabelType::Correlated )

|| ( trackLabelType == TrackLabelEnums::TrackLabelType::FlightPlanTrack );

```

  

Hardcoded by type, not by whether an `extended-tracklabel` was actually

merged in. `TrackLabel.qml`'s `isExtended` (hover/keyboard-shortcut/middle-click

driven, auto-retracts when not hovered/selected) is gated by this — Uncorrelated

and Ground can never enter extended mode, regardless of config.

  

## Visibility of an individual extended field: additive filter, same rules

  

`TrackLabelLineProxyModel::filterAcceptsRow()`

(`source/libs/asd.track.label/model/TrackLabelLineProxyModel.cpp:37-59`):

  

```cpp

if( m_showExtendedFields ) return true;

...

return !isExtended.toBool(); // hide fields with ExtendedFieldRole == true

```

  

`m_showExtendedFields` mirrors the label's `isExtended` state. This is a pure

row filter layered **in front of** the normal per-field visibility pipeline

already documented in `tracklabel-visibility-rules.md`:

  

- Not extended (collapsed): extended fields are filtered out of the model

entirely (as if they don't exist); non-extended fields go through the usual

`shouldBeVisibleWithoutSelection` / `showField` rules unchanged.

- Extended (expanded): extended fields become candidate rows, but still have

to pass the exact same `show-if`/`hide-if`, `visible-in-holding`,

`only-show-on-focus`, and `when-unselected-for-control-states` checks as any

other field — extended-ness does not bypass or shortcut that pipeline.

  

## Selection also does not depend on hover

  

`StaticTrackLabelField.qml:23-31`: the field component only receives

`trackLabelSelected`, never `trackLabelHovered`. `trackLabelHovered` is only

used to gate context menu loading and text-field `active` state

(`TrackLabelField.qml:35,76,166`) — it never widens field visibility. In the

real app only "selected" forces fields visible; "hover" is not an independent

visibility tier. The editor's `VisibilityState::Hover` treated the same as

`Completed` for preview purposes is already flagged as an editor-only

simplification in `tracklabel-visibility-rules.md`.

  

## Answer to the open design question

  

"Completed"/selected-state visibility (`visibleWhenSelected` in

`StaticTrackLabelField.qml:28`, "show if non-empty while selected, no other

condition") is meaningful for **every** variant — it is not extended-specific,

it is the ordinary selected-track-label behaviour that already applies to

Uncorrelated and Ground labels too. There is no real-app concept of hiding

"Completed" for non-extendable variants.

  

What genuinely differs per variant is only the **extended field set**:

Uncorrelated/Ground simply never have extended fields merged in (previous

section), so there is nothing to show/hide there — not because "Completed"

is disabled, but because the field list itself is shorter for those variants.

  

**Recommendation for the editor:** don't gate the "Active"/"Completed"

visibility-state control by `TrackLabelTypeVariant`. Instead, only show the

extended-label toggle/section for Correlated and FlightPlanTrack (mirroring

`canExtend()`), and let every variant's existing visibility-state header work

as-is.
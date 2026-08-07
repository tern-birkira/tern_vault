# Plan: per-item drag/drop and arrow-key move for Field18/Equipment fields

  

Status: **proposed, not implemented**. Written down for review before any code changes.

  

## Problem

  

`Field18Type` and `EquipmentType` are the only two field kinds that own an

internal list of child items (`Field18ItemModel` / `EquipmentItemModel`,

currently kept in `TrackLabelField18Model` / `TrackLabelEquipmentModel`

respectively). Every other field is a flat leaf value.

  

We want the same drag/drop and arrow-key move interactions that already work

for whole fields (see `FieldListModel::requestMoveFieldTo`/`requestMoveField`)

to also work *inside* a Field18/Equipment field, moving its child items

around without moving the field itself, using this selection model:

  

- Single click on a Field18/Equipment field: highlights the *whole* field

(as today) and grays out its inner items; drag/arrow-keys move the field

itself, exactly like any other field.

- Double click on one of the field's inner items: highlights *only* that

item; drag/arrow-keys are now scoped to reordering items within that one

field's own item list - they cannot leave the field.

  

## Why this isn't a small addition

  

`FieldInterface` (the `TrackLabelGenericFieldType` counterpart) is a plain

`QObject`, not a list model - intentionally, since ~90% of field kinds have no

children at all. Making `TrackLabelField18Model`/`TrackLabelEquipmentModel` a

`QAbstractListModel` so QML can repeat over their items would require

`FieldInterface` itself to become one (QML delegates need a single, uniform

model type per `Repeater`/view), which drags list-model machinery onto every

other field kind that will never use it.

  

Dummy data also currently lives per field-type in `TrackLabelTypes.cpp`, one

value per field. Once a field can show multiple inner items, each item needs

its own dummy value, not the field as a whole.

  

## Proposed approach

  

1. **Narrow, opt-in capability instead of a base-class list model.** Give

`FieldInterface` a single virtual accessor:

```cpp

virtual QAbstractListModel* itemsModel() const { return nullptr; }

```

Only `TrackLabelField18Model` and `TrackLabelEquipmentModel` override it,

returning a (new) small `QAbstractListModel` wrapper around their existing

`Field18ItemModel`/`EquipmentItemModel` collections. Every other

`FieldInterface` subclass is untouched and keeps returning `nullptr`.

  

2. **Generic QML repeater for inner items**, e.g. `FieldItemsRepeater.qml`,

modeled on the existing `FieldRowRepeater.qml`. `FieldItem.qml` renders it

only `when: fieldModel.itemsModel`, so plain fields pay no extra cost.

  

3. **Two-level selection**, both concrete indices - no pixel math added:

- `TrackLabelDataController.selectedField` (existing) - the whole field.

- New `TrackLabelDataController.selectedFieldItem` - the row index within

`selectedField.itemsModel()`, set on double-click, cleared whenever

`selectedField` changes.

Single click keeps setting `selectedField` only (whole-field highlight,

inner items rendered `opacity: 0.4` while not individually selected).

Double click additionally sets `selectedFieldItem`, which flips the

*inner* item's `highlighted` on and the outer field's own drag/arrow-key

handling off for the duration.

  

4. **Reuse of the existing move plumbing, scoped to the item list.** The

inner items model gets its own `requestMoveItemTo`/`requestMoveItem`

pair with the exact same shape as `FieldListModel`'s (source index, target

index, direction), but the "row" they operate over is the item list

inside one field, not `TrackLabelModel`'s rows - so a drop target outside

the field's own bounds is simply not offered a `DropArea` at all, which is

what keeps the move confined to "within that field" without needing an

explicit bounds check.

  

5. **Dummy data moves down to the item.** `Field18ItemModel`/

`EquipmentItemModel` each gain their own `dummyData` (or equivalent),

populated the same way `TrackLabelTypes.cpp` currently seeds one value per

field kind, just one entry per item instead of one per field.

  

## What this avoids

  

- No change to `FieldInterface` for fields that never have children.

- No second selection/move mechanism - the new inner-item move signals are

the same shape as the existing field-move signals, just scoped one level

deeper.

- No pixel-position math anywhere in the new code, consistent with the

index-based field move/drag work already in place.

  

## Open questions before implementation

  

- Exact shape of the `itemsModel()` wrapper type (thin adapter around

`QVector<Field18ItemModel*>`/`QVector<EquipmentItemModel*>`, or promote

those collections to own `QAbstractListModel`s directly).

- Whether `selectedFieldItem` should live on `TrackLabelDataController`

(mirroring `selectedField`) or directly on the owning

`TrackLabelField18Model`/`TrackLabelEquipmentModel` instance, given only

one field can have an inner selection at a time.

- Whether the field-properties popup needs a third layout mode for editing

a single inner item's properties, or whether existing per-field editing is

sufficient at this granularity.
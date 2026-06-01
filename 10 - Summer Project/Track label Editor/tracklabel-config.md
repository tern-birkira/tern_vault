# Track Label Configuration — Structure Reference

*How `polaris-asd-tracklabel-config.xml` is parsed and what is and is not valid.*

Sources examined:
- `source/libs/asd.track.label/config/TrackLabelConfig.cpp` / `.h`
- `source/libs/asd.track.label/config/TrackLabelFieldConfig.h`
- `source/libs/asd.track.label/config/TrackLabelLineConfig.h`
- `schemas/tracklabel.xsd`
- `schemas/tracklabel-types.xsd`
- `/usr/share/polaris/config/polaris-asd-tracklabel-config.xml`

---

## Overall File Structure

```xml
<instances xmlns:tlabel="http://tern.is/polaris-asd/tracklabel">
    <instance id="polaris-asd-tracklabel" class-mapping-reference="TrackLabelConfig">
        <tlabel:configuration>
            <tlabel:tracklabel-configuration
                    path="..."
                    radio-callsign-filepath="..."
                    has-external-transfer="false"
                    has-force-act="true"
                    has-flight-rule-change="true">

                <!-- One per track type, one or more total -->
                <tlabel:tracklabel tracklabel-type="Correlated">
                    <tlabel:tracklabel-line>
                        <tlabel:field field-name="callsign" ... />
                        ...
                    </tlabel:tracklabel-line>
                    ...
                </tlabel:tracklabel>

                <!-- Optional: adds extra fields to Correlated and FlightPlanTrack -->
                <tlabel:extended-tracklabel>
                    <tlabel:tracklabel-line> ... </tlabel:tracklabel-line>
                </tlabel:extended-tracklabel>

                <!-- Optional: icons/actions shown beside the label, not inside it -->
                <tlabel:column-0-actions>
                    <tlabel:action type="TransferAction" />
                </tlabel:column-0-actions>

            </tlabel:tracklabel-configuration>
        </tlabel:configuration>
    </instance>
</instances>
```

---

## `<tracklabel-configuration>` Attributes

| Attribute | Type | Required | Default | Notes |
|---|---|---|---|---|
| `path` | string | yes | — | **Deprecated** — required by schema but unused at runtime |
| `radio-callsign-filepath` | string | no | — | Path to radio callsign database file |
| `has-external-transfer` | boolean | no | `true` | Affects how inbound-aircraft transfer actions are shown |
| `has-force-act` | boolean | no | `true` | Affects OLDI ForceAct actions for inbound/assumed flights |
| `has-flight-rule-change` | boolean | no | `false` | Enables VFR↔IFR rule-change from the label context menu |

---

## Track Label Types

Each `<tracklabel>` element has a `tracklabel-type` attribute.

| Type | Description |
|---|---|
| `Correlated` | A track correlated with a flight plan |
| `Uncorrelated` | A track without a flight plan |
| `FlightPlanTrack` | A flight plan track (no radar return) |
| `Ground` | A ground vehicle / surface movement track |

The `<extended-tracklabel>` element uses `tracklabel-type="Extended"` (optional attribute, defaults to `Extended`).

---

## Label Layout: Lines and Fields

A track label is a **vertical stack of lines**. Each line is a **horizontal row of fields**.

```
┌────────────────────────────┐
│  callsign   #  CS  wtc     │  ← tracklabel-line 1
│  AFL  ↑ 027  CFL  ARC      │  ← tracklabel-line 2
│  SPD  ASP  DSPD  APP       │  ← tracklabel-line 3
│  ROUTE  >WPT  DHDG  QNH    │  ← tracklabel-line 4
│  DEST  freeText             │  ← tracklabel-line 5
└────────────────────────────┘
```

Fields within a line flow **left to right** in the order they are declared. There is no column layout.

---

## Field Types Inside a Line

Three kinds of elements can appear inside `<tracklabel-line>`:

### 1. `<field>` — Standard data field

```xml
<tlabel:field field-name="callsign"
              prefix=""
              placeholder="CS"
              toggleable="false"
              blinking="false"
              only-show-on-focus="false"
              font-adjustment="0"
              fixed-width-in-characters="-1"
              left-margin="0"
              bottom-margin="0"
              visible-in-holding="false">
    <!-- optional children -->
    <tlabel:context-menu-item ... />
    <tlabel:visibility ... />
    <tlabel:edit ... />
</tlabel:field>
```

#### `field-name` values (closed enum)

Standard flight data:
`aircraftType`, `assignedVerticalRate`, `callsign`, `calculatedTakeOffTime`, `clearedFlightLevel`, `clearedFlightLevelOrApproach`, `clearedHeadingOrWaypoint`, `clearedHeading`, `currentControllingSector`, `nextControllingSector`, `currentSectorFrequency`, `nextSectorFrequency`, `sectorIndicator`, `waypoint`, `eobtOrCtot`, `clearedSpeed`, `ssrCodeAndCallsign`, `snsInhibitedSsrDot`, `combinedAircraftTypeAndWTC`, `currentFlightLevel`, `destinationAerodrome`, `freeText`, `speed`, `ssrCode`, `transferArrow`, `verticalRateArrow`, `verticalRate`, `wakeTurbulenceCategory`, `alternativeDestinationAeroDrome`, `alternativeDestinationAeroDrome2`, `assignedSSRCode`, `currentFrequency`, `departureAerodrome`, `destinationRunway`, `firExitPoint`, `firExitFlightLevel`, `firExitTime`, `flightRule`, `flightType`, `clearedHoldingPoint`, `holdingTerminationTime`, `numberOfAircraft`, `previousSSRCode`, `requestedFlightLevel`, `route`, `clearedStar`, `clearedApproach`, `measuredFlightLevel`, `actualDepartureTime`, `reportedLevel`, `approachSequenceNumber`, `sectorCoordinatedPointOfExitLevel`, `sectorCoordinatedPointOfExitAndEnrouteCruisingLevel`, `sectorPlannedEntryLevel`, `controlStateString`, `sectorCoordinatedPointOfEntry`, `sectorCoordinatedPointOfExit`, `sectorCoordinatedPointOfExitTime`, `sectorCoordinatedPointOfEntryTime`, `nextRouteElement`, `currentRouteElement`

Downlinked (ADS-B) fields:
`indicatedAirspeed`, `magneticHeading`, `trueAirspeed`, `selectedAltitude`, `finalStateSelectedAltitude`, `flightStatusReportedByAdsB`, `barometricVerticalRate`, `geometricVerticalRate`, `machNumber`, `barometricPressureSettings`

#### `font-adjustment` allowed values (closed set)

`-4`, `-2`, `-1`, `0`, `+1`, `+2`, `+4` (pixel delta relative to base font)

---

### 2. `<field18>` — Field 18 items

Expands into one `TrackLabelFieldConfig` **per `<field18-item>`** child. Shares the same generic field attributes as `<field>`.

```xml
<tlabel:field18 prefix="" placeholder="" ...>
    <tlabel:field18-item type="STS" />
    <tlabel:field18-item type="RMK" />
</tlabel:field18>
```

Each item produces a field with `name="field18"`, `filter=<type>`, and `placeholderText=<type>+"/"`.

Valid `type` values: `PBN`, `COM`, `DAT`, `SUR`, `DOF`, `EET`, `SEL`, `OPR`, `ORGN`, `PER`, `ALTN`, `RALT`, `TALT`, `RIF`, `CODE`, `STS`, `NAV`, `DEP`, `DEST`, `RMK`, `REG`, `TYP`, `DLE`

---

### 3. `<equipment-field>` — Equipment capability items

Expands into one `TrackLabelFieldConfig` **per `<field-equipment-item>`** child (minimum 1 required). Shares the same generic field attributes.

```xml
<tlabel:equipment-field ...>
    <tlabel:field-equipment-item type="surveillanceCapabilities" />
</tlabel:equipment-field>
```

Valid `type` values: `communicationAndNavigationCapabilities`, `communicationCapabilities`, `navigationalCapabilities`, `datalinkCapabilities`, `otherCommunicationCapabilities`, `otherDatalinkCapabilities`, `otherNavigationCapabilities`, `performanceBasedNavigationCapabilities`, `surveillanceCapabilities`, `additionalSurveillanceCapabilities`

---

## Field Child Elements

### `<context-menu-item>` (0 or more per field)

```xml
<tlabel:context-menu-item
    context-menu="ClearanceMenu.CFLMenu"
    mouse="right"
    type="press"
    menu-position="left"
    searchable="false" />
```

| Attribute | Type | Default | Notes |
|---|---|---|---|
| `context-menu` | ContextMenuType | required | See table below |
| `mouse` | `left`/`middle`/`right` | `left` | Which mouse button triggers the menu |
| `type` | `press`/`hold` | `press` | `press` items are ordered before `hold` items regardless of XML order |
| `menu-position` | `left`/`right` | `left` | Popup alignment |
| `searchable` | boolean | `false` | Only valid for a specific subset of menus — see scope below |

Valid `context-menu` values:

| Value | QML component |
|---|---|
| `CallsignMenu.Correlated` | Callsign menu for correlated tracks |
| `CallsignMenu.Uncorrelated` | Callsign menu for uncorrelated tracks |
| `ClearanceMenu.AFLMenu` | Actual flight level clearance |
| `ClearanceMenu.APPMenu` | Approach clearance |
| `ClearanceMenu.ARCMenu` | Assigned rate of climb clearance |
| `ClearanceMenu.ASPMenu` | Assigned speed clearance |
| `ClearanceMenu.CFLMenu` | Cleared flight level |
| `ClearanceMenu.XFLMenu` | Crossed flight level |
| `ClearanceMenu.WaypointMenu` | Waypoint selection |
| `ClearanceMenu.HeadingMenu` | Heading selection |
| `ClearanceMenu.DestinationRWYMenu` | Destination runway |
| `Actions.LevelFilterAction` | Level filter action |
| `Actions.DestinationAerodromeFilterAction` | Destination aerodrome filter |
| `ProfileMenus.OpenProfileMenu` | Open flight profile |
| `SectorMenu.Correlated` | Sector control menu |
| `SectorMenu.ToggleFrequencyAction` | Toggle sector frequency |
| `ManualSequencingMenu` | Manual approach sequencing |

---

### `<visibility>` (0 or 1 per field)

Controls when a field is shown when the track is **not** selected/hovered.

```xml
<tlabel:visibility when-unselected-for-control-states="Assumed TransferOutInitiated">
    <tlabel:show-if flight-property="destinationAerodrome" property-value="BIRK BIKF" />
</tlabel:visibility>
```

| Attribute / Element | Notes |
|---|---|
| `when-unselected-for-control-states` | Space-separated list of `ControlType` values. Field is shown when unselected only if the track is in one of these states. If omitted, field shows for all control states (default: all 11 states). |
| `<show-if flight-property="..." property-value="...">` | Optional additional condition. Field is shown only if the given flight property (a `FieldType` value) matches one of the space-separated `property-value` entries. If `flight-property` is omitted, the condition checks the field's own value. |

Valid `ControlType` values for `when-unselected-for-control-states`:
`NonConcerned`, `Concerned`, `Intruder`, `TransferInInitiated`, `RequestInInitiated`, `Assumed`, `TransferOutInitiated`, `RequestOutInitiated`, `Redundant`, `Completed`, `Unknown` (note: schema also lists `NotSet` and `All` but the C++ code uses the above subset)

---

### `<edit>` (0 or 1 per field)

```xml
<tlabel:edit on-mouse="right" />
```

| Attribute | Type | Required | Notes |
|---|---|---|---|
| `on-mouse` | `left`/`middle`/`right` | yes | Mouse button that triggers inline editing |

---

## Extended Track Label

`<extended-tracklabel>` adds extra fields to the **Correlated** and **FlightPlanTrack** labels. Its lines are merged positionally — line N of the extended label appends its fields to line N of the base label.

- If the base label has fewer lines than the extended label, empty lines are inserted first.
- All fields from the extended label receive `m_extendedField = true` internally.
- `<extended-tracklabel>` has **no effect** on `Uncorrelated` or `Ground` labels.
- Only one `<extended-tracklabel>` block is allowed per configuration.

---

## Column-0 Actions

These are indicators and quick-action buttons shown **beside** (to the left of) the track label, not inside it.

```xml
<tlabel:column-0-actions>
    <tlabel:action type="TransferAction" />
    <tlabel:action type="VFRIndicator" />
</tlabel:column-0-actions>
```

Valid `type` values:

| Value | Description |
|---|---|
| `TransferAction` | Assume / Transfer / Release button (single click) |
| `VFRIndicator` | Shows "V" for VFR flights. Color follows `flightRule` field. |
| `RequestInIndicator` | Shows "ROF" for the sector that requested control |
| `RequestOutIndicator` | Shows "ROF" when another sector requested control |
| `ManualOutboundCoordinationTimeoutAction` | Phone icon when outbound coordination has timed out for Assumed flights |

---

## Scope: What Is and Is Not Configurable

### ✅ In scope (editor should expose these)

- Adding, removing, and reordering **tracklabel-line** elements within each label type
- Adding, removing, and reordering **fields** within a line
- All field attributes: `prefix`, `placeholder`, `toggleable`, `blinking`, `only-show-on-focus`, `font-adjustment`, `fixed-width-in-characters`, `left-margin`, `bottom-margin`, `visible-in-holding`
- Adding/removing `<context-menu-item>` elements per field
- Adding/removing `<visibility>` rules and `<show-if>` conditions
- Configuring `<edit>` on `freeText` fields
- `<extended-tracklabel>` lines
- `<column-0-actions>` list
- Top-level boolean flags: `has-external-transfer`, `has-force-act`, `has-flight-rule-change`
- Property substitution variables (`${property.key}`) in attribute values

### ❌ Out of scope (not supported — do not offer in editor)

| Constraint | Reason |
|---|---|
| **No column layout** | Fields are only laid out horizontally row by row. `tracklabel-line` is the only structural dimension. There is no column grid. |
| **Only one configuration per file** | The file always has exactly one `<instance id="polaris-asd-tracklabel">`. Multiple instances are not loaded. |
| **`<edit>` only on `freeText`** | The C++ code throws `InitializationException` if `<edit>` is placed on any other field type at startup. |
| **Extended label only affects Correlated and FlightPlanTrack** | The `extractExtendedTrackLabel` method only searches for those two types. `Uncorrelated` and `Ground` are not extended. |
| **Only one `<extended-tracklabel>` block** | The schema allows 0 or 1. A second block would be ignored or cause a parse error. |
| **`searchable` only for five context menus** | `searchable="true"` is only valid for `ASPMenu`, `CFLMenu`, `HeadingMenu`, `ARCMenu`, `XFLMenu`. Any other menu throws `InitializationException`. |
| **Closed field-name enum** | `field-name` must be one of the ~60 values in `FieldType`. Custom/arbitrary field names are not allowed. |
| **Closed font-adjustment set** | Only `-4`, `-2`, `-1`, `0`, `+1`, `+2`, `+4` are valid. |
| **Closed tracklabel-type enum** | Only `Correlated`, `Uncorrelated`, `FlightPlanTrack`, `Ground` for `<tracklabel>`. |
| **`path` attribute is irrelevant** | It is required by the schema but the runtime does not use its value. It cannot be removed (schema validation fails), but its content doesn't matter. |
| **Context menus outside the known map produce silent no-ops** | An unknown `context-menu` value returns an empty QML source string — nothing is displayed, but no error is raised. |
| **`show-if flight-property` must be a known FieldType** | The attribute type is `FieldType`. Arbitrary property names are not supported. |
| **Press vs. hold ordering is automatic** | Within a field, `type="press"` context menu items always appear before `type="hold"` items, regardless of XML declaration order. The editor cannot override this ordering. |

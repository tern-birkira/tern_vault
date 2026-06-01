# Track Label Editor

*One of the main deliverables of [[Summer Project Overview]]*  
*Config file: `polaris-asd-tracklabel-config.xml`*

---

## What Is a Track Label?

Every aircraft on the radar display has a **track label** — a small data box next to the blip.

```
┌─────────────────┐
│ TK512           │  ← callsign (line 1)
│ 350↑ 370  N0450 │  ← altitude, climb arrow, CFL, speed (line 2)
│ KEFL            │  ← destination (line 3)
└─────────────────┘
```

The config file defines:
- Which **fields** appear (callsign, altitude, speed, etc.)
- Which **line** each field goes on
- Whether fields are **toggleable** (can be hidden/shown by controller)
- **Visibility rules** (show only when selected, only for certain control states)
- **Context menus** (what happens on right-click)
- **Font size** adjustments

---

## Track Label Types

```xml
<tlabel:tracklabel tracklabel-type="Uncorrelated"> ... </tlabel:tracklabel>
<tlabel:tracklabel tracklabel-type="Correlated">   ... </tlabel:tracklabel>
```

- **Uncorrelated** — track seen on radar but no flight plan matched yet. Shows: SSR code, altitude, vertical rate.
- **Correlated** — matched to a flight plan. Shows full data: callsign, CFL, destination, etc.
- Likely more types (check config for others)

---

## XML Structure Breakdown

```xml
<tlabel:tracklabel tracklabel-type="Correlated">

  <tlabel:tracklabel-line>                          <!-- one row of the label -->

    <tlabel:field field-name="callsign"             <!-- which data to show -->
                  font-adjustment="+4"              <!-- bigger text -->
                  toggleable="false">               <!-- always visible -->

      <tlabel:context-menu-item                   <!-- right-click opens menu -->
          context-menu="CallsignMenu.Correlated"
          mouse="right"/>

    </tlabel:field>

    <tlabel:field field-name="wakeTurbulenceCategory"
                  toggleable="true">           <!-- controller can hide this -->
      <tlabel:visibility
          when-unselected-for-control-states="${pasd.tar.tracklabel.unselected.relevant.cs}"/>
    </tlabel:field>

  </tlabel:tracklabel-line>

</tlabel:tracklabel>
```

---

## Key Field Attributes

| Attribute                   | Meaning                                                      |
| --------------------------- | ------------------------------------------------------------ |
| `field-name`                | Which data field (callsign, currentFlightLevel, speed, etc.) |
| `toggleable`                | Controller can toggle visibility                             |
| `only-show-on-focus`        | Only visible when track is selected                          |
| `font-adjustment`           | `+4` = larger, `-4` = smaller relative to base               |
| `placeholder`               | Text shown when field has no value (e.g., `"CFL"`)           |
| `left-margin`               | Pixel spacing from previous field                            |
| `fixed-width-in-characters` | Fixed column width                                           |
| `visible-in-holding`        | Show when aircraft is in holding pattern                     |

---

## Visibility Rules

Complex rules for when a field shows:
```xml
<tlabel:visibility
    when-unselected-for-control-states="${pasd.tar.tracklabel.unselected.relevant.cs}">
    <tlabel:show-if
        flight-property="destinationAerodrome"
        property-value="${pasd.tar.tracklabel.manual.sequencing.aerodromes}"/>
</tlabel:visibility>
```

→ "Hide when not selected, UNLESS destination is one of these aerodromes"

My editor needs to visualize these rules clearly.

---

## What The Editor Needs to Do

1. **Load** `polaris-asd-tracklabel-config.xml`
2. **Display** each tracklabel-type with its lines and fields visually
3. **Preview** — show what the label looks like (ideally live)
4. **Edit** — add/remove/reorder lines and fields, change attributes
5. **Save** — write back to XML preserving structure + property plac-eholders

---

## `tracklabel-configuration` Top-Level Attributes (TrackLabelListType)

| Attribute | Type | Default | Notes |
|-----------|------|---------|-------|
| `path` | `xs:string` | required | ⚠️ **Deprecated** |
| `radio-callsign-filepath` | `xs:string` | optional | Path to radio callsign DB file |
| `has-external-transfer` | `xs:boolean` | `true` | External sector transfer via messages — affects inbound aircraft actions |
| `has-force-act` | `xs:boolean` | `true` | OLDI in letter of agreement — affects inbound/assumed aircraft actions |
| `has-flight-rule-change` | `xs:boolean` | `false` | Whether flight rule can be changed — affects assumed aircraft actions |

---

## `context-menu-item` Full Attributes (ContextMenuItem)

| Attribute | Type | Default | Notes |
|-----------|------|---------|-------|
| `context-menu` | `ContextMenuType` | required | Which menu to open |
| `mouse` | `MouseButtonType` | `left` | `left` \| `middle` \| `right` |
| `type` | `MousePressType` | `press` | `press` \| `hold` |
| `menu-position` | `MenuPosition` | `left` | Popup alignment: `left` \| `right` |
| `searchable` | `xs:boolean` | `false` | Whether menu has a search bar |

---

## `edit` Field Settings (FieldEditSettings)

Child element of any field. Enables inline editing of the field on mouse click.

| Attribute | Type | Notes |
|-----------|------|-------|
| `on-mouse` | `MouseButtonType` | required — which button triggers edit mode |

---

## `field18` Item Types (Field18Type enum)

Valid values for `<field18-item type="...">`:

`PBN` · `COM` · `DAT` · `SUR` · `DOF` · `EET` · `SEL` · `OPR` · `ORGN` · `PER` · `ALTN` · `RALT` · `TALT` · `RIF` · `CODE` · `STS` · `NAV` · `DEP` · `DEST` · `RMK` · `REG` · `TYP` · `DLE`

---

## `equipment-field` Item Types (EquipmentType enum)

Valid values for `<field-equipment-item type="...">`:

`communicationAndNavigationCapabilities` · `communicationCapabilities` · `navigationalCapabilities` · `datalinkCapabilities` · `otherCommunicationCapabilities` · `otherDatalinkCapabilities` · `otherNavigationCapabilities` · `performanceBasedNavigationCapabilities` · `surveillanceCapabilities` · `additionalSurveillanceCapabilities`

---

## `column-0-actions` Action Types (ActionTypeVariant enum)

Valid values for `<action type="...">` inside `column-0-actions`:

| Value | Description |
|-------|-------------|
| `ManualOutboundCoordinationTimeoutAction` | Phone icon for manually coordinating when outbound coordination timed out (Assumed flights). Color follows `sectorIndicator` field. |
| `RequestInIndicator` | Shows `ROF` text beside label for sector that requested control of the flight. |
| `RequestOutIndicator` | Shows `ROF` text when another sector has requested control. |
| `TransferAction` | Primary transfer action — Assume / Transfer / Release in one click. |
| `VFRIndicator` | Shows `V` beside label for VFR flights. Color follows `flightRule` field. |

---

## Open Questions
- ~~Full list of valid `field-name` values — where is this defined?~~ → `FieldType` enum in `tracklabel-types.xsd` (see below)
- ~~What are all valid `tracklabel-type` values?~~ → `TrackLabelTypeVariant` enum: `Correlated`, `Uncorrelated`, `FlightPlanTrack`, `Ground` + `Extended` (for `extended-tracklabel`)
- How does `context-menu` reference resolve — is there a menu config file? (still open)

---

## Valid `field-name` Values (from `tracklabel-types.xsd` `FieldType` enum)

Standard fields:
`aircraftType` · `assignedVerticalRate` · `callsign` · `calculatedTakeOffTime` · `clearedFlightLevel` · `clearedFlightLevelOrApproach` · `clearedHeadingOrWaypoint` · `clearedHeading` · `currentControllingSector` · `nextControllingSector` · `currentSectorFrequency` · `nextSectorFrequency` · `sectorIndicator` · `waypoint` · `eobtOrCtot` · `clearedSpeed` · `ssrCodeAndCallsign` · `snsInhibitedSsrDot` · `combinedAircraftTypeAndWTC` · `currentFlightLevel` · `destinationAerodrome` · `freeText` · `speed` · `ssrCode` · `transferArrow` · `verticalRateArrow` · `verticalRate` · `wakeTurbulenceCategory` · `alternativeDestinationAeroDrome` · `alternativeDestinationAeroDrome2` · `assignedSSRCode` · `currentFrequency` · `departureAerodrome` · `destinationRunway` · `firExitPoint` · `firExitFlightLevel` · `firExitTime` · `flightRule` · `flightType` · `clearedHoldingPoint` · `holdingTerminationTime` · `numberOfAircraft` · `previousSSRCode` · `requestedFlightLevel` · `route` · `clearedStar` · `clearedApproach` · `measuredFlightLevel` · `actualDepartureTime` · `reportedLevel` · `approachSequenceNumber` · `sectorCoordinatedPointOfExitLevel` · `sectorCoordinatedPointOfExitAndEnrouteCruisingLevel` · `sectorPlannedEntryLevel` · `controlStateString` · `sectorCoordinatedPointOfEntry` · `sectorCoordinatedPointOfExit` · `sectorCoordinatedPointOfExitTime` · `sectorCoordinatedPointOfEntryTime` · `nextRouteElement` · `currentRouteElement`

Downlinked (ADS-B) fields:
`indicatedAirspeed` · `magneticHeading` · `trueAirspeed` · `selectedAltitude` · `finalStateSelectedAltitude` · `flightStatusReportedByAdsB` · `barometricVerticalRate` · `geometricVerticalRate` · `machNumber` · `barometricPressureSettings`

---

## Valid `context-menu` Values (from `ContextMenuType` enum)

`CallsignMenu.Correlated` · `CallsignMenu.Uncorrelated` · `ClearanceMenu.AFLMenu` · `ClearanceMenu.APPMenu` · `ClearanceMenu.ARCMenu` · `ClearanceMenu.ASPMenu` · `ClearanceMenu.CFLMenu` · `ClearanceMenu.XFLMenu` · `Actions.LevelFilterAction` · `ClearanceMenu.WaypointMenu` · `ClearanceMenu.HeadingMenu` · `Actions.DestinationAerodromeFilterAction` · `ProfileMenus.OpenProfileMenu` · `SectorMenu.Correlated` · `SectorMenu.ToggleFrequencyAction` · `ManualSequencingMenu` · `ClearanceMenu.DestinationRWYMenu`

---

## XSD Schema Location

`polaris-asd/schemas/tracklabel.xsd` — imports `tracklabel-types.xsd` + `polaris-shared-types.xsd`

Key hierarchy in XSD:
```
configuration
  tracklabel-configuration (TrackLabelListType)
    tracklabel* (TrackLabelType — has tracklabel-type attr)
      tracklabel-line* (TrackLabelLineType)
        xs:choice: field | field18 | equipment-field
    extended-tracklabel? (ExtendedTrackLabelType)
    column-0-actions? (Column0ActionsType)
```

> ⚠️ `tracklabel-line` uses `xs:choice` — field, field18, equipment-field are 3 different element types. Editor must handle all.

---

*Related: [[ASD Config Overview]] | [[Summer Project Overview]] | [[Flight List Editor]] | [[XSD Field Generation]]*

# Flight List Editor

*One of the main deliverables of [[Summer Project Overview]]*  
*Config file: `polaris-asd-lists-app.xml`*

---

## What Is a Flight List?

A **flight list** is a panel on the ASD showing flights grouped by phase (inbound, departing, etc.)  
Think: a table/list that ATCOs use alongside the radar map.

```
┌─ INBOUND ──────────────────────────────────────────┐
│ CALLSIGN  TYPE  WTC  ETA    ROUTE   RWY  CFL  AFL │
│ TK512     B738  M    14:23  BIKF    01   350  340 │
│ FI123     B757  M    14:45  BIRK    19   220  180 │
└────────────────────────────────────────────────────┘
```

---

## XML Structure Breakdown

```xml
<instances xmlns:tlist="http://tern.is/polaris-asd/flightlist">
  <instance id="polaris-asd-flightlist"
            class-mapping-reference="FlightListConfig">
    <tlist:configuration
        background-color="ebonyClay"
        item-background-color="mineShaftDark"
        default-size="normal">

      <tlist:flight-list-group
          name="MainLists"
          max-height="900"
          vertical-alignment="center">

        <tlist:flight-list
            name=""
            icon=""
            icon-color="#8CBAF0"
            maximum-items-shown="10"
            minimum-items-shown="1"
            vertical-position="370"
            show-marks="true"
            show-freetext="true">

          <tlist:field
              column="0" row="0"
              header="CALLSIGN"
              display-role="callsign"
              font-size="22"
              width="85"
              column-width="85">
            <tlist:context-menu-item
                context-menu="CallsignMenu.Correlated"
                mouse="right"/>
          </tlist:field>

          <!-- more fields... -->

        </tlist:flight-list>
      </tlist:flight-list-group>

    </tlist:configuration>
  </instance>
</instances>
```

---

## Key Concepts

### Groups and Lists
- `flight-list-group` = container (can have multiple lists stacked)
- `flight-list` = one visible list panel (e.g., "Inbound", "Departing")
- Lists have position (`vertical-position`) and size limits

### Fields (Columns)
Each `<tlist:field>` = one column in the list:

| Attribute | Meaning |
|-----------|---------|
| `column` | Column index (0-based) |
| `row` | Row within item (for multi-row items) |
| `header` | Short column header text |
| `header-detailed` | Full tooltip name |
| `display-role` | Which data to show (callsign, estimatedArrivalTime, etc.) |
| `font-size` | Text size |
| `font-size-xl` | Size on larger displays |
| `width` | Column width in pixels |
| `width-xl` | Width on larger displays |
| `column-width` | Total column allocation |

### `flight-list-group` Attributes (FlightListGroupType)

| Attribute | Type | Default | Notes |
|-----------|------|---------|-------|
| `name` | `xs:string` | optional | Group name |
| `max-height` | `xs:int` | optional | Max px height of group |
| `vertical-alignment` | `AlignmentType` | `center` | Vertical alignment within layout |
| `display-side` | `DisplaySideType` | `right` | `left` \| `right` — screen side |

### `flight-list` Full Attributes (FlightListType)

Beyond the basics in the table above:

| Attribute | Type | Default | Notes |
|-----------|------|---------|-------|
| `minimum-items-shown` | `xs:int` | `-1` | Min items for list to be visible |
| `maximum-items-shown` | `xs:int` | required | Max items shown |
| `name` | `xs:string` | required | Display name |
| `icon` | `xs:string` | required | QRC path e.g. `qrc:icons/synchronize-5.svg` |
| `icon-color` | `st:ColorType` | required | Hex color for icon |
| `expandable` | `xs:boolean` | `false` | Expands on hover |
| `moveable` | `xs:boolean` | `false` | Can be dragged around screen edge |
| `mergeable` | `xs:boolean` | `false` | Can merge with other lists |
| `closeable` | `xs:boolean` | `false` | Can be closed by user |
| `collapse-time` | `xs:int` | `1000` | ms before collapse after losing focus (if `expandable`) |
| `vertical-position` | `xs:int` | `0` | px from top (≥0) or bottom (<0) |
| `horizontal-alignment` | `HorizontalAlignment` | `right` | Horizontal list alignment |
| `tab-font-family` | `FontFamily` | `Lato` | Font for tab name |
| `show-marks` | `xs:boolean` | `false` | Show marks in list |
| `show-freetext` | `xs:boolean` | `false` | Show freetext on hover/select |
| `sort-order` | `SortOrderType` | `ascending` | `ascending` \| `descending` |
| `user-sorting-enabled` | `xs:boolean` | `true` | User can adjust sort settings |

### `configuration` Top-Level Attributes

| Attribute | Type | Default | Notes |
|-----------|------|---------|-------|
| `background-color` | `st:StyleSheetColorTypes` | `mineShaftDark` | List background color |
| `item-background-color` | `st:StyleSheetColorTypes` | `abbey` | Row background color |
| `list-panel-background-color` | `st:StyleSheetColorTypes` | optional | Panel background (shared across all groups) |
| `list-panel-background-opacity` | `xs:float` | `1.0` | 0.0–1.0 opacity |
| `fit-height-to-items` | `xs:boolean` | `false` | Fit list height to item count |
| `dynamic-lists-container` | `xs:boolean` | `false` | Container holds dynamically created/deleted lists |
| `group-text-color` | `st:StyleSheetColorTypes` | `silverChalice` | Color of group role text |
| `default-size` | `SizeType` | `normal` | Default size variant |

### Two Size Variants
Many attributes have `-xl` variants (e.g., `font-size-xl`, `width-xl`) for larger screen configurations.

---

## What The Editor Needs to Do

1. **Load** the XML
2. **Display** each flight-list group with its lists and columns
3. **Visual preview** — show a mock-up of what the list looks like
4. **Edit** — reorder columns, change widths/sizes, add/remove fields
5. **Save** — write back to XML

---

## Open Questions
- ~~Full list of valid `display-role` values?~~ → `DisplayRoleType` enum in `polaris-flightlist-types.xsd` (see below)
- ~~What defines the flight list groups (Inbound, Outbound, etc.)?~~ → `name` attr on `flight-list` + `tab` elements define the visible label/tab name. Filtering is done via `tlist:filter` + `tlist:tab` child elements.
- How does `icon` attribute work for a list? (still open — appears to be a QRC path e.g. `qrc:icons/synchronize-5.svg`)

---

## XSD Schema Location

`polaris-asd/schemas/flightlist.xsd` — imports `polaris-flightlist-types.xsd` + `polaris-shared-types.xsd`

> ⚠️ `polaris-lists.xsd` is **deprecated** — use `flightlist.xsd`.

Key hierarchy in XSD:
```
configuration (FlightListConfig)
  flight-list-group* (FlightListGroupType)
    flight-list* (FlightListType)
      field* (FieldAlignmentType)
      tab? (ListTabType — max 2)
```

---

## Valid `display-role` Values (from `DisplayRoleType` in `polaris-flightlist-types.xsd`)

`callsign` · `filedSSRCode` · `rvsmWarning` · `_833khzWarning` · `clearedFlightLevelOrApproach` · `sectorIndicator` · `departureAerodrome` · `destinationAerodrome` · `aircraftType` · `waketurbulencecategory` · `numberOfAircraft` · `calculatedTakeOffTime` · `clearedSectorExitLevel` · `pointOfExit` · `coordinatedPointOfExitTime` · `coordinatedPointOfEntry` · `coordinatedPointOfEntryTime` · `clearedSid` · `clearedStar` · `clearedApproach` · `clearedDepartingSidOrArrivingStar` · `currentControllingSector` · `nextControllingSector` · `currentSectorFrequency` · `nextSectorFrequency` · `estimatedDepartureTime` · `flightRule` · `departureRunway` · `destinationRunway` · `clearedHeadingOrWaypoint` · `clearedHeading` · `waypoint` · `eobtOrCtot` · `estimatedClearedFlightLevel` · `estimatedCrossingConditions` · `coordinatedPointOfEntryLevel` · `coordinatedPointOfExitLevel` · `assignedVerticalRate` · `callsignMismatch` · `clearedFlightLevel` · `clearedSpeed` · `clearedSpeedUnitIsKnots` · `clearedNextWaypoint` · `controllingSectorInAor` · `controlState` · `controlStateString` · `coordinationTimeout` · `correlatedFlightPlan` · `cplMessageStatus` · `currentSector` · `decorrelateFlightPlan` · `departureMessage` · `_833khzWarningStatus` · `rvsmWarningStatus` · `estimatedArrivalTime` · `estimatedSupplementaryFlightLevel` · `flightCourse` · *(and more — check `polaris-flightlist-types.xsd` for full list)*

Special value: `-1` = spacer column (zero-width, no data)

---

## Width Duality

Every size attribute has a normal + XL screen variant:

| Attribute | Meaning |
|-----------|---------|
| `width` | Column content width (normal screen) |
| `width-xl` | Column content width (XL/4K screen) |
| `column-width` | Total column allocation (normal) |
| `column-width-xl` | Total column allocation (XL) |
| `font-size` | Text size (normal) |
| `font-size-xl` | Text size (XL) |

Editor UI needs **both inputs** per size attribute.

---

*Related: [[ASD Config Overview]] | [[Summer Project Overview]] | [[Track Label Editor]] | [[XSD Field Generation]]*

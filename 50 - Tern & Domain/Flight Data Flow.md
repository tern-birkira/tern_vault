# Flight Data Flow

*How a flight moves through ATC systems — end to end*  
*Part of: [[ATC Domain Overview]]*

---

## Overview

```
Pilot files plan → FDPS/SFR → TWR → APP → ACC → (cruise) → ACC → APP → TWR → landed
```

Each handoff = coordination between [[ATC Units]].

---

## Stage 1: Pre-Departure

- Pilot submits flight plan
- Plan enters **[[FDPS]]** (Flight Data Processing System) → creates **[[SFR]]** in [[Polaris]]
- Plan includes: route, altitude, speed, aircraft type, departure/arrival airports

## Stage 2: Departure

- **[[TWR]]** clears aircraft for pushback, taxi, takeoff
- Aircraft airborne → short transition period in TWR airspace
- TWR sends **[[OLDI]]** coordination message to APP: "expect this flight"

## Stage 3: Climb (APP)

- **[[APP]]** takes control from TWR
- Manages climb-out, separation from other traffic in terminal area
- Hands off to ACC at agreed coordination point (defined in [[LoA]])

## Stage 4: Cruise (ACC)

- **[[ACC]]** controls en-route, FL200+
- If crossing multiple ACCs: [[Inter-centre Coordination]] via [[OLDI]]
- Safety nets active: [[STCA]], [[MSAW]], [[APW]], [[CLAM]], [[DSAM]], [[RAM]]

## Stage 5: Descent

- Destination ACC descends aircraft, hands to destination APP
- APP manages arrival sequence
- Hands to TWR for landing

---

## Key Data Artefacts

| Artefact | Description |
|----------|-------------|
| [[SFR]] | System Flight Record — Polaris flight plan |
| [[FDPS]] | Flight Data Processing System (Isavia's system) |
| [[OLDI]] | Protocol for inter-unit coordination messages |
| [[LoA]] | Letter of Agreement — rules between units |
| [[ATIS]] | Automated terminal weather info broadcast to pilots |

---

*Related: [[ATC Units]] | [[OLDI]] | [[Polaris]] | [[Safety Nets]]*

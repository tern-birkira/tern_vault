# ATC Domain Overview

*Foundation knowledge for everything Tern builds*

---

## What Is ATC?

Air Traffic Control = ground-based service ensuring aircraft separation and safe, orderly flow of traffic.

ATCOs (see [[ATCO]]) don't fly planes — they give instructions to pilots.

---

## The Three Layers of ATC

```
ACC  ─ Enroute / Cruising (FL200+)      ← [[Area Control Centre]]
APP  ─ Approach / Departure (lower alt)  ← [[Approach Control]]
TWR  ─ Ground + immediate airspace       ← [[Tower Control]]
```

Each layer = a different [[ATC Units|ATC Unit]]. Aircraft handed off between them.

---

## How a Flight Moves Through ATC

See [[Flight Data Flow]] for full detail.

Short version:
1. Pilot files flight plan → enters [[FDPS]] / [[SFR]]
2. TWR clears takeoff
3. APP takes control after departure
4. ACC takes over for cruise
5. ACC hands to destination ACC → APP → TWR → landed

---

## Key Systems ATCOs Use

| System | Purpose |
|--------|---------|
| ASD (Air Situation Display) | See all traffic on radar-like display |
| Electronic Flight Strips | Digital version of paper strips |
| [[OLDI]] | Automated inter-unit coordination |
| [[Safety Nets]] | Automated alerts (STCA, MSAW, etc.) |

---

## Why This Domain Is Hard (for Software)

- **Safety-critical**: software bugs can kill people
- **Real-time**: data must be processed and displayed with very low latency
- **Regulatory**: must meet EUROCONTROL / ICAO standards
- **Complex state**: tracking hundreds of flights, each with clearances, predicted paths, coordination status

---

## Sub-Topics
- [[ATC Units]] — ACC, APP, TWR in detail
- [[Airspace Structure]] — how airspace is organised
- [[Flight Data Flow]] — lifecycle of a flight through ATC
- [[Safety Nets]] — automated safety alerts
- [[Aviation Acronyms and Definitions]] — full glossary

---

*Related: [[Tern Systems]] | [[Polaris]] | [[ISDS]]*

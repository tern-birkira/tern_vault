# Polaris

**Type:** ATM System  
**Made by:** [[Tern Systems]]  
**Replaces:** [[TAS]]  
**Domain:** [[ATC Domain Overview]]

---

## What Is Polaris?

Polaris = Tern's flagship Air Traffic Management system. Used by [[ATC Units]] for ACC, APP, and TWR operations.

Core job: give ATCOs a unified interface to manage flights — see where they are, see their flight plans, coordinate between sectors/units, receive [[Safety Nets]] alerts.

---

## Key Capabilities

| Capability | Notes |
|-----------|-------|
| Air Situation Display (ASD) | Primary controller interface |
| Electronic Flight Strips | Replaces paper strips |
| [[Inter-sector Coordination]] | Within same ATM unit |
| [[Inter-centre Coordination]] | Between different ATC units via [[OLDI]] |
| [[Safety Nets]] | STCA, MSAW, APW, CLAM, DSAM, RAM |
| Flight Data Processing | Via [[SFR]] (System Flight Record) |

---

## Internal Concepts

- **[[SFR]]** — Polaris internal name for a flight plan / flight data record
- **ASD** — Air Situation Display, primary ATCO interface

---

## Deployment Example
- Budapest: East sector and West sector both run Polaris
  - Enables efficient [[Inter-sector Coordination]] between them

---

## Open Questions
- What language/stack is Polaris built in?
- How does Polaris ingest surveillance data (radar, ADS-B)?
- What's the architecture — monolith, microservices, real-time event system?
- How are [[Safety Nets]] implemented — rule engine, ML, geometry?
- What does a "sector" mean in Polaris data model?

---

*Related: [[Tern Systems]] | [[ISDS]] | [[Safety Nets]] | [[ATC Units]] | [[Flight Data Flow]]*

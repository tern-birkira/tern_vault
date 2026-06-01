# Safety Nets

*Part of: [[ATC Domain Overview]] | Implemented in: [[Polaris]]*

---

## What Are Safety Nets?

Ground-based automated systems that alert [[ATCO]]s to potential hazards. They don't replace controller judgment — they're a last-resort backstop.

---

## The Safety Nets

### [[STCA]] — Short Term Conflict Alert
- Imminent loss of separation between two aircraft
- Look-ahead: ~2 minutes
- Severity escalates as aircraft converge
- *Most critical safety net*

### [[MSAW]] — Minimum Safe Altitude Warning
- Aircraft at risk of hitting terrain or obstacles
- Uses altitude map overlaid on situation display
- Compares transmitted altitude vs local minimum safe altitude

### [[APW]] — Area Proximity Warning
- Aircraft entering prohibited/restricted airspace
- ATM system must have current airspace activations ("volumes") loaded
- Without current data → incoherent warnings

### [[CLAM]] — Cleared Level Adherence Monitoring
- Aircraft deviating from its cleared altitude profile
- E.g. climbing without clearance, not climbing when told to

### [[DSAM]] — Downlinked Selected Altitude Monitor
- Discrepancy between cleared FL and what crew entered in cockpit
- Transmitted via Mode S surveillance
- Catches crew data-entry errors *before* they become incidents
- Also called: Selected Vertical Intent / Final State Selected Altitude

### [[RAM]] — Route Adherence Monitoring
- Aircraft deviating from cleared lateral (horizontal) route
- When triggered: trajectory calculation suspended (since trajectory assumes cleared route)

---

## Relationship to [[MTCD]]

[[MTCD]] (Medium Term Conflict Detection) is a longer-horizon tool (15-25 min), not strictly a "safety net" — more a planning aid. Industry still maturing on this.

---

## Engineering Questions
- How are safety net thresholds configured?
- Are they computed per-aircraft or via spatial indexing?
- How does false-alert rate get tuned?

---

*Related: [[Polaris]] | [[ATC Domain Overview]] | [[ATCO]] | [[Airspace Structure]]*

# Software Architecture at Tern

*My evolving understanding of how Tern's software is built*  
*See also: [[Tech Stack]] | [[Codebase Overview]]*

---

## What I Know So Far

*(Fill this in as you learn — start sparse, add through questions)*

### Polaris Architecture
- [ ] What is the high-level architecture? (monolith / microservices / SOA?)
- [ ] How does real-time surveillance data enter the system?
- [ ] How is state managed across multiple controller workstations?
- [ ] How are [[Safety Nets]] computed — separate process? inline?

### Data Flows
- Surveillance data (radar / ADS-B / Mode S) → ???
- Flight plan data → [[SFR]] / [[FDPS]] → controller display
- [[OLDI]] messages → received, parsed, applied to flight state

### Reliability & Safety
- ATC software is safety-critical — what standards does Tern follow?
- DO-178C? EUROCAE ED-153? EUROCONTROL standards?
- How is fault tolerance handled? Failover? Hot standby?

---

## Questions to Ask My Team

- What's the primary programming language for Polaris?
- How is the system tested? Simulation environments?
- How does Tern handle database consistency for real-time flight state?
- What monitoring/observability exists in production?

---

*Related: [[Polaris]] | [[Tech Stack]] | [[Codebase Overview]] | [[Testing Strategy]]*

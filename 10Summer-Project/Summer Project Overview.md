# 🎯 Summer Project — Configuration Editors for Polaris ASD

**Author of brief:** Davíð Sindri Pétursson  
**Date:** 6 May 2026  
**Repo:** `https://gitlab.com/TernDev/ATM/polaris-asd-editor`

---

## The Problem

Polaris ASD is configured via hundreds of XML/JSON files.  
For complex files (UI layout, aeronautical data) — editing raw XML is error-prone and painful.

**Solution:** Build GUI editors that read, visualize, and write these config files.

---

## What Was Already Built (Previous Interns)

| Tool | Status | Notes |
|------|--------|-------|
| [[Map Layer Editor]] | Prototype | Not yet deployed |
| [[Aeronautical Data Editor]] | Deployed | Live in Hungary, coming to Iceland |

Both live in `polaris-asd-editor` repo.

---

## My Job This Summer

### Phase 1 — Import/Export Library (`3.1`)
Create reusable classes to:
- Parse XML and pick out specific `<instance>` blocks
- Modify them
- Export back while preserving the rest of the file
- Handle `${property.variable}` substitutions

→ Shared library used by ALL editors

### Phase 2 — Field Gen from XSD (`3.2`)
Auto-generate UI form fields from XSD schemas — so editors don't need manual updates when schemas change.

→ See [[XSD Field Generation]] | Hackathon prototype: `schema-editor-generator`

### Phase 3 — Specific Editors (`3.3`)
- **[[Track Label Editor]]** — edit `polaris-asd-tracklabel-config.xml`
- **[[Flight List Editor]]** — edit `polaris-asd-lists-app.xml`
- Possibly: safety nets, aircraft performance

### Phase 4 — WebAssembly (`3.4`)
Egill Magnússon compiles editors to WASM so they run in any browser.  
Success → applied to our editors too.

---

## Key Config Files I'm Editing

| File | What It Controls | My Editor |
|------|-----------------|-----------|
| `polaris-asd-tracklabel-config.xml` | What fields appear on aircraft labels on radar | [[Track Label Editor]] |
| `polaris-asd-lists-app.xml` | Flight list panels (inbound, outbound, etc.) | [[Flight List Editor]] |
| `polaris-asd-ui-config.xml` | Colors, keyboard shortcuts, safety nets, workstation | future |
| `polaris-asd-notifications-config.xml` | Alert/notification behavior | future |

---

## Technology

- **[[Qt Framework]]** — C++ GUI framework used by Polaris and these editors
- **[[WebAssembly]]** — compile Qt apps to run in browser (Egill's work)
- **XML + XSD** — config format + schema
- **[[Config XML Structure]]** — how Polaris XML configs are structured

---

## People

- **Davíð Sindri Pétursson** — project supervisor
- **Egill Magnússon** — WebAssembly track (parallel work)
- **Hayfa Ayadi** — hackathon work on schema-editor-generator + track-label-editor

---

## Open Questions
→ [[70 - Questions & Gaps/Summer Project Questions]]

---

*Related: [[Polaris]] | [[ASD Config Overview]] | [[Track Label Editor]] | [[Flight List Editor]]*

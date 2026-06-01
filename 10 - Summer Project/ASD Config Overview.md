# ASD Config Overview

*How Polaris ASD configuration files are structured*  
*Parent: [[Summer Project Overview]]*

---

## The Big Picture

Polaris ASD reads a set of XML/JSON config files at startup.  
Each file configures a different subsystem of the ASD.

```
polaris/config/
├── polaris-asd-tracklabel-config.xml    ← how aircraft labels look
├── polaris-asd-lists-app.xml            ← flight list panels
├── polaris-asd-lists-fds.xml            ← flight list (FDS variant)
├── polaris-asd-ui-config.xml            ← colors, keyboard, safety nets, workstation
├── polaris-asd-notifications-config.xml ← alert behavior
├── polaris-asd-sfr-editor-config.xml    ← SFR (flight plan) editor layout
├── polaris-asd-header-config-app.xml    ← header bar
├── polaris-asd-field-colors-app.xml     ← field color rules
├── polaris-asd-endpoints-operational.xml← network connections (ASTERIX etc.)
├── polaris-asd-grpc-connections.xml     ← gRPC service endpoints
├── unit-configuration-app.properties   ← key=value properties (color scheme etc.)
└── MapLayers/                           ← map geometry (JSON/XML per layer)
```

---

## XML Config Pattern — `<instances>`

Every XML config file follows this pattern:

```xml
<instances xmlns:tlabel="http://tern.is/polaris-asd/tracklabel">
    <instance id="polaris-asd-tracklabel"
              class-mapping-reference="TrackLabelConfig">
        <tlabel:configuration>
            <!-- actual config here -->
        </tlabel:configuration>
    </instance>
</instances>
```

Key concepts:
- **`<instances>`** — root container, declares XML namespaces
- **`<instance>`** — one configurable component
  - `id` — unique identifier
  - `class-mapping-reference` — maps to a C++ class in Polaris
- **namespace prefix** (`tlabel:`, `tlist:`, etc.) — scopes the XML to a specific schema
- **`${property.name}`** — placeholder resolved from `.properties` files at runtime

---

## Property Substitution

Values like `${pasd.unit.colorscheme}` come from:
- `unit-configuration-app.properties`
- `unit-configuration-fds.properties`

This lets the same XML file work for different customers/units — just swap the properties file.

Example:
```properties
# unit-configuration-app.properties
pasd.unit.colorscheme=Isavia
```

```xml
<!-- polaris-asd-ui-config.xml -->
<col:color-scheme name="${pasd.unit.colorscheme}"/>
```

→ **My editor must handle this** — show resolved values but know the property key behind them.

---

## Config Variants: `-app` vs `-fds`

Some files have two variants:
- `polaris-asd-lists-app.xml` — for APP (approach) workstations
- `polaris-asd-lists-fds.xml` — for FDS (flight data) workstations

Same structure, different content. Editor needs to handle both.

---

## Map Layers

Separate structure: `MapLayers/NN#Name/` folders.
Each layer = JSON or XML geometry file + `.cfg` display config.
Handled by the separate [[Map Layer Editor]].

---

*Related: [[Summer Project Overview]] | [[Track Label Editor]] | [[Flight List Editor]] | [[Config XML Structure]]*

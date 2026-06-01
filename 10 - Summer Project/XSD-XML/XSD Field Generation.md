# XSD Field Generation

*Phase 3.2 of [[Summer Project Overview]]*  
*Goal: auto-generate UI form fields from XSD schemas*

---

## The Idea

Instead of hand-coding a form for every config attribute, **read the XSD schema** and generate the appropriate input widget automatically.

Benefits:
- Editor auto-updates when schema changes
- Less manual code to write
- Consistent validation (schema defines allowed values)

---

## What Is XSD?

**XML Schema Definition** — describes the valid structure of an XML file.

Example: the track label XSD defines that `field-name` must be a string, `toggleable` must be `true|false`, `font-adjustment` must be an integer, etc.

```xml
<!-- example XSD fragment (not real) -->
<xs:element name="field">
  <xs:complexType>
    <xs:attribute name="field-name" type="xs:string" use="required"/>
    <xs:attribute name="toggleable" type="xs:boolean" default="true"/>
    <xs:attribute name="font-adjustment" type="xs:integer"/>
  </xs:complexType>
</xs:element>
```

→ From this, we can auto-generate: text input, checkbox, number spinner.

---

## Hackathon Prototype (Hayfa Ayadi)

Location: `Field_gen_from_XSD_scheme/qt-wasm-exploration/apps/schema-editor-generator`

What it explores:
- Reading an XSD
- Generating Qt form widgets from it
- Possibly displaying in WebAssembly

This is starting code — understand it before building on top.

---

## Relationship to Other Work

The **Aeronautical Data Editor** (already deployed) does the same thing using **protobuf schemas** instead of XSD. Study it as a reference implementation.

---

## XSD Files Location (Resolved)

Both schemas live in `polaris-asd/schemas/`:

| Config File | XSD | Imports |
|------------|-----|---------|
| `polaris-asd-tracklabel-config.xml` | `tracklabel.xsd` | `tracklabel-types.xsd`, `polaris-shared-types.xsd` |
| `polaris-asd-lists-app.xml` | `flightlist.xsd` | `polaris-flightlist-types.xsd`, `polaris-shared-types.xsd` |

> ⚠️ `polaris-lists.xsd` is **deprecated** — `flightlist.xsd` is the real one.

---

## XSD Type → Qt Widget Mapping (Resolved)

| XSD Type | Qt Widget | Notes |
|----------|-----------|-------|
| `xs:boolean` | `QCheckBox` | `toggleable`, `only-show-on-focus`, `blinking`, etc. |
| `xs:string` | `QLineEdit` | Free text |
| `xs:int` | `QSpinBox` | `left-margin`, `fixed-width-in-characters`, `column`, `row`, etc. |
| `xs:float` | `QDoubleSpinBox` | e.g. `list-panel-background-opacity` |
| `xs:restriction` with `xs:enumeration` | `QComboBox` | `field-name`, `tracklabel-type`, `font-adjustment`, `mouse`, etc. |
| `st:StyleSheetColorTypes` | `QComboBox` or color picker | Named color palette (from `polaris-shared-types.xsd`) |
| `st:ColorType` | Color picker / hex input | Hex color string (e.g. `#8CBAF0`) |

### Special case: `FontSizeDelta`
Restricted string enum — NOT an integer. Valid values: `-4`, `-2`, `-1`, `0`, `+1`, `+2`, `+4`.
→ `QComboBox`, not `QSpinBox`.

---

## What XSD Gives You For Free

- **`xs:documentation`** annotations → already written tooltip/help text on most attributes
- **`use="required"`** → mandatory field marker in UI
- **`default="..."`** → pre-fill default value
- **All valid enum values** → no need to hardcode lists in editor code

---

## Constraints / Things XSD Can't Do

- **`${property.variable}` substitutions** — runtime values injected from `.properties` files. Not in schema. Editor must **preserve them as-is** without expanding or validating.
- **Tern DI outer wrapper** (`<instances><instance ...>`) — not part of the content XSD. Editor handles this separately (Phase 1 import/export library).
- **`xs:choice` in tracklabel** — `tracklabel-line` can contain `<field>` OR `<field18>` OR `<equipment-field>`. Editor must handle all 3 types.
- **`width` + `width-xl` duality** in flightlist — every size attribute has a normal and XL-screen variant. UI needs both inputs per field.

---

## Hackathon Prototype (Hayfa Ayadi)

Location: `~/tern_vault/Field_gen_from_XSD_scheme/qt-wasm-exploration/apps/schema-editor-generator`

Key files:
- `src/parsers/XsdSchemaParser.cpp/.h` — parses XSD into `SchemaInfo` / `ElementInfo` / `AttributeInfo`
- `src/parsers/SchemaTypes.h` — data model: `FormField`, `ComplexType`, `AttributeInfo`, `SchemaInfo`
- `src/generators/QmlFormGenerator.cpp/.h` — generates QML form from parsed schema
- `src/models/` — Qt models to drive list views

The `FieldType` enum in `SchemaTypes.h` maps directly to XSD's `FieldType` concepts.

---

## Relationship to Other Work

The **Aeronautical Data Editor** (already deployed) does the same thing using **protobuf schemas** instead of XSD. Study it as a reference implementation for the schema→widget mapping pattern.

---

## Open Questions
- How does the aeronautical editor do schema→widget mapping exactly? (read `polaris-asd-editor` source)
- `st:StyleSheetColorTypes` — full list of named colors? (check `polaris-shared-types.xsd`)

---

*Related: [[Summer Project Overview]] | [[Track Label Editor]] | [[Flight List Editor]] | [[Qt Framework]]*

# Schema-driven UI generation

This is a deep-dive into the `schema-editor-generator` app — what it does, why it exists, and how it maps to the broader pattern used in the `aeronautical-editor`.

## 1. Why this matters

In an air-traffic-control / aeronautical system, "data" is not free-form. It is described by formal schemas:

- **XSD (XML Schema Definition)** for XML configuration (user permissions, sector profiles, runway configs, …).
- **Protobuf `.proto` files** for the aeronautical domain model (flights, tracks, airspace, sectors, …).

These schemas change. New fields get added, enumerations grow, attributes get renamed. If every editor screen is hand-written QML, then **every schema change forces a parallel QML change**, a code review, a rebuild, and a redeploy. That coupling is the bug.

The idea: treat the schema as the **single source of truth** and *derive* the editor UI from it at runtime.

```mermaid
flowchart LR
    A[Schema author edits<br/>XSD / .proto] --> B[Schema file]
    B --> C{Editor app loads schema}
    C --> D[UI regenerates<br/>automatically]
    D --> E[User edits values]
    E --> F[App serializes back<br/>to XML / protobuf]

    style B fill:#234,stroke:#88f,color:#fff
    style D fill:#243,stroke:#8f8,color:#fff
```

Without this, the arrow from `B` → `D` is "developer writes QML by hand."

## 2. The three-stage pipeline

The `schema-editor-generator` app is the minimal, self-contained demonstrator. Its pipeline has three stages:

```mermaid
flowchart TB
    subgraph Stage1["Stage 1: PARSE (C++)"]
        X[XSD text] --> P[XsdSchemaParser]
        P --> S[SchemaInfo<br/>rootFields, complexTypes,<br/>simpleTypes]
    end

    subgraph Stage2["Stage 2: TRANSFORM (C++)"]
        S --> G[QmlFormGenerator]
        G --> M[ElementListModel<br/>+ nested AttributeListModel<br/>+ child ElementListModels]
    end

    subgraph Stage3["Stage 3: RENDER (QML)"]
        M --> R[Repeater over ElementListModel]
        R --> EIG[ElementInputGenerator.qml]
        EIG --> CTRL{model.unit?}
        CTRL -->|enum| CB[ComboBox]
        CTRL -->|simple| TF[TextField]
        CTRL -->|complex| REC[Recursive Loader<br/>child ElementInputGenerator]
    end

    style Stage1 fill:#1a2a3a,stroke:#48a,color:#fff
    style Stage2 fill:#1a3a2a,stroke:#4a8,color:#fff
    style Stage3 fill:#3a2a1a,stroke:#a84,color:#fff
```

### Stage 1 — Parsing the XSD into a typed tree

`XsdSchemaParser` takes an XSD string and walks it in three passes:

1. **Simple types first** — every `<xs:simpleType>` becomes a `FormField` with `FieldType::Enumeration` (if it has `<xs:restriction>` + `<xs:enumeration>` values) or a primitive type. This must come first because complex types reference them.
2. **Complex types** — `<xs:complexType>` becomes a `ComplexType` containing child `FormField`s (from `<xs:sequence>`) and `attributes` (from `<xs:attribute>`).
3. **Root elements** — top-level `<xs:element>`s become entries in `SchemaInfo.rootFields`, each pointing to a complex type by name.

The output is the data structure in `SchemaTypes.h`:

```
SchemaInfo
├── rootElement: "configuration"
├── rootFields: [FormField{name=configuration, type=ComplexType, complexTypeName="configurationType"}]
├── complexTypes:
│     ├── ComplexType{name="configurationType",
│     │     fields=[group-permissions → GroupPermissions, unbounded],
│     │     attributes=[default-permissions → UserPermission, required]}
│     └── ComplexType{name="GroupPermissions",
│           attributes=[group-name: string, permissions: UserPermission]}
└── simpleTypes:
      └── FormField{name="UserPermission",
            type=Enumeration,
            enumValues=["Clearances", "FrequencyManagement", ...]}
```

Crucial detail: the parser **flattens type references**. When it sees an attribute `permissions` of type `UserPermission`, it copies the enum values directly onto the field. The QML layer never has to resolve names — it just sees `model.enumValues`.

### Stage 2 — Schema tree → Qt model tree

`SchemaInfo` is plain C++. QML can't bind to it. So `QmlFormGenerator` converts it into `QAbstractItemModel` subclasses that QML's `Repeater` understands:

```mermaid
classDiagram
    class ElementListModel {
        +rowCount()
        +data(role): elementName
        +getAttributeModel(row) AttributeListModel
        +getChildElementModel(row) ElementListModel
    }
    class AttributeListModel {
        +rowCount()
        +data(role): label, value, unit,<br/>enumValues, isRequired,<br/>defaultValue, description
        +setData(row, value, ValueRole)
    }

    ElementListModel "1" *-- "*" AttributeListModel : per row
    ElementListModel "1" *-- "*" ElementListModel : per row (children)
```

The `unit` role is used as a discriminator the QML layer reads to decide *which input control to instantiate*: `"enum"`, `"complex"`, or anything else (treated as a plain text input). See `ElementInputGenerator.qml`:

```qml
sourceComponent: {
    if (model.unit === "enum")         return enumFieldComponent;   // ComboBox
    else if (model.unit !== "complex") return textFieldComponent;   // TextField
    else                               return null;                 // recurse
}
```

### Stage 3 — Recursive QML rendering

`ElementInputGenerator.qml` is a single component that renders one element:

1. A `Repeater` over the element's `AttributeListModel` emits one row per attribute, each loading a `TextField` or `ComboBox` via a `Loader`.
2. A second `Repeater` over `getChildElementModel(elementIndex)` recurses — each child element triggers another `ElementInputGenerator` instance with `nestingLevel + 1` (controlling indentation and font size).

```mermaid
flowchart TB
    EIG0[ElementInputGenerator<br/>nestingLevel=0<br/>'configuration'] --> A0[default-permissions:<br/>ComboBox UserPermission enum]
    EIG0 --> EIG1[ElementInputGenerator<br/>nestingLevel=1<br/>'group-permissions']
    EIG1 --> A1a[group-name: TextField]
    EIG1 --> A1b[permissions:<br/>ComboBox UserPermission]

    style EIG0 fill:#234,stroke:#88f,color:#fff
    style EIG1 fill:#234,stroke:#88f,color:#fff
```

When the user edits a field, the `TextField.onTextChanged` handler writes back through `attributeModel.setData(idx, text, Qt.UserRole + 7)` — the `ValueRole`. The state lives in the C++ model, not in QML, so `generateXml()` can serialize it on demand.

## 3. The full loop

```mermaid
sequenceDiagram
    participant User
    participant Main as main.qml
    participant App as SchemaEditorApplication (C++)
    participant Parser as XsdSchemaParser
    participant Gen as QmlFormGenerator
    participant Model as ElementListModel
    participant EIG as ElementInputGenerator

    Main->>App: Component.onCompleted: generateElementList()
    App->>App: initializeApplication()
    App->>Parser: parseSchemaString(hardcoded XSD)
    Parser-->>App: SchemaInfo populated
    App->>Gen: generateElementList(schemaInfo)
    Gen->>Model: populate rows + nested models
    Main->>EIG: Repeater binds to elementModel
    EIG->>Model: getAttributeModel(i) / getChildElementModel(i)
    EIG-->>User: render TextField / ComboBox tree

    User->>EIG: edit value
    EIG->>Model: setData(row, value, ValueRole)
    User->>Main: click "Generate XML"
    Main->>App: generateXml()
    App->>Model: walk tree, read values
    App-->>Main: serialized XML string
```

## 4. How this generalises to the aeronautical-editor

The same idea, with a different schema language:

| Concept                | schema-editor-generator       | aeronautical-editor                |
| ---------------------- | ----------------------------- | ---------------------------------- |
| Schema source          | XSD (`<xs:complexType>`, `<xs:simpleType>`, `<xs:enumeration>`) | Protobuf (`message`, `enum`, `repeated`) |
| Type with fields       | `<xs:complexType>` → fields/attributes | `message Foo { ... }` → fields    |
| Enumeration            | `<xs:simpleType>` + `<xs:restriction>` | `enum Bar { ... }`               |
| Repeated value         | `maxOccurs="unbounded"`       | `repeated`                         |
| Required               | `use="required"` / `minOccurs>0` | `required` (proto2) / presence    |
| Documentation/tooltip  | `<xs:annotation><xs:documentation>` | `//` comments or custom options |
| Resulting UI primitive | `TextField` / `ComboBox` / nested group | same set of predefined input widgets |

The protobuf descriptors are introspectable at runtime (every protobuf message carries its own `Descriptor` describing its fields and types). So the aeronautical-editor walks `Descriptor` → builds an equivalent of `SchemaInfo` → reuses the same kind of `ElementListModel` / `AttributeListModel` → renders the same kind of recursive `ElementInputGenerator`-style component.

The big win: when an aeronautical message gains a new field (say a new `RunwayCondition` enum value), the editor picks it up the next time it loads the schema — **no QML edited, no app rebuilt for UI purposes**. Only when you want a *non-default* control for some field (e.g. a map picker for a coordinate) do you write custom QML; everything else is free.

## 5. Limits worth knowing

Looking at the actual code, the demonstrator is intentionally narrow:

- The XSD is **hardcoded** in `getApplicationSchemaString()` — there is no file-loading UI yet. To make this generic you'd wire it to the `load-files-app` pattern or a `WasmFileDialog`.
- `xs:choice` and `xs:all` are treated as `xs:sequence` in `parseComplexType`. No real choice semantics.
- `maxOccurs="unbounded"` is detected in the model (`isArray()`) but the QML doesn't yet render an "Add another" button — a single instance is shown.
- Validation is structural only; there is no XSD `pattern` / `minInclusive` enforcement at input time.
- Date/DateTime fall back to `TextField`.

These are the natural next features when promoting this from a demo into the editor backbone the aeronautical-editor uses for its protobuf equivalent.

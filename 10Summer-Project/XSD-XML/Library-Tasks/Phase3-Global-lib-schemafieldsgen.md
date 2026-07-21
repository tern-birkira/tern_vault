By extracting the architecture from the hackathon prototype (`Field_gen_from_XSD_scheme/qt-wasm-exploration/apps/schema-editor-generator`), we can port its 3-stage metadata pipeline into a permanent, reusable global asset.

The original prototype relied on independent, hardcoded string manipulation. This rewritten plan updates those concepts to utilize the native, high-performance `tern::xml` DOM parsing tools.

To visualize how the hackathon's multi-stage pipeline maps directly into your global shared C++ library classes, see the component breakdown below:

## Phase 3: Global Library — `libs/asd.editor.schemafieldsgen`

This library processes `.xsd` files using the framework's native `tern::xml::XmlDocument` engine to dynamically extract layout structures, metadata validations, and property fields.

### 📋 Detailed Tasks & Subtasks:

#### 🟩 Task 3.1: Module Creation & Build Architecture Alignment

- [ ] **3.1.1:** Build the standardized global directory structure: `libs/asd.editor.schemafieldsgen/src/` and `libs/asd.editor.schemafieldsgen/include/asd.editor.schemafieldsgen/`.
    
- [ ] **3.1.2:** Implement the primary configuration script `libs/asd.editor.schemafieldsgen/CMakeLists.txt` to export the target dependency as `asd::editor::schemafieldsgen`.
    
- [ ] **3.1.3:** Register the directory with the root-level layout mapping file (`libs/CMakeLists.txt`) using the `add_subdirectory(asd.editor.schemafieldsgen)` directive.
    
- [ ] **3.1.4:** Link the library explicitly to the core system compilation dependencies: `asd::editor::commontypes` and the framework's internal `tern::xml` API.
    

#### 🟩 Task 3.2: Implement the Multi-Pass `XsdSchemaParser` (Inspiration: Stage 1 Prototype)

- [ ] **3.2.1:** Create `SchemaTypes.h` inside the public include directory to store the core data containers (`SchemaInfo`, `ComplexType`, and `FormField`) derived from the hackathon data layout.
    
- [ ] **3.2.2:** Structure `XsdSchemaParser.cpp` to use the native DOM loader: `tern::xml::XmlDocument xsdDoc; xsdDoc.load(xsdPath);`.
    
- [ ] **3.2.3:** Establish standard schema namespace evaluations prior to running lookups: `std::string ns = "xs=http://www.w3.org/2001/XMLSchema";`.
    
- [ ] **3.2.4:** **Pass 1 — Simple Types Resolution:** Implement native XPath collection queries via `xsdDoc.selectNodes("//xs:simpleType", ns)` to extract enum definitions (like `FontSizeDelta`). Wrap the native pointer result inside an automated tracking block: `std::unique_ptr<tern::xml::XmlNodeList>`.
    
- [ ] **3.2.5:** **Pass 2 — Complex Types Resolution:** Implement path selections via `xsdDoc.selectNodes("//xs:complexType", ns)` to process properties and layouts (such as elements inside `TrackLabelGenericFieldType`).
    
- [ ] **3.2.6:** **Pass 3 — Root Element Resolution:** Implement lookups via `xsdDoc.selectNodes("//xs:element", ns)` to identify root entries (like the main `<configuration>` element).
    
- [ ] **3.2.7:** **Implement Type-Reference Flattening:** Port the design pattern from the hackathon parser that handles nested type resolution. When processing an element attribute, look up its target type definition and copy the allowable enumeration values array directly onto the field entity descriptor. _This eliminates the need for the QML view layer to perform manual name resolution at runtime._
    

#### 🟩 Task 3.3: Build the Abstract Qt Layout Models (Inspiration: Stage 2 Prototype)

- [ ] **3.3.1:** Create `ElementListModel.h/.cpp` and `AttributeListModel.h/.cpp` as generic subclasses of `QAbstractItemModel` to translate plain C++ structures into layouts that QML view components can easily read.
    
- [ ] **3.3.2:** Configure `AttributeListModel` to expose full metadata configurations via custom Qt item roles:
    
    C++
    
    ```
    enum AttributeRoles {
        LabelRole = Qt::UserRole + 1,
        ValueRole,       // For active values or "${property}" placeholders
        UnitRole,        // Acts as the discriminator ("enum", "complex", "simple")
        EnumValuesRole,  // Direct array of allowable option strings
        IsRequiredRole,  // Maps use="required" constraints
        DefaultValueRole,// Fallback value tracking
        DescriptionRole  // Tooltip descriptions extracted from xs:documentation
    };
    ```
    
- [ ] **3.3.3:** Extract nested tooltip descriptions directly out of the schema definitions using localized native selector lookups: `node.selectSingleNode("xs:annotation/xs:documentation", ns).value();`.
    
- [ ] **3.3.4:** Implement the `unit` discriminator role inside your data models. This parameter tells the QML front-end exactly which input control type to instantiate—such as a dropdown picker for `"enum"`, a nested layout row for `"complex"`, or a text box for standard fields.
    

#### 🟩 Task 3.4: Integrate with Hackathon Architecture References

- [ ] **3.4.1:** Verify that the code logic corresponds with the structure of the prototype located at `Field_gen_from_XSD_scheme/qt-wasm-exploration/apps/schema-editor-generator/src/parsers/`.
    
- [ ] **3.4.2:** Replace the prototype's manual string scanning code with standard framework API calls, like retrieving properties via `XmlElement::getAttribute` and configuring updates via `XmlElement::setAttribute`.
    
- [ ] **3.4.3:** Ensure that `ElementListModel` exposes hierarchical tree methods like `getChildElementModel(int row)` to support clean recursive nesting in your form components.
    


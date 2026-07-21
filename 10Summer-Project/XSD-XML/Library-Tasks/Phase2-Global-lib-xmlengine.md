
This prompt passes along all necessary context regarding your framework's memory rules, the stateless round-trip pattern, and the structural decisions you have established.

### Prompt for LLM Agent: Implementing Phase 2 (`libs/asd.editor.xmlengine`)

**Role:** You are a Senior C++ Software Architect specializing in framework integration and XML processing engine design for high-reliability systems.

**Objective:** Implement the global shared library `libs/asd.editor.xmlengine`. This library is responsible for parsing configuration XML files, extracting specific configuration component blocks (e.g., an individual tracklabel layout matching `tracklabel-type="Correlated"`), and writing modified states back to disk without corrupting or modifying neighboring blocks, whitespace, or file comments.

### 1. Context & Core Framework Contraints (Crucial)

- **The Library Scope:** This library must remain completely **domain-agnostic** and **stateless**. It must not hold internal caches or document states. It takes a file, maps it, extracts the payload, and hands off complete ownership.
    
- **The Framework Lifecycle Reality:** You are wrapping the native `tern::xml` API layer, which acts as a C++ wrapper around a low-level C `libxml2` DOM tree. In this engine, child elements do not own their memory block; the root document completely manages the lifecycle. When a `tern::xml::XmlDocument` goes out of scope, its entire internal element layout is purged.
    
- **The Memory Safeguard:** To prevent dangling pointers, you must package structural results inside the type-safe `asd::editor::commontypes::ConfigurationEnvelope` struct:
    
    C++
    
    ```
    struct ConfigurationEnvelope {
        std::unique_ptr<GenericNode> extractedRoot;
        std::shared_ptr<tern::xml::XmlDocument> masterDocumentContext;
    };
    ```
    
    The `masterDocumentContext` keeps the native DOM tree alive in memory for exactly as long as the consuming application holds onto the envelope.
    

### 2. Implementation Specifications

#### Task A: Implement Pre-Load Schema Validation

Before parsing raw configuration files into editable DOM trees, your importer must run structural validation to ensure input compliance:

1. Instantiate a localized schema compilation workspace using `tern::xml::schema::XmlSchema`.
    
2. Ingest the validation rules using `XmlSchema::read(const std::string& schemaPath)` followed by `XmlSchema::compile()`.
    
3. Validate the incoming target file against this compiled context using `XmlSchema::validate(XmlDocument& doc)`. Trap errors cleanly by wrapping calculations in a try/catch block matching `tern::xml::schema::XmlSchemaValidationException`.
    

#### Task B: Build the Parameter-Targeted `SelectiveImporter`

Implement a parsing module with the following signature:

C++

```
asd::editor::commontypes::ConfigurationEnvelope extractInstance(
    const std::string& xmlFilePath, 
    const std::string& attributeKey, 
    const std::string& targetValue
);
```

1. Initialize an `std::shared_ptr<tern::xml::XmlDocument>` container and run `load(xmlFilePath)` to populate the workspace.
    
2. Build a localized, dynamic XPath selection string targeting the element component by its property attribute rules (e.g., checking `//*[local-name()='tracklabel' and @tracklabel-type='Correlated']`).
    
3. Query the DOM tree using `XmlNode::selectSingleNode`.
    
4. **Fallback Handling for Missing Blocks:** If the query evaluates to `isNull() == true` but the configuration pattern is permitted by the XSD, invoke a fallback snippet handler. Construct a blank, structurally compliant DOM component element layout via `XmlDocument::createElement` and append it to the master root context.
    
5. **Tree Translation:** Recursively parse the selected native subtree into your abstract `GenericNode` format. Map tags via `XmlNode::name()`, attributes via `XmlNode::attributes()`, and loop through sequential row hierarchies using `XmlNode::childNodes()`.
    
    - ⚠️ _Memory Rule:_ Wrap native child collection pointers immediately inside an tracking block (`std::unique_ptr<tern::xml::XmlNodeList>`) to guarantee clean deletion when exiting scope.
        

#### Task C: Build the Component-Level `SelectiveExporter`

Implement a saving module with the following signature:

C++

```
void commitAndSave(
    const asd::editor::commontypes::ConfigurationEnvelope& envelope, 
    const std::string& destinationPath
);
```

1. Avoid granular line/field diff tracking. The engine must re-serialize the _entire modified component block_ (`envelope.extractedRoot`) into a complete native DOM subtree block.
    
2. Convert generic properties and rows back into framework configurations using `XmlElement::setAttribute`.
    
3. Navigate the `masterDocumentContext` to find the stale node entry using your saved configuration parameter selectors.
    
4. Target the item's direct parent using `XmlNode::parentNode()` and replace the entire outdated sub-tree block with the updated element node structure.
    
5. Save the updated configuration cleanly to disk via `XmlDocument::save(destinationPath)`, keeping comments and neighboring sections perfectly intact.
    


## Phase 2: Global Library — `libs/asd.editor.xmlengine`

This library uses the native `tern::xml::XmlDocument` DOM architecture to isolate, parse, and write back targeted configuration component blocks safely.

### 📋 Detailed Tasks & Subtasks:

- [ ] **Task 2.1: Library Setup & Build System Binding**
    
    - [ ] **2.1.1:** Create the directory path: `libs/asd.editor.xmlengine/src/` and `libs/asd.editor.xmlengine/include/`.
        
    - [ ] **2.1.2:** Implement `libs/asd.editor.xmlengine/CMakeLists.txt` exposing target `asd::editor::xmlengine`.
        
    - [ ] **2.1.3:** Link the project target explicitly to the native framework XML dependency libraries.
        
- [ ] **Task 2.2: Implement XSD Pre-Load Schema Validation**
    
    - [ ] **2.2.1:** Create an internal helper module that instantiates `tern::xml::schema::XmlSchema`.
        
    - [ ] **2.2.2:** Load the target validation path using `XmlSchema::read(const std::string& fileName)` followed by `XmlSchema::compile()`.
        
    - [ ] **2.2.3:** Prior to running file parsing operations, execute structural validation against an active configuration document path via `XmlSchema::validate(XmlDocument& doc)`. Trap any validation failures using a clean `try/catch` wrapper over `tern::xml::schema::XmlSchemaValidationException`.
    
    - [ ] **2.2.4** Register valid schema types (for safe internal cpp usage) mapping to xml attr name or xml elem name. Used later for safe XPath searching.
        
- [ ] **Task 2.3: Build the Parameter-Targeted `SelectiveImporter`**
    
    - [ ] **2.3.1:** Implement `SelectiveImporter::extractInstance`, passing the targeted XML path, parameter identifier key (e.g., `"tracklabel-type"`), and target configuration type (e.g., `"Correlated"`).
        
    - [ ] **2.3.2:** Allocate an `std::shared_ptr<tern::xml::XmlDocument>` wrapper context and invoke `load(filename)` to populate the workspace.
        
    - [ ] **2.3.3:** Build a dynamic XPath selector query pattern matching the search payload (e.g., `//*[local-name()='tracklabel' and @tracklabel-type='Correlated']`).
        
    - [ ] **2.3.4:** Execute the target selector lookups using `XmlNode::selectSingleNode`. Verify presence safely by checking `XmlNode::isNull()`.
        
- [ ] **Task 2.4: Convert `tern::xml` Native Subtrees to `GenericNode` Trees**
    
    - [ ] **2.4.1:** Build a iterative transformation method: `std::unique_ptr<GenericNode> transformToGeneric(const tern::xml::XmlNode& nativeNode)`.
        
    - [ ] **2.4.2:** Map element designations via `XmlNode::name()` into your abstract `GenericNode::tagWithNamespace` container.
        
    - [ ] **2.4.3:** Cast valid element assignments via `XmlNode::toElement()` and loop over its properties using `XmlNode::attributes()` to fill your ordered attributes array.
        
    - [ ] **2.4.4:** Query nested child structural paths using `XmlNode::childNodes()`.
        
    - > ⚠️ **CRITICAL MEMORY RULE:** The framework specifies that the caller must delete the resulting list before it goes out of scope. You **must** immediately wrap the output inside an tracking container: `std::unique_ptr<tern::xml::XmlNodeList> children(nativeNode.childNodes());`.
        
    - [ ] **2.4.5:** Loop sequentially across child entries using `children->item(i)` and iteratevly append the transformed references to your configuration payload tree.
        
- [ ] **Task 2.5: Implement the Missing Component "Fallback Generator"**
    *DO NOT THINK THIS IS NEEDED IF VALIDATED CORRECTLY BEFOREHAND* 
    - [ ] **2.5.1:** If `selectSingleNode` evaluates to `isNull() == true` during import, trigger a fallback construction pipeline.
        
    - [ ] **2.5.2:** Read structural rules from the schema database to instantiate a cleanly formatted, structurally compliant template string.
        
    - [ ] **2.5.3:** Convert this raw text snippet into an element node structure via `XmlDocument::createElement` and append it to the master tree layout context so it is available for immediate user customization.
        
- [ ] **Task 2.6: Build the Component-Level `SelectiveExporter`**
    
    - [ ] **2.6.1:** Implement `SelectiveExporter::commitAndSave` to process the updated configuration model.
        
    - [ ] **2.6.2:** (*flat serializeation)* Build a iterative pre-order traversal serialization method: `tern::xml::XmlElement transformToNative(tern::xml::XmlDocument& doc, const GenericNode& genericNode)` to cleanly translate abstract definitions back into native workspace components. 
        
    - [ ] **2.6.3:** Re-map explicit properties and layout arrays back to the target element using `XmlElement::setAttribute`.
        
    - [ ] **2.6.4:** Locate the stale configuration component block inside the master tracking document via your saved XPath context query.
        
    - [ ] **2.6.5:** Swap out the entire old element tree block cleanly by targeting its parent layout node (`XmlNode::parentNode()`) and attaching the updated element node structure.
        
    - [ ] **2.6.6:** Save the finalized configuration data document cleanly to the file system using `XmlDocument::save(filename)`.
        


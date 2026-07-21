

### Phase 1: Core Shared Data Layout (`The Mutual Agreement`)


**Goal:** Create the foundation for cross-module object representation.

> **Prompt:** "Act as a C++ Architect. Implement a library `libs/asd.editor.commontypes` that provides a generic, domain-agnostic tree structure for XML-like configuration data.
> 
> **Requirements:**
> 
> 1. **Data Model:** Create `GenericNode` (with `tag`, `vector<attributes>`, `unique_ptr<children>`) and `GenericAttribute`. Use move-only semantics for `GenericNode` to enforce tree ownership.
>     
> 2. **Envelope:** Implement `ConfigurationEnvelope` containing an `unique_ptr<GenericNode>` (the data) and a `shared_ptr<tern::xml::XmlDocument>` (the context).
>     
> 3. **Forward Declaration:** In `GenericTree.h`, use a forward declaration for `tern::xml::XmlDocument` to avoid header bloat.
> 
> 4. **Context:** The `shared_ptr` to the `XmlDocument` is the 'Memento Token' that keeps the native libxml2 tree memory alive while the UI layer edits the lightweight `GenericNode` tree.
>

#### 🟩 Task 1.1: File System & Build System Integration

- [ ] **1.1.1:** Navigate to your global common types library path: `libs/asd.editor.commontypes/`.
    
- [ ] **1.1.2:** Open or create the domain-agnostic layout header file named `GenericTree.h` inside the include/public directory of that module.
    
- [ ] **1.1.3:** Setup the boilerplate file structure with modern `#ifndef` header guards:
    
    C++
    
    ```
    #ifndef _asd_editor_commontypes_GenericTree_h_
    #define _asd_editor_commontypes_GenericTree_h_
    ```
    
- [ ] **1.1.4:** Include the essential Standard Library dependency headers: `<string>`, `<vector>`, and `<memory>`.
    
- [ ] **1.1.5:** Declare the shared root namespace boundary: `namespace asd::editor::commontypes`.
    
- [ ] **1.1.6:** Open `libs/asd.editor.commontypes/CMakeLists.txt` and ensure `GenericTree.h` is added to the public headers export list so peer modules can discover it during compilation.
    

#### 🟩 Task 1.2: Define the `GenericAttribute` Primitive Struct

- [ ] **1.2.1:** Declare `struct GenericAttribute` inside the namespace to model an explicit XML attribute pair.m
    
- [ ] **1.2.2:** Add `std::string name;` to store the raw attribute name string (e.g., `"font-adjustment"` or `"tracklabel-type"`).
    
- [ ] **1.2.3:** Add `std::string value;` to store the attribute value text. This field must explicitly store un-evaluated properties (such as runtime tokens like `${pasd.tar.tracklabel.flightrules.visible}`) exactly as literal strings to bypass accidental validation failures.
    
- [ ] **1.2.4:** Write a default constructor and a clean parameterized constructor utilizing sink arguments and `std::move` to avoid duplicate heap allocations:
    
    C++
    
    ```
    GenericAttribute(std::string attrName, std::string attrValue)
        : name(std::move(attrName)), value(std::move(attrValue)) {}
    ```
    

#### 🟩 Task 1.3: Define the Hierarchical `GenericNode` Struct

- [ ] **1.3.1:** Declare `struct GenericNode` to act as an abstract element in the object tree.
    
- [ ] **1.3.2:** Add `std::string tagWithNamespace;` to capture raw structural tag descriptions (e.g., `"tlabel:field"` or `"tlabel:tracklabel-line"`).
    
- [ ] **1.3.3:** Add `std::vector<GenericAttribute> attributes;` using an ordered vector to guarantee that attribute placement remains unchanged from disk read to rewrite.
    
- [ ] **1.3.4:** Add `std::vector<std::unique_ptr<GenericNode>> childNodes;` using `std::unique_ptr` to enforce single-ownership semantics and preserve exact sequential ordering of mixed children (like alternating `<tlabel:field>` and `<tlabel:field18>` blocks).
    
- [ ] **1.3.5:** Add an optional `std::string innerText;` field to safely carry plain text values inside elements that don't rely on nested structures.
    
- [ ] **1.3.6:** **Enforce Move-Only Semantics:** Because `std::unique_ptr` cannot be copied, explicitly delete the copy constructor and copy assignment operator to prevent runtime duplication bugs:
    
    C++
    
    ```
    GenericNode(const GenericNode&) = delete;
    GenericNode& operator=(const GenericNode&) = delete;
    ```
    
- [ ] **1.3.7:** Implement default move constructors and move assignment operators so elements can be seamlessly processed across vectors:
    
    C++
    
    ```
    GenericNode(GenericNode&&) noexcept = default;
    GenericNode& operator=(GenericNode&&) noexcept = default;
    ```

- [ ] **1.3.8** implement as a iterative approach, make the tree a bidirectional graph, each child has a parent pointer as a raw pointer (doesn't effect unuiqe ptr).

To make your `GenericNode` tree work flawlessly with `QAbstractItemModel` and provide the "living" interface needed for a high-performance editor, your Mutation API needs to be **atomically safe**.

Because `QAbstractItemModel` requires you to notify it before and after structural changes (using `beginInsertRows`, `endInsertRows`, etc.), your `GenericNode` should not just modify itself; it must support the **Observer Pattern** or provide clear **Hook Points** for your `QAbstractListModel` wrapper to trigger those updates.

Here are the high-level, editor-ready mutation methods:

### 1. Structural Manipulation Methods

These methods modify the tree topology. They must handle the `parent` pointer logic internally to keep the tree state consistent.

- **`void moveChild(int oldIndex, int newIndex)`**
    
    - **Logic:** Reorders the child within the `childNodes` vector.
        
    - **Why:** Essential for list-based UI where a user drags a field up or down to change the visual display order.
        
    - **Editor Hook:** After the move, the parent model must call `dataChanged` for the affected range.
        
- **`void swapNodes(GenericNode* nodeA, GenericNode* nodeB)`**
    
    - **Logic:** Requires `nodeA` and `nodeB` to share the same parent and have identical `tagWithNamespace` strings. It swaps their positions in the `childNodes` vector and updates their internal indices.
        
    - **Constraint:** If the tags do not match, the method **must** throw an `std::invalid_argument` or log a critical failure. Swapping a `<tlabel:field>` with a `<tlabel:tracklabel-line>` would violate your XSD structure.
        
    - **Why:** Allows for rapid layout reordering without full re-serialization.
        
- **`std::unique_ptr<GenericNode> detachChild(GenericNode* node)`**
    
    - **Logic:** Removes the node from the `childNodes` vector and sets its `parent` pointer to `nullptr`.
        
    - **Why:** The "Cut" operation in a Cut/Copy/Paste workflow.
        
    - **Editor Hook:** The parent model must call `beginRemoveRows` before this runs and `endRemoveRows` immediately after.
        
- **`void attachChild(std::unique_ptr<GenericNode> node, int index)`**
    
    - **Logic:** Inserts the node into the `childNodes` vector at the specific index and sets its `parent` pointer to `this`.
        
    - **Why:** The "Paste" or "Drop" operation.
        
    - **Editor Hook:** The parent model must call `beginInsertRows` before and `endInsertRows` after.
        

### 2. Property & Data Mutation Methods

These methods modify the _content_ of a node without changing the tree structure.

- **`void setAttribute(const std::string& name, const std::string& value)`**
    
    - **Logic:** Iterates the `attributes` vector. If the name exists, update the value. If not, push a new `GenericAttribute` object.
        
    - **Editor Hook:** After updating, the model should emit `dataChanged` specifically for the `ValueRole` to refresh the QML input field.
        
- **`void setInnerText(const std::string& text)`**
    
    - **Logic:** Replaces the existing string content.
        
    - **Editor Hook:** Used primarily for elements that do not contain child nodes but store configuration data in the text body.
        

### 3. Safety & Integrity Methods

These methods prevent the editor from entering an invalid state.

- **`bool canAcceptChild(const std::string& tag)`**
    
    - **Logic:** A predicate that checks the schema rules to see if this node type is permitted to contain the proposed child tag.
        
    - **Why:** Prevents the user from dropping an invalid field into a configuration block, effectively enforcing XSD structural rules live in the UI.
        
- **`bool isAncestorOf(GenericNode* potentialDescendant)`**
    
    - **Logic:** A simple upward-traversal loop that walks the `parent` pointers until it hits `nullptr` or matches the target node.
        
    - **Why:** **Critical** for Drag-and-Drop. You must never allow a user to drop a parent node into one of its own children. This check ensures the tree remains a Directed Acyclic Graph (DAG) and never loops.
        

### How to integrate with `QAbstractListModel`

You should **not** call these methods directly from your QML code. Instead:

1. The `QAbstractListModel` subclass wraps a `GenericNode`.
    
2. When the QML UI sends an update request, the Model calls these `GenericNode` mutation methods.
    
3. **Before/After the call:** The Model wraps these methods in the mandatory `beginInsertRows()` / `endInsertRows()` or `beginMoveRows()` / `endMoveRows()` calls.
    

This ensures that the `QML ListView` or `Repeater` always stays perfectly in sync with your C++ memory model without the risk of visual out-of-sync errors.
#### 🟩 Task 1.4: Implement the `ConfigurationEnvelope` Container

- [ ] **1.4.1:** Introduce a clean forward declaration for your native framework XML document class at the top of `GenericTree.h` to decouple your common types header from parsing framework inclusions:
    
    C++
    
    ```
    namespace tern::xml { class XmlDocument; }
    ```
    
- [ ] **1.4.2:** Declare `struct ConfigurationEnvelope` to package configuration components for layout-agnostic round-trip processing.
    
- [ ] **1.4.3:** Add `std::unique_ptr<GenericNode> extractedRoot;` to act as the primary structural handle for the isolated element tree (such as an extracted tracklabel layout block).
    
- [ ] **1.4.4:** **[REWRITTEN]** Replace the unsafe C-style `void*` completely by declaring a type-safe `std::shared_ptr` tracking the native framework master document context:
    
    C++
    
    ```
    std::shared_ptr<tern::xml::XmlDocument> masterDocumentContext;
    ```
    
    _This ensures that as long as the envelope exists in memory, the underlying native libxml2 tree context is safe from premature destruction._
    

#### 🟩 Task 1.5: Compilation Verification Pass

- [ ] **1.5.1:** Write a minimal dummy C++ source file locally or use an existing testing harness to include `GenericTree.h`.
    
- [ ] **1.5.2:** Verify that the structures compile cleanly under your target configuration (C++17/C++20 standards).
    
- [ ] **1.5.3:** Assert that move execution behaves correctly by testing a structural transfer via `std::move`:
    
    C++
    
    ```
    asd::editor::commontypes::ConfigurationEnvelope env;
    env.extractedRoot = std::make_unique<asd::editor::commontypes::GenericNode>();
    // Ensure moving the envelope transfers ownership cleanly
    asd::editor::commontypes::ConfigurationEnvelope destination = std::move(env);
    ```
    
- [ ] **1.5.4:** Complete the verification pass and check for any header inclusion gaps before proceeding directly to the layout implementation of your global `libs/asd.editor.xmlengine` library.


### The Round-Trip Lifecycle in Action (layout-agnostic round-trip processing)

1. **Import:** The XML Engine opens the file into the `masterDocumentContext`. It uses an XPath query to locate the `Correlated` section, clones it, translates it into the lightweight `GenericNode` tree, and assigns it to `extractedRoot`. It hands this complete envelope to your controller.
    
2. **Modify:** Your QML editor modifies properties inside the `extractedRoot` tree (e.g., toggling a checkbox or changing a margin value). The heavy `masterDocumentContext` sits completely untouched in the background.
    
3. **Export:** When the user clicks "Save", your controller hands the envelope back to the XML Engine. The engine translates the modified `extractedRoot` back into native framework elements. It uses the `masterDocumentContext` to find the exact location of the old `Correlated` block, swaps it out entirely with the new one, and saves the master document back to disk.
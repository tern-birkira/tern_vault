
**Adapter/Wrapper architecture**.

## Phase 4: Application Orchestration — `asd.editor.tracklabeleditor`

This phase wires your domain-agnostic processing engines (the `xmlinterface` tree and `xmlengine` stateless bridge) directly into your interactive user interface application views.

### 📋 Detailed Tasks & Subtasks

#### 🟩 Task 4.1: Compilation Linking & Dependency Injection

- [ ] **4.1.1:** Update `apps/track-label-editor/CMakeLists.txt` to include `asd.editor.xmlinterface`, `asd.editor.xmlengine`, and `asd.editor.schemafieldsgen` as mandatory link dependencies.
    
- [ ] **4.1.2:** Configure `target_include_directories` to expose global `include/` paths, ensuring the application resolves `asd.editor.xmlinterface/GenericTree.h` without path ambiguity.
    
- [ ] **4.1.3:** Remove legacy custom XML parsing files from the build pipeline to prevent duplicate configuration initialization.
    
- [ ] **4.1.4:** Update the application context factory registration to ensure the new global modules are initialized before the application controller starts.
    

#### 🟩 Task 4.2: Integrate the Unified File Initialization Pipeline

- [ ] **4.2.1:** Modify `TrackLabelEditorController.cpp` to hold the master instance of the `ConfigurationEnvelope` for the active editing session.
    
- [ ] **4.2.2:** Access target configuration path pointers using the application context manager:
    
    C++
    
    ```
    auto* pathConfig = tern::runtime::ApplicationContext::getInstance<asd::editor::config::PathConfig*>("Editor-paths");
    std::string xmlPath = pathConfig->getImportFilePath() + "/polaris-asd-tracklabel-config.xml";
    ```
    
- [ ] **4.2.3:** Implement a "Load-and-Bind" routine in the controller: Use `xmlengine::SelectiveImporter` to ingest `xmlPath`, generate the `ConfigurationEnvelope`, and pass the resulting tree root pointer to the main `TrackLabelModel`.
    
- [ ] **4.2.4:** Implement a "Write-Back" hook: Register a signal/slot connection between the controller and the UI save button that triggers `xmlengine::SelectiveExporter::commitAndSave(envelope)` to serialize the modified tree back to disk.
    

#### 🟩 Task 4.3: **[REWRITTEN]** Refactor Qt Models into Pure Stateless Adapters

- [ ] **4.3.1:** Open `TrackLabelModel.h` and `FieldListModel.h`. Completely delete the parallel storage vectors `QVector<FieldListModel*> m_rows` and `QVector<FieldInterface*> m_fields` to prevent dual-source-of-truth bugs.
    
- [ ] **4.3.2:** Refactor `FieldInterface` so it does not store internal copies of variables. Give it a constructor that accepts a raw pointer to a backend `GenericNode*`.
    
- [ ] **4.3.3:** Rewrite `FieldInterface` property getters and setters to query the underlying node dynamically using runtime lookups:
    
    C++
    
    ```
    // Data flows directly to/from the single source of truth tree
    QString prefix() const { return QString::fromStdString(m_node->getAttribute("prefix")); }
    ```
    
- [ ] **4.3.4:** Rewrite `FieldListModel::rowCount()` to return `m_lineNode->childNodes.size()` directly from the backend tree. Do the same for `TrackLabelModel::rowCount()` against the configuration rows.
    
- [ ] **4.3.5:** Rewrite `FieldListModel::data()` to fetch the child node matching `index.row()`, wrap it in a lightweight transient `FieldInterface` instance (or look up a view adapter pointer), and expose it via `FieldObjectRole`.
    
- [ ] **4.3.6:** **[CRITICAL]** Refactor `FieldListModel::requestMoveField` to use Qt's native structural view notification hooks wrapped tightly around the backend tree mutation:
    
    C++
    
    ```
    // 1. Notify the QML view a structural change is starting
    beginMoveRows(QModelIndex(), colIndex, colIndex, QModelIndex(), targetIndex);
    // 2. Perform the actual swap EXCLUSIVELY on the underlying backend data tree
    m_lineNode->swapChildren(colIndex, targetIndex);
    // 3. Notify the QML view the structural change is complete
    endMoveRows();
    ```
    

### Why this version is safe

By forcing the tasks to focus on **deleting the duplicate vectors** (`m_rows` and `m_fields`) and routing data access dynamically, your data structure remains completely flexible.

When you implement the **Flight List Editor** later on, you can reuse `asd.editor.xmlengine` and `asd.editor.xmlinterface` without changing a single line of backend storage code, exactly fulfilling your core design requirements.
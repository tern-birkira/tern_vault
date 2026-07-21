# Implementation Tasks: Import/Export Connection for Track Label Editor

  

This document connects the file-system layer (load/save) to the model tree via the existing `TrackLabelLoader`/`TrackLabelExporter` and the `TrackLabelEditorController`.

  

---

  

## Component Separation of Roles

  

```

+-------------------------------------------------------------+

| TrackLabelFileBrowserController |

| - Inherits: FileBrowserControllerBase |

| - Boundary: Handles UI dialogue buttons, paths, filters |

+------------------------------+------------------------------+

|

Triggers File Operation

v

+-------------------------------------------------------------+

| TrackLabelEditorController |

| - Core Orchestrator |

| - Action: Spawns empty canvas, runs stateless Loader |

| - Updates: Lifecycle container and re-binds view layout |

+------------------------------+------------------------------+

|

Mutates / Updates Session

v

+-------------------------------------------------------------+

| TrackLabelEditorInstantiator |

| - Pure Lifecycle Container & QML Property Exposer |

| - Ownership: Keeps heap root config tree alive |

+-------------------------------------------------------------+

  

```

  

---

  

## Phase 1: Instantiator Owns the Config Model

  

### Task 1.1: Add `configModel` property to Instantiator

  

* **File**: `TrackLabelEditorInstantiator.h`

* **Action**: Expose the active configuration model root as a `Q_PROPERTY`. The instantiator owns the lifetime.

* **Implementation**:

```cpp

Q_PROPERTY( models::TrackLabelConfigModel* configModel READ configModel NOTIFY configModelChanged )

  

public:

[[nodiscard]] models::TrackLabelConfigModel* configModel() const { return m_configModel.get(); }

  

/** Replaces the active config tree. Old tree is destroyed. */

void setConfigModel( std::unique_ptr<models::TrackLabelConfigModel> model );

  

signals:

void configModelChanged();

  

private:

std::unique_ptr<models::TrackLabelConfigModel> m_configModel;

```

  

### Task 1.2: Implement `setConfigModel`

  

* **File**: `TrackLabelEditorInstantiator.cpp`

* **Action**:

```cpp

void TrackLabelEditorInstantiator::setConfigModel( std::unique_ptr<models::TrackLabelConfigModel> model )

{

m_configModel = std::move( model );

emit configModelChanged();

}

```

  

---

  

## Phase 2: Controller Wiring

  

### Task 2.1: Keep existing mock allocations (to be deleted later)

  

* **File**: `TrackLabelEditorController.cpp`

* **Action**: Add a comment above `initializeLabelsPerType()` body:

```cpp

// ponytail: delete initializeLabelsPerType() once loader is connected via loadConfiguration()

```

Do NOT delete yet — QML still binds to these layouts.

  

### Task 2.2: Add `syncViewLayoutMappings`

  

* **File**: `TrackLabelEditorController.h`, `TrackLabelEditorController.cpp`

* **Action**: Reads tracklabel children from the config model and populates the existing `m_layouts` hash.

* **Implementation**:

```cpp

void TrackLabelEditorController::syncViewLayoutMappings( models::TrackLabelConfigModel* config )

{

m_layouts.clear();

clearSelection();

  

if( !config )

{

emit currentLayoutChanged();

return;

}

  

for( models::TrackLabelVariantModel* variant : config->tracklabels() )

{

if( !variant )

continue;

  

// Map TrackLabelTypeVariant → LabelType for the existing hash

const auto type = static_cast<commontypes::tracklabel::LabelType>(

static_cast<int>( variant->tracklabelType() ) );

m_layouts[ type ] = variant;

}

  

emit currentLayoutChanged();

reevaluateActiveLayoutVisibility();

}

```

  

Note: The cast between `TrackLabelTypeVariant` and `LabelType` assumes matching enum values. Verify or add an explicit conversion.

  

### Task 2.3: Add `loadConfiguration` method

  

* **File**: `TrackLabelEditorController.h`, `TrackLabelEditorController.cpp`

* **Action**:

```cpp

void TrackLabelEditorController::loadConfiguration( const QString& filePath, const QString& instanceId )

{

auto* instantiator = qobject_cast<TrackLabelEditorInstantiator*>( parent() );

if( !instantiator )

return;

  

instantiator->setIsLoading( true );

  

// Empty canvas — no children allocated by constructor

auto freshConfig = std::make_unique<models::TrackLabelConfigModel>();

  

// Stateless one-shot load

QStringList warnings = serialization::TrackLabelLoader::loadFromFile(

filePath.toStdString(),

instanceId.toStdString(),

freshConfig.get() );

  

// Transfer ownership to instantiator

instantiator->setConfigModel( std::move( freshConfig ) );

  

// Rebuild layout lookup from loaded data

syncViewLayoutMappings( instantiator->configModel() );

  

instantiator->setIsLoading( false );

}

```

  

### Task 2.4: Add `saveConfiguration` method

  

* **File**: `TrackLabelEditorController.h`, `TrackLabelEditorController.cpp`

* **Action**:

```cpp

void TrackLabelEditorController::saveConfiguration( const QString& filePath )

{

auto* instantiator = qobject_cast<TrackLabelEditorInstantiator*>( parent() );

if( !instantiator || !instantiator->configModel() )

return;

  

serialization::TrackLabelExporter::saveToFile(

instantiator->configModel(),

filePath.toStdString() );

}

```

  

---

  

## Phase 3: File Browser Controller

  

### Task 3.1: Create `TrackLabelFileBrowserController`

  

* **File**: `controllers/TrackLabelFileBrowserController.h`

* **Action**: Inherit from `FileBrowserControllerBase`. Route import/export to the orchestrator.

```cpp

#include "asd.editor.filebrowser/FileBrowserControllerBase.h"

  

class TrackLabelFileBrowserController : public asd::editor::filebrowser::FileBrowserControllerBase

{

Q_OBJECT

public:

explicit TrackLabelFileBrowserController(

TrackLabelEditorController* orchestrator,

const QString& importPath,

QObject* parent = nullptr );

  

void onSelectedFilenameChanged() override;

bool exportConfigToSelectedFile() const override;

bool importConfigFromSelectedFile() override;

bool exportNewConfigFile( QString name, QString folderUrl ) override;

  

private:

TrackLabelEditorController* m_orchestrator;

};

```

  

### Task 3.2: Implement File Browser routing

  

* **File**: `controllers/TrackLabelFileBrowserController.cpp`

* **Action**:

```cpp

bool TrackLabelFileBrowserController::importConfigFromSelectedFile()

{

m_orchestrator->loadConfiguration( selectedFilenameWithPath(), "polaris-asd-tracklabel" );

return true;

}

  

bool TrackLabelFileBrowserController::exportConfigToSelectedFile() const

{

m_orchestrator->saveConfiguration( selectedFilenameWithPath() );

return true;

}

  

bool TrackLabelFileBrowserController::exportNewConfigFile( QString name, QString folderUrl )

{

const QString path = QUrl( folderUrl ).toLocalFile() + "/" + name;

m_orchestrator->saveConfiguration( path );

return true;

}

```

  

### Task 3.3: Wire into Instantiator

  

* **File**: `TrackLabelEditorInstantiator.h/cpp`

* **Action**: Add a `TrackLabelFileBrowserController*` member, expose via Q_PROPERTY, construct in the instantiator constructor with the import path from `PathConfig`.

  

---

  

## Execution Order

  

1. Task 1.1–1.2 (Instantiator config ownership)

2. Task 2.1 (Comment mock allocations)

3. Task 2.2–2.4 (Controller load/save/sync)

4. Task 3.1–3.3 (File browser controller)

5. Remove `initializeLabelsPerType()` once QML binds to loaded data
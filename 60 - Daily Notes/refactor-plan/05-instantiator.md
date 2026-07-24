# TrackLabelEditorInstantiator — Refactored

## Responsibility

Factory + coordinator. Creates all controllers, holds shared_ptrs (ownership),
wires cross-controller signals, exposes Q_PROPERTYs to QML.

## Changes Summary

- Remove: `shared_ptr<TrackLabelEditorController> m_trackLabelEditorController`
- Remove: `unique_ptr<TrackLabelConfigModel> m_configModel` (moves to IOController)
- Add: `shared_ptr<TrackLabelRuntimeDataController> m_runtimeDataController`
- Add: `shared_ptr<TrackLabelDataController> m_dataController`
- Add: `shared_ptr<TrackLabelIOController> m_ioController`
- Change: `m_fileBrowserController` constructor args

## Q_PROPERTYs (post-refactor)

```cpp
Q_PROPERTY(bool isCombinedEditor READ isCombinedEditor CONSTANT)
Q_PROPERTY(QUndoStack* undoStack READ undoStack CONSTANT)
Q_PROPERTY(bool isLoading READ isLoading NOTIFY isLoadingChanged)
Q_PROPERTY(bool dataHasBeenLoaded READ dataHasBeenLoaded NOTIFY dataLoaded)

Q_PROPERTY(controllers::TrackLabelRuntimeDataController* runtimeDataController READ runtimeDataController CONSTANT)
Q_PROPERTY(controllers::TrackLabelDataController* dataController READ dataController CONSTANT)
Q_PROPERTY(controllers::TrackLabelIOController* ioController READ ioController CONSTANT)
Q_PROPERTY(controllers::TrackLabelFileBrowserController* fileBrowserController READ fileBrowserController CONSTANT)

Q_PROPERTY(models::FieldsModel* allFields READ allFields CONSTANT)
Q_PROPERTY(models::FieldsProxyModel* fieldsProxyModel READ fieldsProxyModel CONSTANT)
Q_PROPERTY(models::TrackLabelConfigModel* configModel READ configModel NOTIFY configModelChanged)
```

## Construction Order

```cpp
TrackLabelEditorInstantiator::TrackLabelEditorInstantiator(...)
{
    // 1. Runtime data controller (no deps)
    m_runtimeDataController = std::make_shared<controllers::TrackLabelRuntimeDataController>(this);

    // 2. Data controller (depends on runtimeData)
    m_dataController = std::make_shared<controllers::TrackLabelDataController>(
        m_runtimeDataController, this);

    // 3. IO controller (depends on dataController + runtimeDataController)
    m_ioController = std::make_shared<controllers::TrackLabelIOController>(
        m_dataController, m_runtimeDataController, this);

    // 4. Initial default config
    m_ioController->initDefaultConfig();  // creates default TrackLabelConfigModel + syncs to data controller

    // 5. File browser (depends on ioController)
    const QString importPath = QString::fromStdString(m_applicationConfig->getPathConfig().getImportFilePath());
    m_fileBrowserController = std::make_shared<controllers::TrackLabelFileBrowserController>(
        m_ioController, importPath, this);

    // 6. Fields model (unchanged)
    m_allFields = std::make_shared<models::FieldsModel>(this);
    m_fieldsProxyModel = std::make_shared<models::FieldsProxyModel>(this);
    // ... same proxy setup as today ...

    // 7. Signal wiring
    connect(m_runtimeDataController.get(), &controllers::TrackLabelRuntimeDataController::changed,
            m_dataController.get(), &controllers::TrackLabelDataController::reevaluateActiveLayoutVisibility);

    connect(m_dataController.get(), &controllers::TrackLabelDataController::activeTypeChanged,
            m_dataController.get(), &controllers::TrackLabelDataController::reevaluateActiveLayoutVisibility);

    connect(m_fieldsProxyModel.get(), &models::FieldsProxyModel::addFieldRequested,
            m_dataController.get(), &controllers::TrackLabelDataController::addFieldToCurrentLayout);

    connect(m_ioController.get(), &controllers::TrackLabelIOController::configurationLoaded,
            this, [this]() {
                m_allFields->syncAvailability(m_ioController->configModel());
                emit configModelChanged();
                if (!m_dataHasBeenLoaded) {
                    m_dataHasBeenLoaded = true;
                    emit dataLoaded();
                }
            });

    // 8. QML registration
    qmlRegisterSingletonInstance("ASD.Editor.TrackLabelEditor", 1, 0, "TrackLabelEditorInstantiator", this);
}
```

## Removed

- `trackLabelEditorController` Q_PROPERTY and getter
- `setConfigModel()` method (IOController owns it now)
- Direct `m_configModel` member

## QML Migration

QML code that currently uses `TrackLabelEditorInstantiator.trackLabelEditorController` changes to:
- `.trackLabelEditorController.activeType` → `.dataController.activeType`
- `.trackLabelEditorController.currentLayout` → `.dataController.currentLayout`
- `.trackLabelEditorController.selectedField` → `.dataController.selectedField`
- `.trackLabelEditorController.selectField(...)` → `.dataController.selectField(...)`
- `.trackLabelEditorController.clearSelection()` → `.dataController.clearSelection()`

## Implementation Steps

1. Add new member declarations and Q_PROPERTYs
2. Rewrite constructor (new creation order + wiring)
3. Remove old m_trackLabelEditorController references
4. Move configModel ownership delegation to ioController
5. Update isLoading to delegate to ioController
6. Update QML references in .qml files

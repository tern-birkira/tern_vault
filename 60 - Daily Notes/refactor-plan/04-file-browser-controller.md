# TrackLabelFileBrowserController — Refactored

## Responsibility

File selection UI only. Delegates actual IO to IOController.
Matches the pattern: FileBrowserController is a UI adapter, not a data processor.

## Namespace

`asd::editor::tracklabeleditor::controllers`

## File Location (existing, modified)

`controllers/TrackLabelFileBrowserController.h`
`controllers/TrackLabelFileBrowserController.cpp`

## Changes from current

### Current (before refactor)
```cpp
TrackLabelFileBrowserController(
    TrackLabelEditorController* orchestrator,  // ← owns pointer to monolith
    const QString& importPath,
    QObject* parent);
```

Calls `m_orchestrator->loadConfiguration()` / `m_orchestrator->saveConfiguration()`.

### After refactor
```cpp
TrackLabelFileBrowserController(
    std::shared_ptr<TrackLabelIOController> ioController,  // ← delegates to IOController
    const QString& importPath,
    QObject* parent);
```

Calls `m_ioController->loadConfiguration()` / `m_ioController->saveConfiguration()`.

## Interface (post-refactor)

```cpp
class TrackLabelFileBrowserController : public asd::editor::filebrowser::FileBrowserControllerBase
{
Q_OBJECT

public:
    explicit TrackLabelFileBrowserController(
        std::shared_ptr<TrackLabelIOController> ioController,
        const QString& importPath,
        QObject* parent = nullptr);

    void onSelectedFilenameChanged() override;

public slots:
    bool exportConfigToSelectedFile() const override;
    bool importConfigFromSelectedFile() override;
    bool exportNewConfigFile(QString name, QString folderUrl) override;

private:
    std::shared_ptr<TrackLabelIOController> m_ioController;
};
```

## Implementation (trivial delegation)

```cpp
bool TrackLabelFileBrowserController::importConfigFromSelectedFile()
{
    if (!m_ioController) return false;
    m_ioController->loadConfiguration(selectedFilenameWithPath(), "polaris-asd-tracklabel");
    return true;
}

bool TrackLabelFileBrowserController::exportConfigToSelectedFile() const
{
    if (!m_ioController) return false;
    m_ioController->saveConfiguration(selectedFilenameWithPath());
    return true;
}

bool TrackLabelFileBrowserController::exportNewConfigFile(QString name, QString folderUrl)
{
    if (!m_ioController) return false;
    const QString path = QUrl(folderUrl).toLocalFile() + "/" + name;
    m_ioController->saveConfiguration(path);
    return true;
}
```

## Implementation Steps

1. Change constructor: replace `TrackLabelEditorController*` with `shared_ptr<TrackLabelIOController>`
2. Replace `m_orchestrator->loadConfiguration(...)` → `m_ioController->loadConfiguration(...)`
3. Replace `m_orchestrator->saveConfiguration(...)` → `m_ioController->saveConfiguration(...)`
4. Remove include of `TrackLabelEditorController.h`, add `TrackLabelIOController.h`

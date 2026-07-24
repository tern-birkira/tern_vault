# TrackLabelIOController

## Responsibility

Manages ALL IO operations for the track label editor.
Single point of entry for loading/saving configs, loading .properties,
and importing runtime data from external sources.

## Namespace

`asd::editor::tracklabeleditor::controllers`

## File Location

`controllers/TrackLabelIOController.h`
`controllers/TrackLabelIOController.cpp`

## Interface

```cpp
class TrackLabelIOController : public QObject
{
Q_OBJECT
Q_PROPERTY(bool isLoading READ isLoading NOTIFY isLoadingChanged)

public:
    explicit TrackLabelIOController(
        std::shared_ptr<TrackLabelDataController> dataController,
        std::shared_ptr<TrackLabelRuntimeDataController> runtimeDataController,
        QObject* parent = nullptr);

    // --- Config IO ---
    Q_INVOKABLE void loadConfiguration(const QString& filePath, const QString& instanceId);
    Q_INVOKABLE void saveConfiguration(const QString& filePath);

    // --- Loading state ---
    bool isLoading() const;

signals:
    void isLoadingChanged();
    void configurationLoaded();   // emitted after successful load
    void configurationSaved();    // emitted after successful save

private:
    std::shared_ptr<TrackLabelDataController> m_dataController;
    std::shared_ptr<TrackLabelRuntimeDataController> m_runtimeDataController;

    bool m_isLoading = false;
    void setIsLoading(bool loading);
};
```

## Current Implementation (Phase 1 — today)

Extracts load/save from current `TrackLabelEditorController::loadConfiguration()` and
`saveConfiguration()`.

```cpp
void TrackLabelIOController::loadConfiguration(const QString& filePath, const QString& instanceId)
{
    setIsLoading(true);

    QStringList warnings;
    auto freshConfig = serialization::TrackLabelLoader::loadFromFile(
        filePath.toStdString(), instanceId.toStdString(), warnings);

    if (freshConfig)
    {
        // Transfer ownership to Instantiator via signal or direct call
        // DataController gets wired to the new model tree
        m_dataController->syncViewLayoutMappings(freshConfig.get());
    }

    setIsLoading(false);
    emit configurationLoaded();
}
```

### Config model ownership question

Currently `TrackLabelEditorInstantiator` owns the `unique_ptr<TrackLabelConfigModel>`.
The IOController needs to set it on load. Options:
- **A)** IOController gets a raw pointer to Instantiator (same parent-cast pattern as today) ← simple, matches current
- **B)** IOController owns the configModel directly ← cleaner ownership, IOController is the source of truth for loaded data
- **C)** IOController emits signal with the new config, Instantiator receives and stores it

**Decision: B** — IOController owns `unique_ptr<TrackLabelConfigModel>`.
It's the one who loads it, it should own it. Instantiator exposes it via
`Q_PROPERTY(TrackLabelConfigModel* configModel READ configModel NOTIFY configModelChanged)`
that delegates to `ioController->configModel()`.

## Future Extensions (not implemented now, but this is where they land)

### .properties support
```cpp
    // Load placeholder→value mappings from .properties file
    void loadProperties(const QString& filePath);

    // Access the placeholder map (used by serializer)
    QHash<QString, QString> placeholderMap() const;

    // Save .properties alongside config
    void saveProperties(const QString& filePath);
```

### Runtime data import
```cpp
    // Import runtime values from external source into RuntimeDataController
    void importRuntimeValues(const QVariantMap& values);
    // Could accept: controlState, holding, show-if field values, etc.
    // Pushes them into m_runtimeDataController setters
```

## Dependencies

- `shared_ptr<TrackLabelDataController>` (to call syncViewLayoutMappings)
- `shared_ptr<TrackLabelRuntimeDataController>` (for future runtime import)
- `serialization/TrackLabelLoader.h` (existing)
- `serialization/TrackLabelExporter.h` (existing)
- `models/TrackLabelConfigModel.h` (owns it)

## Implementation Steps

1. Create header with load/save declarations + isLoading property
2. Create cpp — move loadConfiguration / saveConfiguration bodies from TrackLabelEditorController
3. Own configModel (move unique_ptr from Instantiator → IOController)
4. Emit configurationLoaded signal (Instantiator can react: update FieldsModel availability, emit dataLoaded)

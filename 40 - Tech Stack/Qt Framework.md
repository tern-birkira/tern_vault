# Qt Framework

*Overview of Qt as used in `polaris-asd-editor`*
*See also: [[C++ Guidelines]] | [[QML Guidelines]] | [[CMake Build System]]*

---

## What Is Qt?

Qt = cross-platform C++ framework for building GUI applications.

Used in Polaris and all its editors because:
- Native-feeling UI on Linux/Windows/macOS
- Strong XML/JSON support
- Built-in model/view architecture (good for config editors)
- Compiles to WebAssembly (for browser deployment — see [[Summer Project Overview]])

---

## Key Modules Used in This Project

| Module | What it provides |
|--------|-----------------|
| `Qt::Core` | QString, QFile, QList, signals/slots, QObject |
| `Qt::Widgets` | Traditional desktop widgets (QTreeView, QTableView, etc.) |
| `Qt::Quick` | QML-based UI (declarative, modern) |
| `Qt::Xml` | `QXmlStreamReader`, `QDomDocument` — XML parsing |
| `Qt::Network` | Network I/O (if needed) |

CMake usage:
```cmake
target_link_libraries(my-target PRIVATE
    Qt6::Core
    Qt6::Widgets
    Qt6::Quick
    Qt6::Xml
)
```

---

## QObject and Signals/Slots

Qt's core pattern: objects communicate via **signals and slots** instead of direct function calls.

```cpp
class Parser : public QObject {
    Q_OBJECT
public:
    void parse(const QString& path);

signals:
    void elementFound(const QString& name);
    void parseFailed(const QString& error);
};

// Connect signal to slot
connect(parser, &Parser::elementFound,
        view,   &View::addElement);
```

- `Q_OBJECT` macro required in every `QObject` subclass
- `signals:` section — declare signals (Qt generates the implementations)
- `emit` — emit a signal
- `connect()` — wire signal → slot

---

## XML Parsing

Two options in Qt:

### QXmlStreamReader (preferred — fast, low memory)
```cpp
QXmlStreamReader xml(&file);
while (!xml.atEnd()) {
    xml.readNext();
    if (xml.isStartElement() && xml.name() == u"field") {
        QString name = xml.attributes().value("field-name").toString();
    }
}
```

### QDomDocument (tree-based — easier for modification)
```cpp
QDomDocument doc;
doc.setContent(&file);
QDomElement root = doc.documentElement();  // <instances>
QDomNodeList fields = root.elementsByTagName("field");
```

For the Polaris editors: `QDomDocument` easier for read-modify-write. `QXmlStreamReader` better for large files.

---

## Model/View Architecture

Qt separates data (model) from display (view):

```
QAbstractItemModel (your data)
    ↓
QTreeView / QTableView / ListView (QML)
    ↓
User sees & edits
```

For config editors:
- Model = parsed XML config (list of fields, their attributes)
- View = tree or table showing the config
- Delegate = custom widget for editing a single field's attributes

---

## Properties in QML ↔ C++

```cpp
class TrackLabelConfig : public QObject {
    Q_OBJECT
    Q_PROPERTY(QString labelType READ labelType NOTIFY labelTypeChanged)

public:
    QString labelType() const { return m_labelType; }

signals:
    void labelTypeChanged();

private:
    QString m_labelType;
};
```

```qml
// In QML
Text { text: config.labelType }
```

---

## Qt Version

> Fill in: `qt --version` or check `CMakeLists.txt` for `find_package(Qt6 ...)`

---

## Useful Qt Docs

- [Qt 6 docs](https://doc.qt.io/qt-6/)
- [QXmlStreamReader](https://doc.qt.io/qt-6/qxmlstreamreader.html)
- [QAbstractItemModel](https://doc.qt.io/qt-6/qabstractitemmodel.html)
- [QML Reference](https://doc.qt.io/qt-6/qmlreference.html)

---

*Related: [[C++ Guidelines]] | [[QML Guidelines]] | [[CMake Build System]] | [[Summer Project Overview]]*

# C++ Guidelines

*Coding standards for C++ work in `polaris-asd-editor`*
*See also: [[QML Guidelines]] | [[Qt Framework]]*

---

## General Principles

- Prefer clarity over cleverness
- RAII everywhere — resources tied to object lifetime
- Prefer value semantics; use pointers only when necessary
- No raw `new`/`delete` — use smart pointers or Qt ownership

---

## Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Classes | `PascalCase` | `XmlImporter`, `TrackLabelConfig` |
| Functions/methods | `camelCase` | `parseInstances()`, `fieldName()` |
| Member variables | `m_camelCase` | `m_fieldName`, `m_isToggleable` |
| Constants / enums | `PascalCase` or `UPPER_SNAKE` | `MaxLineCount`, `PARSE_ERROR` |
| Files | Match class name | `XmlImporter.h` / `XmlImporter.cpp` |
| Namespaces | `lowercase` | `polaris::tracklabel` |

---

## Header Files

```cpp
#pragma once   // preferred over include guards

#include <QObject>   // Qt includes
#include <QString>

#include "MyClass.h" // local includes last

namespace polaris {

class XmlImporter : public QObject
{
    Q_OBJECT

public:
    explicit XmlImporter(QObject* parent = nullptr);

    bool load(const QString& filePath);
    QString fieldName() const;  // const getter

signals:
    void loadFailed(const QString& error);

private:
    QString m_filePath;
};

} // namespace polaris
```

---

## Implementation Files

```cpp
#include "XmlImporter.h"

#include <QFile>
#include <QXmlStreamReader>

namespace polaris {

XmlImporter::XmlImporter(QObject* parent)
    : QObject(parent)
{
}

bool XmlImporter::load(const QString& filePath)
{
    QFile file(filePath);
    if (!file.open(QIODevice::ReadOnly)) {
        emit loadFailed(file.errorString());
        return false;
    }
    // ...
    return true;
}

} // namespace polaris
```

---

## Qt-Specific Rules

### Memory / Ownership
```cpp
// Qt parent-child owns child — don't delete manually
auto* button = new QPushButton("OK", this);  // this owns button

// For non-QObject types, use smart pointers
auto config = std::make_unique<TrackLabelConfig>();
```

### Strings
```cpp
// Use QString for all user-facing / Qt strings
QString label = QStringLiteral("callsign");  // QStringLiteral for literals

// Use std::string only at C++ library boundaries
```

### Signals & Slots
```cpp
// New-style connect (compile-time checked) — always prefer
connect(parser, &XsdParser::elementFound,
        this,   &MyClass::onElementFound);

// Old-style SIGNAL/SLOT macros — avoid for new code
```

---

## Error Handling

```cpp
// Return bool + emit signal for errors (Qt style)
bool XmlImporter::load(const QString& path)
{
    if (path.isEmpty()) {
        emit loadFailed("Path is empty");
        return false;
    }
    // ...
}

// Use Q_ASSERT for programmer errors (not user errors)
Q_ASSERT(!m_fieldName.isEmpty());
```

---

## Modern C++ (C++17)

```cpp
// Range-based for
for (const auto& field : m_fields) { ... }

// Structured bindings
auto [key, value] = parseAttribute(attr);

// auto — use when type is obvious or verbose
auto reader = std::make_unique<QXmlStreamReader>();

// Not when type is unclear
QList<QString> names = getNames();  // explicit is clearer here
```

---

## What to Avoid

```cpp
// ❌ Raw pointers for owned memory
SomeClass* obj = new SomeClass();  // who deletes this?

// ❌ Old-style casts
int x = (int)someFloat;
// ✅
int x = static_cast<int>(someFloat);

// ❌ using namespace std (in headers)
using namespace std;

// ❌ Magic numbers
if (lineCount > 5)  // what's 5?
// ✅
constexpr int kMaxLabelLines = 5;
if (lineCount > kMaxLabelLines)
```

---

## File Organization

```
src/
├── parsers/
│   ├── XmlImporter.h
│   ├── XmlImporter.cpp
│   └── XsdParser.h / .cpp
├── models/
│   ├── TrackLabelConfig.h / .cpp
│   └── FlightListConfig.h / .cpp
└── widgets/
    ├── TrackLabelEditor.h / .cpp
    └── FieldEditWidget.h / .cpp
```

---

*Related: [[QML Guidelines]] | [[Qt Framework]] | [[CMake Build System]]*

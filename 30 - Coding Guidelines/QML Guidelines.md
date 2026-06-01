# QML Guidelines

*Coding standards for QML/Qt Quick work in `polaris-asd-editor`*
*See also: [[C++ Guidelines]] | [[Qt Framework]]*

---

## General Principles

- QML = UI layer only. Business logic stays in C++.
- Expose C++ data/logic via `Q_PROPERTY`, signals, and registered types.
- Keep QML files small and focused — one component per file.
- No complex JavaScript in QML. Move logic to C++ if it's more than a line or two.

---

## File Naming & Structure

```
qml/
├── Main.qml                   # application root
├── components/
│   ├── TrackLabelView.qml     # reusable component
│   ├── FieldEditRow.qml
│   └── PropertyField.qml
└── dialogs/
    └── SaveConfirmDialog.qml
```

- One component per `.qml` file
- File name = component name (`TrackLabelView.qml` exports `TrackLabelView`)
- `PascalCase` for component files

---

## Component Structure

```qml
// TrackLabelView.qml
import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

Item {
    id: root

    // 1. Properties (public interface)
    property string labelType: ""
    property var fields: []
    required property var model  // required = must be set by parent

    // 2. Signals
    signal fieldSelected(string fieldName)

    // 3. Private state
    QtObject {
        id: d  // private by convention
        property int selectedIndex: -1
    }

    // 4. Layout / visual tree
    ColumnLayout {
        anchors.fill: parent
        spacing: 4

        Repeater {
            model: root.fields
            delegate: FieldEditRow {
                field: modelData
                onClicked: root.fieldSelected(modelData.name)
            }
        }
    }
}
```

---

## Naming Conventions

| Thing | Convention | Example |
|-------|-----------|---------|
| Files | `PascalCase` | `TrackLabelView.qml` |
| `id` | `camelCase`, short | `id: root`, `id: listView` |
| Properties | `camelCase` | `property int itemCount` |
| Signals | `camelCase`, verb | `signal itemSelected(int idx)` |
| Handlers | `on` + signal name | `onItemSelected:` |
| JS variables | `camelCase` | `var totalWidth = ...` |

---

## Properties and Bindings

```qml
// ✅ Declarative binding — updates automatically
width: parent.width - 20

// ✅ Explicit property
property int padding: 8
Rectangle { width: parent.width - 2 * padding }

// ❌ Avoid breaking bindings with assignments
Component.onCompleted: {
    width = 100  // breaks the binding above — now it never updates
}
```

---

## Signals and C++ Connection

```qml
// In QML — handle signal from C++ model
Connections {
    target: trackLabelModel  // C++ object registered as context property
    function onLoadFailed(error) {
        errorLabel.text = error
    }
}

// Or on the object directly
Button {
    text: "Load"
    onClicked: trackLabelModel.load(filePath.text)
}
```

---

## Anchors vs Layouts

```qml
// Use layouts for groups of items
RowLayout {
    spacing: 8
    Label { text: "Field:" }
    TextField { Layout.fillWidth: true }
}

// Use anchors for simple positioning relative to parent
Rectangle {
    anchors.centerIn: parent
    width: 200; height: 50
}

// ❌ Don't mix anchors and Layout.* on the same item
```

---

## JavaScript in QML

```qml
// ✅ OK: simple inline expression
opacity: enabled ? 1.0 : 0.3

// ✅ OK: brief handler
onClicked: {
    if (nameField.text.isEmpty()) return
    model.addField(nameField.text)
    nameField.clear()
}

// ❌ Avoid: complex logic — move to C++
onClicked: {
    // 20 lines of parsing/validation logic — belongs in C++
}
```

---

## What to Avoid

```qml
// ❌ Global state / global IDs referenced across files
// (use properties and signals instead)

// ❌ Deep nesting — extract into components
Item {
    Item {
        Item {
            // 5 levels deep — split into components
        }
    }
}

// ❌ Hardcoded colors/sizes — use a Style singleton
Rectangle { color: "#1e1e1e" }
// ✅
Rectangle { color: Style.backgroundDark }
```

---

## Registering C++ Types

In C++:
```cpp
// main.cpp or application setup
qmlRegisterType<TrackLabelModel>("Polaris", 1, 0, "TrackLabelModel");
// or with qmlRegisterSingletonType for singletons
```

In QML:
```qml
import Polaris 1.0

TrackLabelModel {
    id: model
    onLoadFailed: console.warn("Load failed:", error)
}
```

---

*Related: [[C++ Guidelines]] | [[Qt Framework]] | [[CMake Build System]]*

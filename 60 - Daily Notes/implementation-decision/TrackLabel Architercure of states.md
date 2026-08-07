

This document outlines the architecture, data models, and wiring specifications for the refactored, data-driven **Track Label Editor**. It follows the Polaris ASD design philosophy: a decoupled, layered approach where the C++ domain layer drives state validation and complex layout evaluation, keeping the QML tier as a thin presentation layer.

---

## 1. Architectural Blueprint Overview

The editor utilizes a central composition tier to manage lifecycle matrices, field attributes, layout hierarchies, and dynamic visibility rules without offloading business calculations onto JavaScript runtimes.

```
+------------------------------------------------------------------------+
|                      TrackLabelEditorModel (Governor)                  |
|  Tracks Global Filters: activeType, activeVisibility, activeControl    |
|  Tracks Active Selection: selectedField (for Properties Side Panel)    |
+------------------------------------------------------------------------+
                                   |
         +-------------------------+-------------------------+
         | (Orchestrates evaluation & layout swapping)       |
         v                                                   v
+----------------------------------+       +----------------------------------+
|    RowListModel (Uncorrelated)   |  ...  |     RowListModel (Correlated)    |
|  Tracks RowCellModel* children   |       |  Tracks RowCellModel* children   |
+----------------------------------+       +----------------------------------+
         |                                                   |
         v (Index Rows)                                      v (Index Rows)
+----------------------------------+       +----------------------------------+
|          RowCellModel            |       |           RowCellModel           |
|  Owns FieldInterface* instances  |       |  Owns FieldInterface* instances  |
+----------------------------------+       +----------------------------------+
         |                                                   |
         +-------------------------+-------------------------+
                                   v (Index Columns)
                   +-------------------------------+
                   |        FieldInterface         |
                   |  Exposes flat `isVisible`     |
                   |  Aggregates: Text, Layout,    |
                   |              & Appearance     |
                   +-------------------------------+

```

---

## 2. Domain Layer: Field Architecture (Approach A)

To maintain compile-time type safety, avoid memory leaks, and eliminate flat-role mapping boilerplate, all cells are managed as polymorphic `QObject` pointers. Under **Approach A**, the model exposes the complete `FieldInterface*` directly to QML via a single role constant.

### 2.1 Component Architecture Updates

The underlying storage tracks heap-allocated pointers rather than value-type aggregates. Property validation constraints utilize sub-objects managed via smart pointer boundaries (`std::unique_ptr`).

### 2.2 Calculated Field Visibility Logic

Rather than structurally stripping nodes from your view models—which would dynamically collapse layout alignment on screen—the visibility evaluation happens fully in C++. The domain item exposes a flat property `isVisible` which evaluates independent configuration predicates (`onlyShowOnFocus`, `m_controlStateVisibility`) against global shifting environment conditions.

#### `FieldInterface.h`

```cpp
/*****************************************************************************
 * Copyright (c) 1998-2026, Tern Systems Inc.
 * All Rights Reserved.
 *****************************************************************************/

#ifndef _asd_editor_trackLabelEditor_FieldInterface_h_
#define _asd_editor_trackLabelEditor_FieldInterface_h_

#include "FieldText.h"
#include "FieldAppearance.h"
#include "FieldLayout.h"
#include <QObject>
#include <memory>

namespace asd::editor::tracklabelfield
{

class FieldInterface : public QObject
{
    Q_OBJECT
    Q_PROPERTY( FieldText* m_fieldText READ fieldText CONSTANT )
    Q_PROPERTY( FieldAppearance* m_fieldAppearance READ fieldAppearance CONSTANT )
    Q_PROPERTY( FieldLayout* m_fieldLayout READ fieldLayout CONSTANT )
    Q_PROPERTY( bool isVisible READ isVisible NOTIFY visibilityChanged )

public:
    explicit FieldInterface( QObject *parent = nullptr );
    virtual ~FieldInterface() override = default;

    // Getters
    [[nodiscard]] FieldText* fieldText() const { return m_fieldText.get(); }
    [[nodiscard]] FieldAppearance* fieldAppearance() const { return m_fieldAppearance.get(); }
    [[nodiscard]] FieldLayout* fieldLayout() const { return m_fieldLayout.get(); }
    [[nodiscard]] bool isVisible() const { return m_isVisible; }

    /**
     * @brief Core evaluation rules loop executed natively in C++
     */
    void evaluateVisibility(int globalVisibilityMode, int globalControlState)
    {
        bool nextState = true;

        // Rule 1: check 'onlyShowOnFocus' constraints against Normal display mode
        // Mode 0 = Normal, 1 = Hover, 2 = Completed
        if (m_fieldAppearance->onlyShowOnFocus() && globalVisibilityMode == 0) {
            nextState = false;
        }

        // Rule 2: check if global control state is allowed within the specified configuration set
        const auto& allowedStates = m_fieldAppearance->controlStateVisibility();
        if (!allowedStates.empty()) {
            if (std::find(allowedStates.begin(), allowedStates.end(), globalControlState) == allowedStates.end()) {
                nextState = false;
            }
        }

        if (m_isVisible != nextState) {
            m_isVisible = nextState;
            emit visibilityChanged();
        }
    }

signals:
    void visibilityChanged();

private:
    std::unique_ptr< FieldText > m_fieldText;
    std::unique_ptr< FieldAppearance > m_fieldAppearance;
    std::unique_ptr< FieldLayout > m_fieldLayout;
    bool m_isVisible = true;
};

} // namespace asd::editor::tracklabelfield

#endif // _asd_editor_trackLabelEditor_FieldInterface_h_

```

---

## 3. Layout Tier: Qt Models (Option 2 APIs)

To handle polymorphic objects cleanly, **Option 2** is chosen: all structural manipulations are executed via explicit custom C++ domain signatures (`insertField()`, `takeField()`), rather than wrapping actions around loose virtual `insertRows()` primitives. This maintains strict type checks, allows simple lifecycle management inside commands, and simplifies undo stack operations.

### 3.1 Row Cell Model (Inner Grid Array)

Tracks the sequential list of `FieldInterface*` elements belonging to a specific horizontal row channel.

#### `RowCellModel.h`

```cpp
#pragma once
#include <QAbstractListModel>
#include <QVector>
#include "FieldInterface.h"

class RowCellModel : public QAbstractListModel
{
    Q_OBJECT

public:
    enum Roles {
        FieldObjectRole = Qt::UserRole + 1  // Exposes raw FieldInterface* pointer directly
    };
    Q_ENUM(Roles)

    enum MoveDirection {
        MoveLeft,
        MoveRight,
        MoveUp,
        MoveDown
    };
    Q_ENUM(MoveDirection)

    explicit RowCellModel(QObject* parent = nullptr);
    virtual ~RowCellModel() override;

    // QAbstractListModel compliance
    int           rowCount(const QModelIndex& parent = {}) const override;
    QVariant      data(const QModelIndex& index, int role = Qt::DisplayRole) const override;
    QHash<int, QByteArray> roleNames() const override;

    // Option 2 Domain Mutations API (Strict Pointer tracking)
    void insertField(int col, asd::editor::tracklabelfield::FieldInterface* field);
    asd::editor::tracklabelfield::FieldInterface* takeField(int col); 
    void removeField(int col);

    // Read Access
    int count() const { return m_cells.size(); }
    asd::editor::tracklabelfield::FieldInterface* cellAt(int col) const;
    const QVector<asd::editor::tracklabelfield::FieldInterface*>& cells() const { return m_cells; }

    // QML Triggers
    Q_INVOKABLE void requestMoveField(int colIndex, MoveDirection direction);

signals:
    void moveFieldRequested(RowCellModel* sourceRow, int colIndex, MoveDirection direction);

private:
    QVector<asd::editor::tracklabelfield::FieldInterface*> m_cells;
};

```

#### `RowCellModel.cpp`

```cpp
#include "RowCellModel.h"

RowCellModel::RowCellModel(QObject* parent) : QAbstractListModel(parent) {}

RowCellModel::~RowCellModel() { qDeleteAll(m_cells); }

int RowCellModel::rowCount(const QModelIndex& parent) const {
    if (parent.isValid()) return 0;
    return m_cells.size();
}

QVariant RowCellModel::data(const QModelIndex& index, int role) const {
    if (!index.isValid() || index.row() >= m_cells.size()) return {};
    if (role == FieldObjectRole) {
        return QVariant::fromValue(m_cells.at(index.row()));
    }
    return {};
}

QHash<int, QByteArray> RowCellModel::roleNames() const {
    return { { FieldObjectRole, "fieldObject" } };
}

void RowCellModel::insertField(int col, asd::editor::tracklabelfield::FieldInterface* field) {
    col = qBound(0, col, m_cells.size());
    beginInsertRows({}, col, col);
    field->setParent(this);
    m_cells.insert(col, field);
    endInsertRows();
}

asd::editor::tracklabelfield::FieldInterface* RowCellModel::takeField(int col) {
    if (col < 0 || col >= m_cells.size()) return nullptr;
    beginRemoveRows({}, col, col);
    auto* field = m_cells.takeAt(col);
    endRemoveRows();
    if (field) field->setParent(nullptr);
    return field;
}

void RowCellModel::removeField(int col) {
    auto* field = takeField(col);
    if (field) delete field;
}

asd::editor::tracklabelfield::FieldInterface* RowCellModel::cellAt(int col) const {
    return m_cells.at(col);
}

void RowCellModel::requestMoveField(int colIndex, MoveDirection direction) {
    if (colIndex >= 0 && colIndex < m_cells.size()) {
        emit moveFieldRequested(this, colIndex, direction);
    }
}

```

### 3.2 Row List Model (Outer Grid Matrix)

Coordinates the horizontal lines array stack. It serves as the single connection junction catching inner bubble-up move actions.

#### `RowListModel.h`

```cpp
#pragma once
#include <QAbstractListModel>
#include <QVector>
#include "RowCellModel.h"

class RowListModel : public QAbstractListModel
{
    Q_OBJECT

public:
    enum Roles {
        CellModelRole = Qt::UserRole + 1,
        RowIdRole
    };

    explicit RowListModel(QObject* parent = nullptr);
    virtual ~RowListModel() override;

    int      rowCount(const QModelIndex& parent = {}) const override;
    QVariant data(const QModelIndex& index, int role = Qt::DisplayRole) const override;
    QHash<int, QByteArray> roleNames() const override;

    // Mutation tracking wrapper engines
    void appendRow();
    void insertRow(int row);
    void removeRow(int row);
    void moveCell(int fromRow, int fromCol, int toRow, int toCol);
    void notifyRowChanged(int row);

private slots:
    void handleMoveFieldRequest(RowCellModel* sourceRow, int colIndex, RowCellModel::MoveDirection direction);

private:
    QVector<RowCellModel*> m_rows;
    QVector<int> m_rowIds;
    int m_nextRowId = 0;
};

```

#### `RowListModel.cpp`

```cpp
#include "RowListModel.h"

RowListModel::RowListModel(QObject* parent) : QAbstractListModel(parent) {}
RowListModel::~RowListModel() { qDeleteAll(m_rows); }

int RowListModel::rowCount(const QModelIndex& parent) const {
    if (parent.isValid()) return 0;
    return m_rows.size();
}

QVariant RowListModel::data(const QModelIndex& index, int role) const {
    if (!index.isValid() || index.row() >= m_rows.size()) return {};
    switch (role) {
        case CellModelRole: return QVariant::fromValue(m_rows.at(index.row()));
        case RowIdRole:     return m_rowIds.at(index.row());
        default: return {};
    }
}

QHash<int, QByteArray> RowListModel::roleNames() const {
    return { { CellModelRole, "cellModel" }, { RowIdRole, "rowId" } };
}

void RowListModel::insertRow(int row) {
    row = qBound(0, row, m_rows.size());
    beginInsertRows({}, row, row);
    
    auto* cellModel = new RowCellModel(this);
    connect(cellModel, &RowCellModel::moveFieldRequested, this, &RowListModel::handleMoveFieldRequest);

    m_rows.insert(row, cellModel);
    m_rowIds.insert(row, m_nextRowId++);
    endInsertRows();
}

void RowListModel::handleMoveFieldRequest(RowCellModel* sourceRow, int fromCol, RowCellModel::MoveDirection direction) {
    int fromRow = m_rows.indexOf(sourceRow);
    if (fromRow == -1) return;

    int toRow = fromRow;
    int toCol = fromCol;

    switch (direction) {
        case RowCellModel::MoveLeft:  toCol = fromCol - 1; break;
        case RowCellModel::MoveRight: toCol = fromCol + 1; break;
        case RowCellModel::MoveUp:    toRow = fromRow - 1; break;
        case RowCellModel::MoveDown:  toRow = fromRow + 1; break;
    }

    if (toRow < 0 || toRow >= m_rows.size()) return;
    moveCell(fromRow, fromCol, toRow, toCol);
}

void RowListModel::moveCell(int fromRow, int fromCol, int toRow, int toCol) {
    if (fromRow < 0 || fromRow >= m_rows.size() || toRow < 0 || toRow >= m_rows.size()) return;
    if (fromCol < 0 || fromCol >= m_rows[fromRow]->count()) return;

    auto* field = m_rows[fromRow]->takeField(fromCol);
    if (!field) return;

    if (fromRow != toRow) {
        field->setParent(m_rows[toRow]);
    }

    int clampedCol = qBound(0, toCol, m_rows[toRow]->count());
    m_rows[toRow]->insertField(clampedCol, field);

    notifyRowChanged(fromRow);
    if (fromRow != toRow) notifyRowChanged(toRow);
}

void RowListModel::notifyRowChanged(int row) {
    QModelIndex idx = index(row);
    emit dataChanged(idx, idx, { CellModelRole });
}

```

---

## 4. The Master State Governor Model

The top-level `TrackLabelEditorModel` functions as the configuration supervisor. It holds multiple grid session matrices, maintains state selectors, and triggers cascading synchronization across layout parameters.

#### `TrackLabelEditorModel.h`

```cpp
#pragma once
#include <QObject>
#include <QHash>
#include "RowListModel.h"
#include "FieldInterface.h"

namespace asd::editor::tracklabel
{

class TrackLabelEditorModel : public QObject
{
    Q_OBJECT
    Q_PROPERTY(LabelType activeType READ activeType WRITE setActiveType NOTIFY activeTypeChanged)
    Q_PROPERTY(VisibilityState activeVisibility READ activeVisibility WRITE setActiveVisibility NOTIFY activeVisibilityChanged)
    Q_PROPERTY(int activeControlState READ activeControlState WRITE setActiveControlState NOTIFY activeControlStateChanged)
    Q_PROPERTY(RowListModel* currentLayout READ currentLayout NOTIFY currentLayoutChanged)
    Q_PROPERTY(asd::editor::tracklabelfield::FieldInterface* selectedField READ selectedField NOTIFY selectedFieldChanged)

public:
    enum LabelType { Uncorrelated, Correlated, Ground, FlightPlanTrack }; Q_ENUM(LabelType)
    enum VisibilityState { Normal, Hover, Completed }; Q_ENUM(VisibilityState)

    explicit TrackLabelEditorModel(QObject *parent = nullptr);
    virtual ~TrackLabelEditorModel() override = default;

    LabelType activeType() const { return m_activeType; }
    void setActiveType(LabelType type);

    VisibilityState activeVisibility() const { return m_activeVisibility; }
    void setActiveVisibility(VisibilityState state);

    int activeControlState() const { return m_activeControlState; }
    void setActiveControlState(int state);

    RowListModel* currentLayout() const;
    asd::editor::tracklabelfield::FieldInterface* selectedField() const { return m_selectedField; }

    Q_INVOKABLE void selectField(int rowIndex, int colIndex);
    Q_INVOKABLE void clearSelection();

signals:
    void activeTypeChanged();
    void activeVisibilityChanged();
    void activeControlStateChanged();
    void currentLayoutChanged();
    void selectedFieldChanged();

private:
    void reevaluateActiveLayoutVisibility();

    LabelType m_activeType = Uncorrelated;
    VisibilityState m_activeVisibility = Normal;
    int m_activeControlState = 0;

    QHash<LabelType, RowListModel*> m_layouts;
    asd::editor::tracklabelfield::FieldInterface* m_selectedField = nullptr;
};

} // namespace asd::editor::tracklabel

```

#### `TrackLabelEditorModel.cpp`

```cpp
#include "TrackLabelEditorModel.h"

namespace asd::editor::tracklabel
{

TrackLabelEditorModel::TrackLabelEditorModel(QObject *parent) : QObject(parent)
{
    m_layouts[Uncorrelated]    = new RowListModel(this);
    m_layouts[Correlated]      = new RowListModel(this);
    m_layouts[Ground]          = new RowListModel(this);
    m_layouts[FlightPlanTrack] = new RowListModel(this);
    
    // Initial instantiation loop bypasses signals in constructors:
    // e.g., m_layouts[Uncorrelated]->appendRow();
}

void TrackLabelEditorModel::setActiveType(LabelType type) {
    if (m_activeType == type) return;
    m_activeType = type;
    emit activeTypeChanged();
    emit currentLayoutChanged();
    clearSelection();
    reevaluateActiveLayoutVisibility();
}

void TrackLabelEditorModel::setActiveVisibility(VisibilityState state) {
    if (m_activeVisibility == state) return;
    m_activeVisibility = state;
    emit activeVisibilityChanged();
    reevaluateActiveLayoutVisibility();
}

void TrackLabelEditorModel::setActiveControlState(int state) {
    if (m_activeControlState == state) return;
    m_activeControlState = state;
    emit activeControlStateChanged();
    reevaluateActiveLayoutVisibility();
}

RowListModel* TrackLabelEditorModel::currentLayout() const {
    return m_layouts.value(m_activeType, nullptr);
}

void TrackLabelEditorModel::selectField(int rowIndex, int colIndex) {
    auto* layout = currentLayout();
    if (!layout) return;

    auto rowIdx = layout->index(rowIndex);
    auto* cellModel = layout->data(rowIdx, RowListModel::CellModelRole).value<RowCellModel*>();
    if (cellModel && colIndex >= 0 && colIndex < cellModel->count()) {
        m_selectedField = cellModel->cellAt(colIndex);
        emit selectedFieldChanged();
    }
}

void TrackLabelEditorModel::clearSelection() {
    if (m_selectedField) {
        m_selectedField = nullptr;
        emit selectedFieldChanged();
    }
}

void TrackLabelEditorModel::reevaluateActiveLayoutVisibility() {
    auto* layout = currentLayout();
    if (!layout) return;

    for (int r = 0; r < layout->rowCount(); ++r) {
        auto rowIdx = layout->index(r);
        auto* cellModel = layout->data(rowIdx, RowListModel::CellModelRole).value<RowCellModel*>();
        if (!cellModel) continue;

        for (auto* field : cellModel->cells()) {
            field->evaluateVisibility(static_cast<int>(m_activeVisibility), m_activeControlState);
        }
    }
}

}

```

---

## 5. Architectural Extensions Strategy

Because layout mutations are handled via explicit C++ APIs (Option 2) rather than standard index overrides, extensions for Drag & Drop and Undo Stack integrations map cleanly into the domain layer.

### 5.1 Undo/Redo Engine Blueprint (`QUndoCommand`)

Using pointers means commands can extract an existing sub-object from the view matrix, cache it inside the command instance, and push it back into the model if an undo operation is requested.

```cpp
class MoveFieldCommand : public QUndoCommand
{
public:
    MoveFieldCommand(RowListModel* model, int fRow, int fCol, int tRow, int tCol)
        : m_model(model), fromRow(fRow), fromCol(fCol), toRow(tRow), toCol(tCol) {}

    void redo() override {
        m_model->moveCell(fromRow, fromCol, toRow, toCol);
    }

    void undo() override {
        // Reverse spatial vectors perfectly
        m_model->moveCell(toRow, toCol, fromRow, fromCol);
    }
private:
    RowListModel* m_model;
    int fromRow; int fromCol; int toRow; int toCol;
};

```

---

## 6. Presentation Tier Spec: Thin QML Bindings

By handling visibility evaluation and structural modifications entirely in C++, the presentation elements remain fully decoupled from backend logic. The user interface simply hooks directly into the provided state values.

### `TrackLabelField.qml`

```qml
import QtQuick 2.15
import QtQuick.Controls 2.15

Rectangle {
    id: root
    
    // Approach A: single point of entry to all unified aggregated sub-properties
    required property var fieldObject 
    required property int colIndex
    required property int rowIndex

    // The display tier remains thin: no business code or calculation loops needed here
    visible: fieldObject ? fieldObject.isVisible : true
    
    width: fieldObject ? fieldObject.m_fieldLayout.width : 120
    height: fieldObject ? fieldObject.m_fieldLayout.height : 48
    color: "#ffffff"
    border.width: 1

    Text {
        anchors.centerIn: parent
        text: fieldObject ? fieldObject.m_fieldText.prefix + fieldObject.m_fieldText.fieldName : ""
    }

    MouseArea {
        anchors.fill: parent
        onClicked: {
            // Inform the state governor which cell is selected
            trackLabelGovernor.selectField(root.rowIndex, root.colIndex)
        }
    }
}

```
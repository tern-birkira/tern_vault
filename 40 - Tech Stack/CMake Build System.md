# CMake Build System

*How `polaris-asd-editor` is built*
*See also: [[Qt Framework]] | [[C++ Guidelines]]*

---

## What Is CMake?

CMake = cross-platform build system generator. It generates native build files (Makefiles, Ninja, VS project) from `CMakeLists.txt`.

```
CMakeLists.txt → cmake → Makefile/build.ninja → make/ninja → binary
```

---

## Project Structure

```
polaris-asd-editor/
├── CMakeLists.txt          ← root: defines project, finds Qt, adds subdirs
├── CMakePresets.json       ← build presets (debug/release configs)
├── source/
│   ├── CMakeLists.txt      ← defines the main library/executable targets
│   └── ...
└── build/                  ← out-of-source build dir (never commit this)
```

---

## Common Commands

```bash
# Configure (first time or after CMakeLists changes)
cmake -B build -S . -DCMAKE_BUILD_TYPE=Debug

# Or using a preset (from CMakePresets.json)
cmake --preset debug

# Build
cmake --build build --parallel

# Or with preset
cmake --build --preset debug

# Run tests
ctest --test-dir build --output-on-failure
```

---

## Adding a New Source File

In the relevant `CMakeLists.txt`:

```cmake
target_sources(polaris-asd-editor PRIVATE
    src/parsers/XmlImporter.cpp
    src/parsers/XmlImporter.h   # headers optional but good for IDE
)
```

Never add `.cpp` files to multiple targets — causes multiple definition errors.

---

## Adding a Qt Module

```cmake
find_package(Qt6 REQUIRED COMPONENTS Core Widgets Xml Quick)

target_link_libraries(my-target PRIVATE
    Qt6::Core
    Qt6::Xml
)
```

---

## QML Resources

QML files are embedded via Qt resource system:

```cmake
qt_add_qml_module(my-target
    URI "Polaris"
    VERSION 1.0
    QML_FILES
        qml/Main.qml
        qml/components/TrackLabelView.qml
)
```

---

## Debug vs Release

| Build type | Flags | Use for |
|------------|-------|---------|
| `Debug` | `-g`, no optimization | Development, debugging |
| `Release` | `-O2`, no debug info | Deployment |
| `RelWithDebInfo` | `-O2 -g` | Profiling |

Set via: `-DCMAKE_BUILD_TYPE=Debug`

---

## CMakePresets.json

Check existing presets before creating a custom build command:

```bash
cmake --list-presets        # show available presets
cmake --preset debug        # use the debug preset
```

---

*Related: [[Qt Framework]] | [[C++ Guidelines]] | [[Software Architecture at Tern]]*

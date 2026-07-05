# openSUSE Update Applet — Agent Guide

## Build & Run

```bash
# Install dependencies (openSUSE)
sudo zypper install -y qt6-widgets-devel qt6-svg-devel qt6-network-devel qt6-dbus-devel qtkeychain-qt6-devel cmake gcc-c++

# Build (Debug or Release)
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build

# Run (from build dir)
./build/opensuse-update-applet

# Install to system
sudo cmake --install build
```

Single output binary: `opensuse-update-applet`.

## Architecture

- **Entrypoint**: `src/main.cpp` wires all managers together, no dependency injection framework
- **Settings**: singleton (`Settings::instance()`), persists to `~/.config/OpenSUSE/UpdateApplet.conf`
- **UpdateChecker**: runs `zypper list-updates`, `flatpak remote-ls --updates`, `snap refresh --list` via QProcess; emits `passwordRequired` when sudo is needed
- **PasswordManager**: wraps qtkeychain for sudo password; caches in memory, reads/writes via `QKeychain::*PasswordJob`
- **SelfUpdater**: checks GitHub releases API via QNetworkAccessManager
- **LockManager**: wraps `zypper addlock`/`removelock`/`locks`
- **TrayManager**: QSystemTrayIcon with icon states (Idle/Checking/UpdatesAvailable/RebootRequired)
- **MainWindow**: QWidget with 4 tabs (Home, Settings, Locks, About); all UI built in C++ code (no .ui files)
- Communication is Qt signals/slots throughout

## Qt / CMake Conventions

- `CMAKE_AUTOMOC`, `CMAKE_AUTORCC`, `CMAKE_AUTOUIC` are ON — sources are processed automatically
- Qt6 only, C++17
- `#pragma once` for headers, `QStringLiteral` for string constants
- Resources listed in `resources/resources.qrc` (SVG icons + QSS stylesheets)

## Testing & Linting

There are no tests, no CI, no linter/formatter config. The project has no test framework, no test directory, and no workflow files.

## Gitignore

`build/`, `build-release/`, `guide.md`, `releases`, `*.rpm`, and Qt-generated files (`moc_*`, `qrc_*`, `ui_*`) are all gitignored.

## Package

RPM spec at `opensuse-update-applet.spec`; version defined in `CMakeLists.txt` (currently 1.2.0).

# Migration Process for `qgis_coordinate_capture` Plugin

## Overview
This document describes the step‑by‑step procedure followed to migrate the `qgis_coordinate_capture` QGIS plugin from a QGIS 3/Qt5 (PyQt5) codebase to QGIS 4/Qt6 (PyQt6). It captures the changes applied to source files, metadata, and resource handling, and includes commands used during the migration. The goal is to allow a future team member to repeat the migration or to reproduce the state of the repository at any point.

---

## Repository Layout
```
qgis_coordinate_capture/
├── coordinate_capture.py
├── coordinate_capture_dockwidget.py
├── coordinate_capture_map_tool.py
├── __init__.py
├── metadata.txt
├── resources.qrc
├── icons/
│   ├── coordinate_capture.png
│   ├── tracking.svg
│   ├── transformed.svg
│   ├── mIconProjectionEnabled.svg
│   ├── mActionEditCopy.svg
└── README.md
```

## Step‑by‑Step Migration

| Step | Action | Command / Code | Result |
|------|--------|----------------|--------|
| 1 | Inspect current code | `python3 -m py_compile *.py` | All files syntactically valid |
| 2 | Backup original files | `git add .` & `git commit -m "Initial QGIS 3 state"` | Versioned baseline |
| 3 | Update `metadata.txt` | Add `qgisMaximumVersion=4.99` and keep `qgisMinimumVersion=3.14`. | Plugin advertises QGIS 4 compatibility |
| 4 | Update imports to use `qgis.PyQt` | All imports use the `qgis.PyQt.*` namespaces (never bare `PyQt5`/`PyQt6`), with a `try/except` fallback to `PyQt6.QtCore` only for `QSettings`. | |
| 5 | Decouple from compiled Qt resources | Removed `from .resources import *` from `coordinate_capture.py`; all icons are now referenced via local file system paths: `os.path.join(self.plugin_dir, "icons", "<name>")` (e.g. `coordinate_capture.png`, `tracking.svg`, `transformed.svg`, `mIconProjectionEnabled.svg`, `mActionEditCopy.svg`). `icon_path` in `coordinate_capture.py` is likewise a local path. | No more `ModuleNotFoundError: resources` / unresolved `:/plugins/...` URLs |
| 6 | PyQt6 nested‑enum migration | See table below. | No more `AttributeError` on `Qt.*` / `QLineEdit.*` |
| 7 | Float pixel coordinates | Cast `e.position().x()/.y()` to `int()` before calling `QgsMapToPixel.toMapCoordinates()` in `canvasMoveEvent`/`canvasPressEvent` (Qt6 `QPointF` returns `float`, but `toMapCoordinates()` overloads only accept `int` or `QPoint`). | Rubber band + coordinate tracking work |
| 8 | Verify all modules compile again | `python3 -m py_compile *.py` | No syntax errors |
| 9 | Commit changes | `git add .` & `git commit` | |
| 10 | Test in QGIS 4 | Install plugin in QGIS 4 profile, enable, verify toolbar icon, DockWidget, capture coordinates. | All features functional |

### PyQt5 → PyQt6 enum / API changes applied

| PyQt5 (old) | PyQt6 (new) | File |
|-------------|-------------|------|
| `Qt.red` | `QColor(Qt.GlobalColor.red)` (+ `from qgis.PyQt.QtGui import QColor`) | `coordinate_capture_map_tool.py` |
| `Qt.LeftButton` | `Qt.MouseButton.LeftButton` | `coordinate_capture_map_tool.py` |
| `Qt.RightButton` | `Qt.MouseButton.RightButton` | `coordinate_capture_map_tool.py` |
| `Qt.LeftDockWidgetArea` | `Qt.DockWidgetArea.LeftDockWidgetArea` | `coordinate_capture.py` |
| `QLineEdit.TrailingPosition` | `QLineEdit.ActionPosition.TrailingPosition` | `coordinate_capture_dockwidget.py` |
| `toMapCoordinates(e.position().x(), e.position().y())` | `toMapCoordinates(int(e.position().x()), int(e.position().y()))` | `coordinate_capture_map_tool.py` |

### Import block of `coordinate_capture.py` (final state)

```python
import os.path

try:
    from qgis.PyQt.QtCore import QSettings
except ImportError:
    from PyQt6.QtCore import QSettings

from qgis.PyQt.QtCore import QCoreApplication, QTranslator, Qt
from qgis.PyQt.QtGui import QIcon
from qgis.PyQt.QtWidgets import QAction
from qgis.gui import QgsProjectionSelectionDialog
from qgis.core import (QgsUnitTypes, QgsCoordinateTransform,
                       QgsCoordinateReferenceSystem, QgsPointXY)
```

### Icon path pattern (applied to all icons)

```python
self.plugin_dir = os.path.dirname(__file__)
icon = QIcon(os.path.join(self.plugin_dir, "icons", "coordinate_capture.png"))
```

## Important Notes
- **QGIS 4 compatibility**: The plugin now advertises `qgisMaximumVersion=4.99`. If you plan to maintain support for QGIS 5+ in the future, update the version range accordingly.
- **Resources**: The plugin does **not** depend on a compiled `resources.py`. All icon references use local file system paths under the plugin's `icons/` directory, so the `resources.qrc`/`resources.py` build step is optional (the import is kept commented out in `coordinate_capture.py` for possible future use).
- **Dependencies**: No external dependencies were added; the plugin uses only the core QGIS API and PyQt6 (via `qgis.PyQt`).
- **Debugging tip (stale bytecode)**: After editing plugin source files in the repo, always re‑copy them to `~/.local/share/QGIS/QGIS4/profiles/shared/python/plugins/qgis_coordinate_capture/` and delete the `__pycache__` directory in that folder, then restart QGIS. Otherwise QGIS keeps running the old `.pyc` files and tracebacks will point at outdated code lines.
  ```bash
  cp <repo>/*.py ~/.local/share/QGIS/QGIS4/profiles/shared/python/plugins/qgis_coordinate_capture/
  rm -rf ~/.local/share/QGIS/QGIS4/profiles/shared/python/plugins/qgis_coordinate_capture/__pycache__
  ```
- **Testing**: It is recommended to run the plugin on a fresh QGIS 4 installation to confirm no runtime errors.

## Re‑production Checklist
1. Clone the repository.
2. Place the `qgis_coordinate_capture` directory in the QGIS 4 plugin folder as described.
3. Restart QGIS 4 and enable the plugin.
4. Verify the toolbar icon appears and the menu entry works.
5. Click on the map to see coordinate capture in the DockWidget.

---

*Prepared by: ChatGPT, 2026‑10‑26*
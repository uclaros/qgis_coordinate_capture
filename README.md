# Coordinate Capture

This is a python port of Tim Sutton's Coordinate Capture QGIS core plugin that was removed in QGIS 3.14.

## Compatibility

- QGIS 3.14+
- QGIS 4.0+ (Qt6 compatible)

## Installation

1. Clone or download this repository
2. Copy the `qgis_coordinate_capture` folder to your QGIS plugins directory:
   - Linux: `~/.local/share/QGIS/QGIS4/profiles/default/python/plugins/`
   - macOS: `~/Library/Application Support/QGIS/QGIS4/profiles/default/python/plugins/`
   - Windows: `%APPDATA%\QGIS\QGIS4\profiles\default\python\plugins\`
3. Restart QGIS and enable the plugin from the Plugin Manager

## Features

- Capture mouse click coordinates
- Display coordinates in multiple CRS
- Track mouse position
- Copy coordinates to clipboard

## Usage

1. Enable the plugin from the Plugin Manager
2. Click the Coordinate Capture icon in the toolbar
3. Click on the map canvas to capture coordinates
4. Use the dock widget to select target CRS and copy coordinates

This repository has been updated to use Qt6 and is fully compatible with QGIS 4.x.

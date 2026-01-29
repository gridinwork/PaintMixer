# PaintMixer

![PaintMixer UI](images/1.jpeg "PaintMixer UI")

Demo video: https://youtu.be/ryIYmNxW3IQ

## Repository Description (About)
Desktop GUI application for a multi‑tank paint mixing machine. Provides real‑time tank visualization, color configuration, dosing controls, and serial communication with the controller for mixing and calibration tasks.

## Project Description
PaintMixer is a desktop control panel for a paint mixing machine with 10 color tanks. The application connects to the controller over a serial interface, shows tank fill levels, allows operators to select colors and quantities, and sends mixing commands to the hardware. It also supports configuring each tank’s color, name, and dosing time, and persists settings locally for quick reuse.

The GUI is built with PySide2 and is organized around a main mixing view and a settings view. The main screen displays color selectors for each tank and provides convenient add/subtract buttons for dosing. The settings screen lets operators edit color metadata and calibrate parameters, then synchronize those settings back to the controller.

## Key Features
- 10‑tank color selector UI with live fill percentages.
- Quick dosing controls (+1/+5/+10 g and −1/−5/−10 g).
- Mix command generation and dispatch over serial.
- Tank metadata management (name, color, time).
- Live data requests and status refresh from the controller.
- Local persistence in `config_saved.json`.

## Requirements
- Python 3.x
- pip
- PySide2
- pyserial
- numpy
- jsonpickle
- pyrsistent

## Run the App
```bash
python paintmixer.py
```

## File Overview
- `paintmixer.py` — core logic, tank model, and serial commands.
- `MainView.py` — main UI and user interactions.
- `settingsview.py` — settings UI for tank configuration.
- `serialcom.py` — serial communication layer.
- `images/1.jpeg` — UI screenshot for documentation.

## Notes
This project is intended to be used with a compatible paint mixer controller and requires a serial connection to the hardware.
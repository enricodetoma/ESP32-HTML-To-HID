# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESP32-HTML-To-HID is an ESP32 embedded systems project that enables keyboard input control via a web browser interface. The device appears to the host computer as a USB HID (Human Interface Device) keyboard, allowing a web page served by the ESP32 to send keyboard commands and text input to the connected computer.

**Key Use Cases:**
- Remote control via web interface
- Keyboard emulation from a web-connected device

## Hardware Configuration

The project supports multiple ESP32 variants through PlatformIO environments:
- **s3N16r8** (default): ESP32-S3 with 16MB flash, 8MB PSRAM
- **s2Mini**: ESP32-S2 Mini variant

Target board detection and USB/HID functionality requires specific hardware capabilities (native USB interface).

## Build and Deployment

All builds use **PlatformIO**. The source code is in the `htmlToHid/` directory.

**Build commands:**
```bash
# Build default environment (s3N16r8)
platformio run

# Build for specific environment
platformio run -e s2Mini
platformio run -e s3N16r8

# Upload firmware
platformio run --target upload

# Monitor serial output (115200 baud)
platformio run --target monitor

# Clean build
platformio run --target clean
```

## Architecture and Key Components

### Main Entry Point
**htmlToHid/htmlToHid.ino**: Contains the main setup/loop and web server handlers.

### Core Functionality Flow

1. **Initialization** (`genericBaseProject.h:baseProjectSetup()`):
   - SPIFFS filesystem initialization for config persistence
   - Double reset detection for entering config mode
   - WiFi Manager setup with captive portal
   - NTP time synchronization
   - mDNS registration (`fire.local`)

2. **Web Interface** (`webPage.h`):
   - Bootstrap-based responsive UI with directional pad and text input
   - Embedded as a C string in the firmware
   - Sends AJAX requests to ESP32 endpoints without jQuery

3. **USB HID Keyboard** (`htmlToHid.ino`):
   - Uses `USBHIDKeyboard` for keyboard emulation
   - Supports key presses (arrow keys, Escape, Return, F12) via `/command` endpoint
   - Supports text input via `/text` endpoint

### Configuration System
**projectConfig.h** and **wifiManagerHandler.h**:
- Configuration stored in SPIFFS as JSON (`/project_config.json`)
- Settings: timezone, 24-hour clock format, US date format
- WiFi Manager provides captive portal for configuration
- Double Reset Detector allows forcing config mode

### Dependencies
- **WiFiManager** (tzapu): WiFi configuration with captive portal
- **ESP_DoubleResetDetector** (khoih-prog): Hardware reset detection
- **ezTime**: Timezone and NTP time synchronization
- **ArduinoJson**: JSON parsing for config files
- Arduino WebServer (built-in): HTTP server for web interface

### Display Interface
**projectDisplay.h**: Abstract base class for display output. Currently only `serialDisplay.h` (serial console) is implemented. The architecture allows adding hardware display support (e.g., LCD screens) by extending this class.

## Development Notes

- The web server handles three routes:
  - `/`: Returns the HTML webpage
  - `/command?press=<N>`: Maps numeric commands to keyboard keys
  - `/text?text=<string>`: Sends text as keyboard input

- Serial communication is available at 115200 baud for debugging

- Configuration persists across reboots via SPIFFS. Double-reset within 10 seconds triggers config mode.

- The codebase follows a modular header-based structure; most logic is in `.h` files included from the main `.ino` file.

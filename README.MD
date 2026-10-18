# twatch-s3-minimal

A lightweight, debloated, and battery-optimized firmware base for the **LilyGO T-Watch S3** (ESP32-S3, AXP2101 PMU, PCF8563 RTC, ST7789 display).

Refactored from the official [LilyGO TTGO_TWatch_Library](https://github.com/Xinyuan-LilyGO/TTGO_TWatch_Library) factory demo into a clean, practical daily-driver watch face.

---

## Key Improvements & Optimizations

- **Power Optimization:**
  - Scaled CPU clock down from default 160/240 MHz to **80 MHz** (`setCpuFrequencyMhz(80)`), significantly reducing idle current draw while keeping LVGL smooth.
  - Disabled unused hardware peripheral drivers at boot (I2S microphone, LoRa radio, audio DAC/player, and IR transmitter).
  - Multi-rate update cadence: 1-second display refresh for time, staggered 30s/60s intervals for I2C sensor queries (battery, steps, RTC) to minimize bus traffic.
- **Minimalist Watchface (`minimalClock`):**
  - Removed stock bitmap assets, redundant buttons, and QR code screens.
  - High-contrast, clean digital typography using Fira Code.
  - Comprehensive status display:
    - **Top Bar:** Battery percentage & temperature.
    - **Date Line:** Day of week, month, day, 2-digit year, day of year progress, and calendar week (`FRI, OCT 16 '26 | D289/365 W42`).
    - **Center:** Digital time with blinking colon.
    - **Bottom Bar:** Real-time BMA423 step counter.
- **Smart Hardware RTC Sync:**
  - Edge-triggered clock synchronization: triggers automatically upon USB power connection (`isVbusIn`) without running on battery power.
  - Robust **HTTP HEAD** time synchronization bypassing restrictive firewalls that block standard NTP UDP port 123.
  - Multi-network credential support via `secrets.h`.
- **Hardware Button Navigation ("Home / Sleep" Control):**
  - Configured the physical AXP2101 power button as a dedicated multifunctional hardware key.
  - When navigating secondary app pages (Wi-Fi scan, device info, settings), pressing the side button immediately animates back to the `minimalClock` home screen.

---

## Hardware Specifications

| Component | Hardware Model | Interface |
| :--- | :--- | :--- |
| **MCU** | ESP32-S3 (Dual-core Xtensa LX7, 8MB PSRAM, 16MB Flash) | — |
| **PMU** | AXP2101 Power Management Unit | I2C |
| **RTC** | PCF8563 Real-Time Clock | I2C |
| **IMU** | BMA423 Tri-axis Accelerometer (Pedometer) | I2C |
| **Display** | 1.54" IPS LCD (ST7789V, 240×240) | SPI |
| **Touch** | FT6236 Capacitive Touch Controller | I2C |

---

## Getting Started

### 1. Wi-Fi Configuration (`secrets.h`)

This repository ignores `secrets.h` to keep personal Wi-Fi credentials out of version control.

1. Create a `secrets.h` file in the project source directory:
   ```cpp
   #pragma once

   struct WifiCredential {
       const char* ssid;
       const char* password;
   };

   // Add your local Wi-Fi networks here
   const WifiCredential KNOWN_NETWORKS[] = {
       { "YOUR_WIFI_SSID",     "YOUR_WIFI_PASSWORD" },
       { "YOUR_PHONE_HOTSPOT", "HOTSPOT_PASSWORD"   }
   };

   const int KNOWN_NETWORKS_COUNT = sizeof(KNOWN_NETWORKS) / sizeof(KNOWN_NETWORKS[0]);

### 2. Build & Flash (PlatformIO)
This project is configured for VS Code with the PlatformIO extension.

Clone your repository:
```Bash
git clone [https://github.com/nopynospy/twatch-s3-minimal.git](https://github.com/nopynospy/twatch-s3-minimal.git)
cd twatch-s3-minimal
```
Open the project folder in VS Code.

Connect your T-Watch S3 via USB-C.

Build and upload via the PlatformIO toolbar, or run:

```Bash
pio run -e twatch-s3 -t upload
```
Open the serial monitor (baud rate 115200):

```Bash
pio device monitor -b 115200
```
### License
This project is open-source under the Apache-2.0 / MIT licenses corresponding to the upstream components from LilyGO.
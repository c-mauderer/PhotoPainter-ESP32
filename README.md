# ESP32 PhotoPainter Enhancement

**Upgrade your Waveshare PhotoPainter into an internet-connected smart photo frame**

This project provides custom firmware to add WiFi connectivity to the Waveshare PhotoPainter, enabling automatic image fetching and display from the web using an ESP32 microcontroller.

## 🎯 Overview

ESP32 PhotoPainter Enhancement is a hardware/software solution that retrofits onto the [Waveshare PhotoPainter`s](https://www.waveshare.com/wiki/PhotoPainter) SDCard Slot. By utilizing the SD card slot, it seamlessly adds internet connectivity—no permanent hardware modifications required. Firmware for the ESP32(Fetcher) and the PhotoPainter(Renderer) are custom to enable this.

### How It Works

1. **ESP32**: Powered via the PhotoPainter's SD card slot.
2. **I2C Communication**: Uses two SD slot pins for high-speed I2C (600kHz).
3. **Smart Fetching**: Downloads BMP images from the internet and streams them to the PhotoPainter.
4. **RTC Wake-up**: Activates automatically when the PhotoPainter's RTC wakes the system and powers up the SDCard slot. So uses the same RTC deep sleep as the PhotoPainter that cuts of power from SDCard slot when going into deep sleep.
5. **Seamless Display**: Images are processed and shown on the 7.3" e-paper screen.

## 🏗️ Wiring

### Hardware Setup
```
PhotoPainter SD Slot → ESP32 Wemos D1 Mini
├── Power (3.3V)     → VCC
├── Ground           → GND
├── Pin 4 (SDA)      → GPIO 21 (SDA)
└── Pin 5 (SCL)      → GPIO 22 (SCL)

To find out which pins are Pin 4/5 on the PhotoPainter consult a pinout of a microSDCard. The CS(Chip Select) on the SDCard Connector goes to Pin 5 of the PhotoPainter. The MISO port of the SDCard Connector goes to Pin 4 of the PhotoPainter.
```

### Software Components
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   ESP32 Master  │────│   I2C Protocol   │────│ PhotoPainter    │
│   (Fetcher)     │    │   600kHz, 123B   │    │ (Renderer)      │
│                 │    │   chunks         │    │                 │
│ • WiFi Connect  │    │                  │    │ • Image Receive │
│ • HTTP Download │    │                  │    │ • BMP Processing│
│ • Format Convert│    │                  │    │ • E-paper Draw  │
│ • I2C Stream    │    │                  │    │ • RTC Control   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 🚀 Quick Start

### Prerequisites
- Waveshare PhotoPainter device
- ESP32 Wemos D1 Mini or compatible board
- MicroSD card adapter ([example](https://amzn.eu/d/5vNI9uq))
- Basic soldering skills for SD slot connections. I soldered it onto above adapter to create a non permenant connection and also dont destroy the PhotoPainter on soldering misstakes.

### 1. Flash ESP32 Firmware
```bash
# Adopt Platformio.ini with your correct COM Ports
pio run -t upload -e Fetcher
```

### 3. Flash PhotoPainter Firmware
```bash
# Adopt Platformio.ini with your correct COM Ports
pio run -t upload -e Renderer
# Copy firmware.uf2 to PhotoPainter via USB
# See /docs/photopainter-flash.md for details
# The attached script works on windows and sets the PhotoPainter in Flash mode and uploads the firmware file.
```

### 4. Configure WiFi
```bash
# On first boot, ESP32 creates WiFi hotspot "ESP32-PhotoFrame"
# Connect and configure your network
```

### 5.
```bash
# Change hardcoded URL List in fetcher.cpp to point at the ones you want to download.
# I propose to use a webserver with a/some static files and the user overwrites them on the webserver so the client(esp32 fetcher) code can stay the same while rotating images soley over the webserver.
```

## 📁 Project Structure

```
ESP32PhotoFrame/
├── src/
│   ├── fetcher.cpp          # ESP32 firmware (WiFi + I2C master)
│   └── renderer.cpp         # PhotoPainter firmware (I2C slave + display)
├── lib/
│   ├── Config/              # Hardware config & I2C drivers
│   ├── EPaper/              # E-paper display drivers
│   ├── GUI/                 # BMP processing & color conversion
│   ├── Fonts/               # Text rendering
│   └── Examples/            # Test images/sample data
├── ImageConverter/
│   ├── dither.py            # Python image converter
│   ├── requirements.txt     # Python dependencies
│   └── IMAGE_CONVERTER.md   # Converter docs
├── datasheets/              # Hardware docs
├── utils/                   # Upload utilities/scripts
└── platformio.ini           # Build config
```

## 📄 License

This project is dual-licensed under MIT and Apache License 2.0. See [LICENSE](LICENSE) for details.

## 🙏 Acknowledgments

- **Waveshare** - For the excellent PhotoPainter hardware and documentation example project

## 🏗️ Build Software

Make sure you have `platform.io` installed. Then run `pio run`. On Windows, this
will install the tools to `C:\git\.platformio`. On Linux, this will create a
dirctory `C:/git/.platformio` in the current working directory. If you want some
different directory, change the `core_dir` in `platformio.ini`.

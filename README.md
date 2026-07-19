# OLED-BENCH: Video → 1-Bit Frame Encoder

An elegant, self-contained, retro-styled web utility to convert video files into 1-bit packed frame arrays. This tool is designed to easily display animations and video clips on **SSD1306** and **SH1106** OLED screens using **ESP32**, **Arduino Uno**, or **Raspberry Pi**.

---

## 🚀 Features

*   **Self-Contained Web App**: Just open [index.html] in any modern web browser. No local server, `npm install`, or complex build pipeline required.
*   **Live Preview**: Real-time render of the 1-bit display with a simulated retro CRT scanline overlay, phosphor-glow, and custom status indicators.
*   **Video Editing Controls**:
    *   **Trim Range**: Interactively select start and end timestamps.
    *   **Rotation**: Rotate source videos by 90° increments to match your display orientation.
    *   **Scale Modes**: Fit (proportional), Cover (fill + pan), Stretch, or Custom scaling (adjust width, height, and Pan X/Y offsets).
*   **Advanced Image Processing**:
    *   **Dither Modes**: Threshold (pure B/W), Bayer 8x8 ordered dithering, Floyd-Steinberg error diffusion, and Atkinson dithering.
    *   **Adjustments**: Brightness, contrast, B/W threshold bias, outline thickness (stroke modification), and color inversion.
*   **Flash Memory Estimation**: Live calculator indicating frame count and estimated size in KB to avoid compilation/memory overflow issues before conversion.
*   **Multi-Platform Code Generation**:
    1.  `VideoFrame.h`: A C++ header containing `PROGMEM` frame data for Arduino IDE.
    2.  `Main.ino`: Arduino code with timing control and driver initialization for SSD1306 & SH1106 displays.
    3.  `video_oled.py`: A Python script for Raspberry Pi with built-in frame arrays.

---

## 🛠️ Hardware Wiring & Guide

### 1. ESP32 (Recommended)
Excellent memory capacity (4MB+ flash) allows for longer, higher-resolution video sequences. Supports high-speed I2C communication (800kHz).

| OLED Pin | ESP32 Pin |
| :--- | :--- |
| **VCC** | 3.3V |
| **GND** | GND |
| **SCL** | GPIO 22 |
| **SDA** | GPIO 21 |

*   **Required Libraries**: `Adafruit SSD1306` (or `Adafruit SH110X` for SH1106) and `Adafruit GFX Library`.
*   **Upload Instructions**: Paste the generated `Main.ino` into your sketch, create a new tab named `VideoFrame.h`, paste the frame header data there, and upload to your board.

### 2. Arduino Uno
Flash memory is highly constrained (32KB total). A single 128×64 frame occupies 1024 bytes, allowing roughly **~28 frames max** before running out of memory.

| OLED Pin | Uno Pin |
| :--- | :--- |
| **VCC** | 5V |
| **GND** | GND |
| **SCL** | A5 |
| **SDA** | A4 |

*   *Tip*: Use lower resolution presets (e.g., `96x16` or `128x32`), trim the video to a very short duration, and keep target FPS low.
*   *Note*: In `Main.ino`, lower `Wire.setClock(800000)` to `Wire.setClock(400000)` because Uno hardware cannot safely run I2C at 800kHz.

### 3. Raspberry Pi
Runs natively using Python, loading frames dynamically without compiler limit issues.

| OLED Pin | Pi Header Pin |
| :--- | :--- |
| **VCC** | Pin 1 (3.3V) |
| **GND** | Pin 6 (GND) |
| **SCL** | Pin 5 (GPIO 3) |
| **SDA** | Pin 3 (GPIO 2) |

*   **Prerequisites**: Enable I2C interface via `sudo raspi-config` and reboot.
*   **Libraries**: Run `pip install luma.oled pillow`.
*   **Execution**: Download the generated `video_oled.py` (which has the frame array baked directly into it) and execute it using `python3 video_oled.py`.

---

## 📖 How to Use

1.  Open the [index.html](file:///c:/Users/ss357/OneDrive/Desktop/Vscode/ESP32-Video/index.html) file in your web browser.
2.  In the **SOURCE** panel, click **Choose File** and load a video (`.mp4`, `.mov`, etc.).
3.  Set your desired screen resolution under **DISPLAY TARGET**.
4.  Optionally trim the video, pick a scaling method, and apply dithering/image adjustments in the left panels.
5.  Watch the **LIVE PREVIEW** panel to inspect the visual rendering and verify that the estimated flash size fits your microcontroller's storage limit.
6.  Click **⚙ CONVERT VIDEO → CODE**.
7.  Copy or download the generated files from the tabs (`VideoFrame.h`, `Main.ino`, or `video_oled.py`).

---

## 🎨 Under the Hood: Frame Format

The video frames are encoded using a **1-bit per pixel format, MSB-first, packed row-by-row**. 
For a 128×64 screen:
*   Each horizontal line is 128 pixels.
*   With 8 pixels packed per byte, each row requires `128 / 8 = 16` bytes.
*   A complete frame occupies `16 * 64 = 1024` bytes.
*   In C++/Python arrays, `1` represents a lit/white pixel, and `0` represents a dark/black pixel.

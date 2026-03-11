# ESP32 Marauder – Pre-compiled Binary for NodeMCU-32S + Flipper Zero

This is a pre-compiled binary of [ESP32 Marauder](https://github.com/justcallmekoko/ESP32Marauder) configured to work with **Flipper Zero (Momentum firmware)** on a **generic NodeMCU-style ESP32-WROOM-32 board with CP2102 USB chip**.

If you struggled to compile Marauder yourself or couldn't get it to communicate with your Flipper — this is for you.

---

## Target Hardware

| Component | Details |
|-----------|---------|
| **ESP32 Board** | ESP-WROOM-32 with CP2102 (NodeMCU-compatible, 30-pin) |
| **Flipper Firmware** | Momentum (WiFi Marauder scripts) |
| **Marauder Version** | v1.11.0 |
| **Arduino ESP32 Core** | 2.0.9 |
| **Build Target** | `MARAUDER_FLIPPER` |

---

## Wiring – ESP32 → Flipper Zero

> ⚠️ Do **NOT** use the TX0/RX0 (GPIO1/GPIO3) pins labeled on the board — those are shared with the CP2102 USB chip and will block communication with Flipper.

Use **G26 / G27** instead:

| ESP32 Pin | Flipper GPIO Header |
|-----------|---------------------|
| **G26** | Pin 13 (TX) |
| **G27** | Pin 14 (RX) |
| **VIN** | Pin 1 (5V) |
| **GND** | Pin 11 (GND) |

Disconnect the USB cable from PC before using with Flipper.

---

## Flashing the Binary

### Option 1 – ESP Flash Download Tool (Windows, easiest)
1. Download [ESP Flash Download Tool](https://www.espressif.com/en/support/download/other-tools) from Espressif
2. Select chip: `ESP32`
3. Add the `.bin` file at address `0x10000`
4. Set baud rate to `921600`, select your COM port
5. Click **START**

### Option 2 – esptool (cross-platform)
```bash
esptool.py --chip esp32 --port COM9 --baud 921600 write_flash 0x10000 esp32_marauder_flipper_nodemcu32s.bin
```

### Option 3 – web flasher
Use [ESP Web Tools](https://web.esphome.io/) or similar browser-based flasher.

---

## What Was Disabled (and Why)

These features are present in the standard `MARAUDER_FLIPPER` build but were **disabled** because they cause crashes or boot loops on generic NodeMCU-32S hardware:

| Feature | Config Flag | Reason Disabled |
|---------|-------------|-----------------|
| PSRAM | `HAS_PSRAM` | This board has no PSRAM chip — caused `StoreProhibited` guru meditation crash at boot |
| SD Card | `HAS_SD` / `USE_SD` | No SD card slot on this board — caused watchdog reboot loop at boot |
| GPS | `HAS_GPS` | No GPS module — caused watchdog reboot loop at boot |
| Flipper RGB LED | `HAS_FLIPPER_LED` | LED pins (R=GPIO6, G=GPIO5, B=GPIO4) — GPIO6 is reserved for internal SPI flash on ESP32-WROOM-32, using it as OUTPUT causes immediate crash |

---

## Known Limitation

The standard TX0/RX0 pins (GPIO1/GPIO3) on this board are **permanently connected** to the CP2102 USB-to-UART chip, even when no USB cable is plugged in. This causes communication interference when the Flipper tries to talk to the ESP32 over those pins.

The fix is to remap UART to **GPIO26/GPIO27** in firmware (`Serial.begin(115200, SERIAL_8N1, 26, 27)`), which are free from the USB chip.

---

## License

This binary is built from the open-source [ESP32 Marauder](https://github.com/justcallmekoko/ESP32Marauder) project by [@justcallmekoko](https://github.com/justcallmekoko). All credit goes to the original author.

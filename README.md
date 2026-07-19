# Tuya CB3S RGBWW Downlight — ESPHome

> A personal journey of flashing a cheap AliExpress downlight with ESPHome.

## The story

I bought this RGBWW downlight on AliExpress thinking it would be another easy ESPHome flash, like the ESP8266/ESP32 devices I had done before. Spoiler: it was not.

The device is based on a **Tuya CB3S module**, which hides a **Beken BK7231N** SoC. That was my first surprise: this is **not** an ESP chip. Compared to ESP devices, Beken modules are more annoying to deal with: different pin naming, different flash tools, a bootloader that needs to be manually entered by grounding CEN, and a very picky power supply during flashing.

The whole process was quite frustrating for a beginner. I did not have a multimeter, so I could not trace the PCB. There was no exact YAML config available for this specific downlight, and the first attempts left the light stuck in red because the RGB pins were wrong. I learned the hard way that:

- The CB3S module must be powered from the **mains (220 V)** during flashing; powering it from the USB-UART 3.3 V rail is not reliable and can cause random disconnects.
- You must **never connect 3.3 V from the USB-UART adapter** to the Beken module while the mains PSU is also powering it. Only **GND, RX and TX** go to the UART adapter.
- To enter download mode, **bridge CEN to GND for a couple of seconds** right after starting the flashing tool.
- Beken pins are named `P6`, `P7`, `P8`, etc., not `GPIOxx`.
- Finding the correct PWM pins requires patience. I ended up flashing a diagnostic firmware with one monochromatic light per candidate pin, turning each one on and writing down the real color. That was the only way to get the correct mapping.

The correct pinout for my unit is:

| Function | Pin |
|----------|-----|
| Red      | P8  |
| Green    | P24 |
| Blue     | P26 |
| Cold white | P7 |
| Warm white | P6 |
| Touch button | P14 |

## Hardware

- Tuya CB3S module (Beken BK7231N)
- 220 V mains powered LED driver
- RGB + cold/warm white LEDs
- Capacitive touch button on the front cover

## Pinout reference

This project would not have been possible without the CB3S pinout diagram from **LibreTiny**:

[https://docs.libretiny.eu/boards/cb3s/#pinout](https://docs.libretiny.eu/boards/cb3s/#pinout)

Huge thanks to the LibreTiny team for documenting this module. Without that page I would have been completely lost.

## Photos

*Placeholders — photos will be added here.*

| Photo | Description |
|-------|-------------|
| `images/product.jpg` | The downlight as received from the store |
| `images/pcb.jpg` | The CB3S module on the PCB, shield removed |
| `images/wiring.jpg` | Wiring for flashing: GND, RX, TX and the CEN-GND bridge |

## Usage

1. Copy `downlight-cb3s-rgbww.yaml` to your ESPHome config folder.
2. Set your Wi-Fi and secrets in `secrets.yaml`.
3. Compile and flash.
4. If the colors are wrong for your unit, use the diagnostic approach: create one monochromatic light per PWM pin and identify which pin drives each color.

## Files

- `downlight-cb3s-rgbww.yaml` — complete ESPHome configuration example.
- `esphome-device/config.yaml` — hardware-only config for the ESPHome devices site.
- `esphome-device/index.md` — device page for the ESPHome devices site.

## License

MIT — use at your own risk, especially when working with mains voltage.

## Acknowledgements

- [LibreTiny](https://docs.libretiny.eu/) for the CB3S/BK72xx platform and documentation.
- The ESPHome community for making local smart home possible.
- Patience. Lots of it.

# Tuya CB3S RGBWW Downlight — ESPHome

> How two downlights bought by mistake, after a season forgotten in a wardrobe, ended up running ESPHome.

## The story

I bought several units of this Moes RGBWW downlight in its **Zigbee** version… and by mistake, two of them turned out to be the **WiFi** version. Installing Tuya devices that depend on some unknown third-party cloud is not an option for me, so both units went straight into a wardrobe.

Some time later I picked the project back up. The LEDs are really good quality, and it felt like a waste to simply not use them — or to install them as dumb on-off lights and lose the whole color range they offer. They deserved to be set free.

| ![Product](image1.png) | ![Two sizes](image2.png) |
|---|---|
| *The downlight as received from the store* | *It comes in two sizes: 145 mm and 115 mm* |

### First problem: what chip is this?

Neither the enclosure nor the PCB had the chip name printed anywhere. Nothing. Just a module hiding under a metal shield:

| ![Shielded module](image3.jpg) |
|---|
| *The module with its shield on: no way to know what's underneath* |

So there was no choice but to pry off the metal shield to find out which chip it was: a **Beken BK7231N**. Then, thanks to the **LibreTiny** documentation, I was able to verify the pinout and identify the module as a **Tuya CB3S**: https://docs.libretiny.eu/boards/cb3s/

| ![Chip exposed](image4.jpg) |
|---|
| *Shield removed: the Beken BK7231N finally visible* |

![CB3S pinout](cb3s.svg)

*CB3S pinout diagram, taken from the LibreTiny documentation.*

### Bekens are not ESPs

With those obstacles behind me, the "hard" work was done. Or so I thought. Beken chips are a bit different from ESPs, and there are a few things to keep in mind:

1. **The CEN bridge**: you must locate the CEN pin and bridge it to GND before powering the chip… but remove the bridge before the actual writing starts.
2. **Stable 3.3 V supply**: the chip needs a rock-solid 3.3 V supply. If your USB adapter can't provide it, flashing will fail. You can power it from an external 3.3 V supply or plug the downlight itself into mains. **WARNING if you do the latter: make sure the USB adapter's power line is NOT connected. Only GND, RX and TX go to the USB-UART adapter.**
3. **The order that worked for me**: power up the board, plug in the USB adapter, launch the flash command, wait a few seconds, then remove the CEN-to-GND bridge.

A breadboard helps a lot with this kind of juggling. I'm neither an electronics nor an IT expert, so all of this is beginner work… but what matters is reaching the goal.

Since I had downlights of both sizes, I can confirm that **both use the same board and the same software**.

| ![Both sizes wired](image5.jpg) | ![Soldering detail](image6.jpg) |
|---|---|
| *Both downlight sizes wired up for flashing* | *Wires soldered to the CB3S UART pads (GND, RX, TX)* |

### The color puzzle

With the module finally flashed, I tried a "generic" RGB light firmware. It didn't work on the first try: the light got stuck in red because the PWM pins didn't match. There was no exact YAML for this downlight anywhere, so I had to try every pin and combination: I flashed a diagnostic firmware with one monochromatic light per candidate pin, toggled them one by one and wrote down which color actually lit up. Tedious, but it worked. The YAML in this repo is the result and **works 100% with this device**.

The correct pinout for my unit is:

| Function | Pin |
|----------|-----|
| Red      | P8  |
| Green    | P24 |
| Blue     | P26 |
| Cold white | P7 |
| Warm white | P6 |
| Touch button | P14 |

(Mind you: Beken pins are named `P6`, `P7`, `P8`…, not `GPIOxx`.)

## Hardware

- Tuya CB3S module (Beken BK7231N)
- 220 V mains powered LED driver
- RGB + cold/warm white LEDs
- Capacitive touch button on the front cover
- Available in two sizes (145 mm and 115 mm); same board, same firmware

## Usage

1. Copy `downlight-cb3s-rgbww.yaml` to your ESPHome config folder.
2. Set your Wi-Fi in `secrets.yaml`.
3. Compile and flash.
4. If the colors are wrong for your unit, use the diagnostic approach: create one monochromatic light per PWM pin and identify which pin drives each color.

## Files

- `downlight-cb3s-rgbww.yaml` — complete ESPHome configuration example.
- `esphome-device/config.yaml` — hardware-only config for the ESPHome devices site.
- `esphome-device/index.md` — device page for the ESPHome devices site.
- `cb3s.svg` — CB3S pinout diagram (LibreTiny).

## License

AGPL-3.0 — use at your own risk, especially when working with mains voltage.

## Acknowledgements

- [LibreTiny](https://docs.libretiny.eu/) for the CB3S/BK72xx platform and documentation. Without the CB3S page I would have been completely lost.
- The ESPHome community for making local smart home possible.
- Patience. Lots of it.

---

[Versión en castellano](README.es.md)

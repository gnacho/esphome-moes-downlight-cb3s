---
title: Tuya CB3S RGBWW Downlight
description: ESPHome configuration for a Tuya CB3S (Beken BK7231N) based RGBWW downlight.
---

## Product Images

![Product](image1.jpg "Tuya CB3S RGBWW Downlight")
![PCB](image2.jpg "CB3S module on the PCB")
![Wiring](image3.jpg "Flashing wiring")

## Description

This is a mains-powered RGBWW downlight sold under various Tuya-compatible brands, commonly found on AliExpress. It uses the CB3S module with a Beken BK7231N SoC.

Flashing requires connecting only GND, RX and TX from a USB-UART adapter to the CB3S module, powering the downlight from mains (220 V), and bridging CEN to GND briefly to enter download mode.

## GPIO Pinout

| Pin    | Function        |
|--------|-----------------|
| P6     | Warm white PWM  |
| P7     | Cold white PWM  |
| P8     | Red PWM         |
| P24    | Green PWM       |
| P26    | Blue PWM        |
| P14    | Touch button    |

## Basic Configuration

```yaml
{% include "config.yaml" %}
```

## Acknowledgements

Thanks to [LibreTiny](https://docs.libretiny.eu/boards/cb3s/#pinout) for the CB3S pinout documentation.

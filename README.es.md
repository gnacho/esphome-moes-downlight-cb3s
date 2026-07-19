# Downlight RGBWW Tuya CB3S — ESPHome

> Mi historia personal flasheando un downlight barato de AliExpress con ESPHome.

## La historia

Compré este downlight RGBWW en AliExpress pensando que sería otro flasheo sencillo de ESPHome, como los dispositivos ESP8266/ESP32 que ya había hecho antes. Spoiler: no lo fue.

El dispositivo lleva un módulo **Tuya CB3S**, que esconde un SoC **Beken BK7231N**. Esa fue mi primera sorpresa: esto **no** es un ESP. Comparado con los ESP, los Beken son más incómodos: nombran los pines de otra forma, usan herramientas de flasheo distintas, el bootloader hay que meterlo manualmente puentenado CEN, y son muy quisquillosos con la alimentación durante el flasheo.

Todo el proceso fue bastante frustrante para un novato. No tenía multímetro, así que no podía rastrear la PCB. No había un YAML exacto para este downlight, y los primeros intentos dejaron la luz pillada en rojo porque los pines RGB estaban mal. Aprendí a la fuerza que:

- El módulo CB3S hay que alimentarlo desde **220 V** durante el flasheo; alimentarlo desde los 3,3 V del USB-UART no es fiable y provoca desconexiones aleatorias.
- **NUNCA hay que conectar los 3,3 V del adaptador USB-UART** al módulo Beken mientras la fuente de 220 V también lo alimenta. Solo van **GND, RX y TX** al adaptador.
- Para entrar en modo download hay que **puentear CEN a GND unos segundos** justo después de lanzar la herramienta de flasheo.
- Los pines Beken se llaman `P6`, `P7`, `P8`, etc., no `GPIOxx`.
- Encontrar los pines PWM correctos requiere paciencia. Acabé flasheando un firmware de diagnóstico con una luz monocromo por cada pin candidato, encendiendo una a una y apuntando qué color se iluminaba. Fue la única forma de obtener el mapeo correcto.

El pinout correcto de mi unidad es:

| Función | Pin |
|---------|-----|
| Rojo        | P8  |
| Verde       | P24 |
| Azul        | P26 |
| Blanco frío | P7  |
| Blanco cálido | P6  |
| Botón táctil | P14 |

## Hardware

- Módulo Tuya CB3S (Beken BK7231N)
- Driver LED alimentado a 220 V
- LEDs RGB + blanco frío/cálido
- Botón táctil capacitivo en la tapa frontal

## Referencia de pinout

Este proyecto no habría sido posible sin el diagrama de pines del CB3S de **LibreTiny**:

[https://docs.libretiny.eu/boards/cb3s/#pinout](https://docs.libretiny.eu/boards/cb3s/#pinout)

Muchísimas gracias al equipo de LibreTiny por documentar este módulo. Sin esa página me habría perdido por completo.

## Fotos

*Marcadores de posición — aquí se añadirán las fotos.*

| Foto | Descripción |
|------|-------------|
| `images/product.jpg` | El downlight tal como llegó de la tienda |
| `images/pcb.jpg` | El módulo CB3S en la PCB, con el blindaje quitado |
| `images/wiring.jpg` | Cableado para flashear: GND, RX, TX y el puente CEN-GND |

## Uso

1. Copia `downlight-cb3s-rgbww.yaml` a tu carpeta de configuración de ESPHome.
2. Configura tu Wi-Fi y secretos en `secrets.yaml`.
3. Compila y flashea.
4. Si los colores no coinciden en tu unidad, usa el enfoque de diagnóstico: crea una luz monocromo por pin PWM e identifica qué pin controla cada color.

## Archivos

- `downlight-cb3s-rgbww.yaml` — configuración completa de ejemplo para ESPHome.
- `esphome-device/config.yaml` — configuración hardware-only para el sitio de dispositivos de ESPHome.
- `esphome-device/index.md` — página del dispositivo para el sitio de dispositivos de ESPHome.

## Licencia

MIT — úsalo bajo tu propia responsabilidad, especialmente cuando trabajes con tensión de red.

## Agradecimientos

- [LibreTiny](https://docs.libretiny.eu/) por la plataforma CB3S/BK72xx y su documentación.
- A la comunidad de ESPHome por hacer posible el hogar inteligente local.
- A la paciencia. Mucha paciencia.

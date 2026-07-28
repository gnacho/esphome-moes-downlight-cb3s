# Downlight RGBWW Tuya CB3S — ESPHome

> Cómo dos downlights comprados por error, tras una temporada en un armario, acabaron funcionando con ESPHome.

## La historia

Compré varias unidades de este downlight RGBWW de Moes en su versión **Zigbee**… y por error, dos de ellas resultaron ser la versión **WiFi**. Instalar dispositivos Tuya que dependen de un cloud desconocido y ajeno no es una opción para mí, así que las dos unidades se fueron directas a un armario.

Tiempo después retomé el asunto. Los LEDs son de muy buena calidad y era un desperdicio simplemente no usarlos, o instalarlos como luces «tontas» de on-off perdiendo toda la gama de colores que tienen. Había que liberarlos.

| ![Producto](image1.png) | ![Dos tamaños](image2.png) |
|---|---|
| *El downlight tal como llegó de la tienda* | *Existen dos tamaños: 145 mm y 115 mm* |

### Primer problema: ¿qué chip lleva esto?

Ni el encapsulado ni la placa tenían impreso el nombre del chip. Nada. Solo un módulo con su blindaje metálico:

| ![Módulo con blindaje](image3.jpg) |
|---|
| *El módulo con el blindaje puesto: imposible saber qué hay debajo* |

Así que no quedó otra que extraer la protección metálica para averiguar primero el chip: un **Beken BK7231N**. Después, gracias a la documentación de **LibreTiny**, pude verificar el pinout e identificar el módulo como un **Tuya CB3S**: https://docs.libretiny.eu/boards/cb3s/

| ![Chip a la vista](image4.jpg) |
|---|
| *Blindaje retirado: el Beken BK7231N por fin a la vista* |

![Pinout CB3S](cb3s.svg)

*Diagrama de pines del CB3S, extraído de la documentación de LibreTiny.*

### Los Beken no son ESP

Superados estos obstáculos, el trabajo «difícil» ya estaba hecho. O eso creía. Los chips Beken son un poco diferentes a los ESP y hay que tener en cuenta algunos factores:

1. **El puente CEN**: hay que localizar el pin CEN y puentearlo con GND antes de encender el chip… pero quitar el puente antes de que empiece la escritura.
2. **Alimentación 3,3 V estable**: el chip necesita un suministro de 3,3 V sólido. Si el USB no lo puede proporcionar, el flasheo fallará. Puedes alimentarlo con una fuente externa de 3,3 V o conectar el propio downlight a la red eléctrica. **OJO si haces esto último: verifica que la alimentación del USB NO esté conectada. Al adaptador USB-UART solo van GND, RX y TX.**
3. **El orden que a mí me funcionó**: encender la placa, conectar el USB, lanzar el comando de flasheo, esperar unos segundos y quitar el puente CEN a GND.

Una placa de prototipos (protoboard) ayuda muchísimo para estas cosas. Yo no soy experto ni en electrónica ni en informática, así que todo esto es trabajo de novato… pero lo importante es conseguir el objetivo.

Como tenía downlights de los dos tamaños, puedo confirmar que **ambos usan la misma placa y el mismo software**.

| ![Los dos tamaños cableados](image5.jpg) | ![Detalle de soldaduras](image6.jpg) |
|---|---|
| *Los dos tamaños de downlight cableados para el flasheo* | *Cables soldados a los pads UART del CB3S (GND, RX, TX)* |

### El puzle de los colores

Con el módulo por fin flasheado, probé un firmware «genérico» para luces RGB. No funcionó a la primera: la luz se quedaba pillada en rojo porque los pines PWM no coincidían. No había ningún YAML exacto para este downlight en ningún sitio, así que no quedó otra que probar todos los pines y combinaciones: flasheé un firmware de diagnóstico con una luz monocromo por cada pin candidato, fui encendiéndolas una a una y apuntando qué color se iluminaba de verdad. Tedioso, pero funcionó. El YAML de este repo es el resultado y **funciona al 100% con este dispositivo**.

El pinout correcto de mi unidad es:

| Función | Pin |
|---------|-----|
| Rojo        | P8  |
| Verde       | P24 |
| Azul        | P26 |
| Blanco frío | P7  |
| Blanco cálido | P6  |
| Botón táctil | P14 |

(Ojo: los pines Beken se llaman `P6`, `P7`, `P8`…, no `GPIOxx`.)

## Hardware

- Módulo Tuya CB3S (Beken BK7231N)
- Driver LED alimentado a 220 V
- LEDs RGB + blanco frío/cálido
- Botón táctil capacitivo en la tapa frontal
- Disponible en dos tamaños (145 mm y 115 mm); misma placa y mismo firmware

## Cableado para el flasheo

Soldad hilos finos a los pads UART del CB3S: **GND, RX (P10) y TX (P11)** hacia tu adaptador USB-UART, más un puente temporal entre **CEN y GND** (una protoboard ayuda muchísimo aquí).

- **Recomendado**: alimentar la propia placa del downlight a la red eléctrica (su fuente interna es sólida de verdad). En ese caso, **NO conectes la línea 3V3 del adaptador USB: solo van GND, RX y TX.**
- Alternativa: alimentar el chip con una fuente externa de 3,3 V de al menos 500 mA. El pin 3V3 de la mayoría de adaptadores USB no aguanta los picos de corriente del WiFi y el flasheo fallará.

**OJO: estás trabajando con tensión de red. Revisa el cableado antes de enchufar nada y no toques la placa mientras esté alimentada.**

La foto del detalle de soldaduras (image6) muestra exactamente cómo queda.

## Respaldo del firmware original (muy recomendado)

Antes de flashear nada, vuelca el firmware Tuya original. Solo lleva unos minutos y es tu billete de vuelta: con ese fichero puedes devolver el downlight a su estado de fábrica en cualquier momento.

La secuencia que funciona (el orden importa):

1. Placa **apagada**.
2. Puentea **CEN a GND**.
3. Conecta el adaptador USB (solo GND/RX/TX).
4. Lanza el comando de lectura:
   ```bash
   ltchiptool flash read bk7231n moes-downlight-backup.bin -d /dev/ttyUSB0
   ```
5. **Enciende la placa.**
6. **Quita el puente CEN-GND.**
7. Espera a que termine el volcado. El backup debe ocupar exactamente **2 MiB** (2097152 bytes).

Guarda ese fichero a buen recaudo.

Este repo ya incluye mi propio volcado de fábrica: `firmware/moes-downlight-cb3s-stock.bin` (saneado — el bloque de config Tuya con mis credenciales WiFi fue borrado, así que una unidad restaurada arrancará de fábrica pero sin emparejar). Debería funcionar en cualquier unidad de este downlight, pero **hacer tu propio volcado sigue siendo recomendable**.

### Restaurar el firmware original

Si algún día quieres volver a Tuya, escribe el backup con el mismo cableado y la misma secuencia de encendido:

```bash
ltchiptool flash write moes-downlight-backup.bin -d /dev/ttyUSB0
```

## Flasheo de ESPHome

1. Copia `downlight-cb3s-rgbww.yaml` a tu carpeta de configuración de ESPHome.
2. Configura tu Wi-Fi en `secrets.yaml` (`wifi_ssid` / `wifi_password`).
3. Compila el firmware:
   ```bash
   esphome compile downlight-cb3s-rgbww.yaml
   ```
   El `.uf2` queda en `.esphome/build/<nombre-nodo>/.pioenvs/<nombre-nodo>/firmware.uf2`.
4. Flashea con el mismo cableado y la misma secuencia de encendido que el backup:
   ```bash
   ltchiptool flash write firmware.uf2 -d /dev/ttyUSB0
   ```
5. En el primer arranque el downlight se conecta a tu Wi-Fi. Si no puede, levanta un AP de respaldo con portal cautivo para que lo reconfigures.

## Actualizaciones OTA

El flasheo por cable solo hace falta una vez. A partir de ahí, actualiza por WiFi desde el dashboard de ESPHome o con:

```bash
esphome run downlight-cb3s-rgbww.yaml
```

## Resolución de problemas

- **La lectura funciona pero la escritura muere en el primer sector (`0x11000`, "No response received")**: casi siempre es alimentación 3,3 V insuficiente desde el adaptador USB. Alimenta la propia placa y conecta solo GND/RX/TX. Bajar el baudrate no lo arregla.
- **`Timeout attempting to link with chip`**: el chip no está en modo download. Revisa el puente CEN-GND y repite la secuencia de encendido; la temporización importa.
- **Lecturas/escrituras inestables**: hilos cortos, conexiones firmes (solda, no te fíes de duponts sueltos) y un adaptador USB-UART decente.
- **No conecta al WiFi tras flashear**: la config incluye un AP de respaldo con portal cautivo — conéctate y reintroduce tus credenciales. Verifica también que tu `secrets.yaml` estaba junto al YAML al compilar.
- **Los colores no coinciden en tu unidad**: usa el enfoque de diagnóstico descrito arriba — una luz monocromo por cada pin PWM candidato (`P6`, `P7`, `P8`, `P24`, `P26`), enciéndelas una a una y apunta qué color se ilumina de verdad.

## Archivos

- `downlight-cb3s-rgbww.yaml` — configuración completa de ejemplo para ESPHome.
- `esphome-device/config.yaml` — configuración hardware-only para el sitio de dispositivos de ESPHome.
- `esphome-device/index.md` — página del dispositivo para el sitio de dispositivos de ESPHome.
- `cb3s.svg` — diagrama de pines del CB3S (LibreTiny).

## Licencia

AGPL-3.0 — úsalo bajo tu propia responsabilidad, especialmente cuando trabajes con tensión de red.

## Agradecimientos

- [LibreTiny](https://docs.libretiny.eu/) por la plataforma CB3S/BK72xx y su documentación. Sin la página del CB3S me habría perdido por completo.
- A la comunidad de ESPHome por hacer posible el hogar inteligente local.
- A la paciencia. Mucha paciencia.

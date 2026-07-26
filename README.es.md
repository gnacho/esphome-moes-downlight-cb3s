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

## Uso

1. Copia `downlight-cb3s-rgbww.yaml` a tu carpeta de configuración de ESPHome.
2. Configura tu Wi-Fi en `secrets.yaml`.
3. Compila y flashea.
4. Si los colores no coinciden en tu unidad, usa el enfoque de diagnóstico: crea una luz monocromo por pin PWM e identifica qué pin controla cada color.

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

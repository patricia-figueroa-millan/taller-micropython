# Tema 2. Manejo de Entradas y Salidas Digitales

## Introducción
Introducción

Los sistemas embebidos modernos interactúan continuamente con el entorno mediante dispositivos de entrada y salida. Las entradas permiten adquirir información proveniente de sensores, mientras que las salidas permiten comunicar información al usuario o controlar otros dispositivos.

En plataformas como ESP32, MicroPython proporciona mecanismos sencillos para acceder a estos recursos de hardware, facilitando la integración de sensores, pantallas, actuadores y módulos de comunicación dentro de una misma aplicación.

En el contexto de Internet de las Cosas (IoT), las entradas y salidas digitales constituyen el punto de enlace entre el mundo físico y el mundo digital, permitiendo capturar variables del entorno, procesarlas y generar respuestas locales o remotas.

Como proyecto integrador del curso se desarrollará una Red Distribuida de Estaciones Ambientales IoT con MicroPython, conformada por múltiples nodos basados en ESP32 capaces de adquirir información ambiental, comunicarla mediante MQTT, almacenar los datos en la nube y permitir su monitoreo y configuración remota mediante una aplicación web.

A lo largo del curso, la solución será construida de manera incremental. En este tema se desarrollará el primer nodo de la red, implementando una estación ambiental local capaz de medir temperatura y humedad mediante un sensor DHT22 y visualizar la información en una pantalla OLED SSD1306.

Durante el desarrollo del tema, además de aprender a utilizar sensores y dispositivos de visualización, se introducirá una metodología práctica para explorar bibliotecas y objetos de MicroPython mediante el REPL, permitiendo que los participantes puedan descubrir y comprender nuevas funcionalidades de forma autónoma.

---

# 2.1 Programación de Periféricos en MicroPython

## Objetivo

Comprender cómo MicroPython abstrae el hardware mediante bibliotecas y objetos para facilitar el desarrollo de aplicaciones embebidas.

## Del hardware al software

Cuando se desarrolla software embebido tradicionalmente se interactúa con registros y configuraciones de bajo nivel.

MicroPython proporciona una capa de abstracción que permite representar sensores, pantallas y periféricos mediante objetos.

Conceptualmente:

Dispositivo físico → Biblioteca → Clase → Objeto → Métodos

Ejemplo:

```python
from machine import Pin

led = Pin(2, Pin.OUT)
```

El objeto `led` representa un GPIO físico y permite controlarlo mediante métodos como:

```python
led.on()
led.off()
```

## Bibliotecas más utilizadas

- machine
- time / utime
- network
- dht
- ssd1306
- umqtt.simple
- os
- socket
- framebuf

## Actividad 1

Explorar un GPIO desde el REPL:

```python
from machine import Pin

led = Pin(2, Pin.OUT)

type(led)
dir(led)

led.on()
led.off()
```

---

# 2.2 Exploración de Bibliotecas y Objetos desde el REPL

## Objetivo

Aprender a descubrir funcionalidades de bibliotecas y objetos mediante REPL.

### Explorando módulos

```python
import machine

dir(machine)
```

### Explorando clases

```python
from machine import Pin
dir(Pin)
```

```python
from machine import I2C
dir(I2C)
```

### Explorando objetos

```python
from machine import Pin

led = Pin(2, Pin.OUT)

type(led)
dir(led)
```

### Recomendación

Antes de utilizar una biblioteca:

1. Importar el módulo.
2. Ejecutar dir().
3. Crear un objeto.
4. Ejecutar type().
5. Ejecutar dir() sobre el objeto.
6. Identificar métodos.
7. Consultar documentación cuando sea necesario.

---

# 2.3 Instalación y Verificación de Bibliotecas Externas

## Bibliotecas incluidas

MicroPython incorpora bibliotecas como:

```python
machine
time
utime
network
os
dht
```

## Bibliotecas externas

Algunas bibliotecas deben copiarse manualmente al ESP32.

Ejemplo:

```text
ssd1306.py
```

## Obtención de la biblioteca SSD1306

Repositorio oficial:

https://github.com/micropython/micropython-esp32/

Ruta:

```text
drivers/display/ssd1306.py
```

Enlace directo:

https://github.com/micropython/micropython-esp32/blob/esp32/drivers/display/ssd1306.py

## Instalación desde Thonny

1. Abrir Thonny.
2. Crear un archivo nuevo.
3. Copiar el contenido de ssd1306.py.
4. Guardar como:
   
```text
ssd1306.py
```

5. Guardar en la memoria del ESP32.

# 2.4 Integración de Dispositivos mediante I2C: OLED SSD1306

La pantalla OLED SSD1306 utiliza el protocolo de comunicación I2C (Inter-Integrated Circuit), ampliamente empleado en sistemas embebidos para conectar sensores, memorias, pantallas y otros periféricos.

Una de sus principales ventajas es que múltiples dispositivos pueden compartir el mismo bus utilizando únicamente dos líneas de comunicación.

```text
SDA → Datos
SCL → Reloj
```

En esta arquitectura el ESP32 actúa como maestro y coordina la comunicación con los dispositivos conectados al bus.

## Conexión de la pantalla OLED

| OLED | ESP32 |
|------|--------|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO21 |
| SCL | GPIO22 |

## Creación del bus I2C

```python
from machine import Pin, I2C

i2c = I2C(
    0,
    scl=Pin(22),
    sda=Pin(21),
    freq=100000
)
```
En caso de que se muestre un error de =="I2C operation not supported"==, es importante conocer que este generalmente indica una limitación a nivel de hardware con el microcontrolador, una interfaz 12c deshabilitada.

Esta limitación puede ser resuelta cambiando a Software I2C en MicroPython, como se muestra a continuación:

```python
from machine import Pin, SoftI2C

i2c = SoftI2C(
    scl=Pin(22),
    sda=Pin(21),
    freq=400000)

```

## Escaneo de dispositivos

```python
i2c.scan()
```

Resultado esperado:

```python
[60]
```

o

```python
[61]
```

Para esto, es importante realizar la operación usando  

## Creación del objeto OLED

```python
import ssd1306

oled = ssd1306.SSD1306_I2C(
    128,
    64,
    i2c
)
```

## Exploración

```python
type(oled)
dir(oled)
```

## Primera prueba

```python
oled.fill(0)
oled.text("OLED OK",0,0)
oled.show()
```

---

# 2.5 Integración de Sensores Digitales: DHT22

## Conexión

| DHT22 | ESP32 |
|--------|--------|
| VCC | 3.3V |
| GND | GND |
| DATA | GPIO4 |

## Creación del objeto

```python
import dht
from machine import Pin

sensor = dht.DHT22(
    Pin(4)
)
```

## Exploración

```python
type(sensor)
dir(sensor)
```

## Lectura

```python
sensor.measure()

temperatura = sensor.temperature()
humedad = sensor.humidity()
```

## Actividad

```python
sensor.measure()

print(sensor.temperature())
print(sensor.humidity())
```

---

# 2.6 Construcción del Nodo Ambiental Local

## Arquitectura

DHT22 → ESP32 → OLED SSD1306

## Código base

```python
import machine
import dht
import ssd1306
import utime

sensor = dht.DHT22(machine.Pin(4))

i2c = machine.I2C(
    0,
    scl=machine.Pin(22),
    sda=machine.Pin(21)
)

oled = ssd1306.SSD1306_I2C(
    128,
    64,
    i2c
)

while True:

    sensor.measure()

    temp = sensor.temperature()
    hum = sensor.humidity()

    oled.fill(0)
    oled.text("Temp:{:.1f}C".format(temp),0,0)
    oled.text("Hum:{:.1f}%".format(hum),0,12)
    oled.show()

    utime.sleep(2)
```

## Variables de configuración

```python
temp_max = 30
hum_min = 40
hum_max = 80
```

## Evaluación de condiciones

```python
if temp > temp_max:
    estado = "TEMP ALTA"
elif hum < hum_min:
    estado = "HUM BAJA"
elif hum > hum_max:
    estado = "HUM ALTA"
else:
    estado = "NORMAL"
```

## Relación con el proyecto

Este nodo constituye la primera versión funcional de la Red Distribuida de Estaciones Ambientales IoT con MicroPython.

Posteriormente se incorporarán:

1. WiFi.
2. MQTT.
3. Configuración remota.
4. Wildcards MQTT.
5. Supabase.
6. Dashboard web.

# 2.4 Integración de Dispositivos mediante I2C: OLED SSD1306

## Objetivo

Comprender el funcionamiento básico del protocolo I2C e integrar una pantalla OLED SSD1306 utilizando MicroPython.

## ¿Qué es I2C?

I2C (Inter-Integrated Circuit) es uno de los protocolos de comunicación más utilizados en sistemas embebidos para conectar sensores, memorias, pantallas y otros dispositivos periféricos.

Su principal ventaja es que permite conectar múltiples dispositivos utilizando únicamente dos líneas de comunicación.

```text
SDA → Datos
SCL → Reloj
```

El ESP32 actúa como dispositivo maestro y coordina la comunicación con los dispositivos esclavos conectados al bus.

---

## Arquitectura de comunicación I2C

```text
           ESP32
      ┌────────────┐
SDA ──┤ GPIO21     │
SCL ──┤ GPIO22     │
      └─────┬──────┘
            │
     ┌──────┴──────┐
     │ OLED SSD1306│
     └─────────────┘
```

## Conexión de la OLED SSD1306

| OLED | ESP32 |
|--------|--------|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO21 |
| SCL | GPIO22 |

## Creación del bus I2C

```python
from machine import Pin, I2C

i2c = I2C(
    0,
    scl=Pin(22),
    sda=Pin(21),
    freq=100000
)
```

## Explorando el objeto I2C

```python
type(i2c)
```

```python
dir(i2c)
```

Observar métodos como:

```text
scan
readfrom
writeto
readfrom_mem
writeto_mem
```

## Descubriendo dispositivos conectados

```python
i2c.scan()
```

Resultado típico:

```python
[60]
```

o

```python
[61]
```

## Direcciones I2C

| Decimal | Hexadecimal |
|----------|------------|
| 60 | 0x3C |
| 61 | 0x3D |

## Diagnóstico de errores

### Resultado vacío

```python
[]
```

Posibles causas:

- Alimentación incorrecta.
- SDA invertido.
- SCL invertido.
- Conexión floja.
- Pantalla defectuosa.

## Creación del objeto OLED

```python
import ssd1306

oled = ssd1306.SSD1306_I2C(
    128,
    64,
    i2c
)
```

## Explorando el objeto OLED

```python
type(oled)
```

```python
dir(oled)
```

Métodos importantes:

```text
fill
show
text
pixel
line
rect
fill_rect
invert
poweroff
poweron
```

## Primera prueba

```python
oled.fill(0)
oled.text("OLED OK", 0, 0)
oled.show()
```

## Comprendiendo el búfer gráfico

```python
oled.text("Hola",0,0)
```

Modifica el buffer.

```python
oled.show()
```

Actualiza físicamente la pantalla.

## Actividad 4

Mostrar:

```text
MicroPython
ESP32
OLED SSD1306
```

en diferentes posiciones de la pantalla.

---

# 2.5 Integración de Sensores Digitales: DHT22

## Objetivo

Comprender cómo interactuar con sensores digitales utilizando bibliotecas especializadas de MicroPython.

## ¿Qué es el DHT22?

El DHT22 es un sensor digital capaz de medir:

- Temperatura.
- Humedad relativa.

A diferencia de sensores analógicos, el DHT22 transmite la información ya digitalizada.

## Conexión del DHT22

| DHT22 | ESP32 |
|--------|--------|
| VCC | 3.3V |
| GND | GND |
| DATA | GPIO4 |

## Importando la biblioteca

```python
import dht
```

## Explorando el módulo

```python
dir(dht)
```

## Creación del objeto sensor

```python
from machine import Pin

sensor = dht.DHT22(
    Pin(4)
)
```

## Explorando el objeto

```python
type(sensor)
```

```python
dir(sensor)
```

Métodos principales:

```text
measure
temperature
humidity
```

## Comprendiendo el proceso de medición

```python
sensor.measure()
```

Actualiza internamente los valores del sensor.

Posteriormente:

```python
sensor.temperature()
sensor.humidity()
```

## Lectura completa

```python
sensor.measure()

temperatura = sensor.temperature()
humedad = sensor.humidity()

print(temperatura)
print(humedad)
```

## Actividad 6

Realizar cinco mediciones consecutivas y registrar los resultados.

## Diagnóstico de errores

### OSError

```text
OSError: [Errno 110] ETIMEDOUT
```

Posibles causas:

- Cable DATA incorrecto.
- Alimentación insuficiente.
- Conexión defectuosa.

---

# 2.6 Construcción del Nodo Ambiental Local

## Objetivo

Integrar los dispositivos estudiados para construir el primer nodo funcional de la red distribuida.

## Arquitectura

```text
DHT22
   ↓
ESP32
   ↓
OLED SSD1306
```

## Integración inicial

```python
import machine
import dht
import ssd1306
import utime

sensor = dht.DHT22(machine.Pin(4))

i2c = machine.I2C(
    0,
    scl=machine.Pin(22),
    sda=machine.Pin(21)
)

oled = ssd1306.SSD1306_I2C(
    128,
    64,
    i2c
)
```

## Lectura y visualización

```python
while True:

    sensor.measure()

    temp = sensor.temperature()
    hum = sensor.humidity()

    oled.fill(0)

    oled.text(
        "Temp:{:.1f}C".format(temp),
        0,
        0
    )

    oled.text(
        "Hum:{:.1f}%".format(hum),
        0,
        12
    )

    oled.show()

    utime.sleep(2)
```

## Variables de configuración

```python
temp_max = 30
hum_min = 40
hum_max = 80
```

## Evaluación de condiciones

```python
if temp > temp_max:
    estado = "TEMP ALTA"

elif hum < hum_min:
    estado = "HUM BAJA"

elif hum > hum_max:
    estado = "HUM ALTA"

else:
    estado = "NORMAL"
```

## Mostrando estados

```python
oled.text(
    estado,
    0,
    32
)
```

## Actividad 7

Modificar:

```python
temp_max = 25
```

Observar cómo cambia el comportamiento del sistema.

## Preparación para MQTT

Las variables:

```python
temp_max
hum_min
hum_max
```

serán configuradas remotamente mediante MQTT en temas posteriores.

## Relación con el Proyecto Integrador

La estación ambiental construida en este tema representa el primer nodo funcional de la Red Distribuida de Estaciones Ambientales IoT con MicroPython.

## Resultado Final

Al concluir este tema el participante dispondrá de un nodo ambiental capaz de:

- Medir temperatura.
- Medir humedad.
- Mostrar información localmente.
- Generar alertas básicas.
- Utilizar bibliotecas externas.
- Explorar objetos mediante REPL.
- Servir como base para una arquitectura IoT distribuida.

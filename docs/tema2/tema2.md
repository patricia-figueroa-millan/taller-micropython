# Tema 2. Manejo de Entradas y Salidas Digitales

## Introducción

En este tema se desarrollará el primer incremento funcional del proyecto integrador **"Red Distribuida de Estaciones Ambientales IoT con MicroPython"**, mediante la construcción de uno de los nodos que conformarán la red distribuida. Este nodo estará basado en una tarjeta ESP32 y será capaz de adquirir datos de temperatura y humedad utilizando un sensor DHT22, procesar la información localmente y visualizar los resultados en una pantalla OLED SSD1306.

La estación ambiental desarrollada durante este tema constituirá la base sobre la cual, en los siguientes temas, se incorporarán capacidades de comunicación MQTT, configuración remota, almacenamiento en Supabase y visualización mediante un dashboard web.

---

# 2.1 Programación de Periféricos en MicroPython

## Objetivo

Comprender cómo MicroPython permite abstraer y controlar sensores, dispositivos de comunicación y periféricos mediante bibliotecas y objetos, facilitando el desarrollo de aplicaciones embebidas.

## Del hardware al software

Cuando desarrollamos sistemas embebidos, no interactuamos directamente con los registros internos del microcontrolador, como suele hacerse en lenguajes de bajo nivel.

MicroPython proporciona una capa de abstracción que permite representar los componentes físicos mediante objetos y bibliotecas especializadas.

Gracias a esta abstracción, sensores, pantallas y buses de comunicación pueden ser manipulados mediante código Python de manera más sencilla, legible y mantenible.

## Bibliotecas en MicroPython

Las funcionalidades del hardware se encuentran organizadas en módulos o bibliotecas que encapsulan el acceso a los diferentes periféricos.

Algunas de las bibliotecas más utilizadas en aplicaciones embebidas e IoT son:

| Módulo | Función |
|---------|----------|
| machine | Acceso al hardware del microcontrolador |
| time / utime | Temporización y control del tiempo |
| network | Conectividad WiFi |
| dht | Lectura de sensores DHT11 y DHT22 |
| ssd1306 | Control de pantallas OLED SSD1306 |
| umqtt.simple | Comunicación MQTT |

Ejemplo:

```python
import machine
import dht
import ssd1306
```

## Dispositivos físicos representados mediante objetos

En MicroPython, los dispositivos físicos suelen representarse mediante objetos creados a partir de clases incluidas en las bibliotecas.

Por ejemplo:

```python
sensor = dht.DHT22(...)
oled = ssd1306.SSD1306_I2C(...)
```

Estos objetos encapsulan las operaciones necesarias para interactuar con el hardware.

## Resultado esperado

Al finalizar esta sección el participante será capaz de:

- Comprender cómo MicroPython abstrae el hardware mediante bibliotecas.
- Identificar los módulos más utilizados en aplicaciones embebidas.
- Comprender cómo sensores y periféricos son representados mediante objetos dentro del software.

---

# 2.2 Exploración de Bibliotecas y Objetos desde el REPL

## Exploración de bibliotecas y objetos en MicroPython

Una de las ventajas de MicroPython es que permite interactuar de forma dinámica con los objetos desde el REPL. Esto facilita explorar nuevas bibliotecas y descubrir sus funcionalidades sin necesidad de conocer previamente todos sus métodos.

Una vez creado un objeto, es posible inspeccionar sus atributos y métodos utilizando la función integrada `dir()`.

Por ejemplo:

```python
import dht
from machine import Pin

sensor = dht.DHT22(Pin(4))

dir(sensor)
```

Salida esperada:

```python
['humidity', 'measure', 'temperature']
```

Esta información permite identificar las operaciones disponibles para interactuar con el sensor.

## Identificando el tipo de un objeto

También es posible conocer el tipo de un objeto mediante la función `type()`.

```python
type(sensor)
```

Resultado:

```python
<class 'DHT22'>
```

Esto permite comprender qué clase fue utilizada para crear el objeto y facilita consultar posteriormente la documentación correspondiente.

## Explorando otros periféricos

La misma estrategia puede emplearse con cualquier otro dispositivo.

Por ejemplo:

```python
dir(oled)
```

podría mostrar métodos similares a:

```python
[
 'fill',
 'show',
 'text',
 'pixel',
 'line',
 'rect',
 'invert',
 'poweroff',
 'poweron'
]
```

A partir de esta información es posible inferir muchas de las capacidades del dispositivo antes incluso de revisar la documentación.

## Programación exploratoria en MicroPython

Durante el desarrollo de sistemas embebidos es frecuente trabajar con bibliotecas nuevas o periféricos desconocidos.

Por ello, herramientas como:

```python
dir()
type()
```

constituyen mecanismos útiles para explorar objetos, descubrir funcionalidades y comprender la estructura de una biblioteca de forma interactiva desde el REPL.

## Recomendación para el desarrollo del curso

Antes de utilizar cualquier biblioteca nueva, se recomienda:

1. Importar el módulo.
2. Crear un objeto.
3. Ejecutar `type()`.
4. Ejecutar `dir()`.
5. Identificar los métodos disponibles.
6. Consultar la documentación únicamente cuando sea necesario.

## Aplicación al proyecto

Durante este curso esta estrategia se utilizará con:

- dht
- ssd1306
- network
- umqtt.simple

y cualquier otra biblioteca necesaria para construir la Red Distribuida de Estaciones Ambientales IoT con MicroPython.

---

# 2.3 Integración de Sensores Digitales: DHT22

## Objetivo

Comprender cómo utilizar una biblioteca de MicroPython para interactuar con un sensor digital.

## Explorando la biblioteca

```python
import dht

dir(dht)
```

## Creación del objeto sensor

```python
from machine import Pin

sensor = dht.DHT22(Pin(4))
```

## Exploración del objeto

```python
type(sensor)
```

```python
dir(sensor)
```

Los métodos principales son:

```python
measure()
temperature()
humidity()
```

## Lectura de mediciones

```python
sensor.measure()

temperatura = sensor.temperature()
humedad = sensor.humidity()
```

## Validación desde el REPL

El REPL permite verificar rápidamente:

- Funcionamiento del sensor.
- Conexiones.
- Valores obtenidos.

## Resultado esperado

Al finalizar esta sección el participante será capaz de:

- Crear un objeto asociado al DHT22.
- Explorar sus métodos.
- Obtener mediciones de temperatura y humedad.

---

# 2.4 Integración de Dispositivos de Visualización: OLED SSD1306

## Objetivo

Comprender cómo utilizar una biblioteca de MicroPython para interactuar con una pantalla OLED.

## Explorando la biblioteca

```python
import ssd1306

dir(ssd1306)
```

## Creación del objeto OLED

```python
oled = ssd1306.SSD1306_I2C(...)
```

## Exploración del objeto

```python
type(oled)
```

```python
dir(oled)
```

Métodos comunes:

```python
fill()
show()
text()
pixel()
line()
rect()
```

## Primer ejemplo

```python
oled.text("Hola",0,0)
oled.show()
```

## Visualización de información

La pantalla OLED permitirá:

- Mostrar temperatura.
- Mostrar humedad.
- Mostrar estados.
- Mostrar alertas.

## Resultado esperado

Al finalizar esta sección el participante será capaz de:

- Crear un objeto OLED.
- Explorar sus métodos.
- Mostrar información en pantalla.

---

# 2.5 Construcción del Nodo Ambiental Local

## Objetivo

Integrar el sensor DHT22 y la pantalla OLED en una aplicación funcional.

## Arquitectura del nodo

```text
DHT22
   ↓
ESP32
   ↓
OLED SSD1306
```

## Variables de configuración

```python
temp_max = 30
hum_min = 40
hum_max = 80
```

Estas variables serán utilizadas inicialmente de forma local y posteriormente serán configuradas remotamente mediante MQTT.

## Evaluación de condiciones ambientales

```python
if temperatura > temp_max:
    estado = "TEMP ALTA"
```

```python
if humedad < hum_min:
    estado = "HUM BAJA"
```

```python
if humedad > hum_max:
    estado = "HUM ALTA"
```

## Visualización local

Ejemplo:

```text
Temp: 28.4
Hum : 65

NORMAL
```

o

```text
Temp: 34.1
Hum : 50

TEMP ALTA
```

## Preparación para MQTT

Las variables:

```python
temp_max
hum_min
hum_max
```

serán configuradas remotamente en los siguientes temas mediante tópicos MQTT.

## Relación con el Proyecto Integrador

El nodo desarrollado en este tema constituye la primera versión funcional de la Red Distribuida de Estaciones Ambientales IoT con MicroPython.

Durante los siguientes temas se incorporarán:

1. Conectividad WiFi.
2. MQTT.
3. Configuración remota.
4. Wildcards MQTT.
5. Supabase.
6. Dashboard web.

## Resultado final del Tema

Al concluir el Tema 2 el participante dispondrá de un nodo ambiental capaz de:

- Medir temperatura.
- Medir humedad.
- Visualizar información localmente.
- Generar alertas básicas.
- Servir como base para una arquitectura IoT distribuida.

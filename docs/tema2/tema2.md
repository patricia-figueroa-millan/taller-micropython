# Tema 2. Manejo de Entradas y Salidas Digitales

## 2.1 Introducción


Los sistemas embebidos modernos interactúan continuamente con el entorno mediante dispositivos de entrada y salida. Las entradas permiten adquirir información proveniente de sensores, mientras que las salidas permiten comunicar información al usuario o controlar otros dispositivos.

En plataformas como ESP32, MicroPython proporciona mecanismos sencillos para acceder a estos recursos de hardware, facilitando la integración de sensores, pantallas, actuadores y módulos de comunicación dentro de una misma aplicación.

En el contexto de Internet de las Cosas (IoT), las entradas y salidas digitales constituyen el punto de enlace entre el mundo físico y el mundo digital, permitiendo capturar variables del entorno, procesarlas y generar respuestas locales o remotas.

Como proyecto integrador del curso se desarrollará una Red Distribuida de Estaciones Ambientales IoT con MicroPython, conformada por múltiples nodos basados en ESP32 capaces de adquirir información ambiental, comunicarla mediante MQTT, almacenar los datos en la nube y permitir su monitoreo y configuración remota mediante una aplicación web.

A lo largo del curso, la solución será construida de manera incremental. En este tema se desarrollará el primer nodo de la red, implementando una estación ambiental local capaz de medir temperatura y humedad mediante un sensor DHT22 y visualizar la información en una pantalla OLED SSD1306.

Durante el desarrollo del tema, además de aprender a utilizar sensores y dispositivos de visualización, se introducirá una metodología práctica para explorar bibliotecas y objetos de MicroPython mediante el REPL, permitiendo que los participantes puedan descubrir y comprender nuevas funcionalidades de forma autónoma.


## 2.2 Programación de Periféricos en MicroPython

Este tema tine por objetivo comprender cómo MicroPython abstrae el hardware mediante bibliotecas y objetos para facilitar el desarrollo de aplicaciones embebidas.

### 2.2.1 Del hardware al software

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

### 2.2.2 Bibliotecas más utilizadas

- machine
- time / utime
- network
- dht
- ssd1306
- umqtt.simple
- os
- socket
- framebuf

### 2.2.3 Actividad: Exploración de un GPIO desde REPL

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

## 2.3 Exploración de Bibliotecas y Objetos desde el REPL

Este tema, busca que se aprenda a descubrir funcionalidades de bibliotecas y objetos mediante REPL.

### 2.3.1 Exploración de módulos

```python
import machine

dir(machine)
```

### 2.3.2 Exploración de clases

```python
from machine import Pin
dir(Pin)
```

```python
from machine import I2C
dir(I2C)
```

### 2.3.3 Exploración de objetos

```python
from machine import Pin

led = Pin(2, Pin.OUT)

type(led)
dir(led)
```

### 2.3.4 Recomendación para explorar bibliotecas

Antes de utilizar una biblioteca, se sugiere realizar lo siguiente desde el REPL:

1. Importar el módulo.
2. Ejecutar dir().
3. Crear un objeto.
4. Ejecutar type().
5. Ejecutar dir() sobre el objeto.
6. Identificar métodos.
7. Consultar documentación cuando sea necesario.



## 2.4 Instalación y Verificación de Bibliotecas Externas

### 2.4.1 Bibliotecas incluidas

MicroPython incorpora bibliotecas como:

```python
machine
time
utime
network
os
dht
```

### 2.4.2 Bibliotecas externas

Algunas bibliotecas deben copiarse manualmente al ESP32.

Ejemplo:

```text
ssd1306.py
```

### 2.4.3 Obtención de la biblioteca SSD1306

Repositorio oficial:

https://github.com/micropython/micropython-esp32/

Ruta:

```text
drivers/display/ssd1306.py
```

Enlace directo:

https://github.com/micropython/micropython-esp32/blob/esp32/drivers/display/ssd1306.py

### 2.4.4 Instalación desde Thonny

1. Abrir Thonny.
2. Crear un archivo nuevo.
3. Copiar el contenido de ssd1306.py.
4. Guardar como:
   
```text
ssd1306.py
```

5. Guardar en la memoria del ESP32.

## 2.5 Integración de Dispositivos mediante I2C: OLED SSD1306

### 2.5.1 ¿Qué es I2C?

I2C (Inter-Integrated Circuit) es uno de los protocolos de comunicación más utilizados en sistemas embebidos para conectar sensores, memorias, pantallas y otros dispositivos periféricos.

Su principal ventaja es que permite conectar múltiples dispositivos utilizando únicamente dos líneas de comunicación.

```text
SDA → Datos
SCL → Reloj
```

El ESP32 actúa como dispositivo maestro y coordina la comunicación con los dispositivos esclavos conectados al bus.


### 2.5.2 Arquitectura de comunicación I2C

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

### 2.5.3 Conexión de la OLED SSD1306

| OLED | ESP32 |
|--------|--------|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO21 |
| SCL | GPIO22 |

### 2.5.3 Conexión de la pantalla OLED

| OLED | ESP32 |
|------|--------|
| VCC | 3.3V |
| GND | GND |
| SDA | GPIO21 |
| SCL | GPIO22 |

### 2.5.4 Creación del bus I2C

```python
from machine import Pin, I2C

i2c = I2C(
    0,
    scl=Pin(22),
    sda=Pin(21),
    freq=100000
)
```

### 2.5.5 Uso de SoftI2C

En caso de que se muestre un error de =="I2C operation not supported"==, es importante conocer que este generalmente indica una limitación a nivel de hardware con el microcontrolador, una interfaz 12c deshabilitada.

Esta limitación puede ser resuelta cambiando a Software I2C en MicroPython, como se muestra a continuación:

```python
from machine import Pin, SoftI2C

i2c = SoftI2C(
    scl=Pin(22),
    sda=Pin(21),
    freq=400000)

```

### 2.5.5 Descubriendo dispositivos conectados al bus

```python
dispositivos = i2c.scan()
print(dispositivos)
```

Resultados esperados:

```python
[60]
```

o

```python
[61]
```
### 2.5.7 Direcciones I2C

| Decimal | Hexadecimal |
|----------|------------|
| 60 | 0x3C |
| 61 | 0x3D |


### 2.5.8 Exploración del objeto I2C

Una vez creado el bus I2C, es posible inspeccionarlo utilizando las herramientas de exploración de MicroPython vistas anteriormente.

```python
type(i2c)
```
Resultado esperado:

```python
<class 'SoftI2C'>
```
o
```python
<class 'I2C'>
```
dependiendo de la implementación utilizada.

Posteriormente podemos consultar los métodos disponibles:

```python
dir(i2c)
```

La salida puede variar ligeramente dependiendo de la versión de MicroPython, pero normalmente incluye métodos similares a los siguientes:


```text
scan
readfrom
writeto
readfrom_mem
writeto_mem
```
Estos métodos representan operaciones de comunicación de bajo nivel sobre el bus I2C.

### 2.5.9 Métodos principales del objeto I2C

#### scan()
Permite detectar los dispositivos conectados al bus.
```python
i2c.scan()
```
Retorna una lista con las direcciones de los dispositivos encontrados.

Ejemplo:
```python
[60]
```
o
```python
[60, 104]
```

En este caso existirían dos dispositivos conectados al mismo bus.

Este método constituye una de las herramientas más útiles para diagnóstico de conexiones en sistemas embebidos.

#### writeto()
Permite enviar datos directamente a un dispositivo I2C. 
Sintaxis general:

```python
i2c.writeto(direccion, datos)
```

Ejemplo:
```python
i2c.writeto(
    0x3C,
    b'\x00'
)
```
En aplicaciones reales este método suele ser utilizado internamente por bibliotecas como ssd1306.py, por lo que normalmente el programador no necesita utilizarlo directamente.

#### readfrom()
Permite leer datos desde un dispositivo I2C.

Sintaxis:
```python
i2c.readfrom(
    direccion,
    numero_bytes
)
```
Ejemplo:
```python
datos = i2c.readfrom(
    0x3C,
    4
)
```
Retorna un objeto de tipo bytes.

#### writeto_mem()

Muchos dispositivos I2C organizan su información en registros de memoria internos.

Este método permite escribir directamente sobre dichos registros.

Sintaxis:

```python
i2c.writeto_mem(
    direccion,
    registro,
    datos
)
```

Ejemplo conceptual:

```python
i2c.writeto_mem(
    0x68,
    0x00,
    b'\x01'
)
```

#### readfrom_mem()

Permite leer información almacenada en registros específicos del dispositivo.

Sintaxis:

```python
i2c.readfrom_mem(
    direccion,
    registro,
    numero_bytes
)
```

Ejemplo conceptual:

```python
valor = i2c.readfrom_mem(
    0x68,
    0x00,
    1
)
```

Este mecanismo es ampliamente utilizado por sensores, relojes de tiempo real (RTC), memorias EEPROM y otros dispositivos I2C.

#### start() y stop()

Algunas implementaciones de MicroPython exponen métodos para generar explícitamente las condiciones de inicio y finalización de una comunicación I2C.

```python
i2c.start()
```

```python
i2c.stop()
```

Sin embargo, en la mayoría de las aplicaciones estos métodos son gestionados automáticamente por las bibliotecas utilizadas.

#### ¿Por qué normalmente no utilizamos estos métodos directamente?

En este taller se trabajará principalmente con bibliotecas de alto nivel como:

```python
ssd1306
dht
umqtt.simple
```

Estas bibliotecas encapsulan internamente las operaciones de comunicación necesarias.

Por ejemplo:

```python
oled.text("Hola",0,0)
oled.show()
```

internamente ejecuta múltiples llamadas a métodos como:

```python
writeto()
```

y

```python
writeto_mem()
```

sin que el programador tenga que preocuparse por los detalles del protocolo.

No obstante, conocer estos métodos permite comprender mejor cómo funcionan las bibliotecas y facilita el desarrollo de aplicaciones más avanzadas cuando se trabaja con dispositivos para los cuales no existe una biblioteca disponible.


### 2.5.10  Inicialización segura de la pantalla OLED

```python
from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C

i2c = SoftI2C(
    scl=Pin(22),
    sda=Pin(21),
    freq=400000
)

oled = None

try:
    dispositivos = i2c.scan()
    if len(dispositivos) == 0:
        print("Error: no se encontró ningún dispositivo I2C" )
    elif 60 not in dispositivos and 61 not in dispositivos:
        print("No se encontró una pantalla OLED en el bus I2C")
    else:
        oled = SSD1306_I2C(128,64,i2c)
        oled.fill(0)

        oled.text("OLED OK",0,0)
        oled.text("I2C activo",0,15)
        oled.show()

except OSError as e:
    print("Error: fallo de comunicación con la OLED")
    print("Detalle del error:",e)
except Exception as e:
    print("Error inesperado al inicializar la OLED")
    print("Detalle del error:",e)
```


## 2.6 Integración de Sensores Digitales: DHT22


### 2.6.1 Importación de bibliotecas necesarias

```python
import dht
import utime
```

La biblioteca `dht` permite trabajar con sensores DHT11 y DHT22.

La biblioteca `utime` permite controlar intervalos de espera entre mediciones.

### 2.6.2 Conexión del sensor DHT22
| DHT22 | ESP32 |
|--------|--------|
| VCC | 3.3V |
| GND | GND |
| DATA | GPIO4 |

### 2.6.3 Configuración del sensor DHT22

```python
from machine import Pin

dht_pin = Pin(4)
sensor = dht.DHT22(dht_pin)
```

En este fragmento se configura el GPIO4 como pin de datos para el sensor DHT22.

El objeto `sensor` representa al dispositivo físico dentro del programa. A partir de este objeto se podrán ejecutar métodos para solicitar mediciones y recuperar los valores de temperatura y humedad.


### 2.6.4 Exploración del objeto sensor desde REPL

Antes de utilizar el sensor dentro de un programa completo, es recomendable explorarlo desde el REPL.

```python
type(sensor)
```

```python
dir(sensor)
```

Los métodos principales son:

```text
measure
temperature
humidity
```

Esta exploración permite conocer qué operaciones ofrece el objeto asociado al DHT22.


### 2.6.5 Obtención de datos del sensor
El proceso general para obtener datos del sensor DHT22, es el siguiente:

```python
sensor.measure()

temperatura = sensor.temperature()
humedad = sensor.humidity()
```

Sin embargo, se puede crear una función reutilizable para la obtención de datos del sensor, como se muestra a continuación:

```python
def obtener_temperatura():
    sensor.measure()
    temp = sensor.temperature()
    hum = sensor.humidity()
    return temp, hum
```

La función `obtener_temperatura()` encapsula el proceso de adquisición de datos.

Primero se ejecuta:

```python
sensor.measure()
```

Esta instrucción solicita una nueva medición al sensor.

Después se recuperan los valores medidos:

```python
temp = sensor.temperature()
hum = sensor.humidity()
```

Finalmente, ambos valores se devuelven mediante:

```python
return temp, hum
```

Esto permite que la función entregue dos datos al programa principal: temperatura y humedad.


### 2.6.6 Visualización de datos y estado ambiental
De igual manera, el proceso general para vusializar los datos del sensor (estado ambiental) a través de la salida estándar (terminal) es el siguiente:

```python
sensor.measure()

print(sensor.temperature())
print(sensor.humidity())
```

No obstante se puede crear una función específica para la visualización de los datos en la pantalla OLED conectada al I2C, como se muestra a continuación:


```python
def mostrar_emoji(temp, hum):
    oled.fill(0)
    oled.text("Temp: {:.1f}C".format(temp), 0, 0)
    oled.text("Hum: {:.1f}%".format(hum), 0, 10)

    if temp < 15:
        oled.text("FRIO [*]", 30, 30)
    elif temp >= 15 and temp <= 30:
        oled.text("NORMAL :)", 30, 30)
    else:
        oled.text("CALOR [!]", 30, 30)

    oled.show()
```

La función `mostrar_emoji()` recibe como parámetros la temperatura y la humedad.

Su propósito es actualizar la pantalla OLED con:

- Temperatura.
- Humedad.
- Estado ambiental según la temperatura.

La instrucción:

```python
oled.fill(0)
```

limpia la pantalla antes de escribir nuevos valores.

La instrucción:

```python
oled.show()
```

actualiza físicamente la pantalla.


Para evaluar del estado ambiental se puede  utilizar una estructura condicional para clasificar la temperatura:

```python
if temp < 15:
    oled.text("FRIO [*]", 30, 30)
elif temp >= 15 and temp <= 30:
    oled.text("NORMAL :)", 30, 30)
else:
    oled.text("CALOR [!]", 30, 30)
```

Esta lógica permite que el sistema no solo muestre valores, sino que también interprete la información obtenida del sensor.



### 2.6.7 Bucle principal

```python
while True:
    try:
        temperatura, humedad = obtener_temperatura()
        mostrar_emoji(temperatura, humedad)
        utime.sleep(2)
    except OSError as e:
        oled.fill(0)
        oled.text("Error Sensor!", 20, 20)
        oled.show()
        utime.sleep(2)
```

El bucle principal mantiene el sistema en ejecución continua. En cada ciclo:

1. Obtiene temperatura y humedad.
2. Actualiza la pantalla OLED.
3. Espera 2 segundos.
4. Repite el proceso.

### 2.6.8 Manejo de errores del sensor

La lectura del DHT22 puede fallar ocasionalmente por problemas de comunicación, conexión o temporización.

Por ello se utiliza:

```python
try:
```

para intentar ejecutar la lectura normalmente.

Y:

```python
except OSError as e:
```

para capturar errores relacionados con el sensor.

En caso de error, la pantalla muestra:

```text
Error Sensor!
```

Esto evita que el programa se detenga completamente ante una lectura fallida.

### 2.6.9 Código integrado del DHT22

```python
import dht
import utime
from machine import Pin

# Configuración del sensor DHT22
dht_pin = Pin(4)
sensor = dht.DHT22(dht_pin)

# Función para obtener datos del sensor DHT22
def obtener_temperatura():
    sensor.measure()
    temp = sensor.temperature()
    hum = sensor.humidity()
    return temp, hum

# Función para mostrar la temperatura, humedad y el estado
def mostrar_emoji(temp, hum):
    oled.fill(0)
    oled.text("Temp: {:.1f}C".format(temp), 0, 0)
    oled.text("Hum: {:.1f}%".format(hum), 0, 10)

    if temp < 15:
        oled.text("FRIO [*]", 30, 30)
    elif temp >= 15 and temp <= 30:
        oled.text("NORMAL :)", 30, 30)
    else:
        oled.text("CALOR [!]", 30, 30)

    oled.show()

# Bucle principal
while True:
    try:
        temperatura, humedad = obtener_temperatura()
        mostrar_emoji(temperatura, humedad)
        utime.sleep(2)
    except OSError as e:
        oled.fill(0)
        oled.text("Error Sensor!", 20, 20)
        oled.show()
        utime.sleep(2)
```

# 2.7 Estación Ambiental Integrada: Primer Incremento del Proyecto

## 2.7.1 Objetivo

Integrar la comunicación I2C, la pantalla OLED SSD1306 y el sensor DHT22 para construir el primer nodo funcional del proyecto integrador **Red Distribuida de Estaciones Ambientales IoT con MicroPython**.

Además de medir y visualizar datos localmente, este incremento dejará preparada la estructura básica para que, en los siguientes temas, la estación pueda publicar información mediante MQTT y recibir parámetros de configuración remota.


## 2.7.2 Integración de componentes

Hasta este momento se han trabajado por separado los siguientes elementos:

- Comunicación I2C mediante `SoftI2C`.
- Inicialización segura de una pantalla OLED SSD1306.
- Lectura de temperatura y humedad mediante un sensor DHT22.
- Exploración de objetos y bibliotecas mediante REPL.
- Manejo básico de excepciones.

En esta etapa (Practica 2) dichos componentes se integran en una única aplicación capaz de:

1. Medir temperatura.
2. Medir humedad.
3. Evaluar el estado ambiental.
4. Visualizar la información localmente.
5. Preparar datos estructurados para futura publicación MQTT.
6. Utilizar parámetros locales que posteriormente serán configurados de forma remota.


## 2.7.3 Arquitectura del primer incremento

```text
        DHT22
          │
          ▼
       ESP32
          │
          ▼
    OLED SSD1306
```

Este sistema constituye el primer nodo funcional de la red distribuida. A continuación se indican las modificaciones necesarias para dejar el primer incremento del proyecto listo.

## Paso 1. Agregar identificación de la estación

Cada nodo de la red debe tener un identificador único.
Esto permitirá distinguir los datos publicados por cada estación cuando se integren varias tarjetas ESP32 mediante MQTT.

```python
station_id = "estacion_01"
```
Más adelante este identificador podrá utilizarse para construir tópicos MQTT como:

```bash
estaciones/estacion_01/temperatura
estaciones/estacion_01/humedad
estaciones/estacion_01/estado
```

La información relativa a MQTT, funcionamiento y arquitectura se presentará en el Tema 3.

## Paso 2. Agregar parámetros locales configurables

Se agregan variables que por ahora serán locales, pero que posteriormente podrán modificarse mediante mensajes MQTT.

```python
temp_max = 30
hum_min = 40
hum_max = 80
intervalo_lectura = 2
```

Estas variables permiten definir:

* Temperatura máxima permitida.
* Humedad mínima permitida.
* Humedad máxima permitida.
* Tiempo de espera entre mediciones.

En este primer incremento, los valores se escriben directamente en el código. En incrementos posteriores, estos valores podrán cambiarse de forma remota.

## Paso 3. Mantener la inicialización segura de la OLED

Se conserva la lógica de detección de dispositivos I2C antes de crear el objeto OLED.

```python
i2c = SoftI2C(
    scl=Pin(22),
    sda=Pin(21),
    freq=400000
)

oled = None

try:
    dispositivos = i2c.scan()
    if len(dispositivos) == 0:
        print("Error: no se encontró ningún dispositivo I2C")
    elif 60 not in dispositivos and 61 not in dispositivos:
        print("No se encontró una pantalla OLED en dispositivos I2C")
    else:
        oled = SSD1306_I2C(128, 64, i2c)
        oled.fill(0)
        oled.text("OLED OK", 0, 0)
        oled.text("I2C activo", 0, 15)
        oled.show()
        utime.sleep(2)
except OSError as e:
    print("Error: fallo de comunicación con la OLED")
    print("Detalle del error:", e)
except Exception as e:
    print("Error inesperado al inicializar la OLED")
    print("Detalle del error:", e)
```

## Paso 4: Verificar que la OLED fue inicializada

Posteriormente verificar que la pantalla OLED exista, agregando las lineas siguientes despues del código anterior del Paso 3 que ya se tiene.

```python
if oled is None:
    raise Exception("No se puede continuar sin pantalla OLED.")
```
Esto evita que el programa intente ejecutar:
```python
oled.fill(0)
```
si la pantalla no fue inicializada correctamente.

## Paso 5. Crear una función para evaluar el estado ambiental
En lugar de colocar toda la lógica dentro de la función de visualización, se crea una función independiente. Esta función puede agregarse despues de la función obtener_temperatura().

```python
def evaluar_estado(temp, hum):
    if temp > temp_max:
        return "TEMP ALTA"
    elif hum < hum_min:
        return "HUM BAJA"
    elif hum > hum_max:
        return "HUM ALTA"
    else:
        return "NORMAL"
```

Esta separación es importante porque la evaluación del estado podrá reutilizarse después para:

* Mostrar información en pantalla.
* Publicar el estado mediante MQTT.
* Almacenar el estado en Supabase.
* Mostrar alertas en un dashboard.

## Paso 6. Preparar los datos para futura publicación MQTT

Aunque en este incremento todavía no se usa MQTT, se puede preparar una estructura de datos que será útil posteriormente.

```python
def preparar_datos(temp, hum, estado):
    datos = {
        "station_id": station_id,
        "temperatura": temp,
        "humedad": hum,
        "estado": estado,
        "temp_max": temp_max,
        "hum_min": hum_min,
        "hum_max": hum_max
    }

    return datos
```

Esta estructura permitirá visualizar desde ahora qué información podría publicarse más adelante.

## Paso 7. Crear una función para mostrar los datos

```python
def mostrar_datos(temp, hum, estado):
    oled.fill(0)
    oled.text(station_id, 0, 0)
    oled.text("T:{:.1f}C".format(temp), 0, 12)
    oled.text("H:{:.1f}%".format(hum), 0, 24)
    oled.text(estado, 0, 40)
    oled.show()
```

## Paso 8. Integrar el bucle principal
```python
while True:
    try:
        temperatura, humedad = obtener_temperatura()

        estado = evaluar_estado(
            temperatura,
            humedad
        )

        datos = preparar_datos(
            temperatura,
            humedad,
            estado
        )

        mostrar_datos(
            temperatura,
            humedad,
            estado
        )

        print(datos)

        utime.sleep(intervalo_lectura)

    except OSError as e:
        oled.fill(0)
        oled.text("Error Sensor!", 20, 20)
        oled.show()
        print("Error al leer DHT22:", e)
        utime.sleep(intervalo_lectura)
```

En este ciclo:

1. Se leen los datos del DHT22.
2. Se evalúa el estado ambiental.
3. Se preparan los datos en una estructura similar a la que se usará con MQTT.
4. Se actualiza la OLED.
5. Se imprimen los datos en consola.
6. Se espera el intervalo de lectura configurado.

## Código integrado de la Práctica 2 modificado

```python
from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
import dht
import utime

# Identificación de la estación
station_id = "estacion_01"

# Parámetros locales configurables
temp_max = 30
hum_min = 40
hum_max = 80
intervalo_lectura = 2

# Configuración del sensor DHT22
dht_pin = Pin(4)
sensor = dht.DHT22(dht_pin)

# Configuración del bus I2C
i2c = SoftI2C(
    scl=Pin(22),
    sda=Pin(21),
    freq=400000
)

oled = None

try:
    dispositivos = i2c.scan()

    if len(dispositivos) == 0:
        print("Error: no se encontró ningún dispositivo I2C")

    elif 60 not in dispositivos and 61 not in dispositivos:
        print("No se encontró una pantalla OLED en dispositivos I2C")

    else:
        oled = SSD1306_I2C(128, 64, i2c)
        oled.fill(0)
        oled.text("OLED OK", 0, 0)
        oled.text("I2C activo", 0, 15)
        oled.show()
        utime.sleep(2)

except OSError as e:
    print("Error: fallo de comunicación con la OLED")
    print("Detalle del error:", e)

except Exception as e:
    print("Error inesperado al inicializar la OLED")
    print("Detalle del error:", e)

if oled is None:
    raise Exception("No se puede continuar sin pantalla OLED.")

def obtener_temperatura():
    sensor.measure()
    temp = sensor.temperature()
    hum = sensor.humidity()
    return temp, hum

def evaluar_estado(temp, hum):
    if temp > temp_max:
        return "TEMP ALTA"
    elif hum < hum_min:
        return "HUM BAJA"
    elif hum > hum_max:
        return "HUM ALTA"
    else:
        return "NORMAL"

def preparar_datos(temp, hum, estado):
    datos = {
        "station_id": station_id,
        "temperatura": temp,
        "humedad": hum,
        "estado": estado,
        "temp_max": temp_max,
        "hum_min": hum_min,
        "hum_max": hum_max
    }

    return datos

def mostrar_datos(temp, hum, estado):
    oled.fill(0)
    oled.text(station_id, 0, 0)
    oled.text("T:{:.1f}C".format(temp), 0, 12)
    oled.text("H:{:.1f}%".format(hum), 0, 24)
    oled.text(estado, 0, 40)
    oled.show()

while True:
    try:
        temperatura, humedad = obtener_temperatura()

        estado = evaluar_estado(
            temperatura,
            humedad
        )

        datos = preparar_datos(
            temperatura,
            humedad,
            estado
        )

        mostrar_datos(
            temperatura,
            humedad,
            estado
        )

        print(datos)

        utime.sleep(intervalo_lectura)

    except OSError as e:
        oled.fill(0)
        oled.text("Error Sensor!", 20, 20)
        oled.show()
        print("Error al leer DHT22:", e)
        utime.sleep(intervalo_lectura)
```

## Resultado obtenido

Al ejecutar la aplicación, la estación ambiental es capaz de:

* Adquirir temperatura y humedad del entorno.
* Evaluar el estado ambiental con base en umbrales locales.
* Mostrar la información en una pantalla OLED.
* Preparar una estructura de datos lista para futura publicación MQTT.
* Identificarse como una estación específica mediante station_id.
* Operar de forma autónoma como nodo ambiental local.

## Relación con el Proyecto Integrador

La estación ambiental desarrollada en este tema corresponde al primer incremento funcional del proyecto:

Día 2: Nodo Ambiental Local con parámetros locales ✓

En los siguientes temas se incorporarán nuevas capacidades:

```bash
Día 3
WiFi + MQTT
Publicación de datos
        ↓

Día 4
Configuración remota
de umbrales
        ↓

Día 5
Supabase +
Dashboard Web
        ↓

Red Distribuida de
Estaciones Ambientales IoT
```

## Reflexión

Aunque actualmente el sistema funciona de manera local, el código ya está organizado para evolucionar hacia una arquitectura IoT distribuida.

La incorporación de:

* station_id
* temp_max
* hum_min
* hum_max
* intervalo_lectura
* preparar_datos()

permite que el nodo esté listo para integrarse posteriormente con MQTT.

En el siguiente incremento, los valores generados por preparar_datos() podrán publicarse hacia un broker MQTT, y los parámetros locales podrán actualizarse de forma remota mediante mensajes de configuración.
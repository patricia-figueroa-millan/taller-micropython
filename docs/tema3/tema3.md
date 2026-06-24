# Tema 3. Comunicación IoT mediante WiFi y MQTT.

## 3.1 Introducción

En el tema anterior se desarrolló una estación ambiental local basada en ESP32, un sensor DHT22 y una pantalla OLED SSD1306. Esta estación fue capaz de adquirir información del entorno, evaluar el estado ambiental mediante parámetros configurables y visualizar los resultados localmente.

Aunque este tipo de solución resulta útil para monitoreo local, una de las principales características de los sistemas IoT consiste en la capacidad de intercambiar información mediante redes de comunicación, permitiendo que los datos puedan consultarse, almacenarse y procesarse desde ubicaciones remotas.

Para lograr esta conectividad, los dispositivos IoT suelen utilizar protocolos de comunicación ligeros diseñados para operar sobre redes inalámbricas y equipos con recursos limitados. Uno de los protocolos más utilizados en la actualidad es MQTT (Message Queuing Telemetry Transport), ampliamente adoptado en aplicaciones industriales, domótica, monitoreo ambiental, agricultura inteligente y sistemas ciberfísicos.

En este tema se incorporarán capacidades de comunicación inalámbrica al nodo ambiental desarrollado previamente. Inicialmente se establecerá la conexión del ESP32 a una red WiFi utilizando las bibliotecas de red incluidas en MicroPython. Posteriormente se implementará la comunicación mediante MQTT para publicar información ambiental hacia un broker centralizado.

La integración de WiFi y MQTT permitirá que la estación ambiental evolucione de un sistema aislado a un nodo IoT conectado, capaz de compartir información con otros dispositivos y aplicaciones de monitoreo en tiempo real.

Como parte del proyecto integrador Red Distribuida de Estaciones Ambientales IoT con MicroPython, este tema corresponde al segundo incremento funcional de la solución. Cada estación comenzará a transmitir sus mediciones hacia una infraestructura de comunicación común, sentando las bases para la construcción de una red distribuida de monitoreo ambiental.

La arquitectura general que se implementará durante este tema se muestra conceptualmente a continuación:

```text
      DHT22
         │
         ▼
       ESP32
         │
         ▼
       WiFi
         │
         ▼
    Broker MQTT
         │
         ▼
   Clientes MQTT
```

Durante el desarrollo del tema se abordarán los conceptos fundamentales de conectividad WiFi, comunicación MQTT, estructura de tópicos, publicación de mensajes y monitoreo de datos mediante herramientas cliente. Asimismo, se continuará utilizando el REPL como mecanismo de exploración de bibliotecas y objetos, reforzando la capacidad de aprendizaje autónomo sobre nuevas tecnologías y componentes de MicroPython.

Al finalizar el tema, la estación ambiental será capaz de medir temperatura y humedad, visualizar la información localmente y publicar los datos hacia un broker MQTT para su monitoreo remoto en tiempo real.

## 3.2 Conectividad WiFi en MicroPython

### 3.2.1 ¿Qué es WiFi?

WiFi es una tecnología de comunicación inalámbrica basada en los estándares IEEE 802.11 que permite intercambiar información entre dispositivos a través de redes locales sin necesidad de conexiones físicas.

En aplicaciones IoT, WiFi constituye uno de los mecanismos de comunicación más utilizados debido a su amplia disponibilidad, facilidad de implementación y capacidad para conectarse directamente a Internet.

El ESP32 incorpora de forma nativa un módulo WiFi que permite establecer comunicación con redes inalámbricas utilizando las bibliotecas incluidas en MicroPython.

Mediante WiFi, un dispositivo IoT puede:

- Enviar datos hacia servicios en la nube.
- Publicar información mediante MQTT.
- Recibir configuraciones remotas.
- Intercambiar información con otros dispositivos.
- Integrarse con aplicaciones web y móviles.

En este tema se utilizará la conectividad WiFi para permitir que la estación ambiental pueda transmitir información mediante MQTT.


### 3.2.2 Modos de operación WiFi del ESP32

El ESP32 puede operar en distintos modos de red.

Los más utilizados son:

#### Modo Station (STA)

En este modo el ESP32 se conecta a una red WiFi existente como cualquier otro dispositivo.

Por ejemplo:

```text
Router WiFi
     │
 ┌───┴────┐
 │ ESP32  │
 └────────┘
```

En MicroPython este modo se representa mediante:

```python
network.STA_IF
```

Este será el modo utilizado durante el taller.

#### Modo Access Point (AP)

En este modo el ESP32 crea su propia red WiFi.

Otros dispositivos pueden conectarse directamente a él.

```text
        ESP32
          │
     Red WiFi propia
      /          \
 Smartphone    Laptop
```

En MicroPython se representa mediante:

```python
network.AP_IF
```

Este modo suele utilizarse para configuración inicial de dispositivos IoT o aplicaciones donde no existe infraestructura de red disponible.


### 3.2.3 Biblioteca network

MicroPython proporciona la biblioteca `network` para administrar las interfaces de red disponibles.

Importación:

```python
import network
```

Una vez importada, es posible crear objetos que representan interfaces WiFi.

Ejemplo:

```python
wifi = network.WLAN(network.STA_IF)
```

El objeto `wifi` representa la interfaz inalámbrica del ESP32.


### 3.2.4 Exploración de la biblioteca desde REPL

Siguiendo la metodología presentada en el Tema 2, antes de utilizar una nueva biblioteca es recomendable explorar sus componentes mediante el REPL.

Esta práctica permite identificar las clases, constantes y funciones disponibles sin necesidad de consultar inmediatamente la documentación externa.

Para explorar el módulo `network`, se puede ejecutar:

```python
import network

dir(network)
```

La salida puede variar ligeramente dependiendo de la versión de MicroPython instalada, pero normalmente incluye elementos similares a los siguientes:

```text
AP_IF
STA_IF
WLAN
AUTH_OPEN
AUTH_WPA_PSK
AUTH_WPA2_PSK
```

Entre los elementos más importantes destacan:

| Elemento | Descripción |
|-----------|-------------|
| STA_IF | Interfaz WiFi en modo estación (Station Mode). Permite que el ESP32 se conecte a una red WiFi existente. |
| AP_IF | Interfaz WiFi en modo punto de acceso (Access Point). Permite que el ESP32 cree su propia red WiFi. |
| WLAN | Clase principal utilizada para crear y administrar interfaces inalámbricas. |
| AUTH_OPEN | Red inalámbrica sin mecanismo de autenticación ni contraseña. |
| AUTH_WPA_PSK | Red protegida mediante WPA utilizando una clave precompartida (Pre-Shared Key). |
| AUTH_WPA2_PSK | Red protegida mediante WPA2 utilizando una clave precompartida. Es uno de los mecanismos de seguridad más utilizados en redes domésticas y empresariales. |


Por ejemplo, al crear una interfaz WiFi:

```python
wifi = network.WLAN(network.STA_IF)
```

se está utilizando:

- `STA_IF` para indicar que el ESP32 actuará como cliente de una red WiFi existente.
- `WLAN` para crear el objeto que administrará la conexión.

Posteriormente también es posible explorar la clase `WLAN`:

```python
dir(network.WLAN)
```

La salida mostrará los métodos disponibles para administrar la interfaz inalámbrica.

Dependiendo de la versión instalada, aparecerán métodos similares a:

```text
active
connect
disconnect
ifconfig
isconnected
scan
status
config
```

Algunos de los métodos más utilizados son:

| Método | Descripción |
|----------|-------------|
| active() | Activa o desactiva la interfaz WiFi. |
| connect() | Inicia la conexión a una red inalámbrica. |
| disconnect() | Finaliza la conexión actual. |
| isconnected() | Verifica si existe conexión activa. |
| ifconfig() | Obtiene la configuración IP actual. |
| scan() | Busca redes WiFi cercanas. |
| status() | Obtiene información sobre el estado de la conexión. |

La exploración mediante `dir()` permite identificar rápidamente las capacidades de una biblioteca y constituye una estrategia muy útil cuando se trabaja con nuevos módulos, sensores o protocolos de comunicación en MicroPython.

Por otro lado, las constantes relacionadas con autenticación (`AUTH_*`) permiten identificar el tipo de seguridad configurado en una red inalámbrica.

Por ejemplo, cuando se realiza un escaneo de redes mediante:

```python
redes = wifi.scan()
```

MicroPython devuelve información que incluye el mecanismo de autenticación utilizado por cada red detectada. Estas constantes facilitan interpretar dichos valores y conocer el nivel de seguridad disponible en la red.

### 3.2.5 Creación de la interfaz WiFi

Para utilizar la red inalámbrica primero se crea una interfaz de tipo Station.

```python
import network

wifi = network.WLAN(network.STA_IF)
```

Posteriormente se activa:

```python
wifi.active(True)
```

Verificación:

```python
wifi.active()
```

Resultado esperado:

```python
True
```

### 3.2.6 Conexión a una red WiFi

Una vez activada la interfaz se puede iniciar la conexión.

```python
wifi.connect(
    "MiRed",
    "MiPassword"
)
```

Donde:

- `"MiRed"` corresponde al nombre de la red WiFi.
- `"MiPassword"` corresponde a la contraseña.

Ejemplo:

```python
wifi.connect(
    "LaboratorioIoT",
    "12345678"
)
```

La conexión ocurre de forma asíncrona, por lo que normalmente es necesario esperar algunos segundos.


### 3.2.7 Verificación del estado de conexión

La función:

```python
wifi.isconnected()
```

permite verificar si la conexión fue establecida correctamente.

Ejemplo:

```python
wifi.isconnected()
```

Resultado esperado:

```python
True
```

o

```python
False
```

dependiendo del estado de la conexión.


### 3.2.8 Obtención de parámetros de red

Una vez conectado, es posible consultar la configuración IP obtenida.

```python
wifi.ifconfig()
```

Ejemplo de salida:

```python
(
 '192.168.1.15',
 '255.255.255.0',
 '192.168.1.1',
 '8.8.8.8'
)
```

Donde:

| Parámetro | Descripción |
|-----------|-------------|
| IP | Dirección IP asignada |
| Máscara | Máscara de red |
| Gateway | Puerta de enlace |
| DNS | Servidor DNS |

Esta información resulta útil para diagnóstico de problemas de conectividad.


### 3.2.9 Exploración del objeto WiFi

Una vez creada la interfaz, puede inspeccionarse mediante las herramientas ya conocidas.

```python
type(wifi)
```

Resultado esperado:

```python
<class 'WLAN'>
```

Posteriormente:

```python
dir(wifi)
```

Entre los métodos más comunes se encuentran:

```text
active
connect
disconnect
ifconfig
isconnected
scan
status
```

Algunos de éstos ya se mencionaron, para mayor información se puede consultar la información oficial de MicroPython.

### 3.2.10 Escaneo de redes disponibles

El ESP32 puede buscar redes cercanas utilizando:

```python
wifi.scan()
```

Ejemplo:

```python
redes = wifi.scan()

for red in redes:
    print(red)
```

Cada elemento contiene información sobre:

- Nombre de la red (SSID).
- Dirección MAC.
- Canal.
- Intensidad de señal.
- Tipo de seguridad.

Esta función resulta útil para verificar cobertura y disponibilidad de redes.

La salida de wifi.scan(), el quinto campo corresponde al tipo de autenticación (authmode).

Por ejemplo:
```bash
(b'MiRed', b'\xf8\xb1V...', 6, -52, 3, 0)
```
La estructura general es:

```bash
(ssid, bssid, canal, rssi, authmode, hidden)
```

| Campo | Descripción |
|--------|-------------|
| `ssid` | Nombre de la red WiFi. |
| `bssid` | Dirección MAC del punto de acceso. |
| `canal` | Canal utilizado por la red. |
| `rssi` | Intensidad de señal recibida (dBm). |
| `authmode` | Tipo de autenticación o mecanismo de seguridad de la red. |
| `hidden` | Indica si la red se encuentra oculta (`1`) o visible (`0`). |

Los valores comunes de authmode:

| Valor | Constante | Descripción |
|--------|------------|-------------|
| `0` | `AUTH_OPEN` | Red sin contraseña. |
| `1` | `AUTH_WEP` | Red protegida mediante WEP. |
| `2` | `AUTH_WPA_PSK` | Red protegida mediante WPA con clave precompartida. |
| `3` | `AUTH_WPA2_PSK` | Red protegida mediante WPA2 con clave precompartida. |
| `4` | `AUTH_WPA_WPA2_PSK` | Red compatible con WPA y WPA2. |


### 3.2.11 Función reutilizable para conexión WiFi

En aplicaciones reales suele utilizarse una función específica para administrar la conexión.

```python
import network
import utime

def conectar_wifi(ssid, password):
    wifi = network.WLAN(network.STA_IF)
    wifi.active(True)

    if not wifi.isconnected():
        print("Conectando a WiFi...")
        wifi.connect(
            ssid,
            password
        )
        while not wifi.isconnected():
            utime.sleep(1)
            print(".", end="")
    print()
    print("WiFi conectado")
    print("IP:", wifi.ifconfig()[0])

    return wifi
```

Uso:

```python
wifi = conectar_wifi(
    "MiRed",
    "MiPassword"
)
```


### 3.2.12 Manejo básico de errores

Una conexión WiFi puede fallar por diversas razones:

- Contraseña incorrecta.
- Red fuera de alcance.
- Router apagado.
- Interferencias.
- Problemas de infraestructura.

Por esta razón es recomendable implementar un tiempo máximo de espera.

Ejemplo conceptual:

```python
timeout = 15

while not wifi.isconnected() and timeout > 0:
    utime.sleep(1)
    timeout -= 1
```

Si el contador llega a cero:

```python
raise Exception(
    "No fue posible conectar a WiFi"
)
```

Esto evita que el programa quede esperando indefinidamente.


### 3.2.13 Código completo de prueba

```python
import network
import utime

SSID = "MiRed"
PASSWORD = "MiPassword"

wifi = network.WLAN(network.STA_IF)
wifi.active(True)

if not wifi.isconnected():

    print("Conectando a WiFi...")

    wifi.connect(
        SSID,
        PASSWORD
    )

    while not wifi.isconnected():
        utime.sleep(1)
        print(".", end="")

print()
print("Conexión exitosa")

print("IP obtenida:")
print(wifi.ifconfig()[0])
```

### 3.2.14 Integración de WiFi en la estación ambiental

Una vez comprendido el proceso básico de conexión WiFi, el siguiente paso consiste en integrar esta funcionalidad al código de la estación ambiental desarrollada en el Tema 2.

El objetivo de esta integración es que el nodo ambiental pueda conectarse a una red inalámbrica antes de comenzar con la lectura del sensor DHT22 y la visualización de datos en la pantalla OLED.

En este incremento todavía no se publicarán datos mediante MQTT. Primero se agregará la conectividad WiFi para dejar preparada la estación antes de incorporar el broker MQTT.

#### Paso 1. Importar la biblioteca network

Al inicio del programa se agrega la biblioteca `network`:

```python
import network
```

Por lo tanto, las importaciones iniciales quedan de la siguiente manera:

```python
from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
import dht
import utime
import network
```

#### Paso 2. Agregar las credenciales de la red WiFi

Después de la identificación de la estación se agregan las variables correspondientes al nombre de la red y la contraseña.

```python
# Credenciales WiFi
SSID = "MiRed"
PASSWORD = "MiPassword"
```

Estas variables deben modificarse de acuerdo con la red WiFi disponible durante la práctica.

#### Paso 3. Crear una función para conectar a WiFi

Se agrega una función reutilizable llamada `conectar_wifi()`.

```python
def conectar_wifi(ssid, password):
    wifi = network.WLAN(network.STA_IF)
    wifi.active(True)

    if not wifi.isconnected():
        print("Conectando a WiFi...")
        wifi.connect(ssid, password)

        timeout = 15

        while not wifi.isconnected() and timeout > 0:
            utime.sleep(1)
            timeout -= 1
            print(".", end="")

    print()

    if wifi.isconnected():
        print("WiFi conectado")
        print("IP:", wifi.ifconfig()[0])
        return wifi
    else:
        raise Exception("No fue posible conectar a WiFi")
```

Esta función realiza las siguientes acciones:

1. Crea la interfaz WiFi en modo estación.
2. Activa la interfaz inalámbrica.
3. Intenta conectarse a la red indicada.
4. Espera un máximo de 15 segundos.
5. Muestra la dirección IP obtenida.
6. Detiene el programa si no se logra la conexión.

#### Paso 4. Conectar la estación antes del bucle principal

Después de validar que la OLED fue inicializada correctamente, se llama a la función `conectar_wifi()`.

```python
wifi = conectar_wifi(
    SSID,
    PASSWORD
)
```

Esta instrucción debe ejecutarse antes del ciclo `while True`, ya que la estación debe estar conectada a la red antes de iniciar su operación continua.

#### Paso 5. Mostrar el estado de conexión en la OLED

Para confirmar visualmente que el nodo se conectó a WiFi, se puede mostrar un mensaje en la pantalla OLED.

```python
oled.fill(0)
oled.text("WiFi OK", 0, 0)
oled.text(wifi.ifconfig()[0], 0, 15)
oled.show()
utime.sleep(2)
```

Esto permite verificar desde la estación que la conexión fue exitosa y que el ESP32 obtuvo una dirección IP.

##### Código integrado con WiFi

```python
from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
import dht
import utime
import network

# Identificación de la estación
station_id = "estacion_01"

# Credenciales WiFi
SSID = "MiRed"
PASSWORD = "MiPassword"

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

def conectar_wifi(ssid, password):
    wifi = network.WLAN(network.STA_IF)
    wifi.active(True)

    if not wifi.isconnected():
        print("Conectando a WiFi...")
        wifi.connect(ssid, password)

        timeout = 15

        while not wifi.isconnected() and timeout > 0:
            utime.sleep(1)
            timeout -= 1
            print(".", end="")

    print()

    if wifi.isconnected():
        print("WiFi conectado")
        print("IP:", wifi.ifconfig()[0])
        return wifi
    else:
        raise Exception("No fue posible conectar a WiFi")

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

wifi = conectar_wifi(
    SSID,
    PASSWORD
)

oled.fill(0)
oled.text("WiFi OK", 0, 0)
oled.text(wifi.ifconfig()[0], 0, 15)
oled.show()
utime.sleep(2)

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

#### Resultado esperado

Al ejecutar este incremento, la estación ambiental deberá:

- Inicializar la pantalla OLED.
- Conectarse a una red WiFi.
- Mostrar la dirección IP obtenida.
- Leer temperatura y humedad.
- Evaluar el estado ambiental.
- Mostrar los datos en la OLED.
- Imprimir en consola la estructura de datos preparada.

Con esta modificación, la estación ambiental ya no funciona únicamente como un nodo local, sino como un dispositivo conectado a una red inalámbrica. Esta conectividad será la base para publicar los datos mediante MQTT en la siguiente sección.


## 3.3 Fundamentos de MQTT y uso de EMQX

### 3.3.1 ¿Qué es MQTT?

MQTT, por sus siglas en inglés *Message Queuing Telemetry Transport*, es un protocolo de comunicación ligero diseñado para el intercambio de mensajes entre dispositivos conectados en red.

Este protocolo es ampliamente utilizado en aplicaciones de Internet de las Cosas debido a que permite enviar y recibir información con bajo consumo de recursos, lo cual resulta adecuado para microcontroladores, sensores, actuadores y sistemas embebidos.

A diferencia de otros esquemas de comunicación donde un dispositivo se comunica directamente con otro, MQTT utiliza una arquitectura basada en publicación y suscripción.

Esto significa que los dispositivos no necesitan conocerse directamente entre sí. En su lugar, todos se comunican a través de un servidor intermediario llamado broker MQTT.

En el contexto del proyecto integrador, MQTT permitirá que la estación ambiental basada en ESP32 publique los datos de temperatura, humedad y estado ambiental hacia un broker, para que posteriormente puedan ser consultados desde otras aplicaciones o dispositivos.


### 3.3.2 Arquitectura Publisher / Broker / Subscriber

MQTT funciona mediante tres elementos principales:

| Elemento | Descripción |
|----------|-------------|
| Publisher | Dispositivo o aplicación que publica mensajes. |
| Broker | Servidor encargado de recibir, administrar y distribuir los mensajes. |
| Subscriber | Dispositivo o aplicación que se suscribe a uno o más tópicos para recibir mensajes. |

La arquitectura general puede representarse de la siguiente manera:

```text
        Publisher
           │
           ▼
      Broker MQTT
           │
           ▼
       Subscriber
```

En esta arquitectura:

- El publicador envía información hacia un tópico.
- El broker recibe el mensaje.
- El suscriptor recibe el mensaje si está suscrito al tópico correspondiente.

Por ejemplo, en el proyecto:

```text
ESP32 ─────────────► Broker MQTT ─────────────► Cliente MQTT
Publicador              EMQX                     Suscriptor
```

La estación ambiental actuará inicialmente como publicador, ya que enviará los valores obtenidos por el sensor DHT22.


### 3.3.3 ¿Qué es un broker MQTT?

Un broker MQTT es el componente central de la comunicación MQTT.

Su función principal es recibir los mensajes enviados por los publicadores y reenviarlos a los clientes que se encuentran suscritos a los tópicos correspondientes.

El broker permite desacoplar la comunicación entre dispositivos. Esto significa que el publicador no necesita saber quién recibirá el mensaje, y el suscriptor no necesita saber qué dispositivo lo generó.

Por ejemplo:

```text
estaciones/estacion_01/temperatura
```

Si el ESP32 publica un valor en ese tópico, cualquier cliente suscrito a ese mismo tópico podrá recibirlo.

Esto facilita el desarrollo de sistemas IoT distribuidos, ya que múltiples dispositivos pueden publicar y consumir información de manera organizada.


### 3.3.4 EMQX como broker MQTT gratuito

Para realizar las pruebas del taller se utilizará el broker público gratuito de EMQX.

EMQX proporciona un broker MQTT público orientado a pruebas, aprendizaje y prototipado, sin necesidad de instalar un servidor propio.

Los datos básicos de conexión son:

| Parámetro | Valor |
|----------|-------|
| Broker | `broker.emqx.io` |
| Puerto TCP | `1883` |
| Seguridad | Sin cifrado |
| Autenticación | No requiere usuario ni contraseña |
| Uso recomendado | Pruebas, aprendizaje y prototipos |

El puerto `1883` corresponde al puerto TCP estándar para comunicación MQTT sin cifrado.

Es importante considerar que este broker es público. Por lo tanto, no debe utilizarse para enviar información sensible, contraseñas reales, datos personales o información privada.



### 3.3.5 Datos de conexión a EMQX

En MicroPython, los datos de conexión pueden definirse mediante variables.

```python
MQTT_BROKER = "broker.emqx.io"
MQTT_PORT = 1883
```

Como el broker público no requiere autenticación básica para las pruebas iniciales, no será necesario definir usuario ni contraseña.

Más adelante, al utilizar un broker privado o una instancia propia de EMQX, podrían agregarse variables como:

```python
MQTT_USER = "usuario"
MQTT_PASSWORD = "password"
```


### 3.3.6 Concepto de tópico MQTT

En MQTT, los mensajes se organizan mediante tópicos.

Un tópico es una cadena de texto que funciona como una ruta lógica donde se publica o se recibe información.

Ejemplo:

```text
estaciones/estacion_01/temperatura
```

Los tópicos permiten organizar la información de forma jerárquica.

```text
estaciones
 └── estacion_01
      ├── temperatura
      ├── humedad
      └── estado
```


### 3.3.7 Estructura de tópicos para la estación ambiental

| Dato | Tópico |
|------|--------|
| Temperatura | `estaciones/estacion_01/temperatura` |
| Humedad | `estaciones/estacion_01/humedad` |
| Estado ambiental | `estaciones/estacion_01/estado` |

```python
station_id = "estacion_01"

topic_temp = "estaciones/" + station_id + "/temperatura"
topic_hum = "estaciones/" + station_id + "/humedad"
topic_estado = "estaciones/" + station_id + "/estado"
```


### 3.3.8 Publicación de mensajes

Publicar un mensaje significa enviar información hacia un tópico específico.

```text
Publicar:
Tópico  → estaciones/estacion_01/temperatura
Mensaje → 28.5
```

Ejemplo conceptual en MicroPython:

```python
cliente.publish(
    topic_temp,
    str(temperatura)
)
```


### 3.3.9 Suscripción a tópicos

Suscribirse a un tópico significa indicar al broker que se desea recibir los mensajes publicados en esa ruta.

```text
ESP32 publica:
estaciones/estacion_01/temperatura → 28.5

Cliente suscrito recibe:
28.5
```


### 3.3.10 Uso de comodines en tópicos MQTT

Cuando una aplicación necesita monitorear múltiples dispositivos, puede resultar poco práctico suscribirse individualmente a cada tópico.

Para resolver este problema, MQTT incorpora comodines (*wildcards*) que permiten suscribirse simultáneamente a varios tópicos relacionados.

Los comodines sólo pueden utilizarse en operaciones de suscripción y no en operaciones de publicación.

Los dos comodines más utilizados son:

| Comodín | Descripción |
|---------|-------------|
| `+` | Sustituye exactamente un nivel del tópico. |
| `#` | Sustituye uno o más niveles del tópico. Debe colocarse al final del tópico. |

#### Comodín de nivel único (+)

El símbolo `+` permite reemplazar un único nivel dentro de la jerarquía del tópico.

Por ejemplo:

```text
estaciones/+/temperatura
```

Coincidirá con:
```bash
estaciones/estacion_01/temperatura
estaciones/estacion_02/temperatura
estaciones/estacion_03/temperatura
```
Pero no coincidirá con:

```bash
estaciones/estacion_01/humedad
estaciones/estacion_01/configuracion/temperatura
```
porque el comodín + sólo sustituye un nivel.

La estructura puede visualizarse así:

```text
estaciones
     │
     ├── estacion_01
     │         │
     │         └── temperatura
     │
     ├── estacion_02
     │         │
     │         └── temperatura
     │
     └── estacion_03
               │
               └── temperatura
```
Este tipo de suscripción resulta útil cuando se desea monitorear la temperatura de todas las estaciones de una red distribuida.

#### Comodín multinivel (#)

El símbolo # permite sustituir uno o más niveles del tópico.

Debe colocarse al final de la ruta.

Por ejemplo:
```text
estaciones/#
```

Coincidirá con:

```text
estaciones/estacion_01/temperatura
estaciones/estacion_01/humedad
estaciones/estacion_01/estado
estaciones/estacion_02/temperatura
estaciones/estacion_02/humedad
estaciones/estacion_02/estado
estaciones/estacion_03/temperatura
```

La estructura puede visualizarse así:

```text
estaciones
 ├── estacion_01
 │     ├── temperatura
 │     ├── humedad
 │     └── estado
 │
 ├── estacion_02
 │     ├── temperatura
 │     ├── humedad
 │     └── estado
 │
 └── estacion_03
       ├── temperatura
       ├── humedad
       └── estado
```

Al suscribirse a:
```text
estaciones/#
```

se recibirán todos los mensajes publicados dentro de la jerarquía estaciones.

Ejemplos aplicados al proyecto integrador

Supongamos que existen tres estaciones ambientales:
```text
estacion_01
estacion_02
estacion_03
```

Cada una publica:
```text
estaciones/estacion_01/temperatura
estaciones/estacion_01/humedad
estaciones/estacion_01/estado

estaciones/estacion_02/temperatura
estaciones/estacion_02/humedad
estaciones/estacion_02/estado

estaciones/estacion_03/temperatura
estaciones/estacion_03/humedad
estaciones/estacion_03/estado
```

Si un cliente desea monitorear únicamente las temperaturas de todas las estaciones, puede suscribirse a:

```text
estaciones/+/temperatura
```

Si desea recibir toda la información generada por todas las estaciones, puede suscribirse a:

```text
estaciones/#
```

#### Importancia de los comodines
s comodines permiten construir aplicaciones escalables sin necesidad de conocer previamente cuántos dispositivos existen en la red.

En sistemas IoT con decenas o cientos de dispositivos, el uso de comodines facilita:

* Monitoreo centralizado.
* Dashboards en tiempo real.
* Sistemas de análisis de datos.
* Supervisión de múltiples estaciones.
* Detección de eventos y alertas.

En el proyecto integrador, los comodines serán especialmente útiles cuando se implemente el monitoreo simultáneo de varias estaciones ambientales desde una única aplicación cliente MQTT.

### 3.3.11 Calidad de servicio en MQTT

| QoS | Descripción |
|-----|-------------|
| 0 | El mensaje se envía una vez, sin confirmación de entrega. |
| 1 | El mensaje se entrega al menos una vez. |
| 2 | El mensaje se entrega exactamente una vez. |

En este taller se utilizará principalmente QoS 0, dado que es el nivel de calidad soportado por EQMX de manera gratuita.


### 3.3.12 Relación de MQTT con el proyecto integrador

La evolución del proyecto puede representarse así:

```text
Tema 2
Lectura local + OLED
        │
        ▼
Tema 3
WiFi + MQTT
Publicación de datos
        │
        ▼
Tema 4
Configuración remota
de parámetros
        │
        ▼
Tema 5
Supabase + Dashboard
```

En este segundo incremento, la estación ambiental seguirá midiendo y visualizando datos localmente, pero además podrá enviar información al broker MQTT.


### 3.3.13 Resultado esperado de esta sección

Al finalizar esta sección, el participante deberá comprender que:

- MQTT es un protocolo ligero utilizado en aplicaciones IoT.
- La comunicación MQTT se basa en publicación y suscripción.
- El broker MQTT recibe y distribuye los mensajes.
- EMQX será utilizado como broker público gratuito para pruebas.
- Los datos se organizan mediante tópicos.
- La estación ambiental publicará temperatura, humedad y estado ambiental.
- Los comodines permiten monitorear múltiples estaciones.
- MQTT será la base para evolucionar hacia configuración remota y almacenamiento en la nube.

### 3.3.14 Código integrado con WiFi + MQTT usando EMQX

```python
# Con WiFi + MQTT + OLED I2C
from machine import Pin, SoftI2C
from ssd1306 import SSD1306_I2C
from umqtt.simple import MQTTClient
import dht
import utime
import network
import ubinascii
import machine
import socket

# Identificación de la estación
station_id = "estacion_01"

# Credenciales WiFi
SSID = "MiRed"
PASSWORD = "MiPassword"

# Configuración MQTT - EMQX público
MQTT_BROKER = "broker.emqx.io"
MQTT_PORT = 1883
MQTT_CLIENT_ID = station_id + "_" + ubinascii.hexlify(machine.unique_id()).decode()

# Parámetros locales configurables
temp_max = 30
hum_min = 40
hum_max = 80
intervalo_lectura = 2

# Tópicos MQTT
topic_temp = "estaciones/" + station_id + "/temperatura"
topic_hum = "estaciones/" + station_id + "/humedad"
topic_estado = "estaciones/" + station_id + "/estado"

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


def conectar_wifi(ssid, password):
    wifi = network.WLAN(network.STA_IF)
    wifi.active(True)

    if not wifi.isconnected():
        print("Conectando a WiFi...")
        wifi.connect(ssid, password)

        timeout = 20

        while not wifi.isconnected() and timeout > 0:
            utime.sleep(1)
            timeout -= 1
            print(".", end="")

    print()

    if wifi.isconnected():
        config = wifi.ifconfig()

        # Forzar DNS público
        wifi.ifconfig((
            config[0],
            config[1],
            config[2],
            "8.8.8.8"
        ))

        print("WiFi conectado")
        print("IP:", wifi.ifconfig()[0])
        print("DNS:", wifi.ifconfig()[3])

        return wifi
    else:
        raise Exception("No fue posible conectar a WiFi")


def probar_dns():
    print("Probando DNS...")

    try:
        info = socket.getaddrinfo(MQTT_BROKER, MQTT_PORT)
        print("DNS OK:")
        print(info)
        return True

    except OSError as e:
        print("Error DNS:")
        print(e)
        return False


def conectar_mqtt():
    try:
        cliente = MQTTClient(
            client_id=MQTT_CLIENT_ID,
            server=MQTT_BROKER,
            port=MQTT_PORT,
            keepalive=60
        )

        cliente.connect()

        print("Conectado a broker MQTT")
        print("Broker:", MQTT_BROKER)
        print("Puerto:", MQTT_PORT)
        print("Client ID:", MQTT_CLIENT_ID)

        return cliente

    except OSError as e:
        print("Error al conectar con MQTT")
        print("Broker:", MQTT_BROKER)
        print("Puerto:", MQTT_PORT)
        print("Client ID:", MQTT_CLIENT_ID)
        print("Detalle:", e)
        raise


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


def publicar_datos(cliente, temp, hum, estado):
    cliente.publish(topic_temp.encode(), str(temp).encode())
    cliente.publish(topic_hum.encode(), str(hum).encode())
    cliente.publish(topic_estado.encode(), estado.encode())

    print("Datos publicados por MQTT")


# Conexión WiFi
wifi = conectar_wifi(
    SSID,
    PASSWORD
)

oled.fill(0)
oled.text("WiFi OK", 0, 0)
oled.text(wifi.ifconfig()[0], 0, 15)
oled.show()
utime.sleep(2)

# Prueba DNS antes de MQTT
dns_ok = probar_dns()

if not dns_ok:
    oled.fill(0)
    oled.text("Error DNS", 0, 0)
    oled.text("Revisar red", 0, 15)
    oled.show()
    raise Exception("No se pudo resolver el broker MQTT.")

# Conexión MQTT
cliente_mqtt = conectar_mqtt()

oled.fill(0)
oled.text("MQTT OK", 0, 0)
oled.text("EMQX", 0, 15)
oled.show()
utime.sleep(2)

# Bucle principal
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

        publicar_datos(
            cliente_mqtt,
            temperatura,
            humedad,
            estado
        )

        print(datos)

        utime.sleep(intervalo_lectura)

    except OSError as e:
        oled.fill(0)
        oled.text("Error!", 0, 20)
        oled.show()
        print("Error durante ejecucion:", e)
        utime.sleep(intervalo_lectura)
```
Los tópicos publicados quedan así:

```text
estaciones/estacion_01/temperatura
estaciones/estacion_01/humedad
estaciones/estacion_01/estado
````




# Tema 4. Configuración remota mediante MQTT

## 4.1 Introducción
## 4.1 Introducción

En el tema anterior se incorporó la conectividad WiFi y la comunicación MQTT a la estación ambiental basada en ESP32. Con ello, el nodo dejó de funcionar únicamente como una estación local y comenzó a publicar datos de temperatura, humedad y estado ambiental hacia un broker MQTT, permitiendo su monitoreo remoto en tiempo real.

Sin embargo, en muchos sistemas IoT no basta con enviar información desde el dispositivo hacia una aplicación externa. También es necesario que el dispositivo pueda recibir instrucciones, parámetros o comandos desde la red para modificar su comportamiento sin necesidad de reprogramarlo físicamente.

En este tema se ampliará la funcionalidad de la estación ambiental para que, además de publicar datos, pueda suscribirse a tópicos MQTT y recibir mensajes de configuración remota. Esto permitirá modificar parámetros como la temperatura máxima permitida, los límites de humedad y el intervalo de lectura desde un cliente MQTT externo.

De esta manera, la estación evolucionará de un nodo de monitoreo hacia un nodo IoT configurable, capaz de ajustar su operación en tiempo real a partir de instrucciones recibidas mediante MQTT.

Como parte del proyecto integrador **Red Distribuida de Estaciones Ambientales IoT con MicroPython**, este tema corresponde al tercer incremento funcional de la solución. En esta etapa se incorporará la comunicación entre la estación ambiental y el broker MQTT, sentando las bases para procesos de supervisión, control remoto y administración de múltiples estaciones.

La arquitectura general que se implementará durante este tema se muestra conceptualmente a continuación:

```text
        Cliente MQTT
             │
             ▼
        Broker MQTT
             │
             ▼
           ESP32
             │
     ┌───────┴────────┐
     ▼                ▼
  DHT22             OLED
```

Durante el desarrollo del tema se abordarán los conceptos de suscripción MQTT, recepción de mensajes, funciones *callback*, tópicos de configuración y validación básica de comandos. Asimismo, se continuará utilizando la estación ambiental desarrollada previamente, integrando nuevas funciones de manera incremental.

Al finalizar el tema, la estación ambiental será capaz de medir temperatura y humedad, visualizar la información localmente, publicar datos hacia el broker MQTT y recibir parámetros remotos para modificar su comportamiento durante la ejecución.

## 4.2 Del monitoreo remoto al control remoto

En el Tema 3, la estación ambiental adquirió la capacidad de enviar información hacia un broker MQTT, permitiendo que otros dispositivos o aplicaciones pudieran monitorear en tiempo real las variables ambientales medidas por el sensor DHT22. En esta etapa, la comunicación se realizó únicamente en un sentido: desde la estación hacia el broker.

Este modelo de comunicación resulta adecuado para aplicaciones donde únicamente se requiere observar el comportamiento de un proceso. Sin embargo, en muchos sistemas IoT reales también es necesario enviar instrucciones desde una aplicación remota hacia los dispositivos desplegados en campo.

Esta capacidad transforma un sistema de monitoreo en un sistema de supervisión y control, permitiendo modificar parámetros de operación, cambiar configuraciones o ejecutar acciones sin necesidad de acceder físicamente al dispositivo.

En el proyecto integrador, la estación ambiental mantendrá la publicación periódica de temperatura, humedad y estado ambiental, pero además comenzará a escuchar determinados tópicos MQTT para recibir nuevas configuraciones enviadas por un cliente remoto.

La comunicación pasará de ser unidireccional a bidireccional.

```text
                 Tema 3
        Monitoreo remoto

        Publicación MQTT

ESP32 ----------------------► Broker
                               │
                               ▼
                        Cliente MQTT
```

```text
                 Tema 4
      Monitoreo + Control remoto

             Publicación
ESP32 ----------------------► Broker
  ▲                            │
  │                            ▼
  └──────── Configuración ◄──── Cliente MQTT
```

Gracias a esta nueva capacidad será posible modificar, por ejemplo:

- La temperatura máxima permitida (`temp_max`).
- El límite inferior de humedad (`hum_min`).
- El límite superior de humedad (`hum_max`).
- El intervalo entre lecturas (`intervalo_lectura`).

Estos cambios se aplicarán mientras el programa se encuentra en ejecución, sin necesidad de detener la estación ni volver a cargar el código en la ESP32.

Este mecanismo constituye una de las principales ventajas de los sistemas IoT modernos, ya que permite administrar dispositivos distribuidos desde una ubicación central, facilitando las tareas de supervisión, mantenimiento y ajuste de parámetros de operación.

En las siguientes secciones se estudiará cómo MQTT implementa esta comunicación bidireccional mediante operaciones de suscripción, funciones *callback* y recepción de mensajes publicados en tópicos específicos de configuración.

## 4.3 Publicación y suscripción en MQTT

En el Tema 3 se estudió que MQTT utiliza un modelo de comunicación basado en **publicación** (*publish*) y **suscripción** (*subscribe*). Gracias a este modelo, los dispositivos intercambian información a través de un broker MQTT sin necesidad de comunicarse directamente entre sí.

Hasta este momento, la estación ambiental únicamente ha actuado como **publicador**, enviando periódicamente los valores de temperatura, humedad y estado ambiental hacia los tópicos definidos para cada variable.

```text
ESP32
(Publisher)
     │
     │ Publica datos
     ▼
Broker MQTT
     │
     ▼
Clientes MQTT
(Subscribers)
```

Sin embargo, MQTT también permite que un mismo dispositivo desempeñe simultáneamente el papel de **publicador** y **suscriptor**.

En este caso, la estación ambiental continuará publicando sus mediciones, pero además se suscribirá a un **único tópico de configuración**, donde esperará recibir mensajes en formato JSON enviados desde un cliente MQTT.

La arquitectura de comunicación será la siguiente:

```text
                   Publicación
ESP32 --------------------------------► Broker MQTT
  ▲                                       │
  │                                       ▼
  └──────────── Suscripción ◄──────── Cliente MQTT
```

De esta forma, el ESP32 mantendrá una comunicación bidireccional con el broker.

Mientras publica información ambiental, permanecerá escuchando continuamente el tópico de configuración para detectar si algún cliente ha enviado una nueva configuración de operación.

Conceptualmente, el flujo de trabajo será el siguiente:

1. La estación mide temperatura y humedad.
2. Publica los datos mediante MQTT.
3. El broker distribuye la información a los clientes suscritos.
4. Un cliente MQTT envía un mensaje JSON con la nueva configuración.
5. El broker entrega dicho mensaje al ESP32.
6. La estación actualiza únicamente los parámetros incluidos en el mensaje y continúa operando con los nuevos valores.

Este comportamiento puede representarse de la siguiente manera:

```text
                Temperatura
ESP32 ------------------------------► Broker
                                      │
                                      ▼
                               Cliente MQTT

Cliente MQTT ------------------------► Broker
        JSON de configuración          │
                                       ▼
                                     ESP32
```

Una de las principales ventajas de este modelo es que el publicador y el suscriptor permanecen desacoplados. El cliente que envía la configuración no necesita conocer la dirección IP del ESP32 ni establecer una conexión directa con él; únicamente debe publicar el mensaje JSON en el tópico de configuración correspondiente. Del mismo modo, la estación ambiental sólo necesita permanecer suscrita a dicho tópico para recibir automáticamente cualquier actualización enviada por el broker.

Esta arquitectura facilita el desarrollo de sistemas IoT escalables, ya que un mismo broker puede administrar simultáneamente la comunicación entre decenas o incluso cientos de dispositivos distribuidos.

## 4.4 Tópicos de configuración remota

## 4.4 Tópicos de configuración remota

En el Tema 3 se definieron los tópicos utilizados para publicar las variables ambientales de la estación, como la temperatura, la humedad y el estado del sistema.

Para implementar la configuración remota podría definirse un tópico independiente para cada parámetro de operación. Sin embargo, en este proyecto se utilizará una estrategia diferente: un **único tópico de configuración** que recibirá mensajes en formato JSON.

Esta aproximación reduce el número de suscripciones necesarias y permite modificar uno o varios parámetros mediante un solo mensaje.

La estructura general de los tópicos queda de la siguiente manera:

```text
estaciones
└── estacion_01_xxxxxxxxxxxx
    ├── temperatura
    ├── humedad
    ├── estado
    └── config
```

donde `estacion_01_xxxxxxxxxxxx` corresponde al identificador único almacenado en la variable `MQTT_CLIENT_ID`.

Los tópicos utilizados por la estación serán:

| Información | Tópico MQTT |
|-------------|-------------|
| Temperatura | `estaciones/<MQTT_CLIENT_ID>/temperatura` |
| Humedad | `estaciones/<MQTT_CLIENT_ID>/humedad` |
| Estado ambiental | `estaciones/<MQTT_CLIENT_ID>/estado` |
| Configuración remota | `estaciones/<MQTT_CLIENT_ID>/config` |

En MicroPython, estos tópicos se definen de la siguiente manera:

```python
topic_temp = "estaciones/" + MQTT_CLIENT_ID + "/temperatura"
topic_hum = "estaciones/" + MQTT_CLIENT_ID + "/humedad"
topic_estado = "estaciones/" + MQTT_CLIENT_ID + "/estado"

topic_config = "estaciones/" + MQTT_CLIENT_ID + "/config"
```

Gracias al uso de `MQTT_CLIENT_ID`, cada estación genera automáticamente un conjunto de tópicos únicos, evitando interferencias cuando varias estaciones utilizan simultáneamente el mismo broker MQTT.

La configuración remota ya no se envía como un valor aislado asociado a un tópico específico. En su lugar, el cliente MQTT publica un mensaje JSON sobre el tópico de configuración.

Por ejemplo:

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45,
        "hum_max": 75,
        "intervalo_lectura": 5
    }
}
```

Este mensaje puede contener uno o varios parámetros. La estación únicamente actualizará aquellos que estén presentes dentro de la sección `config`, manteniendo sin cambios el resto de la configuración.

Posteriormente, el ESP32 se suscribirá únicamente al tópico `config`, permaneciendo a la espera de nuevos mensajes enviados desde cualquier cliente MQTT autorizado.

El uso de un único tópico de configuración constituye una estrategia ampliamente utilizada en aplicaciones IoT, ya que simplifica el protocolo de comunicación, facilita su mantenimiento y permite extender el sistema incorporando nuevas opciones de configuración sin necesidad de crear tópicos adicionales.

## 4.5 Función callback en MQTT

Hasta este momento, la estación ambiental ha enviado información mediante el método `publish()`. Sin embargo, para que el ESP32 pueda recibir mensajes desde el broker MQTT es necesario definir un mecanismo que procese automáticamente cada mensaje recibido.

En MQTT, este mecanismo se implementa mediante una **función callback**.

Una función *callback* es una función que no se ejecuta directamente desde el programa principal, sino que es invocada automáticamente por otra función o biblioteca cuando ocurre un determinado evento.

En el caso de MQTT, el evento corresponde a la recepción de un mensaje en alguno de los tópicos a los que el cliente se encuentra suscrito.

El flujo general de funcionamiento puede representarse de la siguiente manera:

```text
Cliente MQTT
      │
      │ Publica mensaje
      ▼
Broker MQTT
      │
      │ Entrega mensaje
      ▼
ESP32
      │
      ▼
Función callback
      │
      ▼
Procesa el mensaje
```

La biblioteca `umqtt.simple` permite registrar una función callback mediante el método:

```python
cliente.set_callback(mi_callback)
```

donde `mi_callback` es el nombre de la función que será ejecutada automáticamente cuando llegue un nuevo mensaje.

Una estructura básica de una función callback es la siguiente:

```python
def callback(topic, msg):
    print("Topico:", topic)
    print("Mensaje:", msg)
```

Esta función recibe dos argumentos:

| Parámetro | Descripción |
|-----------|-------------|
| `topic` | Tópico MQTT en el que se recibió el mensaje. |
| `msg` | Contenido del mensaje recibido. |

Es importante considerar que tanto el tópico como el mensaje son recibidos en formato **bytes**, por lo que normalmente es necesario convertirlos antes de procesarlos.

El tópico se convierte a una cadena de texto mediante:

```python
topic = topic.decode()
```

Mientras que el contenido del mensaje se mantiene en formato `bytes`, ya que será procesado directamente por la biblioteca `ujson`.

Por ejemplo, si un cliente MQTT publica el siguiente mensaje:

**Tópico**

```text
estaciones/estacion_01_xxxxxxxxxxxx/config
```

**Mensaje**

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45
    }
}
```

La función callback recibirá internamente:

```python
topic = b'estaciones/estacion_01_xxxxxxxxxxxx/config'

msg = b'{"config":{"temp_max":28,"hum_min":45}}'
```

Posteriormente, el mensaje JSON se convierte automáticamente en un diccionario de MicroPython mediante:

```python
datos = ujson.loads(msg)
```

obteniendo una estructura como la siguiente:

```python
{
    "config": {
        "temp_max": 28,
        "hum_min": 45
    }
}
```

A partir de este momento, el programa ya no necesita analizar el nombre del tópico para determinar qué parámetro debe modificar. En su lugar, únicamente recorre el diccionario contenido en la clave `config`.

Conceptualmente, el procesamiento puede representarse de la siguiente manera:

```text
Mensaje recibido
        │
        ▼
Convertir JSON
        │
        ▼
Diccionario
        │
        ▼
Recorrer config
        │
        ▼
Actualizar únicamente
los parámetros recibidos
```

Esta estrategia ofrece una mayor flexibilidad, ya que un mismo mensaje puede modificar uno o varios parámetros simultáneamente sin necesidad de utilizar múltiples tópicos de configuración.

En la siguiente sección se implementará la función `callback()` completa, la cual recorrerá el diccionario recibido, validará los valores de cada parámetro y actualizará la configuración de la estación ambiental durante la ejecución del programa.

## 4.6 Recepción de mensajes desde el broker

Una vez definida la función *callback*, el siguiente paso consiste en indicar al cliente MQTT a qué tópico desea suscribirse y verificar periódicamente si el broker ha enviado nuevos mensajes.

Este proceso se realiza en tres etapas:

1. Registrar la función *callback*.
2. Suscribirse al tópico de configuración.
3. Revisar continuamente si existen mensajes pendientes.

### Registrar la función callback

Después de crear la conexión con el broker MQTT, se registra la función que procesará los mensajes recibidos.

```python
cliente_mqtt.set_callback(callback)
```

A partir de este momento, cualquier mensaje recibido en el tópico de configuración será enviado automáticamente a la función `callback()`.

### Suscribirse al tópico de configuración

Posteriormente, el cliente MQTT debe indicar al broker cuál es el tópico que desea escuchar.

En este proyecto la estación ambiental se suscribirá únicamente al tópico de configuración definido previamente.

```python
cliente_mqtt.subscribe(topic_config.encode())
```

Al realizar esta suscripción, el broker registrará que la estación ambiental desea recibir cualquier mensaje publicado en dicho tópico.

Conceptualmente, el proceso puede representarse así:

```text
ESP32
  │
  └── Suscribe → estaciones/<MQTT_CLIENT_ID>/config
                     │
                     ▼
                Broker MQTT
```

A partir de este momento, cualquier cliente MQTT podrá publicar un mensaje JSON en el tópico de configuración y el broker lo enviará automáticamente a la estación correspondiente.

### Verificar la llegada de nuevos mensajes

La biblioteca `umqtt.simple` no ejecuta automáticamente la función *callback* cuando llega un mensaje. Es necesario que el programa consulte periódicamente al broker para verificar si existen mensajes pendientes.

Para ello se utiliza el método:

```python
cliente_mqtt.check_msg()
```

Este método realiza las siguientes acciones:

- Consulta al broker si existe algún mensaje pendiente.
- Si no hay mensajes, continúa la ejecución del programa.
- Si existe un mensaje, lo recibe y ejecuta automáticamente la función *callback* registrada previamente.

Por esta razón, es recomendable llamar a `check_msg()` dentro del ciclo principal del programa.

Por ejemplo:

```python
while True:

    cliente_mqtt.check_msg()

    temperatura, humedad = obtener_temperatura()

    estado = evaluar_estado(
        temperatura,
        humedad
    )

    publicar_datos(
        cliente_mqtt,
        temperatura,
        humedad,
        estado
    )

    utime.sleep(intervalo_lectura)
```

De esta forma, en cada iteración del programa la estación realizará dos tareas principales:

1. Verificar si existe una nueva configuración enviada mediante MQTT.
2. Continuar con la lectura del sensor y la publicación de los datos ambientales.

El flujo general de operación será el siguiente:

```text
Inicio del ciclo
        │
        ▼
check_msg()
        │
        ▼
¿Llegó un mensaje?
        │
   ┌────┴────┐
   │         │
 No         Sí
   │         │
   │         ▼
   │    Ejecutar callback()
   │         │
   └────┬────┘
        ▼
Leer DHT22
        │
        ▼
Evaluar estado
        │
        ▼
Publicar datos MQTT
        │
        ▼
Esperar intervalo
        │
        ▼
Repetir
```

Gracias a este mecanismo, la estación ambiental puede seguir publicando sus mediciones periódicamente mientras permanece atenta a cualquier mensaje de configuración enviado desde un cliente MQTT, logrando una comunicación bidireccional sin interrumpir su funcionamiento continuo.

## 4.7 Modificación remota de parámetros locales

Una vez que la estación ambiental es capaz de recibir mensajes desde el broker MQTT, el siguiente paso consiste en interpretar el contenido del mensaje y utilizarlo para modificar los parámetros locales que controlan el funcionamiento del programa.

En este proyecto, los parámetros que podrán modificarse de forma remota son:

- Temperatura máxima (`temp_max`).
- Humedad mínima (`hum_min`).
- Humedad máxima (`hum_max`).
- Intervalo entre lecturas (`intervalo_lectura`).

Todos estos parámetros fueron definidos previamente como variables globales al inicio del programa.

```python
temp_max = 30
hum_min = 40
hum_max = 80
intervalo_lectura = 2
```

Cuando el ESP32 reciba un mensaje en el tópico de configuración, la función `callback()` convertirá el contenido JSON en un diccionario y recorrerá los parámetros incluidos dentro de la clave `config`.

Una implementación simplificada es la siguiente:

```python
config = datos["config"]

for parametro, valor in config.items():

    valor = float(valor)

    if parametro == "temp_max":
        temp_max = valor

    elif parametro == "hum_min":
        hum_min = valor

    elif parametro == "hum_max":
        hum_max = valor

    elif parametro == "intervalo_lectura":
        intervalo_lectura = valor
```

En esta función se utiliza la palabra reservada `global` para indicar que las variables modificadas corresponden a las definidas fuera de la función y no a variables locales.

A diferencia de la implementación basada en múltiples tópicos, ahora el programa no necesita identificar qué parámetro se desea modificar a partir del nombre del tópico. Toda la información necesaria se encuentra dentro del documento JSON recibido.

Por ejemplo, si un cliente MQTT publica el siguiente mensaje:

```json
{
    "config": {
        "temp_max": 28
    }
}
```

la variable:

```python
temp_max
```

pasará automáticamente de:

```text
30
```

a:

```text
28
```

De forma similar, si se publica:

```json
{
    "config": {
        "intervalo_lectura": 5
    }
}
```

la variable:

```python
intervalo_lectura
```

cambiará su valor de:

```text
2
```

a:

```text
5
```

También es posible modificar varios parámetros mediante un único mensaje.

Por ejemplo:

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45,
        "hum_max": 75,
        "intervalo_lectura": 5
    }
}
```

En este caso, la función `callback()` recorrerá el diccionario `config` y actualizará únicamente los parámetros presentes en el mensaje, manteniendo sin cambios cualquier otro parámetro de configuración.

El proceso completo puede representarse mediante el siguiente diagrama:

```text
Cliente MQTT
       │
       │ Publica JSON
       ▼
Broker MQTT
       │
       ▼
ESP32
       │
       ▼
callback()
       │
       ▼
Convierte JSON
       │
       ▼
Recorre config
       │
       ▼
Actualiza variables
       │
       ▼
El programa continúa utilizando
los nuevos valores
```

Este mecanismo constituye una de las principales ventajas de utilizar mensajes JSON para la configuración remota. Además de reducir el número de tópicos MQTT necesarios, permite agrupar múltiples parámetros en un solo mensaje, simplifica la comunicación entre dispositivos y facilita la incorporación de nuevas opciones de configuración conforme evoluciona el sistema IoT.

## 4.8 Validación de comandos recibidos

Aunque la estación ambiental ya puede recibir mensajes y modificar sus parámetros locales, es importante considerar que no todos los mensajes enviados desde un cliente MQTT necesariamente serán correctos.

Por ejemplo, un cliente podría enviar un mensaje con una estructura JSON incorrecta, utilizar un nombre de parámetro inexistente o proporcionar un valor fuera del rango permitido.

Por ejemplo, el siguiente mensaje contiene un valor no numérico:

```json
{
    "config": {
        "temp_max": "abc"
    }
}
```

Mientras que el siguiente contiene un valor fuera del rango esperado:

```json
{
    "config": {
        "temp_max": 120
    }
}
```

En ambos casos, la estación debe evitar modificar su configuración con información inválida.

Por esta razón, la función `callback()` incorpora mecanismos básicos de validación antes de actualizar las variables del programa.

La validación realizada contempla los siguientes aspectos:

- Verificar que el mensaje recibido tenga formato JSON válido.
- Comprobar que exista la clave `config`.
- Validar que cada parámetro recibido sea reconocido.
- Convertir cada valor al tipo de dato esperado.
- Verificar que el valor se encuentre dentro del rango permitido.

Una parte de esta validación puede observarse en el siguiente fragmento del programa:

```python
for parametro, valor in config.items():

    valor = float(valor)

    if parametro == "temp_max":
        if 0 <= valor <= 60:
            temp_max = valor
        else:
            print("Valor fuera de rango:", parametro)

    elif parametro == "hum_min":
        if 0 <= valor <= 100:
            hum_min = valor
        else:
            print("Valor fuera de rango:", parametro)

    elif parametro == "hum_max":
        if 0 <= valor <= 100:
            hum_max = valor
        else:
            print("Valor fuera de rango:", parametro)

    elif parametro == "intervalo_lectura":
        if 1 <= valor <= 60:
            intervalo_lectura = valor
        else:
            print("Valor fuera de rango:", parametro)

    else:
        print("Parametro desconocido:", parametro)
```

Toda esta lógica se encuentra protegida mediante un bloque `try-except`.

```python
try:
    datos = ujson.loads(msg)

    ...

except Exception as e:
    print("Error al procesar JSON:", e)
```

De esta manera, si el mensaje recibido no tiene un formato JSON válido o alguno de sus valores no puede convertirse correctamente al tipo de dato esperado, el programa continúa ejecutándose sin interrumpir la operación de la estación ambiental.

Los rangos utilizados para validar cada parámetro son los siguientes:

| Parámetro | Rango permitido |
|------------|-----------------|
| `temp_max` | 0 a 60 °C |
| `hum_min` | 0 a 100 % |
| `hum_max` | 0 a 100 % |
| `intervalo_lectura` | 1 a 60 segundos |

Por ejemplo, si se recibe el siguiente mensaje:

```json
{
    "config": {
        "temp_max": 120
    }
}
```

la salida será similar a:

```text
Valor fuera de rango: temp_max
```

En cambio, si se recibe un parámetro no reconocido:

```json
{
    "config": {
        "temperatura_maxima": 28
    }
}
```

la estación mostrará:

```text
Parametro desconocido: temperatura_maxima
```

y continuará funcionando normalmente.

La validación de los mensajes constituye una práctica fundamental en aplicaciones IoT, ya que evita configuraciones incorrectas, reduce la posibilidad de errores durante la ejecución y mejora la confiabilidad del sistema.

En aplicaciones reales, este mecanismo puede complementarse con autenticación del broker, conexiones cifradas mediante TLS, control de acceso a los tópicos MQTT, confirmación de cambios y almacenamiento persistente de la configuración recibida.


## 4.9 Código integrado WiFi + MQTT + OLED + configuración remota mediante JSON

A continuación se presenta el código integrado de la estación ambiental. En esta versión, el ESP32 se conecta a una red WiFi, publica la temperatura, la humedad y el estado ambiental mediante MQTT, y además permanece suscrito **al tópico de configuración remota**.

A diferencia de la versión desarrollada en el tema anterior, la configuración ya no se recibe mediante varios tópicos independientes. En su lugar, se utiliza un único tópico que transporta un mensaje en formato JSON. Esta estrategia simplifica la administración de la estación y permite modificar uno o varios parámetros utilizando un solo mensaje.

El tópico de configuración utilizado será:

```text
estaciones/<MQTT_CLIENT_ID>/config
```

donde `<MQTT_CLIENT_ID>` corresponde al identificador único generado para cada estación.

El formato esperado del mensaje es el siguiente:

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45,
        "hum_max": 75,
        "intervalo_lectura": 5
    }
}
```

También es posible modificar únicamente uno de los parámetros:

```json
{
    "config": {
        "temp_max": 28
    }
}
```

Para procesar mensajes JSON en MicroPython se utiliza la biblioteca `ujson`.

```python
import ujson
```

### Definición del tópico de configuración

En lugar de definir un tópico por cada parámetro, ahora se utiliza un único tópico para recibir toda la configuración.

```python
topic_config = "estaciones/" + MQTT_CLIENT_ID + "/config"
```

### Función callback

La función `callback()` recibe el mensaje publicado por el broker MQTT, convierte el contenido JSON en un diccionario de MicroPython y posteriormente recorre los parámetros recibidos para actualizar únicamente aquellos que fueron enviados por el cliente MQTT.

Durante este proceso también se realiza una validación básica de cada parámetro para evitar asignar valores fuera de los rangos esperados.

```python
def callback(topic, msg):

    global temp_max
    global hum_min
    global hum_max
    global intervalo_lectura

    topic = topic.decode()

    if topic != topic_config:
        return

    try:
        datos = ujson.loads(msg)

        if "config" not in datos:
            print("No existe la clave config")
            return

        config = datos["config"]

        for parametro, valor in config.items():

            valor = float(valor)

            if parametro == "temp_max":
                if 0 <= valor <= 60:
                    temp_max = valor
                    print(parametro, "=", valor)
                else:
                    print("Valor fuera de rango:", parametro)

            elif parametro == "hum_min":
                if 0 <= valor <= 100:
                    hum_min = valor
                    print(parametro, "=", valor)
                else:
                    print("Valor fuera de rango:", parametro)

            elif parametro == "hum_max":
                if 0 <= valor <= 100:
                    hum_max = valor
                    print(parametro, "=", valor)
                else:
                    print("Valor fuera de rango:", parametro)

            elif parametro == "intervalo_lectura":
                if 1 <= valor <= 60:
                    intervalo_lectura = valor
                    print(parametro, "=", valor)
                else:
                    print("Valor fuera de rango:", parametro)

            else:
                print("Parametro desconocido:", parametro)

        print("Configuración actualizada")

    except Exception as e:
        print("Error al procesar JSON:", e)
```

### Suscripción al tópico de configuración

Después de establecer la conexión con el broker MQTT, el cliente se suscribe al tópico de configuración.

```python
cliente.subscribe(topic_config.encode())
```

Por lo tanto, la función `conectar_mqtt()` queda de la siguiente manera:

```python
def conectar_mqtt():
    try:
        cliente = MQTTClient(
            client_id=MQTT_CLIENT_ID,
            server=MQTT_BROKER,
            port=MQTT_PORT,
            keepalive=60
        )

        cliente.set_callback(callback)
        cliente.connect()

        cliente.subscribe(topic_config.encode())

        print("Conectado a broker MQTT")
        print("Broker:", MQTT_BROKER)
        print("Puerto:", MQTT_PORT)
        print("Client ID:", MQTT_CLIENT_ID)

        print("Suscrito al topico:")
        print(topic_config)

        return cliente

    except OSError as e:
        print("Error al conectar con MQTT")
        print("Broker:", MQTT_BROKER)
        print("Puerto:", MQTT_PORT)
        print("Client ID:", MQTT_CLIENT_ID)
        print("Detalle:", e)
        raise
```

Gracias a esta estructura, el ESP32 puede actualizar únicamente los parámetros enviados por el cliente MQTT, manteniendo sin cambios el resto de la configuración.

El uso de un único tópico y mensajes en formato JSON simplifica la comunicación, reduce el número de suscripciones necesarias y hace que el protocolo sea más flexible para futuras ampliaciones del proyecto.

Esta ampliación permite que la estación ambiental evolucione de un sistema de monitoreo hacia un nodo IoT capaz de recibir configuraciones remotas durante su operación, sentando las bases para aplicaciones de automatización y control que se desarrollarán en temas posteriores.

### Código integrado final

A continuación se presenta el código integrado completo de la estación ambiental con soporte para WiFi, MQTT, publicación de datos ambientales y configuración remota mediante mensajes JSON.

```python
# Con WiFi + MQTT + OLED I2C + Configuracion remota mediante JSON

# (Código completo mostrado en la siguiente sección)
```

En esta versión del programa se realizaron los siguientes cambios respecto al código desarrollado en el Tema 3:

- Se agregó la biblioteca `ujson` para procesar mensajes en formato JSON.
- Se sustituyeron los cuatro tópicos de configuración por un único tópico general.

```python
topic_config = "estaciones/" + MQTT_CLIENT_ID + "/config"
```

- Los tópicos de publicación utilizan `MQTT_CLIENT_ID`, permitiendo que cada estación publique sobre rutas únicas dentro del broker MQTT.

```python
topic_temp = "estaciones/" + MQTT_CLIENT_ID + "/temperatura"
topic_hum = "estaciones/" + MQTT_CLIENT_ID + "/humedad"
topic_estado = "estaciones/" + MQTT_CLIENT_ID + "/estado"
```

- La función `callback()` ahora recibe un mensaje JSON con la clave `config`.
- El contenido de `config` se recorre mediante un diccionario utilizando `items()`.
- Se incorporó validación de rangos antes de actualizar cada parámetro.
- La estación se suscribe únicamente al tópico `topic_config`.
- Se mantiene la revisión continua de mensajes mediante `cliente_mqtt.check_msg()` dentro del ciclo principal.

El siguiente mensaje JSON permite modificar varios parámetros simultáneamente:

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45,
        "hum_max": 75,
        "intervalo_lectura": 5
    }
}
```

Este mensaje debe publicarse en el tópico:

```text
estaciones/<MQTT_CLIENT_ID>/config
```

donde `<MQTT_CLIENT_ID>` corresponde al identificador único generado por la estación.

### Reto propuesto

Como actividad adicional, se propone extender el protocolo de comunicación para incorporar una nueva sección denominada `command`, destinada a ejecutar acciones remotas sobre actuadores conectados a la ESP32.

Por ejemplo, controlar el LED integrado conectado al pin GPIO 2 mediante mensajes como los siguientes:

```json
{
    "command": {
        "device": "led",
        "action": "on"
    }
}
```

o

```json
{
    "command": {
        "device": "led",
        "action": "off"
    }
}
```

El objetivo del reto es modificar la función `callback()` para que, además de procesar la sección `config`, también sea capaz de interpretar la sección `command` y ejecutar la acción correspondiente sobre el dispositivo indicado.

## 4.10 Prueba desde cliente MQTT

Una vez que la estación ambiental se encuentra conectada al broker MQTT, es posible verificar su funcionamiento utilizando un cliente MQTT externo.

Durante este taller se utilizará el cliente web **MQTTX Web Client**, el cual permite conectarse a un broker MQTT, visualizar los mensajes publicados por la estación y enviar configuraciones remotas sin necesidad de instalar software adicional.

### Cliente web MQTT recomendado

El cliente web puede abrirse desde la siguiente dirección:

```text
https://mqttx.app/web-client
```

### Configuración de la conexión

Después de abrir la página, se debe crear una nueva conexión utilizando los siguientes parámetros:

| Parámetro | Valor |
|-----------|-------|
| Nombre de conexión | `Estación Ambiental` |
| Host | `broker.emqx.io` |
| Puerto | `8084` |
| Path | `/mqtt` |
| SSL/TLS | Activado |
| Username | Dejar vacío |
| Password | Dejar vacío |

Una vez configurados estos parámetros, se puede establecer la conexión con el broker MQTT.

### Monitoreo de los datos publicados

Para observar la información enviada por la estación ambiental, se recomienda suscribirse al siguiente tópico:

```text
estaciones/<MQTT_CLIENT_ID>/#
```

donde `<MQTT_CLIENT_ID>` corresponde al identificador generado por la ESP32.

Gracias al comodín `#`, el cliente recibirá todos los mensajes publicados por la estación:

```text
estaciones/<MQTT_CLIENT_ID>/temperatura
estaciones/<MQTT_CLIENT_ID>/humedad
estaciones/<MQTT_CLIENT_ID>/estado
```

Cada vez que la ESP32 realice una nueva lectura del sensor DHT22, los valores aparecerán automáticamente en el cliente MQTT.

### Envío de una configuración remota

Para modificar la configuración de la estación, se debe publicar un mensaje sobre el tópico:

```text
estaciones/<MQTT_CLIENT_ID>/config
```

utilizando como contenido el siguiente documento JSON:

```json
{
    "config": {
        "temp_max": 28,
        "hum_min": 45,
        "hum_max": 75,
        "intervalo_lectura": 5
    }
}
```

También es posible modificar únicamente uno de los parámetros.

Por ejemplo:

```json
{
    "config": {
        "temp_max": 26
    }
}
```

Una vez publicado el mensaje, la estación ambiental recibirá automáticamente la nueva configuración, actualizará los parámetros correspondientes y continuará operando sin necesidad de reiniciar el programa.

### Resultado esperado

Si la comunicación se realizó correctamente, en la consola del ESP32 se observará una salida similar a la siguiente:

```text
temp_max = 28
hum_min = 45
hum_max = 75
intervalo_lectura = 5

Configuracion actualizada
```

Asimismo, en el cliente MQTT continuarán visualizándose las publicaciones periódicas de temperatura, humedad y estado ambiental, ahora utilizando los nuevos parámetros de operación.

Con esta prueba se verifica que la estación ambiental es capaz de mantener una comunicación bidireccional mediante MQTT: por un lado publica continuamente la información del sensor y, por otro, recibe configuraciones remotas en tiempo real utilizando un único tópico y mensajes en formato JSON.

## 4.11 Resultado esperado del incremento

Al finalizar este incremento, la estación ambiental habrá evolucionado de un nodo IoT capaz únicamente de publicar información hacia un dispositivo con comunicación bidireccional mediante MQTT.

En esta versión, la estación será capaz de:

- Conectarse a una red WiFi.
- Publicar periódicamente la temperatura, la humedad y el estado ambiental mediante MQTT.
- Suscribirse a un tópico de configuración remota.
- Recibir mensajes JSON enviados desde un cliente MQTT.
- Procesar la información contenida en la sección `config` del mensaje.
- Validar los valores recibidos antes de modificar la configuración.
- Actualizar en tiempo real los parámetros de operación sin reiniciar el programa.
- Continuar monitoreando el entorno mientras permanece atenta a nuevas configuraciones remotas.

El comportamiento general de la estación puede representarse mediante el siguiente diagrama:

```text
                 Cliente MQTT
           (Monitoreo y Configuración)
                    ▲           │
                    │           │
     Publicación    │           │ JSON
                    │           ▼
               Broker MQTT (EMQX)
                    ▲
                    │
                    │
                  ESP32
          ┌─────────┴─────────┐
          ▼                   ▼
       Sensor DHT22       Pantalla OLED
```

Con este incremento, el proyecto integrador da un paso importante hacia la construcción de un sistema IoT distribuido, ya que las estaciones ambientales pueden ser administradas de forma remota sin necesidad de intervención física sobre cada dispositivo.

Además, la utilización de MQTT como mecanismo de comunicación bidireccional y el empleo de mensajes en formato JSON sientan las bases para incorporar funcionalidades más avanzadas, como la sincronización de configuraciones entre múltiples estaciones, el envío de comandos de actuación y la integración con plataformas de almacenamiento y visualización de datos.

En el siguiente tema, las estaciones ambientales dejarán de depender únicamente del broker MQTT para el intercambio de información y comenzarán a integrarse con una base de datos en la nube mediante Supabase. Esto permitirá almacenar históricamente las mediciones, consultar información desde aplicaciones web y construir un tablero de monitoreo centralizado para toda la red distribuida de estaciones ambientales.
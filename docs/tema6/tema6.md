# Aplicación IoT en Streamlit: Suscriptor MQTT, Panel Web y Almacenamiento en Supabase

## Introducción

En el tema anterior se configuró una base de datos en Supabase para almacenar las mediciones generadas por las estaciones ambientales IoT. Con ello, el proyecto ya cuenta con una infraestructura en la nube capaz de conservar el historial de datos.

En este tema se desarrollará una aplicación web utilizando **Streamlit**, un framework de Python que permite construir interfaces web de forma rápida y sencilla sin necesidad de conocimientos avanzados de desarrollo web.

La aplicación tendrá tres funciones principales:

- Suscribirse al broker MQTT para recibir las mediciones publicadas por la estación IoT.
- Mostrar la información recibida en un panel web interactivo.
- Almacenar automáticamente las mediciones en la base de datos de Supabase.

La arquitectura implementada será la siguiente:

```text
ESP32 → MQTT → Streamlit → Supabase
```

De esta manera, la ESP32 permanecerá como un nodo IoT ligero encargado únicamente de adquirir y publicar las mediciones, mientras que la aplicación desarrollada en Streamlit concentrará las tareas de recepción, visualización y almacenamiento de la información.

A diferencia de otros entornos de desarrollo web, Streamlit permite crear aplicaciones completas escribiendo únicamente código en Python, lo que lo convierte en una excelente herramienta para desarrollar paneles de monitoreo, aplicaciones científicas y sistemas IoT.

En este tema la aplicación se construirá de manera incremental. En cada paso se incorporará una nueva funcionalidad hasta obtener un sistema capaz de comunicarse con las estaciones IoT, visualizar las mediciones y generar el historial de datos en Supabase.

---

# Objetivo

Desarrollar una aplicación web en Streamlit capaz de recibir las mediciones publicadas por las estaciones IoT mediante MQTT, mostrarlas en un panel interactivo y almacenarlas automáticamente en una base de datos Supabase.

---

# Competencias a desarrollar

- Comprender la arquitectura de una aplicación IoT basada en MQTT y Streamlit.
- Crear aplicaciones web utilizando Streamlit.
- Configurar un cliente MQTT en Python.
- Mostrar información en tiempo real mediante una interfaz web.
- Integrar aplicaciones Python con Supabase utilizando su API REST.
- Construir un sistema IoT completo de adquisición, visualización y almacenamiento de datos.

---

# Material requerido

- Computadora con acceso a Internet.
- Python 3.10 o superior instalado.
- Visual Studio Code.
- Proyecto desarrollado durante los temas anteriores.
- Broker MQTT configurado.
- Proyecto de Supabase creado en el tema anterior.

---

# Desarrollo

## Paso 1. Crear el proyecto de Streamlit

Antes de comenzar a desarrollar la aplicación, se preparará la estructura del proyecto donde se almacenarán todos los archivos necesarios.

Durante las siguientes prácticas esta carpeta irá incorporando nuevos archivos conforme se agreguen funcionalidades a la aplicación.

---

### 1.1 Crear la carpeta del proyecto

En la computadora, cree una carpeta con el siguiente nombre:

```text
station_dashboard
```

Esta carpeta contendrá todos los archivos relacionados con la aplicación web.

---

### 1.2 Abrir la carpeta en Visual Studio Code

Abra **Visual Studio Code** y seleccione:

```text
File → Open Folder
```

Seleccione la carpeta creada:

```text
station_dashboard
```

Una vez abierta, Visual Studio Code mostrará la carpeta vacía en el panel lateral izquierdo.

---

### 1.3 Crear el archivo principal

Dentro de la carpeta `station_dashboard`, cree un archivo llamado:

```text
app.py
```

Este archivo contendrá el programa principal de la aplicación Streamlit.

La estructura del proyecto será inicialmente la siguiente:

```text
station_dashboard/
│
└── app.py
```

---

### 1.4 Crear el archivo de dependencias

Dentro de la misma carpeta cree un archivo llamado:

```text
requirements.txt
```

En este archivo se registrarán las bibliotecas necesarias para ejecutar la aplicación.

Después de crearlo, la estructura del proyecto será:

```text
station_dashboard/
│
├── app.py
└── requirements.txt
```

> **Nota:** Aunque durante este primer paso el archivo `requirements.txt` permanecerá vacío, más adelante se utilizará para registrar todas las dependencias del proyecto, facilitando su instalación y distribución.

---

## Resultado esperado

Al finalizar este paso se deberá contar con:

- Una carpeta llamada `station_dashboard`.
- Un archivo principal llamado `app.py`.
- Un archivo de dependencias llamado `requirements.txt`.
- El proyecto abierto en Visual Studio Code.

En el siguiente paso se instalarán las bibliotecas necesarias y se ejecutará la primera aplicación desarrollada con Streamlit.
# ⚡ Proyectos MicroPython en ESP32

Este repositorio contiene una colección de proyectos sencillos para el microcontrolador ESP32 utilizando el firmware MicroPython. El desarrollo se realiza utilizando Visual Studio Code (VS Code) y herramientas de línea de comandos para la carga del código.

## 🛠️ Herramientas y Configuración Inicial

Para comenzar a trabajar con el ESP32 y MicroPython, necesitas configurar tu entorno de desarrollo.

### 1. Requisitos de Hardware

- Placa ESP32 (cualquier modelo: DevKit, Wemos, etc.)
- Cable USB adecuado para la placa

### 2. Instalación de Herramientas de Software

Instala las siguientes herramientas en tu PC:

| Herramienta | Propósito | Instalación (Python) |
|-------------|-----------|---------------------|
| Python | Lenguaje base para MicroPython y las herramientas | [Descargar Python 3+](https://www.python.org/downloads/) |
| VS Code | Editor de código principal | [Descargar VS Code](https://code.visualstudio.com/) |
| ampy | Herramienta de línea de comandos para subir archivos y ejecutar scripts | `pip install adafruit-ampy` |
| esptool | Utilidad oficial de Espressif para flashear el firmware | `pip install esptool` |

#### 📹 Videos Tutoriales

- **Instalar Python y ampy:** [Ver tutorial en YouTube](https://www.youtube.com/watch?v=01SMgraNqUM&list=PL-Hb9zZP9qC6MBRJEdyIqE_rZbZOzxIpK&index=10)
- **Uso de ampy:** [Ver tutorial en YouTube](https://www.youtube.com/watch?v=y3uBKBWvh7Y&list=PL-Hb9zZP9qC6MBRJEdyIqE_rZbZOzxIpK&index=11)

#### Origen e Instalación de esptool.py

`esptool.py` es la herramienta de línea de comandos oficial, de código abierto, desarrollada por Espressif Systems (los fabricantes del ESP32). Su propósito principal es interactuar a bajo nivel con el bootloader del chip para flashear (escribir) firmware (como MicroPython) y manipular la memoria flash.

Se instala fácilmente a través del gestor de paquetes de Python, pip:

```bash
pip install esptool
```

### 3. Identificación del Puerto Serial (COM)

Antes de usar ampy o esptool, debes saber a qué puerto serial se ha conectado tu ESP32.

#### 🔎 En Windows

1. Abre el **Administrador de Dispositivos** (presiona `Windows + X` y selecciónalo)
2. Busca y expande la sección **Puertos (COM y LPT)**
3. Conecta tu placa ESP32 y observa la nueva entrada que aparece (ej. `Silicon Labs CP210x... (COM5)`)
4. El número entre paréntesis (ej. `COM5`) es el que usarás como `COMX` en tus comandos

#### 💻 En Linux/macOS

Ejecuta el siguiente comando en la terminal para listar los dispositivos:

```bash
ls /dev/tty.*
# o
ls /dev/ttyUSB*
```

### 4. Flasheo de MicroPython en el ESP32

Antes de subir código, el ESP32 debe tener el firmware de MicroPython.

1. **Descarga el firmware:** Obtén el archivo `.bin` para tu ESP32 desde el [sitio oficial de MicroPython](https://micropython.org/download/esp32/)

2. **Flashea el firmware:** Utiliza `esptool.py` (reemplaza `COMX` por tu puerto y `firmware.bin` por tu archivo):

```bash
# ⚠️ Opcional: Borrar completamente la memoria antes de flashear
esptool.py --port COMX erase_flash 

# Flashear el firmware
esptool.py --port COMX --baud 460800 write_flash --flash_size=detect 0 firmware.bin
```

## 💻 Carga y Ejecución de Código con ampy

Una vez que MicroPython está instalado, usa `ampy` para interactuar con el dispositivo.

> **NOTA:** Reemplaza `COMX` por el puerto serial de tu ESP32 (ej. `COM5`).

### 1. Subir un Archivo (Persistente)

Para enviar tu código local (`tu_script.py`) a la memoria flash del ESP32:

```bash
ampy -p COMX put tu_script.py
```

### 2. Ejecutar un Archivo (Temporal para Prueba)

Para ejecutar un script directamente sin guardarlo en la flash:

```bash
ampy -p COMX run tu_script.py
```

## 📂 Proyectos Desarrollados

### 1. Proyecto: LED Intermitente (Blink)

El programa fundamental para verificar que el código se ejecuta. Utiliza el pin GPIO 2 (común para el LED integrado).

**Archivo sugerido:** `main.py`

```python
# main.py
from machine import Pin
import time

# Configuración: Pin 2 para el LED integrado
LED_PIN = 2
led = Pin(LED_PIN, Pin.OUT)

print("Programa de LED intermitente iniciado en el ESP32.")

while True:
    # Encender y esperar
    led.value(1)
    print("LED ENCENDIDO")
    time.sleep(0.5)  
    
    # Apagar y esperar
    led.value(0)
    print("LED APAGADO")
    time.sleep(0.5)
```

### 2. Proyecto: Cliente Wi-Fi (Modo STA)

Conecta el ESP32 a una red Wi-Fi existente (Modo Station/Cliente).

**Archivo sugerido:** `wifi_sta.py`

```python
# wifi_sta.py
from machine import Pin
import network
import time

# --- CREDENCIALES (MODIFICAR) ---
SSID = 'Bazinga'         
PASSWORD = 'Bazinga321'  

# --- CONEXIÓN ---
wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(SSID, PASSWORD) 

max_intentos = 10
intento = 0
print(f"Intentando conectar a la red '{SSID}'...")
while not wlan.isconnected() and intento < max_intentos:
    print(f"Esperando conexion... ({intento + 1}/{max_intentos})")
    time.sleep(1)
    intento += 1

# --- RESULTADO FINAL ---
if wlan.isconnected():
    ip_config = wlan.ifconfig()
    print("\nConexion Wi-Fi establecida con exito")
    print(f"IP asignada: {ip_config[0]}")
else:
    print("\nFallo al conectar. Verifique SSID y contrasenia.")
    wlan.active(False)
```

### 3. Proyecto: Punto de Acceso Wi-Fi (Modo AP)

Configura el ESP32 para crear su propia red Wi-Fi (Modo Access Point). La clave para la visibilidad es configurar `channel=6`.

**Archivo sugerido:** `wifi_ap.py`

```python
# wifi_ap.py
import network
import time

# --- CONFIGURACIÓN DEL AP ---
AP_SSID = 'ESP32_PuntoDeAcceso' 
AP_PASSWORD = 'password123' 
AP_IP = '192.168.4.1'
AP_SUBNET = '255.255.255.0'
AP_GATEWAY = '192.168.4.1'

ap = network.WLAN(network.AP_IF)
ap.active(False) 
time.sleep(1) 
ap.active(True) 

# Configuración con 'hidden=False' y 'channel=6' para asegurar la visibilidad
ap.config(essid=AP_SSID, password=AP_PASSWORD, hidden=False, channel=6) 
ap.ifconfig((AP_IP, AP_SUBNET, AP_GATEWAY, AP_IP)) 

# --- BUCLE PRINCIPAL Y MONITOREO ---
if ap.active():
    print("---------------------------------------")
    print(f"AP configurado con exito.")
    print(f"  SSID: {ap.config('essid')}")
    print(f"  IP del AP: {ap.ifconfig()[0]}")
    print("---------------------------------------")
else:
    print("ERROR: No se pudo activar el modo Access Point.")
    
while True:
    clientes = ap.status('stations')
    if clientes:
        print(f"Clientes conectados: {len(clientes)}")
    else:
        print("Esperando cliente...")
    time.sleep(5)
```

## 📝 Notas Adicionales

- Asegúrate de que el ESP32 esté correctamente conectado y que el driver USB esté instalado
- Cierra cualquier programa de terminal serial (como Thonny o PuTTY) antes de usar `ampy`, ya que bloquean el puerto COM
- Si tienes problemas de conexión, prueba reducir la velocidad de baudios a 115200

## 📄 Licencia

Este proyecto está disponible para uso educativo y de aprendizaje.

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Por favor, abre un issue o pull request para sugerencias y mejoras.

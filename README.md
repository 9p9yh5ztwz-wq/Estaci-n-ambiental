# Estaci-n-ambiental
Prototipo IoT para monitoreo ambiental – Taller de Diseño en Ingeniería
Estación Ambiental IoT

Proyecto Taller de Diseño en Ingeniería.

Este proyecto consiste en una estación ambiental de bajo costo diseñada para medir variables del entorno de manera local y visualizar los datos mediante una interfaz web.

El sistema usa un Arduino UNO para adquirir las mediciones de los sensores y un ESP32-S3 para recibir los datos, generar una red Wi-Fi local y mostrarlos en una página web accesible desde un celular, tablet o computador.

Arquitectura del sistema

DHT11 ───────┐
MQ-9 ────────┼──> Arduino UNO ── UART ──> ESP32-S3 ── Wi-Fi ──> Interfaz web
DSM501A ─────┘

Componentes principales

* Arduino UNO
* ESP32-S3-N16R8
* DHT11
* MQ-9
* DSM501A
* Protoboard
* Resistencias 220V, 10V, 4.5V
* Cables Dupont
* Fuente USB / powerbank de 5 V
* LED

Sensores:
|DHT11

Mide:
* Temperatura en °C
* Humedad relativa en %

Conexión:
S   -> D4 Arduino
+   -> 5V
-   -> GND

|MQ-9

Sensor utilizado para detectar variaciones asociadas principalmente a monóxido de carbono y gases combustibles.

Conexión:
AO  -> A0 Arduino
VCC -> 5V
GND -> GND
DO  -> Sin conectar

La lectura utilizada actualmente corresponde al valor ADC del Arduino entre 0 y 1023.
El sensor todavía no se encuentra calibrado, por lo que estos valores no deben interpretarse directamente como ppm.

|DSM501A
Sensor de partículas basado en pulsos digitales.

Conexión utilizada:
Pin 3 VCC   -> 5V
Pin 4 Vout1 -> D7 Arduino
Pin 5 GND   -> GND

El sistema calcula el porcentaje de tiempo durante el cual la señal permanece en nivel bajo dentro de una ventana de 30 segundos.
Actualmente esta lectura se considera experimental y todavía está en proceso de validación.

Comunicación Arduino - ESP32

El Arduino envía los datos al ESP32 mediante comunicación serial UART.

Arduino D11 TX -> divisor de voltaje -> GPIO17 RX ESP32
ESP32 GPIO18 TX -> Arduino D10 RX
Arduino GND -> ESP32 GND

Se utiliza un divisor de voltaje en la señal Arduino -> ESP32 porque el Arduino trabaja con lógica de 5 V y el ESP32 con 3,3 V.

La comunicación funciona a:
9600 baudios


Formato de datos

El Arduino envía una trama CSV con el formato:
Temperatura,Humedad,MQ,DSM,EstadoDHT

Ejemplo:
23.4,55.0,95,12.453,1

Donde:
* 23.4 = temperatura en °C
* 55.0 = humedad relativa en %
* 95 = lectura ADC del MQ-9
* 12.453 = porcentaje de ocupación temporal del DSM501A
* 1 = lectura válida del DHT11


|ESP32 y Wi-Fi
El ESP32 funciona como punto de acceso Wi-Fi.

Red:
EstacionAmbiental

La interfaz puede abrirse desde:
http://192.168.4.1

El sistema funciona como una red local, por lo que no necesita conexión a Internet para mostrar los datos.

Interfaz web

La página web muestra:
* Temperatura
* Humedad
* Lectura analógica del MQ-9
* Ocupación temporal del DSM501A
* Estado de comunicación con el Arduino

Los datos se actualizan automáticamente aproximadamente una vez por segundo.


|Estructura del repositorio

Estacion-Ambiental/
│
├── Arduino_Estacion_Ambiental.ino
├── ESP32_Estacion_Ambiental.ino
├── README.md
│
└── documentacion/
    ├── diagrama_conexiones.png
    └── fotos_prototipo/

Librerías utilizadas

Arduino UNO

#include <SoftwareSerial.h>
#include <DHT.h>
#include <util/atomic.h>
#include <avr/interrupt.h>
#include <math.h>

ESP32

#include <WiFi.h>
#include <WebServer.h>
#include <math.h>
#include <stdlib.h>
#include <string.h>

Estado actual del prototipo

Actualmente se encuentra funcionando:

* Lectura de temperatura mediante DHT11
* Lectura de humedad mediante DHT11
* Lectura analógica del MQ-9
* Recepción de señal del DSM501A
* Comunicación entre Arduino UNO y ESP32
* Red Wi-Fi local
* Servidor web
* Visualización de datos en tiempo real

Limitaciones actuales

* El MQ-9 todavía no está calibrado para entregar concentraciones en ppm.
* El MQ-9 no mide CO₂.
* La lectura del DSM501A todavía está en proceso de validación.
* Los valores obtenidos corresponden a un prototipo experimental y no deben considerarse mediciones ambientales certificadas.

Próximos pasos

* Calibrar y validar los sensores.
* Mejorar la interpretación de los datos del DSM501A.
* Integrar el sistema dentro de una caja impresa en 3D.
* Alimentar el prototipo mediante una powerbank.
* Realizar pruebas en diferentes ubicaciones.
* Mejorar la robustez y presentación final del prototipo.

Objetivo final

Desarrollar una estación ambiental compacta, de bajo costo y replicable que permita obtener información localizada sobre las condiciones ambientales de distintos sectores.

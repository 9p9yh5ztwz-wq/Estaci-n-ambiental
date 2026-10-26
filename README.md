# Estación ambiental
Prototipo IoT para monitoreo ambiental – Taller de Diseño en Ingeniería
Estación Ambiental IoT

Proyecto Taller de Diseño en Ingeniería.

Descripción:

Este proyecto consiste en una estación ambiental de bajo costo diseñada para medir variables del entorno de manera local y visualizar los datos mediante una interfaz web.

El sistema utiliza un Arduino UNO para adquirir y procesar las mediciones de los sensores y un ESP32-S3 para recibir esos datos, generar una red Wi-Fi local y mostrarlos en una página web accesible desde un celular, tablet o computador.

Además, el sistema incorpora un LED como actuador visual, que responde automáticamente a un umbral experimental de humedad.

⸻

Arquitectura del sistema

DHT11 ───────┐
MQ-9 ────────┼──> Arduino UNO ── UART ──> ESP32-S3 ── Wi-Fi ──> Interfaz web
DSM501A ─────┘        │
                      └──> LED actuador


El flujo general del sistema es:

Sensores
   ↓
Arduino UNO
   ↓
Procesamiento de datos
   ↓
├── LED actuador
│
└── Comunicación UART
       ↓
     ESP32-S3
       ↓
     Wi-Fi local
       ↓
Página web de visualización

⸻

|Componentes principales:

* Arduino UNO
* ESP32-S3-N16R8
* DHT11
* MQ-9
* DSM501A
* LED
* Resistencia de 220 Ω
* Protoboard
* Resistencias de 10 kΩ y 4,7 kΩ para comunicación Arduino–ESP32
* Cables Dupont
* Cable USB
* Fuente USB o powerbank de 5 V

⸻

|Sensores utilizados:

DHT11:

Sensor utilizado para medir:

* Temperatura en °C
* Humedad relativa en %

Conexión

DHT11      Arduino UNO
S      ->  D4
+      ->  5V
-      ->  GND

El Arduino realiza una lectura aproximadamente cada 2,5 segundos.

⸻

MQ-9:

Sensor utilizado para detectar variaciones asociadas principalmente a:

* Monóxido de carbono (CO)
* Gases combustibles

Conexión

MQ-9       Arduino UNO
AO     ->  A0
VCC    ->  5V
GND    ->  GND
DO     ->  Sin conectar

Actualmente se utiliza la salida analógica AO.

El Arduino obtiene un valor ADC entre:

0 - 1023

Estas lecturas todavía no están calibradas, por lo que no se interpretan directamente como concentraciones en ppm.

El MQ-9 no mide CO₂.

⸻

DSM501A:

Sensor utilizado para detectar partículas mediante pulsos digitales.

Conexión

DSM501A        Arduino UNO
Pin 3 VCC   -> 5V
Pin 4 Vout1 -> D7
Pin 5 GND   -> GND

Los pines 1 y 2 no se utilizan.

El sistema mide cuánto tiempo permanece la señal del DSM501A en nivel LOW dentro de una ventana de:

30 segundos

A partir de esto se calcula un porcentaje de ocupación temporal.

Actualmente esta lectura se considera experimental y todavía está en proceso de validación.

⸻

LED actuador:

El sistema incorpora un LED conectado al Arduino UNO.

Conexión

Arduino D8
   |
Resistencia 220 Ω
   |
LED
   |
GND

Funcionamiento

Se utiliza un umbral experimental de humedad:

Humedad >= 60 % -> LED ENCENDIDO
Humedad < 60 %  -> LED APAGADO

El objetivo de este umbral es demostrar la integración:

Sensor -> Procesamiento -> Actuador

El 60 % se utiliza únicamente como criterio experimental para demostrar el funcionamiento del prototipo y no representa un nivel peligroso de humedad.

⸻

Distribución de alimentación:

La protoboard se utiliza para distribuir:

5V
GND

El Arduino UNO alimenta el riel positivo y negativo de la protoboard.

Desde estos rieles se alimentan:

* DHT11
* MQ-9
* DSM501A

Esquema:

Arduino 5V
   ↓
Riel positivo protoboard
   ├── DHT11
   ├── MQ-9
   └── DSM501A
Arduino GND
   ↓
Riel negativo protoboard
   ├── DHT11
   ├── MQ-9
   ├── DSM501A
   └── GND ESP32

⸻

Comunicación Arduino UNO - ESP32

El Arduino envía los datos obtenidos de los sensores al ESP32 mediante comunicación serial UART.

Pines utilizados

Arduino D11 TX
      |
Divisor de voltaje
      |
ESP32 GPIO17 RX

También se comparte:

Arduino GND -> ESP32 GND

El código actual utiliza:

Arduino D10 = RX
Arduino D11 = TX

y en el ESP32:

GPIO17 = RX
GPIO18 = TX

Actualmente la comunicación principal es unidireccional:

Arduino -> ESP32

⸻

Divisor de voltaje Arduino - ESP32

El Arduino UNO trabaja con lógica de 5 V y el ESP32 con lógica de 3,3 V.

Por esta razón se utiliza un divisor de voltaje en la señal:

Arduino D11
   |
 10 kΩ
   |
   +------> GPIO17 ESP32
   |
 10 kΩ
   |
 4,7 kΩ
   |
  GND

De esta manera se reduce el voltaje antes de ingresar al ESP32.

⸻

Formato de comunicación:

El Arduino genera una trama CSV con cinco campos:

Temperatura,Humedad,MQ,DSM,EstadoDHT

Ejemplo:

23.4,55.0,95,12.453,1

Donde:

23.4   -> Temperatura en °C
55.0   -> Humedad relativa en %
95     -> Lectura ADC del MQ-9
12.453 -> Ocupación temporal DSM501A
1      -> Lectura válida del DHT11

Si el DHT11 presenta un error, se envía:

EstadoDHT = 0

⸻

Código del Arduino UNO

El Arduino se encarga principalmente de:

1. Leer temperatura y humedad del DHT11.
2. Leer el valor analógico del MQ-9.
3. Medir los pulsos del DSM501A.
4. Controlar el LED actuador.
5. Crear la trama CSV.
6. Enviar los datos al ESP32.

⸻

Librerías utilizadas en Arduino

#include <SoftwareSerial.h>
#include <DHT.h>
#include <util/atomic.h>
#include <avr/interrupt.h>
#include <math.h>

SoftwareSerial:

Permite crear una comunicación serial adicional entre Arduino y ESP32 utilizando los pines D10 y D11.

DHT:

Permite realizar las lecturas de temperatura y humedad del sensor DHT11.

util/atomic.h:

Permite proteger variables utilizadas tanto por el programa principal como por las interrupciones.

avr/interrupt.h:

Permite utilizar interrupciones del microcontrolador ATmega328P.

math.h:

Se utiliza para funciones matemáticas y validación de datos.

⸻

Variables principales del Arduino

enlace

Comunicación serial con el ESP32.

dht

Objeto utilizado para controlar el sensor DHT11.

MQ_PIN

Pin analógico A0 utilizado por el MQ-9.

DSM_PIN

Pin digital D7 utilizado por el DSM501A.

LED_PIN

Pin digital D8 utilizado por el LED actuador.

UMBRAL_HUMEDAD

Umbral experimental de 60 % utilizado para encender el LED.

VENTANA_US

Define una ventana de 30 segundos para el análisis del DSM501A.

ocupacion

Guarda el porcentaje de tiempo que la señal del DSM501A permanece en LOW.

trama

Almacena la información que posteriormente se envía al ESP32.

⸻

Funciones principales del Arduino

setup()

Inicializa:

* Monitor Serie
* Comunicación con ESP32
* DHT11
* DSM501A
* LED actuador
* Timer1

⸻

ISR(TIMER1_COMPA_vect)

Rutina de interrupción utilizada para monitorear la señal del DSM501A.

Permite medir los pulsos sin bloquear el funcionamiento del resto del sistema.

⸻

loop()

Realiza continuamente:

Lectura sensores
      ↓
Control LED
      ↓
Procesamiento datos
      ↓
Creación trama CSV
      ↓
Envío al ESP32

El programa evita utilizar delay() para mantener el sistema funcionando de manera continua.

⸻

ESP32-S3

El ESP32 se utiliza principalmente como módulo de conectividad.

Sus funciones son:

* Recibir los datos enviados por Arduino
* Validar las tramas recibidas
* Crear una red Wi-Fi local
* Crear un servidor web
* Mostrar los datos mediante una interfaz

⸻

|Librerías utilizadas en ESP32

#include <WiFi.h>
#include <WebServer.h>
#include <math.h>
#include <stdlib.h>
#include <string.h>

WiFi:

Permite utilizar la conectividad Wi-Fi del ESP32.

WebServer:

Permite crear el servidor HTTP utilizado por la interfaz web.

math.h:

Permite validar valores numéricos.

stdlib.h:

Permite convertir cadenas de texto a valores numéricos.

string.h:

Permite trabajar con cadenas de caracteres.

⸻

|Red Wi-Fi

El ESP32 funciona como punto de acceso.

Nombre de la red

EstacionAmbiental

Dirección local

http://192.168.4.1

La interfaz funciona dentro de una red local.

Por lo tanto:

NO necesita conexión a Internet.

Un celular, tablet o computador puede conectarse directamente a la red generada por el ESP32.

⸻

|Interfaz web

La página muestra:

* Temperatura
* Humedad
* Lectura ADC del MQ-9
* Ocupación temporal del DSM501A
* Estado de comunicación con Arduino
* Estado de lectura del DHT11

Los datos se actualizan automáticamente aproximadamente cada segundo.

⸻

|Validación de datos

Antes de mostrar las mediciones, el ESP32 verifica que la trama enviada por Arduino tenga el formato correcto.

Se validan los siguientes rangos:

Temperatura: -40 a 80 °C
Humedad: 0 a 100 %
MQ-9: 0 a 1023
DSM501A: -1 o entre 0 y 100 %

Si una trama está incompleta o contiene datos inválidos, se descarta.

⸻

|Estructura del sistema

SENSORES
│
├── DHT11
├── MQ-9
└── DSM501A
       ↓
ARDUINO UNO
       │
       ├── LED actuador
       │
       └── UART
            ↓
         ESP32-S3
            ↓
          Wi-Fi
            ↓
       Página web
            ↓
Celular / Tablet / Computador

⸻

|Estado actual del prototipo

Actualmente el sistema permite:

* Leer temperatura con DHT11
* Leer humedad con DHT11
* Obtener lectura analógica del MQ-9
* Recibir pulsos del DSM501A
* Controlar automáticamente un LED
* Comunicar Arduino UNO con ESP32
* Crear una red Wi-Fi local
* Ejecutar un servidor web
* Mostrar datos en una interfaz
* Actualizar las mediciones automáticamente

⸻

|Limitaciones actuales

El prototipo todavía tiene algunas limitaciones:

* El MQ-9 no está calibrado para entregar concentraciones en ppm.
* El MQ-9 no mide CO₂.
* La lectura del DSM501A todavía está en proceso de validación.
* Las mediciones actuales son experimentales.
* El sistema no corresponde a una estación ambiental certificada.
* El umbral del LED es experimental.

⸻

|Encapsulado propuesto

El sistema está pensado para integrarse dentro de una caja compacta impresa en 3D.

Características propuestas:

* Color verde oscuro
* Montaje en muro o poste
* Protección frente al entorno
* Ventilación para los sensores
* Acceso para mantenimiento
* Espacio para Arduino UNO
* Espacio para ESP32
* Espacio para sensores
* Espacio para powerbank

Dimensiones conceptuales aproximadas:

15 cm x 23 cm x 6 cm

⸻

Alimentación autónoma

Para una futura implementación se propone utilizar una powerbank de aproximadamente:

10.000 mAh
5 V

Esto permitiría que el sistema funcione sin estar conectado permanentemente a un computador.

⸻

|Próximos pasos

* Validar correctamente el DSM501A
* Calibrar el MQ-9
* Mejorar la estabilidad de las mediciones
* Integrar todo dentro del encapsulado
* Alimentar el sistema mediante powerbank
* Realizar pruebas en distintos lugares
* Comparar zonas con diferente cobertura vegetal
* Mejorar la interfaz web
* Evaluar nuevos sensores en futuras iteraciones

⸻

|Objetivo final

El objetivo final es desarrollar una estación ambiental:

* Compacta
* De bajo costo
* Replicable
* Fácil de instalar
* Capaz de entregar información ambiental localizada

La propuesta busca permitir la comparación de condiciones ambientales entre distintos sectores y apoyar futuras decisiones relacionadas con calidad ambiental y planificación urbana.

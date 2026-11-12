# Overview

Esta biblioteca proporciona un conjunto de funciones y capas de abstracción en C++ para crear y procesar paquetes del protocolo DYNAMIXEL con el fin de gestionar servomotores inteligentes DYNAMIXEL en tiempo real. Está diseñada como un controlador modular con soporte nativo para ROS 2 Jazzy (mediante nodos C++ y tópicos estándar) así como compatibilidad para Python mediante wrappers (pybind11).
---

## Especificaciones de Soporte

Supported DYNAMIXEL:

**Exclusivamente la serie AX** 

|                    Modelos |
| :---:|
|                 **AX-12A / AX-12+**, **AX-12W** y **AX-18A** |


## Protocolos soportados

**Protocolo DYNAMIXEL 1.0.**

## Dispositivos Soportados

* Convertidores USB a Serie/RS485

* U2D2
* USB2AX
* Adaptadores basados en **FTDI** (ej. `/dev/ttyUSB0`).


## Supported Operating Systems

**Linux** (probado principalmente en entornos Ubuntu / Debian).
**ROS 2 Jazzy** (Integración nativa mediante ament_cmake y colcon).


## Supported Languages
* **C++** (Lenguaje base del controlador).
* **Python** (Wrappers con pybind11)



### Funcionalidades clave: Telemetría y Monitoreo en Tiempo Real
#### 1. Control Articular y de Rotación Continua

* Modo Articulación (Joint Mode): Control preciso de ángulos objetivo en radianes (rango de 0 a 300) con interpolación por tiempo o disparo a velocidad física máxima (Moving Speed = 1023 para tiempos menores o iguales a 0.1 s).
* Modo Rueda (Wheel Mode): Soporte para rotación continua infinita en sentido horario (CW) y antihorario (CCW) mediante control directo de velocidad angular.
* Estabilidad y Escalabilidad: Declaración explícita de los IDs de los motores en el bus RS-485 para prevenir desbordamientos de pila (stack smashing), permitiendo escalar de 2 a 3, 5 o más articulaciones de forma segura.

#### 2. Interfaz ROS 2 y Telemetría en Tiempo Real
A través del nodo arm_ax_node, el sistema conecta el hardware directamente con la red de ROS 2 mediante mensajes estándar (sensor_msgs/msg/JointState):

* Telemetría (/joint_states a 10 Hz): Lectura continua de posiciones actuales (convertidas de valores raw 0–1023 a radianes), además de acceso a parámetros de diagnóstico como velocidad angular, voltaje de entrada, temperatura interna y estado de movimiento (isMoving).

* Comandos Dinámicos (/joint_commands): Ejecución en tiempo real de posiciones objetivo y perfiles de velocidad sin necesidad de recompilar el código fuente C++.


#### Para mas detalle en sus funcionalidades consulte
[Arquitectura por capas de la libreria](<Arquitectura por capas.md>)


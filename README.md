# Library-Dynamixel (`dynamixel-ctrl`)

Controlador modular en C++ con integración nativa para **ROS 2 Jazzy** y soporte para wrappers en Python (`pybind11`), diseñado para la gestión en tiempo real de servomotores inteligentes **Dynamixel serie AX (AX-12A / AX-18A)** sobre **Protocolo 1.0 a 1 Mbps**.

---

## Índice de Documentación

Explora las guías técnicas detalladas según la etapa de configuración o desarrollo:

1. **[Overview](Overview.md)** — Resumen general, especificaciones de hardware soportado (serie AX), middleware y arquitectura de comunicaciones.
2. **[Arquitectura por Capas](<Arquitectura por capas.md>)** — Estructura interna del controlador (`DynamixelSDK` $\rightarrow$ `DynamixelManager` $\rightarrow$ `armAX` $\rightarrow$ `ArmAXNode`).
3. **[Device Setup](<Device Setup.md>)** — Detección e identificación del puerto USB-Serial (`/dev/ttyUSB0`), permisos (`dialout`) y configuración específica para **NVIDIA Jetson Orin**.
4. **[Virtual Environment](Virtual_Environment.md)** — Gestión de entornos Python y requisitos del sistema nativo (Python 3.12) para evitar conflictos de compilación en ROS 2 Jazzy.
5. **[Download](Download.md) & [Download SDK](<Download SDK.md>)** — Descarga del repositorio, dependencias del sistema y preparación de cabeceras del SDK de Dynamixel.
6. **[Library Setup](<Library Setup.md>)** — Implementación del código C++ (`armAX`, nodo ROS 2), configuración de IDs de motores (2, 3 o 5), modos de operación (*Joint Mode* / *Wheel Mode*), `CMakeLists.txt`, compilación con `colcon` y ejecución en 3 terminales.
7. **[Basic Read / Write](<Basic Read_write.md>)** — Ejemplo *standalone* en C++ puro (sin ROS 2) para lectura y escritura directa sobre el bus RS-485.
8. **[API Reference](API_Reference.md)** — Referencia rápida de clases, métodos y parámetros disponibles en la librería.

---

## Resumen de Especificaciones

* **Servomotores Soportados:** Exclusivamente serie AX (**AX-12A / AX-12+**, **AX-12W**, **AX-18A**).
* **Protocolo y Velocidad:** Protocolo DYNAMIXEL 1.0 a **1,000,000 bps (1 Mbps)**.
* **Interfaces Hardware:** U2D2, USB2AX y adaptadores USB-Serial basados en FTDI / CH340.
* **Sistemas Operativos:** Ubuntu 24.04 LTS (x86_64) y **NVIDIA Jetson Orin** (JetPack / ARM64).
* **Middleware y Lenguajes:** **ROS 2 Jazzy** con núcleo en **C++** (principal) y bindings opcionales en **Python** (`pybind11`).

---

## Inicio Rápido (ROS 2 Jazzy)

Una vez compilado el workspace siguiendo la guía de **[Library Setup](<Library Setup.md>)**, abre 3 terminales para operar el brazo robótico:

* **Terminal 1 — Nodo Driver Hardware:**
  ```bash
  source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
  sudo chmod 666 /dev/ttyUSB0
  ros2 run arm_ax_cpp_pkg arm_ax_node
  ```

  * **Terminal 2 — Monitoreo de Telemetría (/joint_states):**
```bash
source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
ros2 topic echo /joint_states
  ```

  * **Terminal 3 — Envío de Comandos (/joint_commands):**
```bash
source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
ros2 topic pub --once /joint_commands sensor_msgs/msg/JointState "{position: [0.0, 0.5], velocity: [0.1]}"
```
*(Nota: Si decides mover todos tus archivos `.md` a una subcarpeta `docs/`, únicamente antepón `docs/` a cada enlace en el índice, por ejemplo: `[Library Setup](<docs/Library Setup.md>)`).*

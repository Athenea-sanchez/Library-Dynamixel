# Ruta de Aprendizaje e Integración Paso a Paso (Linux x86_64 / Jetson Orin)

Sigue los documentos de la carpeta `docs/` en este orden estricto para configurar hardware, compilar la librería y ejecutar el controlador en **ROS 2 Jazzy**:

---

### 1. [`Overview.md`](Overview.md) (Visión General y Arquitectura)
* **Objetivo:** Comprender las capacidades de la librería `dynamixel-ctrl`, los modelos soportados de la **Serie AX** (AX-12A / AX-18A) y la arquitectura para **PC (x86_64)** y **NVIDIA Jetson Orin** mediante conversor USB-FTDI / U2D2 (`/dev/ttyUSB0`) a 1 Mbps.
* **Lo que harás:** Verificar que tu hardware y distribución (Ubuntu 24.04 + ROS 2 Jazzy) cumplen con los requisitos del sistema.

---

### 2. [`Virtual_Environment.md`](Virtual_Environment.md) (Gestión del Entorno Python)
* **Objetivo:** Asegurar que el sistema utilice el intérprete nativo de Python 3.12 requerido por ROS 2 Jazzy.
* **Lo que harás:** Desactivar cualquier entorno virtual (`deactivate`) para evitar conflictos de paquetes (`empy`, `catkin_pkg`) al compilar con `colcon`.

---

### 3. [`Download.md`](Download.md) y [`Download SDK.md`](<Download SDK.md>) (Descarga e Integración del SDK)
* **Objetivo:** Obtener el código fuente de `dynamixel-ctrl` y preparar las cabeceras del **DYNAMIXEL SDK** (Protocolo 1.0).
* **Lo que harás:**
  1. Clonar el repositorio principal con `git clone`.
  2. Verificar que en `src/brazo/armAX/lib/` se encuentra la librería compartida (`libdxl_x64_cpp.so`) y que las cabeceras `dynamixel_sdk/` estén ubicadas dentro de `include/`.

---

### 4. [`Device Setup.md`](<Device Setup.md>) (Configuración del Puerto y Hardware)
* **Objetivo:** Preparar los servomotores, la alimentación y los permisos del sistema operativo.
* **Lo que harás:**
  1. Alimentar el bus Dynamixel con fuente externa de 12V DC y conectar el adaptador USB-Serial (ver diagrama en [`conexion.jpg`](conexion.jpg)).
  2. Otorgar permisos de lectura/escritura al puerto `/dev/ttyUSB0` (`sudo usermod -aG dialout $USER` y `sudo chmod 666 /dev/ttyUSB0`).
  3. *(En Jetson Orin)* Verificar la carga de módulos kernel (`ftdi_sio` / `ch341`) y modo de energía (`nvpmodel -m 0`).

---

### 5. [`Library Setup.md`](<Library Setup.md>) (Compilación y Despliegue en ROS 2 Jazzy)
* **Objetivo:** Configurar los IDs de los motores activos, compilar el paquete C++ (`arm_ax_cpp_pkg`) y operar el brazo en ROS 2.
* **Lo que harás:**
  1. Copiar las cabeceras `dynamixel_sdk/` y los binarios `.so`/`.a` hacia `src/brazo/armAX_ws/src/arm_ax_cpp_pkg/`.
  2. Declarar los IDs de los motores activos en `src/armAX.cpp` (evitando *stack smashing*) y elegir entre **Modo Articulación** (`setJointMode`) o **Modo Rueda** (`setWheelMode`).
  3. Compilar el workspace:
     ```bash
     cd ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws
     rm -rf build/arm_ax_cpp_pkg install/arm_ax_cpp_pkg
     colcon build --packages-select arm_ax_cpp_pkg --symlink-install
     ```
  4. Ejecutar el flujo operativo en **3 terminales**:
     * **Terminal 1 (Driver):** `ros2 run arm_ax_cpp_pkg arm_ax_node`
     * **Terminal 2 (Telemetría):** `ros2 topic echo /joint_states`
     * **Terminal 3 (Comandos):** `ros2 topic pub --once /joint_commands sensor_msgs/msg/JointState "{position: [0.0, 0.5], velocity: [0.1]}"`

---

### 6. [`Basic Read_write.md`](<Basic Read_write.md>) (Tutorial Standalone C++ sin ROS 2)
* **Objetivo:** Probar el control directo de un servomotor individual en C++ puro mediante `DynamixelManager` y `DynamixelAXControl`.
* **Lo que harás:** Compilar `my_read_write.cpp` con `g++` enlazando `-lDynamixelAXControl -lDynamixelManager -ldxl_x64_cpp` y ejecutarlo usando `LD_LIBRARY_PATH`.

---

### 7. [`Arquitectura por capas.md`](<Arquitectura por capas.md>) y [`API_Reference.md`](API_Reference.md) (Referencia Técnica)
* **Objetivo:** Profundizar en el diseño interno de la librería y las firmas de sus métodos públicos.
* **Lo que harás:** Consultar el diagrama de capas (`DynamixelSDK` $\rightarrow$ `DynamixelManager` $\rightarrow$ `armAX` $\rightarrow$ `ArmAXNode`) y la documentación detallada de cada función disponible.

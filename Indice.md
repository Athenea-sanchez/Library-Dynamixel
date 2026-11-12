# Ruta de Aprendizaje e Integración Paso a Paso para PC (Linux x86_64)

Sigue los documentos de la carpeta `docs/` en este orden estricto para configurar el entorno en tu PC:

---

### 1. 🗋 `Overview.md` (Visión General)
* **Objetivo:** Comprender las capacidades de la librería `dynamixel-ctrl`, los modelos de la Serie AX (AX-12A / AX-18A) y la arquitectura para **PC (x86_64)** con comunicación por conversor USB-FTDI / U2D2 (`/dev/ttyUSB0`)[cite: 1].
* **Lo que harás:** Verificar que tu hardware cumple con los requisitos del sistema.

---

### 2. 🗋 `Download_SDK.md` (Descarga e Instalación del DYNAMIXEL SDK)
* **Objetivo:** Obtener e instalar las capas base oficiales del **DYNAMIXEL SDK** de ROBOTIS en tu PC.
* **Lo que harás:** 
  1. Descargar el SDK mediante Git (`git clone https://github.com/ROBOTIS-GIT/DynamixelSDK.git`), Releases o paquetes de ROS (`ros-$ROS_DISTRO-dynamixel-sdk`).
  2. Comprender la estructura modular del SDK para compilar y enlazar las librerías dinámicas (`.so`).

---

### 3. 🗋 `Download.md` (Descarga y Estructura x86_64 del Repositorio)
* **Objetivo:** Obtener el código fuente de la librería `dynamixel-ctrl` y entender el árbol de carpetas.
* **Lo que harás:**
  1. Clonar el repositorio principal con `git clone`.
  2. Verificar que en la carpeta `brazo/armAX/lib/` se encuentra la librería compartida compilada para PC: **`libdxl_x64_cpp.so`**[cite: 1].

---

### 4. 🗋 `Device_Setup.md` (Configuración del Puerto y Calibración)
* **Objetivo:** Preparar los servomotores y los permisos del sistema operativo.
* **Lo que harás:**
  1. Alimentar el bus con fuente externa de 12V DC.
  2. Conectar el adaptador USB-FTDI o U2D2 a la PC.
  3. Ejecutar `sudo usermod -aG dialout $USER` para dar permisos de lectura/escritura al puerto `/dev/ttyUSB0` sin requerir `sudo`.
  4. Asignar IDs únicas a cada motor con DYNAMIXEL Wizard 2.0 y configurar la velocidad de transmisión a 1,000,000 bps.

---

### 5. 🗋 `Library_Setup.md` (Compilación Nativa en PC)
* **Objetivo:** Compilar la librería base C++ (`armAX`) y los bindings de Python en x86_64.
* **Lo que harás:**
  1. Instalar prerrequisitos: `sudo apt install build-essential cmake git python3-dev python3-pip`.
  2. Compilar la librería base:
     ```bash
     cd brazo/armAX
     mkdir -p build && cd build
     cmake ..
     make -j$(nproc)
     ```
  3. Compilar el espacio de trabajo de ROS 2 (opcional si usas ROS 2 en PC):
     ```bash
     cd brazo/armAX_ws
     colcon build --symlink-install
     source install/setup.bash
     ```

---

### 6. 🗋 `Sync_Read_Write.md` (Tutorial C++ en PC)
* **Objetivo:** Crear y ejecutar tu primer programa C++ de control síncrono del brazo en PC.
* **Lo que harás:**
  1. Crear el archivo `my_sync_read_write.cpp` incluyendo `armAX.h` y `DynamixelManager.h`.
  2. Compilar el ejecutable enlazando la librería de PC `libdxl_x64_cpp.so`:
     ```bash
     g++ my_sync_read_write.cpp -o my_sync_read_write \
         -I../../armAX/include \
         -L../../armAX/lib \
         -larmAX -lDynamixelAXControl -lDynamixelManager -ldxl_x64_cpp -lpthread
     ```
  3. Ejecutar especificando la variable de entorno `LD_LIBRARY_PATH`:
     ```bash
     LD_LIBRARY_PATH=../../armAX/lib ./my_sync_read_write
     ```

---

### 7. 🗋 `ROS2_Integration.md` (Integración ROS 2 en PC)
* **Objetivo:** Ejecutar los nodos de control de alto nivel si utilizas ROS 2 en tu PC.
* **Lo que harás:**
  1. Ejecutar el nodo en C++: `ros2 run arm_ax_cpp_pkg arm_ax_cpp_node`.
  2. Probar el envío de comandos publicando en el tópico `/arm/set_positions` y monitorear telemetría en `/joint_states`.

---

### 🗋 `API_Reference.md` (Consulta Técnica)
* **Uso:** Manual de referencia permanente durante el desarrollo para consultar clases, métodos y parámetros de `DynamixelManager`, `DynamixelAXControl` y `armAX`.
# Library Setup: `dynamixel-ctrl`

Esta sección describe cómo instalar las dependencias, configurar el entorno virtual de Python 3.10 mediante un script automatizado, preparar los permisos del puerto serie y compilar tanto la librería C++/Python base como el espacio de trabajo de ROS 2 en sistemas Linux.

---

## 1. Detección e Identificación del Puerto USB

Identifica el puerto asignado al convertidor USB-Serial (ej. U2D2, FTDI, CH340):

```bash
ls /dev/ttyUSB* /dev/ttyACM* 2>/dev/null
```
*(Generalmente se asignará como `/dev/ttyUSB0` o `/dev/ttyACM0`).*

### Permisos del Puerto Serial USB
Otorga acceso de lectura y escritura al puerto para trabajar sin `sudo`:

```bash
# Otorgar permiso temporal al puerto
sudo chmod 666 /dev/ttyUSB0

# O bien, agregar tu usuario al grupo dialout (acceso permanente)
sudo usermod -aG dialout $USER
```
---
### Caso de Detección e Identificación del Puerto USB en ORIN

Conecta el convertidor USB-Serial (U2D2, FTDI o CH340) a uno de los puertos USB de la Jetson Orin e identifica la ruta asignada

``` bash 
ls /dev/ttyUSB* /dev/ttyACM* 2>/dev/null 
```

## 2. Código Fuente Integrado

### A. Librería Base C++

Gestiona la conexión y comunicación por el bus RS-485 a 1 Mbps. 
Declara explícitamente la lista segura de motores para prevenir errores de desbordamiento de pila (stack smashing) y administra el control de velocidad física cuando el tiempo enviado es igual o menor a 0.1s. 

*En este codigo se modifica el numero de motores y el modo de articulación*

#### Guía de Modificación de Motores y Modos de Operación
Para evitar hacer escaneos inestables en el bus RS-485 que provoquen fallos de memoria (stack smashing), los motores activos se declaran directamente en el constructor de src/armAX.cpp.

*Ejemplo para 5 motores*

```cpp
motors = {
    { 1 , DynamixelAXControl( this->manager , 1 ) },
    { 2 , DynamixelAXControl( this->manager , 2 ) },
    { 3 , DynamixelAXControl( this->manager , 3 ) },
    { 4 , DynamixelAXControl( this->manager , 4 ) },
    { 5 , DynamixelAXControl( this->manager , 5 ) }
};
```
#### Modo Articulación (Joint Mode) vs Modo Rueda (Wheel Mode)
* Modo Articulación (Joint Mode - Configuración actual): El servomotor busca alcanzar un ángulo objetivo en radianes (rango mecánico de 0° a 300°).

Al llegar a la posición deseada, el servo se detiene automáticamente.

* Modo Rueda (Wheel Mode - Rotación Continua):
Desactiva los límites angulares para que el servo actúe como motor de tracción y gire de forma continua (infinita).

Para activarlo, se cambia p.second.setJointMode(...) por p.second.setWheelMode() en src/armAX.cpp. En este modo, la función setSpeed() controla el sentido (0-1023 horario, 1024-2047 antihorario) y la velocidad continua de giro.


```bash
cat << 'EOF' > ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/src/arm_ax_cpp_pkg/src/armAX.cpp
#include "armAX_cpp_pkg/armAX.h"
#include <iostream>

armAX::armAX(DynamixelManager* puerto) : manager(puerto) {
    // -------------------------------------------------------------------------
    // Para agregar o quitar motores, edita este mapa de IDs (ej. IDs 1, 2, 3, etc, como se explico anteriormente).
    // -------------------------------------------------------------------------
    motors = {
        { 1 , DynamixelAXControl( this->manager , 1 ) },
        { 2 , DynamixelAXControl( this->manager , 2 ) }
    };
}

armAX::~armAX() {}

std::string armAX::get_messages() { return messages; }

// -------------------------------------------------------------------------
// CONEXIÓN Y CONFIGURACIÓN DE MODO DE OPERACIÓN
// -------------------------------------------------------------------------
bool armAX::connect(bool home) {
    messages.clear();

    for (auto& p : motors) {
        // Configuración predeterminada: MODO ARTICULACIÓN (Joint Mode: 0° a 300°)
        if (!p.second.setJointMode(1.5707963268, -1.5707963268)) {
            messages.append("[ERROR] Falló JointMode en motor ID: " + std::to_string(p.first) + "\n");
            return false;
        }
        
        // NOTA PARA MODO RUEDA (ROTACIÓN CONTINUA):
        // Si se requiere rotación infinita en lugar de ángulos fijos, reemplazar
        // la línea anterior por: p.second.setWheelMode();
    }
    return true;
}

bool armAX::pingServo(int id) {
    messages.clear();
    return manager->pingServo(id);
}

// -------------------------------------------------------------------------
// LECTURA DE POSICIONES (TELEMETRÍA RAW 0 - 1023)
// -------------------------------------------------------------------------
std::vector<float> armAX::getAllPosition() {
    messages.clear();
    std::vector<float> positions;
    for (auto& p : motors) {
        positions.push_back(p.second.getPosition());
        messages.append(p.second.get_message());
    }
    return positions;
}

// -------------------------------------------------------------------------
// ESCRITURA DE POSICIONES Y CONTROL DE VELOCIDAD
// -------------------------------------------------------------------------
bool armAX::setPositionArray(std::vector<int> ids, std::vector<float> positions, std::vector<float> times) {
    messages.clear();
    if ((ids.size() == positions.size()) && (times.size() == positions.size())) {
        for (size_t i = 0; i < ids.size(); i++) {
            int m = ids[i];
            if (motors.find(m) == motors.end()) return false;
            
            // Si el tiempo enviado es <= 0.1s, fuerza velocidad máxima física (Moving Speed = 1023)
            if (times[i] <= 0.1f) {
                motors.at(m).setSpeed(1023); 
                if (!motors.at(m).setPosition(positions[i])) {
                    messages.append(motors.at(m).get_message());
                    return false;
                }
            } else {
                if (!motors.at(m).setPositionInTime(positions[i], times[i])) {
                    messages.append(motors.at(m).get_message());
                    return false;
                }
            }
        }
        return true;
    }
    return false;
}
EOF
```

### Archivos de Compilación (CMakeLists.txt y package.xml)

#### CMake
* Para compilar el proyecto, este archivo le indica a CMake y g++ cómo generar el ejecutable vinculando los módulos de ROS 2, las cabeceras de C++ y la SDK de Dynamixel, registrando las librerías estáticas, enlazando las dependencias necesarias e instalando el binario final en el entorno local para su ejecución.

```bash
cat << 'EOF' > ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/src/arm_ax_cpp_pkg/CMakeLists.txt
cmake_minimum_required(VERSION 3.8)
project(arm_ax_cpp_pkg)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(arm_ax_interfaces REQUIRED)

set(LIBRARY_DIR ${PROJECT_SOURCE_DIR}/lib)

include_directories(
  include
  include/dynamixel_sdk
  ${rclcpp_INCLUDE_DIRS}
  ${sensor_msgs_INCLUDE_DIRS}
  ${arm_ax_interfaces_INCLUDE_DIRS}
)

link_directories(
  ${LIBRARY_DIR}
)

add_library(ax_manager STATIC IMPORTED)
set_target_properties(ax_manager PROPERTIES
  IMPORTED_LOCATION ${LIBRARY_DIR}/libDynamixelAXControl.a
)

add_library(dxl_manager STATIC IMPORTED)
set_target_properties(dxl_manager PROPERTIES
  IMPORTED_LOCATION ${LIBRARY_DIR}/libDynamixelManager.a
)

add_executable(arm_ax_node 
  src/arm_ax_cpp_node.cpp
  src/armAX.cpp
)

ament_target_dependencies(arm_ax_node
  rclcpp
  sensor_msgs
  arm_ax_interfaces
)

target_link_libraries(arm_ax_node
  ax_manager
  dxl_manager
  arm_ax_interfaces::arm_ax_interfaces__rosidl_typesupport_cpp
  dxl_x64_cpp
  pthread
)

install(TARGETS arm_ax_node DESTINATION lib/${PROJECT_NAME})
install(DIRECTORY include/ DESTINATION include)
install(DIRECTORY lib/ DESTINATION lib)

ament_package()
EOF
```
#### package.xml

* El archivo `package.xml` funciona como el manifiesto de dependencias que informa al gestor de ROS 2 sobre los requisitos del paquete, utilizando `buildtool_depend` para indicar el sistema de compilación `ament_cmake` y `<depend>` para declarar las librerías y mensajes indispensables (como `rclcpp` y `dynamixel_sdk`), asegurando que la compilación se detenga y notifique si falta algún componente.v

``` bash 
cat << 'EOF' > ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/src/arm_ax_cpp_pkg/package.xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>arm_ax_cpp_pkg</name>
  <version>0.0.0</version>
  <description>Paquete C++ para control dinámico de servomotores Dynamixel AX-12A en ROS 2 Jazzy</description>
  <maintainer email="joseplascencia346@aragon.unam.mx">Jose Plascencia</maintainer>
  <license>MIT</license>

  <depend>rclcpp</depend>
  <depend>sensor_msgs</depend>
  <depend>rcl_interfaces</depend>
  <depend>arm_ax_interfaces</depend>
  <depend>dynamixel_sdk</depend> 
  <buildtool_depend>ament_cmake</buildtool_depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>

  <export>
    <build_type>ament_cmake</build_type>
  </export>
</package>
EOF
```

### Compilación del Workspace 

``` bash 
cd ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws
rm -rf build/arm_ax_cpp_pkg install/arm_ax_cpp_pkg
colcon build --packages-select arm_ax_cpp_pkg --symlink-install
```
#### Flujo de las terminales 

* Terminal 1 (Driver C++ / Hardware Node): Ejecuta el nodo principal arm_ax_node que habla directamente por el puerto serie USB (/dev/ttyUSB0) a 1 Mbps.

``` bash 
source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
sudo chmod 666 /dev/ttyUSB0
ros2 run arm_ax_cpp_pkg arm_ax_node
```
* Terminal 2 (Telemetría / Telemetry): Escucha el tópico /joint_states para ver en tiempo real qué ángulos tienen los servomotores en la red de ROS 2.

``` bash 
source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
ros2 topic echo /joint_states
```

* Terminal 3: Terminal 3 (Comandos / Action): Envía órdenes en el tópico /joint_commands para mover los motores (cambiando posiciones o velocidad).

``` bash 
source ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX_ws/install/setup.bash
ros2 topic pub --once /joint_commands sensor_msgs/msg/JointState "{position: [0.0, 0.5], velocity: [0.1]}"
```
























## 
```bash
ls -l build/
# Deberías visualizar: armAX.so, DynamixelAXControl.so y DynamixelManager.so
```

### Configuración e Identificación de los Motores
Antes de ejecutar los scripts de control, verifica que la lista de IDs en tu código coincida con la configuración física de tus servomotores (por ejemplo, [3, 4]).
```bash
nano examples/example.py
```


### Ejecución de Pruebas
Para ejecutar el ejemplo comprobando la comunicación:

```bash
PYTHONPATH=build LD_LIBRARY_PATH=lib:build python3 examples/example.py
```

---

## 6. Compilación del Workspace de ROS 2 (`armAX_ws`)

Para compilar la integración con ROS 2:

```bash
# 1. Navegar al workspace de ROS 2
cd ../armAX_ws

# 2. Compilar los paquetes con colcon
colcon build --symlink-install

# 3. Cargar el entorno de ROS 2
source install/setup.bash
```

### Ejecución de Nodos
```bash
# Ejecutar nodo en C++
ros2 run arm_ax_cpp_pkg arm_ax_cpp_node

# Ejecutar nodo en Python
ros2 run arm_ax_py_pkg arm_ax_py_node
```

---

## 7. Comandos de Limpieza Frecuentes

- **Limpiar compilación base:** `cd brazo/armAX && make clean`
- **Limpiar workspace ROS 2:** `rm -rf brazo/armAX_ws/build brazo/armAX_ws/install brazo/armAX_ws/log`
- **Salir del entorno virtual:** `deactivate`

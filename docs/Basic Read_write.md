# Basic Read/Write C++: Individual Servo Control

Esta sección describe cómo escribir un programa *standalone* en C++ (sin ROS 2) utilizando las clases `DynamixelManager` y `DynamixelAXControl` para conectarse a un solo servomotor Dynamixel AX, configurar su modo articular, enviar posiciones en radianes y monitorear su posición actual.

## 1. Crea la estructura del proyecto

Crea una carpeta de trabajo e inicia tu archivo fuente C++:

```bash
mkdir -p my_dxl_project/src
cd my_dxl_project/src
nano my_read_write.cpp
```

## 2. Incluir las cabeceras 

Incluye las cabeceras DynamixelManager.h y DynamixelAXControl.h para gestionar el puerto serie y el control del servomotor:

```cpp
#include "DynamixelManager.h"
#include "DynamixelAXControl.h"
#include <iostream>
#include <cmath>

```

## 3. Inicializar el Gestor de Puerto y Motor

Abre el puerto serie (/dev/ttyUSB0) a 1000000 bps con Protocolo 1.0 mediante DynamixelManager y pasa su referencia al constructor de DynamixelAXControl:

C++

```cpp
int main() {
    uint8_t dxl_id = 1;
    const char* port_name = "/dev/ttyUSB0";
    int baudrate = 1000000;

    // 1. Abrir puerto serie (Puerto, Baudrate, Protocolo 1.0)
    DynamixelManager dxl_mgr(port_name, baudrate, 1.0f);
    if (dxl_mgr.connect() != 0) {
        std::cout << "¡Error al abrir el puerto " << port_name << "!" << std::endl;
        return -1;
    }
    std::cout << "¡Puerto abierto exitosamente!" << std::endl;

    // 2. Instanciar el motor asociado al gestor
    DynamixelAXControl dxl(&dxl_mgr, dxl_id);

```

## 4. Configurar Modo Articular y Mover el Motor

Activa el modo articular (setJointMode), solicita un ángulo objetivo en radianes (rango válido aprox. -2.61 a 2.61 rad, donde 0.0 es el centro en 512) y lee la posición cruda (0–1023):

```cpp
// Configurar Joint Mode (-90° a +90° en radianes)
    if (!dxl.setJointMode(1.5707963268f, -1.5707963268f)) {
        std::cout << "Error al configurar Joint Mode en el motor." << std::endl;
        dxl_mgr.disconnect();
        return -1;
    }

    float target_rad = 0.0f;
    while (true) {
        std::cout << "\nIngresa el angulo objetivo en radianes (-1.57 ~ 1.57) [99 para salir]: ";
        std::cin >> target_rad;

        if (target_rad == 99.0f) {
            break;
        } else if (target_rad < -2.61f || target_rad > 2.61f) {
            std::cout << "El angulo debe estar entre -2.61 y 2.61 radianes." << std::endl;
            continue;
        }

        // Escribir la posicion objetivo (en radianes) en un tiempo de 1.0 segundo
        dxl.setPositionInTime(target_rad, 1.0f);

        // Convertir radianes a valor raw (0-1023) para verificar llegada
        int target_raw = static_cast<int>((target_rad * (180.0f / M_PI) * (1023.0f / 300.0f)) + 512.0f);
        int present_raw = 0;

        do {
            present_raw = static_cast<int>(dxl.getPosition());
            std::cout << "Posicion Actual (raw 0-1023): " << present_raw << std::endl;
        } while (std::abs(target_raw - present_raw) > 5);
    }

    // Cerrar puerto serie al salir
    dxl_mgr.disconnect();
    return 0;
}
```

### Ejemplo de codigo fuente 

``` cpp
#include "DynamixelManager.h"
#include "DynamixelAXControl.h"
#include <iostream>
#include <cmath>

int main() {
    uint8_t dxl_id = 1;
    const char* port_name = "/dev/ttyUSB0";
    int baudrate = 1000000;

    // 1. Inicializar gestor de puerto serie (Protocolo 1.0)
    DynamixelManager dxl_mgr(port_name, baudrate, 1.0f);
    if (dxl_mgr.connect() != 0) {
        std::cout << "Error: No se pudo abrir el puerto " << port_name << std::endl;
        return -1;
    }
    std::cout << "Exito: Puerto " << port_name << " conectado correctamente." << std::endl;

    // 2. Crear objeto de control AX asociado al puerto
    DynamixelAXControl dxl(&dxl_mgr, dxl_id);

    // 3. Configurar Joint Mode (habilita torque y límites angulares)
    if (!dxl.setJointMode(1.5707963268f, -1.5707963268f)) {
        std::cout << "Error al configurar Joint Mode en motor ID: " << (int)dxl_id << std::endl;
        dxl_mgr.disconnect();
        return -1;
    }
    std::cout << "Motor ID " << (int)dxl_id << " listo en Joint Mode." << std::endl;

    float target_rad = 0.0f;
    while (true) {
        std::cout << "\nIngresa el angulo objetivo en radianes (-1.57 ~ 1.57) [99 para salir]: ";
        std::cin >> target_rad;

        if (target_rad == 99.0f) {
            break;
        } else if (target_rad < -2.61f || target_rad > 2.61f) {
            std::cout << "Error: El angulo debe estar dentro del rango [-2.61, 2.61] rad." << std::endl;
            continue;
        }

        // Escritura: Mover a target_rad en 1.0 segundo
        if (!dxl.setPositionInTime(target_rad, 1.0f)) {
            std::cout << "Error al enviar la posicion objetivo." << std::endl;
            continue;
        }

        // Lectura: Monitorear Present Position (raw 0 - 1023)
        int target_raw = static_cast<int>((target_rad * (180.0f / M_PI) * (1023.0f / 300.0f)) + 512.0f);
        int present_raw = 0;

        do {
            present_raw = static_cast<int>(dxl.getPosition());
            std::cout << "Posicion Actual (raw): " << present_raw << std::endl;
        } while (std::abs(target_raw - present_raw) > 5);
    }

    // Limpieza y cierre de puerto al salir
    dxl_mgr.disconnect();
    std::cout << "Puerto cerrado correctamente." << std::endl;

    return 0;
}

```

## Se compila y se ejecuta en Linux 

***Compila enlazando con la librería precompilada libdxl_x64_cpp.so o con las librerías construidas dentro de brazo/armAX/***

``` bash
# Compilar con g++ enlazando DynamixelAXControl, DynamixelManager y el SDK
g++ my_read_write.cpp -o my_read_write \
    -I../../armAX/include \
    -I../../armAX/include/dynamixel_sdk \
    -L../../armAX/lib \
    -lDynamixelAXControl -lDynamixelManager -ldxl_x64_cpp -lpthread

# Ejecutar cargando la librería compartida en el PATH
LD_LIBRARY_PATH=../../armAX/lib ./my_read_write
```


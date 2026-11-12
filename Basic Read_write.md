# Basic Read/Write C++: Individual Servo Control
*Esta sección describe cómo escribir un programa en C++ utilizando la clase DynamixelAXControl para conectarse a un solo servomotor Dynamixel AX, habilitar torque, enviar posiciones y monitorear la posición actual.

## 1.  Crea la estructura del proyecto 

Crea una carpeta de trabajo e inicia tu archivo fuente C++

```bash  
mkdir -p my_dxl_project/src
cd my_dxl_project/src
nano my_read_write.cpp
``` 

*En este caso el nombre de la carpeta de trabajo es my_dxl_project.*

## 2. Incluir las cabeceras 

Incluye la cabecera DynamixelAXControl.h para acceder a la clase de control de los servomotores

```cpp
#include "DynamixelAXControl.h"
#include <iostream>
#include <cmath>

```

## 3. Inicializar el Gestor de Puerto y Motor

Crea la instancia de control definiendo el ID del motor, el puerto serie (dev/ttyUSB0) y el baudrate (1000000 o 57600 según tu configuración)


```cpp
int main() {
    uint8_t dxl_id = 1;
    const char* port_name = "/dev/ttyUSB0";
    int baudrate = 1000000;

    DynamixelAXControl dxl(dxl_id, port_name, baudrate);

    // Abrir puerto y verificar conexión
    if (!dxl.init()) {
        std::cout << "¡Error al abrir el puerto " << port_name << "!" << std::endl;
        return -1;
    }
    std::cout << "¡Puerto abierto exitosamente!" << std::endl;

```

## 4. Habilitar Torque y Mover el Motor

Habilita el torque e interactúa con el usuario para enviar comandos de posición

```cpp
// Habilitar Torque
    dxl.setTorqueEnable(true);

    int target_position;
    while (true) {
        std::cout << "\nIngresa la posicion objetivo (0 ~ 1023) [-1 para salir]: ";
        std::cin >> target_position;

        if (target_position == -1) {
            break;
        } else if (target_position < 0 || target_position > 1023) {
            std::cout << "La posicion debe estar entre 0 y 1023." << std::endl;
            continue;
        }

        // Escribir la posicion objetivo (Goal Position)
        dxl.setGoalPosition(target_position);

        // Leer la posicion actual (Present Position) hasta llegar al objetivo
        int present_position = 0;
        do {
            present_position = dxl.getPresentPosition();
            std::cout << "Posicion Actual: " << present_position << std::endl;
        } while (std::abs(target_position - present_position) > 5);
    }

    // Deshabilitar Torque y cerrar puerto al salir
    dxl.setTorqueEnable(false);
    dxl.closePort();
    return 0;
}
```

### Ejemplo de codigo fuente 

``` cpp
#include "DynamixelAXControl.h"
#include <iostream>
#include <cmath>

int main() {
    uint8_t dxl_id = 1;
    const char* port_name = "/dev/ttyUSB0";
    int baudrate = 1000000;

    // Crear objeto de control AX
    DynamixelAXControl dxl(dxl_id, port_name, baudrate);

    // Intentar abrir la comunicación serie
    if (!dxl.init()) {
        std::cout << "Error: No se pudo abrir el puerto " << port_name << std::endl;
        return -1;
    }
    std::cout << "Exito: Puerto " << port_name << " conectado correctamente." << std::endl;

    // Habilitar Torque en el motor Dynamixel AX
    if (!dxl.setTorqueEnable(true)) {
        std::cout << "Error al habilitar el torque." << std::endl;
        return -1;
    }
    std::cout << "Torque habilitado en el motor ID: " << (int)dxl_id << std::endl;

    int target_position = 0;
    while (true) {
        std::cout << "\nIngresa la posicion objetivo (0 ~ 1023) [-1 para salir]: ";
        std::cin >> target_position;

        if (target_position == -1) {
            break;
        } else if (target_position < 0 || target_position > 1023) {
            std::cout << "Error: La posicion debe estar dentro del rango [0, 1023]." << std::endl;
            continue;
        }

        // Escritura: Enviar Goal Position
        if (!dxl.setGoalPosition(target_position)) {
            std::cout << "Error al enviar la posicion objetivo." << std::endl;
            continue;
        }

        // Lectura: Monitorear Present Position
        int present_position = 0;
        do {
            present_position = dxl.getPresentPosition();
            std::cout << "Posicion Actual: " << present_position << std::endl;
        } while (std::abs(target_position - present_position) > 5);
    }

    // Limpieza al salir
    dxl.setTorqueEnable(false);
    dxl.closePort();
    std::cout << "Puerto cerrado y torque deshabilitado." << std::endl;

    return 0;
}

```
## Se compila y se ejecuta en Linux 

***Compila enlazando con la librería precompilada libdxl_x64_cpp.so o con las librerías construidas dentro de brazo/armAX/***

``` bash
# Compilar con g++ enlazando las librerías de armAX
g++ my_read_write.cpp -o my_read_write \
    -I../../armAX/include \
    -L../../armAX/lib \
    -ldxl_x64_cpp -lpthread

# Ejecutar cargando la librería compartida en el PATH
LD_LIBRARY_PATH=../../armAX/lib ./my_read_write 

```


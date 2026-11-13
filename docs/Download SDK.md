# Descarga e Instalación del DYNAMIXEL SDK (PC)

Esta sección explica cómo obtener el DYNAMIXEL SDK oficial de ROBOTIS para entornos de desarrollo en PC (Linux x86_64, Windows o macOS) [^1]. 

El SDK proporciona las capas fundamentales de comunicación serie y gestión del protocolo DYNAMIXEL para compilar las librerías compartidas (.so, .dll o .dylib) [^2]. 
También contiene las cabeceras requeridas (`dynamixel_sdk/dynamixel_sdk.h`) para compilar los módulos de la librería `dynamixel-ctrl` [^3].

---

## Opciones de Descarga del SDK

Existen tres métodos principales para instalar el DYNAMIXEL SDK en tu PC de desarrollo:

### Opción 1: Clonar el código fuente mediante Git (Recomendado)
Para obtener la última versión directamente desde el repositorio oficial de ROBOTIS [^3]:

```git clone https://github.com/ROBOTIS-GIT/DynamixelSDK.git```

### Opción 2: Descarga directa desde la página de Releases
Puedes descargar el paquete comprimido (.zip o .tar.gz) con el código fuente compilado desde la página oficial de releases
[^github_releases].

### Opción 3: Instalación mediante paquetes de ROS / ROS 2
Disponible únicamente en entornos donde ROS o ROS 2 esté instalado previo a la compilación [^4]:

```sudo apt-get install ros-$ROS_DISTRO-dynamixel-sdk```

---

## Estructura de Archivos del SDK

Al extraer o clonar el SDK, encontrarás la estructura modular organizada por lenguajes de programación y plataformas [^5]:
```text 
DynamixelSDK/
├── c/                          # Implementación base en lenguaje C
│   ├── build/                  # Scripts Makefiles y soluciones de Visual Studio
│   └── include/                # Cabeceras (.h) de bajo nivel
├── c++/                        # Implementación nativa en C++
│   ├── build/                  # Makefiles para Linux/macOS y proyectos VS para Windows
│   ├── include/                # Cabeceras (.hpp / .h) de C++
│   └── src/                    # Código fuente del puerto y manejadores de paquetes
├── python/                     # Wrapper y paquete de Python
└── ros/                        # Integración con metapaquetes de ROS
```
### Gestión Multiplataforma (PortHandler)
Mientras que el módulo PortHandler (encargado de la gestión del entorno de comunicación serie) se implementa de forma independiente según el sistema operativo (Linux, macOS o Windows) [^6], el resto del código fuente del SDK está diseñado de forma multiplataforma (cross-platform).

Puedes compilar estos archivos utilizando un Makefile (para Linux/macOS) o Microsoft Visual Studio (para Windows), dependiendo de si tu plataforma es de 32 bits o 64 bits [^7]. 

Los ejemplos incluidos demuestran cómo utilizar las funciones del SDK para controlar uno o más actuadores DYNAMIXEL aprovechando las librerías dinámicas precompiladas (.so, .dll o .dylib)[^8].

---

## Inclusión de Cabeceras en `brazo/armAX/include/`

Para evitar el error de compilación `fatal error: dynamixel_sdk/dynamixel_sdk.h: No such file or directory` al ejecutar `make` en `brazo/armAX`, copia la carpeta de cabeceras del SDK directamente dentro de la carpeta `include/` de `brazo/armAX`:

1. Clonar el SDK de ROBOTIS (si aún no lo has hecho):
   git clone https://github.com/ROBOTIS-GIT/DynamixelSDK.git

2. Copiar la carpeta de cabeceras a la ruta de tu proyecto:
   cp -r DynamixelSDK/c++/include/dynamixel_sdk ~/Dynamixel/dynamixel-ctrl/src/brazo/armAX/include/

3. Verificar que la estructura en `brazo/armAX/include/` quede configurada así:
   brazo/armAX/include/
   ├── armAX.h
   ├── DynamixelAXControl.h
   ├── DynamixelManager.h
   └── dynamixel_sdk/
       └── dynamixel_sdk.h

Una vez realizada esta copia, ejecuta nuevamente `make` dentro de `brazo/armAX` para compilar sin errores de cabecera.

---
## Referencias Documentales 

[^1]: ROBOTIS Co., Ltd., "DYNAMIXEL SDK e-Manual", ROBOTIS Customer Support, 2024. [En línea]. Disponible en: https://emanual.robotis.com/docs/en/software/dynamixel/dynamixel_sdk/overview/

[^2]: I.A. Glover y P.M. Grant, Digital Communications, 3a ed. Harlow: Prentice Hall, 2009.

[^3]: ROBOTIS-GIT, "DynamixelSDK Source Code Repository", GitHub, 2024. [En línea]. Disponible en: https://github.com/ROBOTIS-GIT/DynamixelSDK

[^github_releases]: ROBOTIS-GIT, "DynamixelSDK Releases", GitHub, 2024. [En línea]. Disponible en: https://github.com/ROBOTIS-GIT/DynamixelSDK/releases

[^4]: ROS.org, "dynamixel_sdk - ROS Wiki", Open Robotics. [En línea]. Disponible en: http://wiki.ros.org/dynamixel_sdk

[^5]: ROBOTIS Co., Ltd., "DYNAMIXEL SDK Library Setup Guide", e-Manual ROBOTIS, 2023.

[^6]: J. B. Kuipers, Quaternions and Rotation Sequences. Princeton, NJ: Princeton University Press, 2002. (Capítulo 5, Sección 5.14 “Quaternions to Matrices”, pág. 125).

[^7]: H.-L. Pham, V. Perdereau, B. Adorno, y P. Fraisse, "Position and Orientation Control of Robot Manipulators Using Dual Quaternion Feedback", IEEE International Conference on Robotics and Automation, pp. 658–663, 2010.

[^8]: National Instruments, "Interfacing C/C++ Shared Libraries (.so/.dll) with LabVIEW and External Languages", Application Note 114, 2021.

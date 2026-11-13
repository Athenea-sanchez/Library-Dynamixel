# Descargar Biblioteca 
Esta sección explica cómo obtener el código fuente e instalar la biblioteca del controlador para el brazo robótico.

## Existen 3 maneras de instalar

### 1. Descargar el código fuente directamente mediante Git clone.

Dentro de terminal: 
```$ git clone https://github.com/tu-usuario/ ```<!repositorio>

### 2. Descarga directa desde la página de versiones (Releases) de GitHub.

Descargar la última versión comprimida (.zip o .tar.gz) directamente desde la sección de Releases del repositorio oficial.

### 3. Compilación e integración local de la librería C++ / Python.

Para entornos sin paquetes precompilados, se genera la librería compilando directamente con CMake y pybind11:

```$ mkdir build && cd build ```

```$ cmake .. && make -j$(nproc)```

## Estructura de los archivos 
```text
├── 🗀 brazo
│   ├── 🗀 armAX
│   │   ├── 🗀 examples
│   │   │   ├── 🗋 example.cpp
│   │   │   └── 🗋 example.py
│   │   ├── 🗀 include
│   │   │   ├── 🗋 armAX.h
│   │   │   ├── 🗋 DynamixelAXControl.h
│   │   │   └── 🗋 DynamixelManager.h
│   │   ├── 🗀 lib
│   │   │   └── 🗋 libdxl_x64_cpp.so
│   │   ├── 🗀 Makefile
│   │   ├── 🗀 Makes
│   │   │   └── 🗋 Makefile_Prueba
│   │   ├── 🗀 obj
│   │   └── 🗀 src
│   │       ├── 🗋 armAX.cpp
│   │       └── 🗋 armAXLinkPython.cpp
│   ├── 🗀 armAX_ws
│   │   └── 🗀 src
│   │       ├── 🗀 arm_ax_cpp_pkg
│   │       │   ├── 🗋 CMakeLists.txt
│   │       │   ├── 🗀 include
│   │       │   │   └── 🗀 armAX_cpp_pkg
│   │       │   │       ├── 🗋 armAX.h
│   │       │   │       ├── 🗋 DynamixelAXControl.h
│   │       │   │       └── 🗋 DynamixelManager.h
│   │       │   ├── 🗀 lib
│   │       │   │   ├── 🗋 libarmAX.a
│   │       │   │   ├── 🗋 libdxl_x64_cpp.so
│   │       │   │   ├── 🗋 libDynamixelAXControl.a
│   │       │   │   └── 🗋 libDynamixelManager.a
│   │       │   ├── 🗀 LICENSE
│   │       │   ├── 🗋 package.xml
│   │       │   └── 🗀 src
│   │       │       └── 🗋 arm_ax_cpp_node.cpp
│   │       ├── 🗀 arm_ax_interfaces
│   │       │   ├── 🗋 CMakeLists.txt
│   │       │   ├── 🗀 LICENSE
│   │       │   ├── 🗀 msg
│   │       │   │   └── 🗋 SetOnePosition.msg
│   │       │   ├── 🗋 package.xml
│   │       │   └── 🗀 srv
│   │       │       └── 🗋 GetOnePosition.srv
│   │       └── 🗀 arm_ax_py_pkg
│   │           ├── 🗀 arm_ax_py_pkg
│   │           │   ├── 🗋 __init__.py
│   │           │   ├── 🗀 __pycache__
│   │           │   │   ├── 🗋 __init__.cpython-310.pyc
│   │           │   │   └── 🗋 arm_ax_py_node.cpython-310.pyc
│   │           │   ├── 🗋 arm_ax_py_node.py
│   │           │   └── 🗀 dynamixel_libs
│   │           │       ├── 🗋 armAX.so
│   │           │       ├── 🗋 DynamixelAXControl.so
│   │           │       └── 🗋 DynamixelManager.so
│   │           ├── 🗀 LICENSE
│   │           ├── 🗋 package.xml
│   │           ├── 🗀 resource
│   │           │   └──  arm_ax_py_pkg
│   │           ├── 🗋 setup.cfg
│   │           ├── 🗋 setup.py
│   │           └── 🗀 test
│   │               ├── 🗋 test_copyright.py
│   │               ├── 🗋 test_flake8.py
│   │               └── 🗋 test_pep257.py
│   ├── 🗀 build
│   └── 🗀 tests
│       ├── 🗀 bin
│       │   └── 🗀 prueba
│       ├── 🗀 build
│       │   ├── 🗋 armAX.so
│       │   ├── 🗋 DynamixelAXControl.so
│       │   └── 🗋 DynamixelManager.so
│       ├── 🗀 include
│       │   ├── 🗋 armAX.h
│       │   ├── 🗋 DynamixelAXControl.h
│       │   └── 🗋 DynamixelManager.h
│       ├── 🗀 lib
│       │   ├── 🗋 libarmAX.a
│       │   ├── 🗋 libdxl_x64_cpp.so
│       │   ├── 🗋 libDynamixelAXControl.a
│       │   └── 🗋 libDynamixelManager.a
│       ├── 🗀 Makefile
│       └── 🗀 src
│           ├── 🗋 prueba.cpp
│           ├── 🗋 prueba.py
│           └── 🗋 test.py
└── 🗀 drivers
    ├── 🗀 DriverAX
    │   ├── 🗀 build
    │   │   ├── 🗋 DynamixelAXControl.so
    │   │   └── 🗋 DynamixelManagerP1.so
    │   ├── 🗀 examples
    │   │   ├── 🗋 example.cpp
    │   │   └── 🗋 example.py
    │   ├── 🗀 include
    │   │   ├── 🗋 DynamixelAXControl.h
    │   │   └── 🗋 DynamixelManagerP1.h
    │   ├── 🗀 lib
    │   │   ├── 🗋 libdxl_x64_cpp.so
    │   │   ├── 🗋 libDynamixelAXControl.a
    │   │   └── 🗋 libDynamixelManagerP1.a
    │   ├── 🗀 Makefile
    │   ├── 🗀 obj
    │   │   ├── 🗋 DynamixelAXControl.o
    │   │   └── 🗋 DynamixelLinkPython.o
    │   └── 🗀 src
    │       ├── 🗋 DynamixelAXControl.cpp
    │       └── 🗋 DynamixelLinkPython.cpp
    ├── 🗀 DriverXL
    │   ├── 🗀 build
    │   │   └── 🗋 DynamixelManagerP2.so
    │   ├── 🗀 examples
    │   │   ├── 🗋 example.cpp
    │   │   └── 🗋 example.py
    │   ├── 🗀 include
    │   │   ├── 🗋 DynamixelManagerP2.h
    │   │   └── 🗋 DynamixelXLControl.h
    │   ├── 🗀 lib
    │   │   ├── 🗋 libdxl_x64_cpp.so
    │   │   └── 🗋 libDynamixelManagerP2.a
    │   ├── 🗀 Makefile
    │   └── 🗀 src
    │       ├── 🗋 DynamixelLinkPython.cpp
    │       └── 🗋 DynamixelXLControl.cpp
    ├── 🗀 DynamixelManagerP1
    │   ├── 🗀 build
    │   │   └── 🗋 DynamixelManagerP1.so
    │   ├── 🗀 examples
    │   │   ├── 🗋 example.cpp
    │   │   └── 🗋 example.py
    │   ├── 🗀 include
    │   │   ├── 🗀 dynamixel_sdk
    │   │   │   ├── 🗋 dynamixel_sdk.h
    │   │   │   ├── 🗋 group_bulk_read.h
    │   │   │   ├── 🗋 group_bulk_write.h
    │   │   │   ├── 🗋 group_fast_bulk_read.h
    │   │   │   ├── 🗋 group_fast_sync_read.h
    │   │   │   ├── 🗋 group_handler.h
    │   │   │   ├── 🗋 group_sync_read.h
    │   │   │   ├── 🗋 group_sync_write.h
    │   │   │   ├── 🗋 packet_handler.h
    │   │   │   ├── 🗋 port_handler.h
    │   │   │   ├── 🗋 port_handler_arduino.h
    │   │   │   ├── 🗋 port_handler_linux.h
    │   │   │   ├── 🗋 port_handler_mac.h
    │   │   │   ├── 🗋 port_handler_windows.h
    │   │   │   ├── 🗋 protocol1_packet_handler.h
    │   │   │   └── 🗋 protocol2_packet_handler.h
    │   │   └── 🗋 DynamixelManagerP1.h
    │   ├── 🗀 lib
    │   │   ├── 🗋 libdxl_x64_cpp.so
    │   │   └── 🗋 libDynamixelManagerP1.a
    │   ├── 🗀 Makefile
    │   ├── 🗀 obj
    │   │   ├── 🗋 DynamixelLinkPython.o
    │   │   └── 🗋 DynamixelManagerP1.o
    │   └── 🗀 src
    │       ├── 🗋 DynamixelLinkPython.cpp
    │       └── 🗋 DynamixelManagerP1.cpp
    ├── 🗀 DynamixelManagerP2
    │   ├── 🗀 build
    │   │   └── 🗋 DynamixelManagerP2.so
    │   ├── 🗀 examples
    │   │   ├── 🗋 example.cpp
    │   │   └── 🗋 example.py
    │   ├── 🗀 include
    │   │   ├── 🗀 dynamixel_sdk
    │   │   │   ├── 🗋 dynamixel_sdk.h
    │   │   │   ├── 🗋 group_bulk_read.h
    │   │   │   ├── 🗋 group_bulk_write.h
    │   │   │   ├── 🗋 group_fast_bulk_read.h
    │   │   │   ├── 🗋 group_fast_sync_read.h
    │   │   │   ├── 🗋 group_handler.h
    │   │   │   ├── 🗋 group_sync_read.h
    │   │   │   ├── 🗋 group_sync_write.h
    │   │   │   ├── 🗋 packet_handler.h
    │   │   │   ├── 🗋 port_handler.h
    │   │   │   ├── 🗋 port_handler_arduino.h
    │   │   │   ├── 🗋 port_handler_linux.h
    │   │   │   ├── 🗋 port_handler_mac.h
    │   │   │   ├── 🗋 port_handler_windows.h
    │   │   │   ├── 🗋 protocol1_packet_handler.h
    │   │   │   └── 🗋 protocol2_packet_handler.h
    │   │   └── 🗋 DynamixelManagerP2.h
    │   ├── 🗀 lib
    │   │   ├── 🗋 libdxl_x64_cpp.so
    │   │   └── 🗋 libDynamixelManagerP2.a
    │   ├── 🗀 Makefile
    │   ├── 🗀 obj
    │   │   ├── 🗋 DynamixelLinkPython.o
    │   │   └── 🗋 DynamixelManagerP2.o
    │   └── 🗀 src
    │       ├── 🗋 DynamixelLinkPython.cpp
    │       └── 🗋 DynamixelManagerP2.cpp
    └── 🗀 tests
        ├── 🗀 bin
        │   └── 🗀 prueba
        ├── 🗀 build
        │   ├── 🗋 DynamixelAXControl.so
        │   ├── 🗋 DynamixelManagerP1.so
        │   └── 🗋 DynamixelManagerP2.so
        ├── 🗀 include
        │   ├── 🗋 DynamixelAXControl.h
        │   ├── 🗋 DynamixelManagerP1.h
        │   └── 🗋 DynamixelManagerP2.h
        ├── 🗀 lib
        │   ├── 🗋 libdxl_x64_cpp.so
        │   ├── 🗋 libDynamixelAXControl.a
        │   ├── 🗋 libDynamixelManagerP1.a
        │   └── 🗋 libDynamixelManagerP2.a
        ├── 🗀 Makefile
        └── 🗀 src
            ├── 🗋 prueba.cpp
            └── 🗋 prueba.py
```

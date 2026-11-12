# API Reference: `dynamixel-ctrl`
Esta referencia de API detalla las clases, métodos y estructuras que componen la librería dynamixel-ctrl para el control de servomotores Dynamixel de la serie AX.

## Clase DynamixelManager (Capa de Comunicación)

Encapsula la gestión del puerto serie (/dev/ttyUSB0) y la configuración de velocidad en baudios utilizando internamente los handlers del Dynamixel SDK.

Actúa como la capa de abstracción de comunicación de bajo nivel. 


### Constructor y Destructor

| Método / Función | Sintaxis | Descripción |
| :--- | :--- | :--- |
| **Constructor** | `DynamixelManager(const char* port_name, int baudrate)` | Inicializa los parámetros de conexión con el puerto serie especificado (ej. `/dev/ttyUSB0`) y la velocidad en baudios (ej. `1000000`). |
| **Destructor** | `~DynamixelManager()` | Cierra automáticamente el puerto serie y libera los recursos en memoria al destruir el objeto. |

### Métodos de la Clase

| Método | Sintaxis | Parámetros | Retorno | Descripción Detallada |
| :--- | :--- | :--- | :--- | :--- |
| **`openPort`** | `bool openPort()` | *Ninguno* | `bool` | Abre el puerto serie configurado en el sistema operativo. Devuelve `true` si la conexión se establece con éxito y `false` en caso de error. |
| **`closePort`** | `void closePort()` | *Ninguno* | `void` | Cierra el descriptor del puerto serie y libera el bus de comunicación. |
| **`setBaudRate`** | `bool setBaudRate(int baudrate)` | `baudrate` (`int`): Tasa de baudios deseada. | `bool` | Modifica la velocidad de transmisión del puerto serie (ej. `57600`, `1000000`). |
| **`getPortName`** | `const char* getPortName()` | *Ninguno* | `const char*` | Retorna el nombre del puerto serie asignado a la instancia. |
| **`getBaudRate`** | `int getBaudRate()` | *Ninguno* | `int` | Retorna la velocidad actual en baudios configurada en el gestor. |

---

## 2. Clase `DynamixelAXControl`

La clase `DynamixelAXControl` representa la abstracción de un servomotor individual de la serie AX (AX-12A / AX-18A) e interactúa con su tabla de memoria RAM/EEPROM.

### Constructor

| Método | Sintaxis | Descripción |
| :--- | :--- | :--- |
| **Constructor** | `DynamixelAXControl(uint8_t dxl_id, const char* port_name, int baudrate)` | Inicializa el control de un servomotor asignando su ID individual, la ruta del puerto y el baudrate. |

### Métodos Directos (Control por Motor)

| Método | Sintaxis | Parámetros | Retorno | Descripción Detallada |
| :--- | :--- | :--- | :--- | :--- |
| **`init`** | `bool init()` | *Ninguno* | `bool` | Verifica la presencia física del motor en el bus enviando un paquete de prueba (*Ping*). |
| **`setTorqueEnable`** | `bool setTorqueEnable(bool enable)` | `enable` (`bool`): `true` (ON), `false` (OFF) | `bool` | Enciende o apaga el torque del motor escribiendo en la dirección de memoria EEPROM/RAM. |
| **`setGoalPosition`** | `bool setGoalPosition(int position)` | `position` (`int`): Valor entre `0` y `1023` | `bool` | Envía la posición de destino (*Goal Position*) al servomotor. |
| **`getPresentPosition`** | `int getPresentPosition()` | *Ninguno* | `int` | Lee y devuelve la posición actual en tiempo real (`0`–`1023`). Retorna `-1` si falla la lectura. |
| **`setMovingSpeed`** | `bool setMovingSpeed(int speed)` | `speed` (`int`): Velocidad (`0`–`1023`) | `bool` | Configura la velocidad de movimiento de la articulación. |
| **`getMovingSpeed`** | `int getMovingSpeed()` | *Ninguno* | `int` | Lee la velocidad actual del motor registrada en la tabla de control. |
| **`setTorqueLimit`** | `bool setTorqueLimit(int limit)` | `limit` (`int`): Límite de torque (`0`–`1023`) | `bool` | Establece el límite máximo de torque entregado por el motor. |

---

## 3. Clase `armAX`

La clase `armAX` representa la capa de control de más alto nivel para el brazo robótico completo. Encapsula las operaciones síncronas (*Sync Read / Sync Write*) sobre vectores de motores.

### Constructor

| Método | Sintaxis | Descripción |
| :--- | :--- | :--- |
| **Constructor** | `armAX(DynamixelManager& dxl_mgr, const std::vector<uint8_t>& motor_ids)` | Crea una instancia del brazo asociándola a un gestor de puerto `DynamixelManager` y definiendo el listado de IDs que componen el brazo. |

### Métodos Coordinados (Brazo Completo)

| Método | Sintaxis | Parámetros | Retorno | Descripción Detallada |
| :--- | :--- | :--- | :--- | :--- |
| **`setTorqueEnableAll`** | `bool setTorqueEnableAll(bool enable)` | `enable` (`bool`): `true` (ON), `false` (OFF) | `bool` | Activa o des
# Guía de Configuración de un entorno virtual.

Esta guía detalla el procedimiento para instalar **Python 3.10** en una máquina virtual con una versión superior de Ubuntu.

---

## 1. Agregar el Repositorio Deadsnakes
Dado que las versiones recientes de Ubuntu integran Python 3.12, es necesario agregar el repositorio `deadsnakes` para acceder a versiones anteriores.

Abre la terminal y ejecuta:

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
```

---

## 2. Instalar Python 3.10 y Dependencias
Instala los binarios de Python 3.10 junto con el módulo de entorno virtual (`venv`) y las herramientas de desarrollo (`dev`):

```bash
sudo apt install python3.10 python3.10-venv python3.10-dev -y
```

Para verificar que la instalación fue exitosa:

```bash
python3.10 --version
```

---

## 3. Crear el Entorno Virtual
Ubícate en el directorio donde desees almacenar el entorno virtual (o en la raíz de tu proyecto) y ejecútalo especificando la versión requerida:

```bash
python3.10 -m venv mi_entorno
```

> **Nota:** Puedes reemplazar `mi_entorno` por el nombre que prefieras (ej. `.venv`, `env_dynamixel`, etc.).

---

## 4. Activar el Entorno Virtual
Para utilizar el entorno virtual en tu sesión actual de la terminal:

```bash
source mi_entorno/bin/activate
```

Una vez activado, verás el nombre del entorno al inicio del prompt de la terminal:
```text
(mi_entorno) usuario@ubuntu:~$
```

### Verificación del entorno
Comprueba que los comandos apuntan a la versión correcta:

```bash
python --version   # Debería mostrar Python 3.10.x
pip --version      # Debería mostrar la ruta dentro de mi_entorno
```

---

## 5. Uso del Entorno Virtual

* **Navegación:** Puedes navegar a cualquier carpeta o repositorio (por ejemplo, tus paquetes de ROS o el proyecto `dynamixel-ctrl-main`) y la terminal mantendrá activo **Python 3.10**.
* **Instalación de paquetes:** Todo paquete instalado con `pip` quedará almacenado únicamente en el entorno virtual:
  ```bash
  pip install --upgrade pip
  pip install pyserial
  ```
* **Nueva Terminal:** El entorno se activa únicamente por pestaña/ventana de terminal. Si abres una nueva sesión, debes volver a ejecutar `source <ruta_al_entorno>/bin/activate`.

---

## 6. Desactivar el Entorno
Cuando termines de trabajar y quieras volver al entorno por defecto del sistema:

```bash
deactivate
```

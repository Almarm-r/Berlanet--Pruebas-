# BeerLaNet — Instalación y configuración

Este documento describe cómo configurar el entorno de ejecución de **BeerLaNet** utilizando **Miniconda**.

Para garantizar la compatibilidad con el proyecto, se recomienda utilizar:

* **Python 3.10.21**
* **Miniconda**
* El entorno virtual definido en `environment.yml`

> **Importante:** Antes de ejecutar cualquier código del proyecto, debes activar el entorno `beerlanet`.

---

## 1. Requisitos

Antes de comenzar, instala **Miniconda** en tu sistema.

Puedes descargar Miniconda desde:

https://docs.conda.io/projects/miniconda/en/latest/

Después de instalarlo, abre:

* **Anaconda Prompt** en Windows, o
* una terminal en Linux/macOS.

Comprueba que Conda está disponible:

```bash
conda --version
```

Deberías obtener una salida similar a:

```text
conda 25.x.x
```

---

# 2. Clonar el repositorio

Clona el repositorio de BeerLaNet:

```bash
git clone https://github.com/Almarm-r/Berlanet--Pruebas-
```

Entra en la carpeta del proyecto:

```bash
cd Berlanet--Pruebas
```

La estructura principal del proyecto debería ser similar a:

```text
Berlanet--Pruebas/
│
├── src/
│   ├── BeerLaNet.py
│   └── ...
│
├── experiments/
│   └── ...
│
├── environment.yml
├── README.md
└── README2.md
```

---

# 3. Crear el entorno Conda

El proyecto utiliza **Python 3.10.21**.

Si el repositorio incluye el archivo:

```text
environment.yml
```

puedes crear automáticamente el entorno ejecutando:

```bash
conda env create -f environment.yml
```

Este comando instalará las dependencias especificadas en el archivo.

---

# 4. Verificar la versión de Python

Activa el entorno:

```bash
conda activate beerlanet
```

Luego verifica la versión de Python:

```bash
python --version
```

Debe aparecer:

```text
Python 3.10.21
```

También puedes verificar qué Python está siendo utilizado:

### Windows

```bash
where python
```

### Linux/macOS

```bash
which python
```

La ruta debe corresponder al entorno `beerlanet`.

---

# 5. Si el entorno no existe

Si por alguna razón no puedes utilizar `environment.yml`, puedes crear el entorno manualmente:

```bash
conda create -n beerlanet python=3.10.21
```

Activa el entorno:

```bash
conda activate beerlanet
```

Después instala las dependencias necesarias del proyecto.

Si existe un archivo `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

# 6. Activar el entorno antes de trabajar

**Cada vez que vayas a ejecutar BeerLaNet debes activar el entorno.**

Ejecuta:

```bash
conda activate beerlanet
```

Puedes comprobar que el entorno está activo con:

```bash
conda info --envs
```

El entorno activo aparecerá marcado con `*`:

```text
# conda environments:

base          C:\Users\usuario\miniconda3
beerlanet   * C:\Users\usuario\miniconda3\envs\beerlanet
```

También puedes comprobarlo ejecutando:

```bash
python --version
```

y:

```bash
conda list
```

---

# 7. Verificar las dependencias

Para comprobar las dependencias instaladas:

```bash
conda list
```

Si algunas dependencias fueron instaladas mediante `pip`, puedes comprobarlas con:

```bash
pip list
```

También puedes comprobar que PyTorch está disponible:

```bash
python -c "import torch; print(torch.__version__)"
```

Y comprobar que OpenCV está disponible:

```bash
python -c "import cv2; print(cv2.__version__)"
```

---

# 8. Ejecutar BeerLaNet

Una vez activado el entorno:

```bash
conda activate beerlanet
```

puedes ejecutar los experimentos del proyecto.

Por ejemplo:

```bash
cd experiments
```

y posteriormente ejecutar el script correspondiente:

```bash
python nombre_del_experimento.py
```

La ejecución debe realizarse siempre dentro del entorno `beerlanet`.

---

# 9. Uso desde Jupyter Notebook

Si los experimentos se realizan mediante Jupyter Notebook, primero activa el entorno:

```bash
conda activate beerlanet
```

Luego puedes iniciar Jupyter:

```bash
jupyter notebook
```

También puedes registrar el entorno como un kernel:

```bash
python -m ipykernel install --user --name beerlanet --display-name "Python 3.10.21 (BeerLaNet)"
```

En Jupyter selecciona:

```text
Python 3.10.21 (BeerLaNet)
```

como kernel.

---

# 10. Problemas frecuentes

## Conda no reconoce el comando `conda`

Si aparece:

```text
'conda' is not recognized as an internal or external command
```

abre **Anaconda Prompt** y vuelve a ejecutar los comandos desde allí.

En Linux/macOS también puedes inicializar Conda con:

```bash
conda init
```

Después reinicia la terminal.

---

## Se está utilizando otra versión de Python

Comprueba:

```bash
python --version
```

Si no aparece:

```text
Python 3.10.21
```

comprueba que el entorno esté activado:

```bash
conda activate beerlanet
```

y vuelve a verificar:

```bash
python --version
```

---

## El entorno ya existe

Si aparece un mensaje indicando que `beerlanet` ya existe, simplemente actívalo:

```bash
conda activate beerlanet
```

No es necesario crearlo nuevamente.

---

## Actualizar el entorno

Si el archivo `environment.yml` cambia, puedes actualizar el entorno mediante:

```bash
conda env update -f environment.yml --prune
```

La opción `--prune` elimina paquetes que ya no aparecen en el archivo de configuración.

---

# 11. Desactivar el entorno

Cuando termines de trabajar:

```bash
conda deactivate
```

Esto devuelve la terminal al entorno anterior.

---

# 12. Resumen de instalación

La instalación desde cero puede resumirse en:

```bash
git clone <URL_DEL_REPOSITORIO>

cd BeerLaNet

conda env create -f environment.yml

conda activate beerlanet

python --version
```

La última instrucción debe mostrar:

```text
Python 3.10.21
```

A partir de ese momento, **mantén activado el entorno `beerlanet` mientras ejecutes el proyecto**.

---

## Entorno utilizado

| Componente         | Versión           |
| ------------------ | ----------------- |
| Python             | 3.10.21           |
| Gestor de entornos | Miniconda         |
| Entorno Conda      | `beerlanet`       |
| Dependencias       | `environment.yml` |

---

## Nota sobre reproducibilidad

El archivo `environment.yml` permite definir las dependencias necesarias para reproducir el entorno de BeerLaNet.

Se recomienda **no modificar manualmente las versiones de las librerías** si el objetivo es reproducir los experimentos originales. Si necesitas realizar cambios en las dependencias, crea una copia del entorno o actualiza el archivo `environment.yml` de forma controlada.

---

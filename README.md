# Evaluación 4 – Machine Learning supervisado y no supervisado sobre UNSW-NB15

**Equipo 1** 
· Integrante 1
· Integrante 2 
· Integrante 3 
· Integrante 4

## 1. Fuente de datos

Trabajamos con una versión del **UNSW-NB15**, un conjunto de datos de tráfico de red creado por el Australian Centre for Cyber Security (UNSW Canberra). El tráfico se generó en un laboratorio que mezcla actividad real con ataques sintéticos, y cada registro resume una conexión: cuánto duró, cuántos paquetes y bytes viajaron en cada sentido, tiempos de vida (TTL), retardos y contadores de conexiones parecidas en los últimos minutos.

El archivo que usamos (`UNSW-NB15_Dataset2.csv`) tiene estas características:

| Característica | Valor |
|---|---|
| Registros | 340.693 |
| Variables de entrada | 39, todas numéricas |
| Variable objetivo | `attack` (5 tipos de ataque) |
| Valores faltantes | 0 |

| Ataque | Registros originales |
|---|---:|
| Backdoor | 95.494 |
| Probe | 72.789 |
| Worm | 60.855 |
| Generic | 57.309 |
| DoS | 54.246 |

Link de descarga: https://www.kaggle.com/datasets/ucimachinelearning/unsw-nb15-dataset

<br>


Dos diferencias frente al UNSW-NB15 original condicionan el planteamiento:

- **No hay tráfico normal.** Todas las filas son ataques, así que no se puede plantear "normal vs. malicioso". El problema principal es identificar **qué tipo de ataque** es cada conexión.
- **No vienen `proto`, `service` ni `state`.** El análisis se hace con las variables de conexión y comportamiento disponibles (`sttl`, `dttl`, `ct_state_ttl`, `swin`, `dwin`, `trans_depth`, `ct_flw_http_mthd`, `is_ftp_login`, entre otras).

## 2. Estructura del proyecto

```
├── .gitattributes                  # finales de línea iguales en Windows/Linux/macOS
├── README.md
└── evaluacion 4/
    ├── .gitignore
    ├── 01_supervisado_unsw_nb15.ipynb
    ├── requirements.txt
    ├── resultados/
    │   └── supervisado/
    └── visualizaciones/
        └── supervisado/
```

## 3. Cómo ejecutar

Funciona en Windows, Linux y macOS con Python 3.10 o superior. Todos los comandos se ejecutan dentro de `evaluacion 4/`.

### 3.1 Crear el entorno e instalar dependencias

El `.venv` es propio de cada sistema operativo y no se sube al repo, así que cada integrante crea el suyo.

**Windows (PowerShell)**
```powershell
cd "evaluacion 4"
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```
> Si PowerShell bloquea el script de activación: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.
> En `cmd` se activa con `.venv\Scripts\activate.bat`.

**Linux / macOS**
```bash
cd "evaluacion 4"
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```
> En Debian/Ubuntu, si falla la creación del entorno: `sudo apt install python3-venv`.

### 3.2 Descargar el dataset

El CSV no se sube al repo, así que cada integrante debe descargarlo. Si no está, el notebook falla en la celda que carga los datos.

1. Entrar a https://www.kaggle.com/datasets/ucimachinelearning/unsw-nb15-dataset, iniciar sesión y pulsar **Download**. Kaggle entrega un `.zip`.
2. Descomprimirlo y copiar `UNSW-NB15_Dataset2.csv` en `evaluacion 4/`. También puede ir en `evaluacion 4/data/` o en la raíz del repo; el notebook lo busca en esos tres lugares.

**Windows (PowerShell)**, desde la raíz del repo:
```powershell
Expand-Archive "$HOME\Downloads\archive.zip" -DestinationPath "$HOME\Downloads\unsw"
Copy-Item "$HOME\Downloads\unsw\UNSW-NB15_Dataset2.csv" "evaluacion 4\"
```

**Linux / macOS**, desde la raíz del repo:
```bash
unzip ~/Downloads/archive.zip -d ~/Downloads/unsw      # en Linux en español puede ser ~/Descargas
cp ~/Downloads/unsw/UNSW-NB15_Dataset2.csv "evaluacion 4/"
```

> El nombre del `.zip` (`archive.zip`) puede variar según el navegador. Si es así, ajústalo en el comando.

### 3.3 Ejecutar el notebook

**Opción A: VS Code o Cursor**
1. Instalar las extensiones *Python* y *Jupyter*.
2. Abrir `01_supervisado_unsw_nb15.ipynb`.
3. Arriba a la derecha: **Select Kernel → Python Environments → `.venv`**.
4. **Run All**.

**Opción B: JupyterLab en el navegador** (con el entorno activado)
```bash
jupyter lab 01_supervisado_unsw_nb15.ipynb
```

**Opción C: sin interfaz** (ejecuta todo y guarda las salidas en el mismo archivo)
```bash
jupyter nbconvert --to notebook --execute --inplace 01_supervisado_unsw_nb15.ipynb
```

Las carpetas `resultados/` y `visualizaciones/` se crean solas.

Se usa una semilla fija (`RANDOM_STATE = 42`) en particiones, muestreos y modelos.

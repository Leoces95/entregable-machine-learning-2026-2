# Evaluación 4 – Machine Learning supervisado y no supervisado sobre UNSW-NB15

**Equipo 1** 
· Harold Restrepo
· Leonel Martinez 
· Yulieth Urrego 
· Jalvi Villegas

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

Dos diferencias frente al UNSW-NB15 original condicionan el planteamiento:

- **No hay tráfico normal.** Todas las filas son ataques, así que no se puede plantear "normal vs. malicioso". El problema principal es identificar **qué tipo de ataque** es cada conexión.
- **No vienen `proto`, `service` ni `state`.** El análisis se hace con las variables de conexión y comportamiento disponibles (`sttl`, `dttl`, `ct_state_ttl`, `swin`, `dwin`, `trans_depth`, `ct_flw_http_mthd`, `is_ftp_login`, entre otras).

## 2. Estructura del proyecto

```
evaluacion 4/
├── .gitignore
├── 01_supervisado_unsw_nb15.ipynb
├── 02_no_supervisado_unsw_nb15.ipynb
├── requirements.txt
├── README.md
├── resultados/
│   ├── supervisado/
│   └── no_supervisado/
└── visualizaciones/
    ├── supervisado/
    └── no_supervisado/
```

## 3. Cómo ejecutar

1. Crear el entorno e instalar dependencias:
   ```bash
   python -m venv .venv
   .venv\Scripts\activate          # Windows  (Linux/Mac: source .venv/bin/activate)
   python -m pip install -r requirements.txt
   ```
2. Copiar `UNSW-NB15_Dataset2.csv` en la carpeta del notebook, en `data/` o en la carpeta padre. El notebook lo busca en esos tres lugares.
3. Ejecutar los notebooks. Son independientes y pueden correr en cualquier orden:
   ```bash
   python 01_supervisado_unsw_nb15.ipynb 
   python 02_no_supervisado_unsw_nb15.ipynb 
   ```
   Las carpetas `resultados/` y `visualizaciones/` se crean solas.

Se usa una semilla fija (`RANDOM_STATE = 42`) en particiones, muestreos y modelos.

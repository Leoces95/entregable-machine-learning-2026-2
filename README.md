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

## 4. Análisis exploratorio (común a ambos notebooks)

- **Duplicados:** el 58 % de las filas son copias exactas. Se eliminan antes de dividir los datos; si no, la misma conexión podría quedar en entrenamiento y en prueba y las métricas saldrían infladas.
- **Desbalance:** al quitar duplicados el balance cambia mucho y Backdoor pasa a dominar.
- **Asimetría:** variables como `sbytes`, `dbytes`, `sload` y `dload` tienen colas muy largas, por eso se usan escalas logarítmicas en las gráficas y en el clustering.
- **Variables sin sentido para modelar distancias:** `stcpb` y `dtcpb` son números de secuencia TCP iniciales, prácticamente aleatorios.
- **Correlación:** se usa Spearman porque las relaciones no son lineales y hay valores extremos.

## 5. Aprendizaje supervisado (`01_supervisado_unsw_nb15.ipynb`)

### Metodología

1. División estratificada 70 / 15 / 15: entrenamiento 99.708, validación 21.366 y prueba 21.366.
2. El escalado y la selección de variables se ajustan **solo con entrenamiento**.
3. Selección de variables con **información mutua** (`mutual_info_classif`).
4. El desbalance se maneja con `class_weight="balanced"`.
5. Hiperparámetros con `GridSearchCV` sobre submuestras estratificadas del entrenamiento.
6. Todos los modelos se evalúan sobre **el mismo conjunto de prueba**, que no se usa para ajustar nada.

### Modelos y justificación

| Modelo | Por qué se usa | Limitación |
|---|---|---|
| Regresión Logística | Modelo base, rápido e interpretable | Solo separa las clases con fronteras lineales |
| SVM con kernel RBF | Captura fronteras no lineales entre ataques | Costoso con muchos datos; se entrena con submuestra |
| One-Class SVM | Aprende un solo tipo de tráfico y marca lo que se sale de él | Solo resuelve problemas de "una clase vs. el resto" |

### Resultados – clasificación multiclase (5 ataques, conjunto de prueba)

| Modelo | Accuracy | Balanced Acc. / Recall macro | Precision macro | F1 macro | ROC-AUC OvR | PR-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Regresión Logística | 0,695 | 0,749 | 0,601 | 0,600 | 0,940 | 0,698 |
| **SVM RBF** | **0,871** | **0,833** | **0,748** | **0,768** | 0,936 | **0,764** |

El SVM es el mejor modelo: sube el F1 macro de 0,60 a 0,77. La regresión logística tiene un ROC-AUC un poco más alto (0,940 frente a 0,936), es decir, ordena bien las probabilidades, pero sus fronteras lineales confunden más clases al momento de decidir. El *balanced accuracy* coincide con el *recall* macro porque los dos son el promedio del recall de cada clase.

### Resultados – análisis complementario Generic vs. resto

| Modelo | Umbral | Precision | Recall | F1 | ROC-AUC | PR-AUC |
|---|---:|---:|---:|---:|---:|---:|
| Regresión Logística | 0,245 | 0,9957 | 0,9954 | 0,9955 | 0,9976 | 0,9999 |
| SVM RBF | 0,766 | 0,9973 | 0,9996 | 0,9985 | 0,9989 | 0,9999 |
| One-Class SVM | 0,0075 | 0,9996 | 0,9998 | 0,9997 | 0,9999 | 1,0000 |

Los umbrales se ajustaron sobre **validación** maximizando F1 y luego se aplicaron a prueba. Generic se separa casi perfectamente del resto porque tiene un patrón muy marcado en TTL y tamaño de paquetes; por eso los tres modelos superan 0,995.

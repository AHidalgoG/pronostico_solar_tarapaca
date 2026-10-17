# Pronóstico multisitio de radiación solar en Tarapacá

**Pronóstico horario de irradiancia global horizontal (GHI) mediante modelos estadísticos y de aprendizaje profundo.**

Este repositorio reúne los datos, el flujo ETL en KNIME y los experimentos de un estudio que compara **regresión lineal múltiple, SARIMA, MLP y LSTM** para predecir el GHI **una hora hacia adelante** en **Pozo Almonte, Pica y Colchane**, Región de Tarapacá, Chile.

El objetivo es comparar su **precisión y comportamiento entre localidades**, considerando también la complejidad de cada enfoque.

## 1. Datos y metodología

Los datos proceden del [Explorador Solar del Ministerio de Energía de Chile](https://solar.minenergia.cl/). Se utilizaron series **horarias modeladas por ubicación** del periodo **2004–2016**; no deben confundirse con mediciones instrumentales continuas realizadas en los tres sitios.

El conjunto original contiene **341.856 observaciones**, distribuidas entre las tres localidades. El flujo **KNIME** realiza la preparación de datos, genera variables predictoras y exporta los conjuntos utilizados por los modelos.

| Conjunto | Periodo | Archivo |
|---|---|---|
| Entrenamiento | 2004–2013 | `ghi_multisitio_entrenamiento.csv` |
| Validación | 2014–2015 | `ghi_multisitio_validacion.csv` |
| Prueba | 2016 | `ghi_multisitio_prueba.csv` |

El objetivo es `ghi_t1` (GHI de la siguiente hora). Entre las variables construidas se incluyen rezagos de GHI, representaciones cíclicas del tiempo e indicadores de localidad. Los modelos multivariados también emplean variables meteorológicas; **SARIMA utiliza únicamente la historia del GHI y se ajusta por localidad**.

Para reproducir la preparación de datos desde los CSV originales, consultar **[ETL en KNIME](etl/README.md)**.

## 2. Estructura del proyecto

```text
pronostico_solar_tarapaca/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/                    # CSV originales por localidad
│   └── processed/              # Entrenamiento, validación y prueba
├── etl/
│   ├── etl-series.knwf         # Flujo de KNIME exportado
│   └── README.md
├── notebooks/
│   ├── 01_eda.ipynb
│   ├── 02_regresion_lineal.ipynb
│   ├── 03_sarima.ipynb
│   ├── 04_mlp.ipynb
│   └── 05_lstm.ipynb
├── models/                      # Modelos entrenados y escaladores, si se guardan
├── results/
│   ├── figures/               # Gráficos
│   ├── metrics/               # Métricas exportadas
│   └── predictions/           # Predicciones exportadas
└── docs/                       # Artículo y documentación metodológica
```

> Los nombres anteriores representan la **estructura objetivo**. Verificar que coincidan con los archivos reales antes de publicar. La conversión de los notebooks a scripts `.py` y la incorporación de `src/` y `main.py` son pasos posteriores, todavía no implementados.

## 3. Instalación y ejecución

Se requiere **Python**, **VS Code con las extensiones Python y Jupyter** (o una instalación de Jupyter compatible) y los CSV del proyecto. KNIME solo es necesario si se quiere **regenerar** `data/processed/` a partir de `data/raw/`.

Desde la **raíz del repositorio**, crear el entorno virtual:

```powershell
python -m venv .venv
```

Instalar las dependencias en ese entorno (en Windows, sin necesidad de activarlo):

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

Para ejecutar los experimentos:

1. Abrir la carpeta del repositorio en VS Code.
2. Abrir el notebook deseado de `notebooks/` y seleccionar como *kernel* `.venv\Scripts\python.exe`.
3. Comprobar la presencia de los CSV en `data/raw/` (EDA) y `data/processed/` (modelos).
4. Ejecutar el notebook completo desde el inicio (**Restart Kernel and Run All Cells**).
5. Revisar las métricas y las salidas guardadas en `results/` y `models/`, según el notebook.

**Los notebooks de los cuatro modelos son independientes:** no hace falta ejecutar primero los otros modelos. Para conocer o reproducir el procesamiento previo, ejecutar el flujo KNIME siguiendo su propio README.

El reentrenamiento de SARIMA, MLP y LSTM puede llevar un tiempo considerable. Los scripts `.py` y el futuro `main.py` reutilizarán **las mismas bibliotecas** de `requirements.txt`, siempre que conserven las mismas dependencias.

## 4. Resultados de referencia

Métricas globales del **conjunto de prueba** publicadas en el estudio:

| Modelo | MAE (W/m²) | RMSE (W/m²) |
|---|---:|---:|
| Regresión lineal | 44,76 | 80,08 |
| SARIMA | 31,84 | 73,89 |
| MLP | 17,73 | 55,10 |
| **LSTM** | **16,24** | **54,67** |

La **LSTM obtuvo la mejor precisión global**, aunque la mejora frente a MLP fue pequeña. El desempeño fue diferente entre localidades, especialmente en Colchane. El resultado corresponde al pronóstico **a una hora** y no demuestra superioridad universal de una arquitectura.

Las métricas se calcularon con las predicciones negativas truncadas a cero y consideran también las horas nocturnas. El conjunto evaluado en el artículo contiene **25.989 predicciones** (8.663 por localidad), después de excluir observaciones sin objetivo válido.

## 5. Reproducibilidad y alcance

- **Partición temporal:** entrenamiento (2004–2013), validación (2014–2015) y prueba (2016); no se utiliza división aleatoria.
- **ETL:** el workflow KNIME no requiere semillas aleatorias. Sus instrucciones de importación, rutas y ejecución están en [`etl/README.md`](etl/README.md).
- **Regresión y SARIMA:** no requieren semilla de inicialización aleatoria en los experimentos implementados.
- **MLP y LSTM:** utilizan `tf.keras.utils.set_random_seed(42)`. Aun así, la versión de TensorFlow, el hardware y determinadas operaciones pueden introducir diferencias al reentrenar.
- **Estado de verificación:** las adaptaciones de rutas y la ejecución completa de los notebooks en un entorno nuevo deben comprobarse antes de declarar el repositorio totalmente reproducible.
- **Versiones de dependencias:** `requirements.txt` declara paquetes directos, **sin fijar versiones todavía**. Después de validar el entorno, registrar las versiones efectivamente utilizadas para una reproducción más estricta.

Los resultados provienen de datos históricos modelados, no de una evaluación operativa contra estaciones de medición independientes. Asimismo, los modelos utilizan distintos conjuntos de predictores y no se verificó la disponibilidad en tiempo real de todas las variables meteorológicas.

## 6. Referencia y uso de datos

**Artículo:** *Pronóstico multisitio de radiación solar global horizontal en la Región de Tarapacá mediante modelos estadísticos y de aprendizaje profundo*, de **Alonso Hidalgo Gutiérrez, Francisco Meza y Manuel Alejandro Goyo** (INFONOR 2026).

**Fuente de datos:** [Explorador Solar — Ministerio de Energía de Chile](https://solar.minenergia.cl/).

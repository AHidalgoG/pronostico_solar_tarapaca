# ETL de radiación solar multisitio — KNIME

## 1. Resumen

Este workflow de **KNIME Analytics Platform** prepara datos horarios de radiación solar de **Colchane, Pica y Pozo Almonte (2004–2016)**, obtenidos del [Explorador Solar del Ministerio de Energía de Chile](https://solar.minenergia.cl/). Son **datos modelados por ubicación**, no mediciones directas de estaciones en las tres localidades.

Genera los conjuntos utilizados por los modelos de **regresión lineal, SARIMA, MLP y LSTM** para pronosticar la irradiancia global horizontal (GHI) con una hora de anticipación.

## 2. Importación, configuración y ejecución
**Requisitos:**

- **KNIME Analytics Platform**. El archivo de flujo suministrado utiliza nodos cuya versión registrada es **5.11.0**; se recomienda comenzar con esa versión o una compatible.
- Nodos estándar de KNIME, incluidos **KNIME Base Nodes** y **KNIME Java Snippet Nodes** (para los separadores basados en reglas).
- Tres archivos CSV originales en `data/raw/`, con los nombres indicados a continuación.

Mantener la siguiente organización:

```text
pronostico-solar-tarapaca/
├── data/
│   ├── raw/
│   │   ├── colchane.csv
│   │   ├── pica.csv
│   │   └── pozo_almonte.csv
│   └── processed/
└── etl/
    ├── etl-series.knwf
    └── README.md
```

1. **Importar el workflow:** descargar el repositorio y abrir el archivo `etl/etl-series.knwf` mediante KNIME Analytics Platform.

2. **Verificar las rutas de entrada y salida:** comprobar que los nodos `CSV Reader` utilicen los archivos originales ubicados en `data/raw/` y que los nodos `CSV Writer` exporten los resultados a `data/processed/`. Si el flujo se importa fuera del directorio del proyecto, será necesario ajustar las rutas según su ubicación.

3. **Ejecutar el flujo:** ejecutar el workflow completo y verificar que todos los nodos finalicen correctamente. Como resultado, se generarán tres archivos CSV correspondientes a los conjuntos de entrenamiento, validación y prueba.

**Nota:** los nodos `CSV Writer` pueden sobrescribir los archivos procesados existentes. Se recomienda conservar una copia antes de ejecutar nuevamente el flujo.

## 3. Etapas del flujo e información relevante

| Etapa | Operaciones principales |
|---|---|
| **Extracción** | Lectura de los CSV, identificación de localidad y unión de los registros. |
| **Preprocesamiento** | Tratamiento de fecha/hora, filtrado de GHI faltante, control de duplicados y ordenamiento temporal. |
| **Transformación** | Variables temporales cíclicas, rezagos `ghi_lag_1` y `ghi_lag_24`, objetivo `ghi_t1` (GHI de la hora siguiente) e indicadores de localidad. Los rezagos se calculan por localidad. |
| **Vista minable** | Selección de variables, división cronológica y exportación de los conjuntos. |

**Archivos generados y valores de referencia:**

| Archivo | Periodo | Filas |
|---|---|---:|
| `ghi_multisitio_entrenamiento.csv` | 2004–2013 | 262.944 |
| `ghi_multisitio_validacion.csv` | 2014–2015 | 52.560 |
| `ghi_multisitio_prueba.csv` | 2016 (hasta el 26 de diciembre) | 25.992 |

**Consideraciones:** el ETL no utiliza aleatoriedad ni semillas; las particiones se definen por fechas fijas. Para verificar la reproducción deben compararse también columnas, fechas y valores, no solo las cantidades de filas.
# ETL de radiación solar multisitio — KNIME

Este directorio contiene el flujo de extracción, transformación y preparación de datos utilizado en el estudio **«Pronóstico multisitio de radiación solar global horizontal en la Región de Tarapacá mediante modelos estadísticos y de aprendizaje profundo»**.

El flujo procesa registros horarios de **Colchane, Pica y Pozo Almonte** (2004–2016), procedentes del [Explorador Solar del Ministerio de Energía de Chile](https://solar.minenergia.cl/), y genera los conjuntos empleados para entrenar y evaluar los modelos de regresión lineal, SARIMA, MLP y LSTM. Los registros corresponden a datos estimados/modelados disponibles en dicha plataforma; no deben describirse como mediciones instrumentales directas de las tres localidades.

## 1. Requisitos

- **KNIME Analytics Platform**. El archivo de flujo suministrado utiliza nodos cuya versión registrada es **5.11.0**; se recomienda comenzar con esa versión o una compatible.
- Nodos estándar de KNIME, incluidos **KNIME Base Nodes** y **KNIME Java Snippet Nodes** (para los separadores basados en reglas).
- Tres archivos CSV originales en `data/raw/`, con los nombres indicados a continuación.

Este ETL **no necesita Python ni `requirements.txt`** para ejecutarse.

## 2. Organización de archivos

```text
pronostico-solar-tarapaca/
├── data/
│   ├── raw/
│   │   ├── colchane.csv
│   │   ├── pica.csv
│   │   └── pozo_almonte.csv
│   └── processed/
│       ├── ghi_multisitio_entrenamiento.csv
│       ├── ghi_multisitio_validacion.csv
│       └── ghi_multisitio_prueba.csv
└── etl/
    ├── etl-series.knwf
    └── README.md
```

El archivo `.knwf` es un **flujo exportado** de KNIME. Al importarlo, KNIME crea una copia editable del flujo dentro de su espacio de trabajo (*workspace*). La ubicación real de esa copia determina cómo se resuelven las rutas relativas.

## 3. Importación y configuración

1. Descarga el repositorio completo, manteniendo las carpetas `data/` y `etl/`.
2. Abre KNIME Analytics Platform. Para el procedimiento seguido en este proyecto, configura como *workspace* la carpeta `pronostico-solar-tarapaca/` y abre **Local Space**.
3. Utiliza **Import workflow** para importar `etl/etl-series.knwf` al espacio de trabajo. **No es necesario abrir el `.knwf` con doble clic desde Windows.**
4. Abre el flujo importado. En los tres **CSV Reader** de la etapa *Extracción*, pulsa el selector de archivos y selecciona **Local space → Choose relative to: Workflow**. Apunta a los archivos de `data/raw/`.
5. En los tres **CSV Writer** de *Vista Minable*, utiliza el mismo criterio de ubicación relativa y apunta a `data/processed/` con los nombres señalados en la sección 2.
6. Guarda la configuración del flujo.

**Las rutas dependen de dónde se haya importado el workflow.** Por ejemplo:

| Ubicación real del flujo importado | Ruta de lectura de Colchane relativa al workflow |
| --- | --- |
| `pronostico-solar-tarapaca/ETL_pronostico_solar/` | `../data/raw/colchane.csv` |
| `pronostico-solar-tarapaca/etl/ETL_pronostico_solar/` | `../../data/raw/colchane.csv` |

Para los escritores, sustituye `data/raw/colchane.csv` por el archivo correspondiente dentro de `data/processed/`.

**No utilices rutas absolutas** como `C:\\Users\\...` ni confundas la ubicación del `.knwf` exportado con la ubicación del flujo importado. Si KNIME ofrece un selector visual relativo al workflow, es preferible utilizarlo.

**Si se utiliza el repositorio como workspace de KNIME, agrega `.metadata/` al `.gitignore` de la raíz**, para no publicar archivos internos de KNIME.

## 4. Etapas del flujo

| Etapa | Operaciones principales |
| --- | --- |
| **Extracción** | Lectura de los tres CSV, incorporación de la variable `comuna` y concatenación. |
| **Preprocesamiento (PPD)** | Conversión y renombrado de fecha/hora, filtrado de GHI válido, control de duplicados y ordenamiento por localidad y tiempo. |
| **Transformación** | Extracción de componentes temporales, creación de variables cíclicas, generación de `ghi_lag_1`, `ghi_lag_24` y del objetivo `ghi_t1`, trabajando por localidad, y codificación de la localidad. |
| **Vista minable** | Selección de las columnas finales, división cronológica y exportación de tres CSV. |

El objetivo **`ghi_t1`** es el GHI de la **hora siguiente**. Los rezagos `ghi_lag_1` y `ghi_lag_24` corresponden a valores históricos de una y 24 horas, respectivamente. Los indicadores binarios `colchane` y `pica` identifican localidades; **Pozo Almonte es la categoría de referencia**.

## 5. Ejecución

1. Comprueba que los tres **CSV Reader** pueden leer sus archivos originales.
2. Verifica que la carpeta `data/processed/` existe y que los **CSV Writer** apuntan a ella.
3. Si quieres preservar una ejecución anterior, haz una copia de los CSV procesados: los escritores están configurados para **sobrescribir archivos existentes**.
4. Restablece (*Reset*) el flujo si hay nodos ejecutados con configuraciones anteriores.
5. Ejecuta el flujo completo (**Execute**) y comprueba que finaliza sin errores.
6. Revisa los archivos resultantes en `data/processed/`.

**No hay particiones aleatorias ni nodos que requieran una semilla**: los registros se dividen mediante condiciones de fecha fijas. Con los mismos datos, configuraciones y un entorno compatible, el ETL debería producir resultados equivalentes.

## 6. Salidas y comprobaciones

Los archivos procesados contienen **16 columnas**:

```text
fecha_hora, ghi, temp, vel, cloud, comuna, dia_anio,
hora_sin, hora_cos, dia_anio_cos, dia_anio_sin,
ghi_lag_1, ghi_t1, ghi_lag_24, colchane, pica
```

La partición se realiza por la **fecha de la observación `fecha_hora`**, sin barajar los registros:

| Archivo | Periodo de referencia | Filas esperadas |
| --- | --- | ---: |
| `ghi_multisitio_entrenamiento.csv` | 02-01-2004 a 31-12-2013 | 262.944 |
| `ghi_multisitio_validacion.csv` | 01-01-2014 a 31-12-2015 | 52.560 |
| `ghi_multisitio_prueba.csv` | 01-01-2016 a 26-12-2016 | 25.992 |
| **Total** | | **341.496** |

Estos conteos corresponden a los CSV de referencia del experimento y sirven para verificar que el flujo produce la misma estructura y cantidad de filas. **Una ejecución completada sin errores no demuestra por sí sola la igualdad de todos los valores**: para una verificación rigurosa también deben compararse las columnas, fechas, localidades y valores exportados con los CSV de referencia.

### Observaciones sobre los límites temporales

- El conjunto de prueba conserva **tres filas finales con `ghi_t1` faltante** (una por localidad), ya que no existe una observación posterior disponible. Esas filas deben excluirse al evaluar pronósticos supervisados; **25.989** observaciones de prueba tienen objetivo disponible.
- Al construir `ghi_t1` antes de dividir por fecha, las últimas filas de entrenamiento y validación pueden utilizar como etiqueta el GHI de la primera hora del periodo siguiente (**seis casos entre ambas fronteras y las tres localidades**). El flujo original se conserva para documentar fielmente los experimentos publicados; si se decide modificar esta decisión metodológica, deben regenerarse los datos y reevaluarse los modelos, dejando constancia del cambio.

## 7. Actualización del flujo publicado

Después de modificar y probar rutas u otros parámetros en KNIME, **vuelve a exportar el workflow actualizado** y reemplaza `etl/etl-series.knwf` en el repositorio. Editar el flujo importado en el workspace **no actualiza automáticamente** el archivo `.knwf` original.

El flujo analizado inicialmente contenía rutas absolutas, por lo que es especialmente importante publicar **la exportación realizada después de corregirlas**. Para una comprobación final, importa el `.knwf` publicado en una instalación o espacio de trabajo limpio y repite la ejecución.

## 8. Procedencia y uso de datos

**Fuente:** [Explorador Solar — Ministerio de Energía de Chile](https://solar.minenergia.cl/).

Antes de distribuir públicamente los archivos CSV originales o derivados, comprueba las condiciones de uso y redistribución establecidas por la fuente de datos. Este README documenta el procedimiento técnico, pero no concede por sí mismo una licencia sobre los datos de terceros.

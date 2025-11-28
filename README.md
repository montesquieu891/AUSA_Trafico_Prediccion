# 🚦 Análisis y Predicción de Tráfico Urbano (AUSA - CABA)

## Resumen Ejecutivo

Este proyecto aplica técnicas avanzadas de Series Temporales (Time Series Analysis) para predecir el flujo vehicular por hora en la red de autopistas de la Ciudad de Buenos Aires (CABA). El foco se puso en la **AU 4 Lugones (Sentido A)**, identificada en la etapa exploratoria como la vía de **mayor caudal de tráfico promedio**.

El modelo **Prophet** (de Meta) fue entrenado con éxito para capturar la estacionalidad diaria, semanal y anual del tráfico, confirmando picos laborales críticos (8:00h y 17:00h) y revelando una **tendencia de crecimiento sostenido** en el volumen vehicular. La solución ofrece una herramienta clave para que AUSA pueda anticipar la congestión y optimizar la gestión de recursos.

---

## 🎯 Objetivos y Alcance del Proyecto

1.  **Limpieza Robusta:** Preparar y estandarizar datos históricos de flujo vehicular (147k+ registros) con inconsistencias de formato (fechas con separadores mixtos y strings numéricos).
2.  **Análisis Exploratorio (EDA):** Identificar patrones de congestión, horas pico y la vía más crítica para la predicción.
3.  **Modelado:** Construir y entrenar un modelo de Series Temporales (TSA) para generar un pronóstico de tráfico a 7 días.
4.  **Validación:** Descomponer la predicción para validar la lógica del modelo frente a fenómenos conocidos (ej. vacaciones de invierno).

---

## 🛠️ Metodología y Tecnologías

| Etapa | Enfoque Técnico | Herramientas Clave |
| :--- | :--- | :--- |
| **ETL & Limpieza** | Filtrado por Máscara Booleana. Estandarización de Separadores (`str.replace`), Manejo de Nulos. | **Pandas**, NumPy |
| **Análisis Exploratorio** | Agregación Temporal (`.groupby().mean()`). Creación de Variables Cíclicas (`dt.hour`, `dt.dayofweek`). | **Seaborn**, Matplotlib |
| **Modelado (TSA)** | Modelo Aditivo con Estacionalidad Semanal, Diaria y Anual. Uso de `make_future_dataframe`. | **Prophet** (Meta), Pandas |
| **Entorno** | Google Colab | Python 3.10+ |

---

## 📈 Resultados Clave del Modelado

El modelo entrenado reveló las siguientes dinámicas en el tráfico de la AU 4 Lugones (A):

| Patrón | Hallazgo Confirmado | Implicación |
| :--- | :--- | :--- |
| **Pico Diario** | La estacionalidad diaria es **bimodal**, con el pico máximo de congestión ocurriendo consistentemente entre las **17:00h y 18:00h**. | El control operativo y el peaje deben optimizarse para este intervalo de tiempo. |
| **Tendencia** | Se observa una **clara tendencia ascendente** en el volumen base de tráfico a lo largo de 2024. | La capacidad vial es un problema creciente que debe abordarse estratégicamente. |
| **Estacionalidad Anual** | El modelo asigna un valor predictivo **negativo significativo** al tráfico durante los meses de **Julio/Agosto** (período vacacional). | Confirma que la reducción del tráfico se debe al receso escolar/laboral. |

---

## 🚀 Próximos Pasos y Refinamiento Técnico (Hoja de Ruta)

Se identificaron limitaciones técnicas en el modelo actual. Las futuras modificaciones se enfocarán en incrementar la precisión predictiva (reducir la varianza residual) y abordar las restricciones del modelo aditivo estándar.

### 1. Refinamiento del Modelo Arquitectónico (Limitación $y \ge 0$)

El modelo aditivo actual genera predicciones negativas en los valles nocturnos.

* **A. Transformación Logarítmica:** Implementar una **transformación logarítmica** en la variable objetivo $y$ ($\ln(y+1)$) antes del entrenamiento. Esto estabiliza la varianza y asegura que la predicción transformada inversa ($y = e^{y'} - 1$) se mantenga **no negativa**, resolviendo el error de subestimación en los valles.
* **B. Modelo Multiplicativo:** Probar explícitamente el modelo multiplicativo de Prophet (`seasonality_mode='multiplicative'`), ya que la magnitud de los picos de tráfico puede ser proporcional al nivel base de la serie.

### 2. Reducción de la Varianza Residual (Regresores Externos)

La alta dispersión de los puntos históricos ($y - \hat{y} \ne 0$) indica que factores externos no modelados están afectando la predicción.

* **A. Datos Climáticos:** Integrar un dataset de **clima histórico** (ej. API de OpenWeatherMap) para crear regresores continuos:
    * `temperatura_maxima`: El calor extremo puede reducir la actividad.
    * `precipitacion_acumulada`: La **lluvia fuerte** (variable categórica/dummy) afecta negativamente la velocidad y el flujo.
* **B. Eventos Exógenos (Regresores Dummy):** Añadir regresores binarios (`extra_regressors`):
    * `es_feriado_puente`: Para aislar el impacto de los fines de semana largos.
    * `evento_masivo`: Marcar días con grandes conciertos o eventos deportivos que distorsionan el patrón normal.

### 3. Exploración de Modelos Alternativos para Datos de Conteo

El tráfico es inherentemente una serie de **datos de conteo** (números enteros no negativos).

* **A. Modelos de Conteos:** Explorar modelos estadísticos que manejen esta distribución, como la **Regresión de Poisson** o, más precisamente, la **Regresión Binomial Negativa** (`statsmodels` o librerías especializadas). Estos modelos ajustan la varianza de conteo de manera nativa sin requerir transformaciones manuales de la variable objetivo.

### 4. Operacionalización (MLOps Básico)

* **A. Pipeline de Producción:** Configurar un flujo de trabajo (workflow) automatizado (ej. usando **GitHub Actions** o **Airflow**) que se ejecute diariamente para:
    * Extraer nuevos datos de AUSA (si están disponibles en un endpoint).
    * Ejecutar el script de limpieza (`df.rename`, `pd.to_datetime`).
    * Generar el pronóstico de 7 días.
    * Guardar la predicción en una base de datos o almacenamiento en la nube (`AWS S3` o `Google Cloud Storage`).

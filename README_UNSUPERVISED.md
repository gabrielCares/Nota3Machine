# README - Módulo de Segmentación No Supervisada (Clustering)

## 1. Descripción Técnica y Justificación

En esta fase del proyecto, aplicamos técnicas de **Aprendizaje No Supervisado** para explorar la estructura subyacente de los datos de solicitantes de crédito. A diferencia del modelo supervisado, aquí no utilizamos la variable `TARGET` para entrenar, sino que buscamos agrupar a los clientes basándonos exclusivamente en la similitud de sus características financieras y demográficas.

### Técnica Seleccionada: K-Means Clustering
Se eligió el algoritmo **K-Means** por las siguientes razones técnicas y de negocio:

1.  **Naturaleza de los Datos:** Las variables más determinantes del dataset (`AMT_INCOME_TOTAL`, `AMT_CREDIT`, `AMT_ANNUITY`) son numéricas y continuas. K-Means, que opera mediante distancias euclidianas en un espacio vectorial, es ideal para este tipo de datos.
2.  **Escalabilidad:** K-Means es computacionalmente eficiente ($O(n)$), lo cual es crucial dado el volumen de registros en el dataset *Home Credit*.
3.  **Separabilidad de Negocio:** El objetivo es crear "Arquetipos de Clientes" (Personas). K-Means fuerza la creación de grupos distintos y no superpuestos, lo que facilita asignar una estrategia comercial única a cada segmento.

### Preprocesamiento Crítico
Antes del modelado, se ejecutó un pipeline estricto:
* **Imputación de Valores Nulos:** Se utilizó `SimpleImputer` (Estrategia: Mediana) para manejar datos faltantes sin perder registros, ya que K-Means no tolera valores `NaN`.
* **Estandarización (StandardScaler):** Paso obligatorio. Dado que el *Ingreso* (escala de millones) y la *Cantidad de Hijos* (escala de unidades) tienen magnitudes muy diferentes, se estandarizaron todas las variables a media 0 y desviación estándar 1. Sin esto, la variable de mayor magnitud dominaría falsamente la distancia y la formación de clusters.

---

## 2. Instrucciones de Ejecución

Este módulo es independiente del pipeline de predicción, pero genera insumos para él.

### Requisitos Previos
Asegúrese de tener instaladas las librerías listadas en `requirements.txt`:
* `pandas`, `numpy` (Manipulación de datos)
* `scikit-learn` (KMeans, PCA, StandardScaler)
* `matplotlib`, `seaborn` (Visualización)

### Pasos para ejecutar
1.  **Carga de Datos:** Coloque el archivo `application_.parquet` en el directorio `/content/` o en la raíz del proyecto.
2.  **Ejecución del Script:**
    Abra y ejecute el notebook correspondiente o el script de Python:
    ```bash
    # Si es un script .py
    python src/clustering_model.py
    ```
3.  **Flujo del Algoritmo:**
    * El script cargará las variables financieras clave.
    * Se ejecutará el **Método del Codo (Elbow Method)** para visualizar la inercia y ayudar a decidir el número óptimo de $k$ (se sugiere $k=4$ o $k=5$ para este dataset).
    * Se entrenará el modelo y se asignarán las etiquetas (`Cluster_0`, `Cluster_1`, etc.) a cada cliente.
    * Se generará una proyección **PCA (Principal Component Analysis)** para visualizar los grupos en 2 dimensiones.

---

## 3. Análisis e Interpretación de Resultados

Tras la ejecución del modelo con $k=4$ clusters, se han identificado los siguientes perfiles financieros (basados en los centroides promedio):

| Cluster | Perfil Sugerido | Características Dominantes |
| :--- | :--- | :--- |
| **0** | **Base Masiva (Bajo Riesgo)** | Ingresos medios-bajos, montos de crédito conservadores. Es el grupo más numeroso. Suelen tener empleo estable y pocos hijos. |
| **1** | **Alto Valor (Premium)** | Ingresos muy superiores al promedio (>2.5 desviaciones estándar). Solicitan créditos altos pero tienen amplia capacidad de pago (Ratio Endeudamiento bajo). |
| **2** | **Alto Riesgo / Sobreendeudados** | Ingresos bajos o inestables, pero con solicitudes de crédito (`AMT_CREDIT`) desproporcionadamente altas. Este grupo suele correlacionar con una mayor tasa de morosidad en el análisis posterior. |
| **3** | **Familias Jóvenes / Consumo** | Clientes más jóvenes, con mayor cantidad de hijos (`CNT_CHILDREN`) y créditos de consumo a corto plazo (`AMT_ANNUITY` alta en relación al crédito total). |

**Validación Visual:**
La reducción de dimensionalidad mediante **PCA** muestra una separación clara entre el Cluster "Premium" y la "Base Masiva", aunque existe cierta superposición en las fronteras de los grupos de riesgo medio, lo cual es esperable en datos financieros reales.

---

## 4. Discusión: Incorporación al Proyecto Final

**¿Se debe incorporar este método al modelo final de Scoring Supervisado?**

**Decisión: SÍ.**

**Justificación de la Integración:**

1.  **Ingeniería de Características (Feature Engineering):**
    El Cluster asignado no es solo un resultado descriptivo; se convierte en una **nueva variable predictiva** (`Cluster_ID`). Al alimentar esta variable al modelo supervisado (Random Forest/XGBoost), le estamos dando al algoritmo una versión "resumida" de la posición socioeconómica del cliente, lo que a menudo mejora el rendimiento del modelo.

2.  **Captura de No-Linealidades:**
    Los modelos lineales tradicionales a veces fallan en detectar interacciones complejas (ej: *Tener altos ingresos es bueno, PERO si tienes altos ingresos y pides un crédito pequeño a corto plazo, podrías ser sospechoso de fraude*). El clustering captura estas interacciones multidimensionales y las condensa en una sola etiqueta.

3.  **Estrategias de Negocio Diferenciadas:**
    Más allá de la predicción de `0` o `1`, el clustering permite al banco aplicar políticas matizadas:
    * *Para el Cluster 1 (Premium):* Ofrecer tasas preferenciales o Cross-selling (Venta cruzada).
    * *Para el Cluster 2 (Riesgo):* Solicitar avales adicionales o limitar el monto máximo, incluso si el modelo de scoring les da un "Aprobado" marginal.

**Conclusión:**
La segmentación K-Means se integra como un paso de preprocesamiento avanzado. La etiqueta del cluster se utilizará como *input* para el modelo final de clasificación de riesgo.
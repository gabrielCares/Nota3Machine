# README - Módulo de Segmentación No Supervisada (Clustering) y Análisis de Riesgo

## 1. Descripción Técnica y Justificación

En esta fase del proyecto se aplicaron técnicas de **Aprendizaje No Supervisado** (K-Means) para segmentar a los clientes, complementadas con un modelo **Supervisado** (Random Forest) para validar la capacidad predictiva de las variables y el riesgo asociado a cada segmento.

### Técnica Principal: K-Means Clustering
Se eligió el algoritmo **K-Means** por las siguientes razones:
1.  **Naturaleza de los Datos:** Las variables financieras críticas (`AMT_CREDIT`, `AMT_INCOME`) son continuas, ideales para la distancia euclidiana.
2.  **Separabilidad de Negocio:** Permite crear "Arquetipos de Clientes" distintos para aplicar políticas de riesgo diferenciadas.

### Preprocesamiento
* **Imputación:** Estrategia de Mediana para manejo de nulos.
* **Estandarización:** `StandardScaler` para evitar que variables de gran magnitud (Ingresos) dominen el cálculo de distancias.

---

## 2. Instrucciones de Ejecución

### Requisitos
Instalar dependencias: `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`.

### Pasos
1.  **Carga de Datos:** Asegúrese de tener `application_.parquet` y `bureau.parquet` en la ruta del proyecto.
2.  **Ejecución:**
    ```bash
    python src/clustering_analysis.py
    ```
3.  **Salidas:** El script genera visualizaciones PCA, la asignación de clusters y el reporte de clasificación supervisada.

---

## 3. Análisis e Interpretación de Resultados

A continuación se detallan los hallazgos basados en la ejecución del modelo con los datos reales:

### A. Segmentación de Clientes (Perfiles Identificados)
El análisis detectó 4 clusters con comportamientos de riesgo muy diferenciados, destacando la presencia de datos atípicos.

| Cluster | Perfil del Cliente | Análisis de Riesgo |
| :--- | :--- | :--- |
| **Cluster 0** | **Base Masiva** (136k clientes)<br>Ingresos medios/bajos, créditos moderados. | **ALTO RIESGO (9.52% Mora).** Es el grupo con la mayor tasa de impago. Requiere políticas de aprobación estrictas. |
| **Cluster 1** | **Bajo Riesgo** (42k clientes)<br>Créditos medios, perfil conservador. | **BAJO RIESGO (5.38% Mora).** Clientes seguros. Candidatos ideales para *cross-selling* y aprobación rápida. |
| **Cluster 2** | **Premium / Alto Valor** (67k clientes)<br>Ingresos altos y montos de crédito muy elevados (> 1M). | **RIESGO MEDIO (6.83% Mora).** A pesar de tener altos ingresos, el alto endeudamiento eleva su riesgo por encima del Cluster 1. |
| **Cluster 3** | **Outlier (Anomalía)**<br>Solo 1 cliente con ingreso extremo ($117M). | **DATO ATÍPICO.** Este registro distorsiona el análisis y debe ser eliminado en la próxima iteración de limpieza. |

### B. Validación Supervisada (Capacidad Predictiva)
Para confirmar la utilidad de estos datos, se entrenó un modelo Random Forest obteniendo las siguientes métricas:

* **Capacidad de Discriminación (AUC = 0.61):** El modelo tiene una capacidad moderada para distinguir morosos. Funciona mejor como herramienta de apoyo ("Semáforo de Riesgo") que como decisor automático único.
* **Detección de Morosos (Recall = 0.50):** Se logra identificar correctamente al **50% de los morosos reales** (2,462 casos detectados).
* **Costo de Oportunidad (Precision = 0.11):** La política agresiva de detección genera un alto número de falsos positivos (clientes buenos marcados como riesgo).

### C. Factores Determinantes del Riesgo (Feature Importance)
El análisis confirma que el riesgo no depende de *quién* es el cliente, sino de *su deuda*:
1.  **Carga Financiera (~70% de peso):** Las variables `AMT_CREDIT` (36%) y `AMT_ANNUITY` (33%) son las más decisivas. **Conclusión:** Un cliente se vuelve riesgoso cuando su cuota es desproporcionada, sin importar sus ingresos.
2.  **Ingresos (`AMT_INCOME` 14%):** Es un factor secundario frente al nivel de deuda.

---

## 4. Conclusión y Estrategia de Integración

**Decisión:** Incorporar la segmentación al modelo final de Scoring.

**Razones de Negocio:**
1.  **Estrategia Diferenciada:** Los resultados validan que no se puede tratar igual al "Cluster 0" (9.5% riesgo) que al "Cluster 1" (5.3% riesgo). El cluster actúa como una variable categórica potente para el modelo final.
2.  **Acción Recomendada:** Utilizar el modelo para **solicitar garantías adicionales** a los perfiles de riesgo detectados (especialmente en Clusters 0 y 2), en lugar de rechazar automáticamente, dado el margen de precisión actual.
3.  **Limpieza:** Es imperativo filtrar el "Cluster 3" (outliers) para estabilizar los centros de los grupos.

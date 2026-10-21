# Churn Prediction – End-to-End ML Project (Sector Bancario)

Modelo de Machine Learning que predice qué clientes de un banco tienen mayor probabilidad de irse (churn), los segmenta por nivel de riesgo y estima el valor de una campaña de retención enfocada.

## 🎯 Resultados clave

| Indicador | Resultado |
|---|---|
| Modelo final | XGBoost con tuning de hiperparámetros |
| ROC-AUC (test) | **0.871** (0.866 en validación cruzada de 5 folds) |
| Segmento de riesgo alto (20% de los clientes) | concentra el **63% de las fugas**, ~3 veces más que una selección al azar |
| Contactar al 30% de mayor riesgo | alcanza el **76% de las fugas** |
| Principales factores de fuga | edad, inactividad y número de productos |

## 💼 Problema de negocio

Retener a un cliente cuesta menos que captar uno nuevo, pero contactar a toda la cartera es caro e ineficiente. El objetivo es responder a tres preguntas:

1. ¿Qué clientes tienen mayor probabilidad de irse?
2. ¿Qué características los diferencian?
3. ¿A cuántos clientes conviene contactar para maximizar el retorno de una campaña de retención?

## 📊 Dataset

- **Fuente:** Kaggle – *Churn Modelling* (`Churn_Modelling.csv`), clientes de un banco europeo (Francia, España y Alemania).
- **Registros:** 10,000 clientes, 14 columnas (edad, balance, antigüedad, n.º de productos, actividad, país, etc.).
- **Target:** `Exited` (1 = el cliente se fue, 0 = se quedó). Tasa de churn ≈ 20%.
- **Calidad:** sin valores faltantes.

## 🔬 Metodología

### 1. Preprocesamiento
- Se eliminaron identificadores sin valor predictivo (`RowNumber`, `CustomerId`, `Surname`).
- `Gender` se codificó como 0/1 y `Geography` con one-hot encoding (`drop_first`).
- División estratificada 80/20 (`random_state=42`): 8,000 clientes para entrenamiento y 2,000 para test.

### 2. Corrección de data leakage
La primera versión ajustaba el `StandardScaler` **antes** de dividir los datos, de modo que información del test se filtraba al entrenamiento. En la versión final:
- Se eliminó el escalado, innecesario para modelos de árboles.
- La evaluación se hace con validación cruzada de 5 folds sobre el set de entrenamiento.

### 3. Comparación de modelos

| Modelo | ROC-AUC |
|---|---|
| Logistic Regression | 0.766 |
| Random Forest | 0.853 |
| LightGBM | 0.858 |
| XGBoost | 0.860 |
| **XGBoost tuneado** | **0.866 (CV) / 0.871 (test)** |

### 4. Tuning y desbalance de clases
- `RandomizedSearchCV` con 40 combinaciones y validación cruzada de 5 folds.
- Para el desbalance se usó `scale_pos_weight` en lugar de SMOTE, porque SMOTE rindió peor en validación cruzada (0.841 frente a 0.866).
- El tuning redujo la brecha entre entrenamiento y validación de ~0.12 a 0.04 (menos sobreajuste).

## 📈 Segmentación por riesgo

Los 2,000 clientes de test se ordenaron por su puntaje de riesgo y se dividieron en tres segmentos:

| Segmento | % de clientes | Churn real | % del total de fugas |
|---|---|---|---|
| 🔴 Alto | 20% | 64.0% | 63% |
| 🟡 Medio | 30% | 18.0% | 27% |
| 🟢 Bajo | 50% | 4.3% | 10% |

## 🔍 Factores de fuga (SHAP y perfiles)

- **Edad:** es el factor más fuerte. El segmento Alto promedia 47.5 años, frente a 34.7 en el Bajo.
- **Número de productos:** relación no lineal. Con 1 producto el churn es de 26.7%, con 2 baja a 8.4%, y con 3–4 sube a 85–100% (aunque son solo 69 clientes).
- **Inactividad:** solo el 32% del segmento Alto es miembro activo, frente al 61% del Bajo.
- **País:** Alemania tiene casi el doble de churn (~30% frente a ~17% en el resto).
- **Salario y credit score** casi no diferencian a quienes se van.

## 💰 Estimación del valor de una campaña de retención

> ⚠️ **Supuestos ilustrativos**, no datos reales: valor de 1,500 EUR por cliente retenido, costo de 50 EUR por contacto y 25% de éxito de la campaña.

| Estrategia | Fugas alcanzadas | Beneficio neto estimado |
|---|---|---|
| **Contactar al 30% de mayor riesgo** | **76%** | **85,875 EUR** |
| Contactar a todos los clientes | 100% | 52,625 EUR |
| Contactar a un 30% elegido al azar | ~30% | ~15,800 EUR |

Enfocar la campaña con el modelo genera más beneficio que contactar a toda la cartera, con menos del tercio del costo.

## 📊 Dashboard en Power BI

Archivo: `dashboard/ChurnDashboard.pbix`. Se alimenta de `customers_risk.csv` (2,000 clientes de test con puntaje, segmento y `CustomerId`) y `model_metrics.csv`. Las medidas DAX (total de clientes, churners, tasa de churn, clientes en riesgo alto, % de fugas capturadas y un mensaje clave dinámico) se verificaron contra los resultados en Python.

- ✅ **Página 1 – Resumen ejecutivo:** KPIs principales, churn por segmento, distribución de las fugas y filtro por país.
- 🔄 **Página 2 – Drivers del churn** (en progreso)
- 🔄 **Página 3 – Clientes en riesgo y simulador de ROI** (en progreso)

## ⚠️ Limitaciones

- Los supuestos económicos del análisis de ROI son ilustrativos.
- Por el uso de `scale_pos_weight`, el puntaje del modelo sirve para **ordenar** a los clientes por riesgo, no como una probabilidad calibrada.
- Los factores identificados muestran **asociaciones, no causalidad**.
- El género es una variable sensible: su uso en decisiones reales requeriría una revisión de sesgo y equidad.
- Dataset público de un banco europeo; los patrones podrían no trasladarse a otros mercados.

## 📁 Estructura del proyecto

```
ChurnPrediction/
├── data/
│   ├── raw/                 # Dataset original (Churn_Modelling.csv)
│   └── processed/           # Datos procesados y salidas para el dashboard
├── notebooks/
│   ├── 01_EDA.ipynb
│   ├── 02_Data_Preprocessing.ipynb
│   ├── 03_Modeling.ipynb
│   ├── 03b_Tuning.ipynb
│   └── 04_Analysis.ipynb
├── models/
│   └── xgboost_tuned.pkl    # Modelo final
├── dashboard/
│   └── ChurnDashboard.pbix
└── requirements.txt
```

## 🛠️ Tech Stack

- **Lenguaje:** Python 3.10+
- **Procesamiento de datos:** Pandas, NumPy
- **Visualización:** Matplotlib, Seaborn, Power BI (DAX)
- **Machine Learning:** scikit-learn, XGBoost, LightGBM
- **Interpretabilidad:** SHAP
- **Entorno:** Jupyter Notebook

## 🚀 Cómo ejecutar

1. Instala las dependencias:
   ```bash
   pip install -r requirements.txt
   ```
2. Abre Jupyter:
   ```bash
   jupyter notebook
   ```
3. Ejecuta los notebooks de `notebooks/` en orden: `01_EDA` → `02_Data_Preprocessing` → `03_Modeling` → `03b_Tuning` → `04_Analysis`.
4. Abre `dashboard/ChurnDashboard.pbix` en Power BI Desktop.

## 📋 Estado del proyecto

- [x] Fase 0: Setup y planificación
- [x] Fase 1: EDA
- [x] Fase 2: Preprocesamiento
- [x] Fase 3: Modelado y tuning
- [x] Fase 4: Análisis de resultados (segmentación, SHAP y ROI)
- [ ] Fase 5: Dashboard en Power BI (página 1 de 3 terminada)
- [ ] Fase 6: Documentación final y presentación

## 👤 Autor

**Ysrael Zurita** – Estudiante de Ingeniería de Sistemas, Universidad de Lima
[LinkedIn](https://www.linkedin.com/in/ysrael-zurita-solano) · [GitHub](https://github.com/YsraelZ14)

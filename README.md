# Churn Prediction - End-to-End ML Project

## 📊 Descripción
Proyecto de predicción de churn (abandono de clientes) en el sector bancario utilizando técnicas de Machine Learning.

**Objetivo:** Predecir qué clientes tienen mayor probabilidad de abandonar el banco y proporcionar insights para estrategias de retención.

## 📁 Estructura del Proyecto
```
ChurnPrediction/
├── data/ # Datos (raw y procesados)
├── notebooks/ # Jupyter notebooks con análisis
├── src/ # Código Python (funciones reutilizables)
├── models/ # Modelos entrenados (.pkl)
├── reports/ # Reportes, figuras y resultados
└── requirements.txt # Dependencias de Python 
```

## 🛠️ Tech Stack
- **Lenguaje:** Python 3.10+
- **Data Processing:** Pandas, NumPy
- **Visualización:** Matplotlib, Seaborn
- **Machine Learning:** scikit-learn, XGBoost, LightGBM
- **Interpretabilidad:** SHAP
- **Notebooks:** Jupyter

## 📋 Fases del Proyecto

1. **FASE 0:** Setup & Planning ✅
2. **FASE 1:** EDA (Exploratory Data Analysis)
3. **FASE 2:** Preprocessing
4. **FASE 3:** Modelado
5. **FASE 4:** Análisis de Resultados
6. **FASE 5:** PowerBI Dashboard
7. **FASE 6:** Documentación & Presentación

## 🚀 Cómo ejecutar

1. Instala dependencias:
```bash
   pip install -r requirements.txt
```

2. Abre Jupyter:
```bash
   jupyter notebook
```

3. Abre los notebooks en `notebooks/` en orden (01_EDA, 02_Preprocessing, etc.)

## 📊 Dataset
- **Fuente:** Kaggle - Bank Customer Churn Modeling
- **Registros:** 10,000 clientes
- **Features:** 14 columnas (edad, balance, antigüedad, etc.)
- **Target:** Exited (1 = cliente se fue, 0 = se quedó)

## 📝 Autor
Tu Nombre - Ingeniería de Sistemas, Universidad de Lima

---

*Proyecto para prácticas pre-profesionales en Data Analytics - Sector Bancario*
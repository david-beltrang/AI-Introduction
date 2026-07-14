# FourthProject — Clasificación de Customer Churn con ML supervisado

## Objetivo

Predecir si un cliente de telecomunicaciones va a cancelar su servicio (*churn*) usando cuatro modelos de clasificación distintos. El objetivo es comparar su rendimiento y entender qué variables del perfil del cliente influyen más en la decisión de irse.

---

## Dataset

**Archivo:** `WA_Fn-UseC_-Telco-Customer-Churn.csv`  
**Fuente:** IBM Watson Analytics Sample Data  
**Tamaño original:** 7 043 filas × 21 columnas → 7 010 filas × 20 columnas tras limpieza  
**Variable objetivo:** `Churn` (Yes/No → 1/0)  
**Desbalance de clases:** ~73.5% No Churn / ~26.5% Churn

El dataset representa clientes de una empresa de telecomunicaciones. Cada fila es un cliente con información sobre los servicios que contrató, su tipo de contrato, método de pago y cuánto tiempo lleva como cliente. La tarea es predecir si ese cliente se va a ir o no.

---

## Preprocesamiento común (todos los notebooks)

Todos los modelos parten del mismo pipeline de limpieza:

1. **Eliminar `customerID`** — es un identificador aleatorio, no aporta nada al modelo.
2. **Convertir `TotalCharges` a número** — viene como texto (`object`) porque los clientes con 0 meses de antigüedad tienen el campo vacío. Se usa `pd.to_numeric(..., errors='coerce')` y se eliminan esas ~11 filas.
3. **Eliminar duplicados** — se encontraron 22 filas duplicadas.
4. **One-Hot Encoding** — la mayoría de variables son categóricas (Yes/No, tipo de contrato, etc.). Se convierten a columnas de 0s y 1s para que el modelo pueda usarlas.
5. **División train/test** — 80% para entrenar, 20% para evaluar, usando `stratify=y` para mantener la proporción de clases en ambos conjuntos.
6. **Escalado con `StandardScaler`** — obligatorio para KNN y SVM, que son sensibles a la escala de los datos. El árbol de decisión no lo necesita pero algunos notebooks lo incluyen igual.

---

## Modelos implementados

### 1. Árbol de Decisión — `decision_tree_model.ipynb`

Un árbol de decisión aprende una serie de reglas del estilo *"si el contrato es mes a mes Y el tenure es menor a 12 meses → probablemente se va"*. Divide el dataset de forma recursiva según la variable que mejor separa las clases en cada paso, usando el índice Gini como criterio de impureza.

**Ventaja:** fácil de interpretar, se puede visualizar el árbol completo y ver exactamente qué reglas usa.  
**Riesgo principal:** overfitting — si el árbol crece demasiado, memoriza el conjunto de entrenamiento en lugar de generalizar.

**Lo que hace el notebook:**
- Entrena 4 árboles con distintas configuraciones de `max_depth`, `min_samples_split` y `min_samples_leaf` para controlar el overfitting.
- Compara el accuracy en train vs test para detectar sobreajuste.
- Visualiza el árbol del mejor modelo.
- Muestra la importancia de cada variable según cuánto contribuyó a las divisiones del árbol.

**Parámetros clave usados:**
- `criterion='gini'`
- `max_depth=5`
- `min_samples_split=15`
- `min_samples_leaf=10`

---

### 2. K-Nearest Neighbors (KNN) — `knn_model.ipynb`

KNN no construye un modelo explícito. Para clasificar un cliente nuevo, busca los K clientes más similares en el conjunto de entrenamiento (usando distancia euclidiana) y asigna la clase mayoritaria entre ellos. Es un algoritmo *lazy* — todo el cómputo ocurre en el momento de predecir.

**Ventaja:** simple de entender y no hace suposiciones sobre la distribución de los datos.  
**Riesgo principal:** muy sensible a la escala de las variables (por eso se escala obligatoriamente) y lento con datasets grandes. Además, el valor de K afecta mucho el resultado.

**Lo que hace el notebook:**
- Aplica transformación logarítmica a `TotalCharges` para reducir el sesgo de la distribución.
- Usa el **método del codo** para encontrar el K óptimo: entrena KNN con K de 1 a 20 y grafica el error para identificar el punto donde agregar más vecinos ya no mejora el modelo.
- Entrena dos versiones: una con todas las features y otra con reducción de dimensionalidad usando `ExtraTreesClassifier` para seleccionar solo las variables más importantes.
- Compara ambas versiones con matriz de confusión, métricas (precision, recall, F1) y curvas ROC.

---

### 3. Support Vector Machine (SVM) — `svm_model.ipynb`

Una SVM busca el hiperplano que mejor separa las dos clases (Churn / No Churn) dejando el mayor margen posible entre ellas. Para datos que no son linealmente separables, usa una función *kernel* que transforma los datos a un espacio de mayor dimensión donde sí se pueden separar.

**Ventaja:** funciona bien en espacios de alta dimensión y es robusto cuando hay pocas muestras.  
**Riesgo principal:** lento de entrenar con datasets grandes y sensible a la escala y al desbalance de clases.

**Lo que hace el notebook:**
- Entrena 4 variantes: kernel RBF y polinomial, con y sin `class_weight='balanced'` (para compensar el desbalance de clases).
- Usa `GridSearchCV` con validación cruzada de 5 pliegues para encontrar los mejores hiperparámetros (`C`, `kernel`).
- Aplica reducción de features con `SelectFromModel` + `ExtraTreesClassifier` y reentrena la SVM con las variables más importantes.
- Incluye una sección de log-transform en `TotalCharges` para reducir el sesgo antes de escalar.
- Evalúa cada variante con accuracy, reporte de clasificación y matriz de confusión.

---

### 4. Red Neuronal (MLP) — `neural_network_model.ipynb`

Una red neuronal de tipo MLP (Multi-Layer Perceptron) aprende representaciones complejas de los datos pasándolos por capas de neuronas con funciones de activación no lineales. Cada neurona aprende una combinación de las entradas y el proceso se repite capa a capa hasta producir una probabilidad de churn en la salida.

**Ventaja:** puede capturar relaciones no lineales complejas entre variables que otros modelos no detectan.  
**Riesgo principal:** más difícil de interpretar, necesita más datos y es más sensible a los hiperparámetros.

**Arquitectura usada:**
```
Entrada (30 variables)
    → Capa densa: 64 neuronas, activación ReLU
    → Dropout (0.3)
    → Capa densa: 32 neuronas, activación ReLU
    → Dropout (0.2)
    → Salida: 1 neurona, activación Sigmoid → probabilidad de churn
```

**Lo que hace el notebook:**
- Aplica `np.log1p` a `TotalCharges` antes de escalar (la distribución tiene sesgo positivo).
- Usa `stratify=y` en el split por el desbalance de clases.
- Entrena el modelo base y evalúa los resultados (accuracy ~0.80, pero Recall de Churn solo ~0.50).
- Reentrrena con `class_weight={0: 1.0, 1: 2.8}` — el peso 2.8 viene de la proporción del desbalance (1031/371 ≈ 2.78). Esto le dice al optimizador que equivocarse con un cliente que se va cuesta 2.8 veces más que equivocarse con uno que se queda. El Recall de Churn sube a ~0.74.
- Prueba una tercera variante con Dropout reducido (0.1) para ver si retener más información mejora el resultado.
- Monitorea el entrenamiento con curvas de pérdida y accuracy para detectar overfitting.

---

## Comparación de enfoques

| Modelo | Interpretable | Necesita escalado | Maneja desbalance | Velocidad |
|---|---|---|---|---|
| Árbol de decisión | Sí (visualizable) | No | Parcial | Rápido |
| KNN | No | Sí (obligatorio) | No directamente | Lento en predicción |
| SVM | No | Sí (obligatorio) | Con `class_weight` | Medio |
| Red neuronal | No | Sí | Con `class_weight` | Medio (con GPU rápido) |

---

## Cómo ejecutar

Los notebooks están preparados para Google Colab. Cada uno carga el dataset directamente desde una URL pública.

```bash
# En Colab: abrir el notebook y ejecutar todas las celdas (Ctrl+F9)

# En local:
pip install pandas numpy matplotlib seaborn scikit-learn tensorflow kagglehub
jupyter notebook
```

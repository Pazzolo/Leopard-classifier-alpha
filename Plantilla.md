# Proyecto ML — Plantilla genérica

> Extraído de Notion el 2026-11-15
> Fuente: https://pointy-whistle-142.notion.site/Proyecto-ML-734cbbc3faae41c9a3f2425105baab07
> Contenido: página principal · base de datos **Ítems Proyecto** (11 ítems) · base de datos **Criterios** (rúbrica, 7 niveles)

---

## 1. Página principal

El objetivo de este proyecto es usar machine learning para resolver un problema usando la metodología correcta aprendida en el curso.

Se deberá implementar el proyecto en una notebook de Google Colab donde se consideren los siguientes aspectos:

Recuerde que no existe necesidad de hacer un informe asociado a su proyecto. Todo debe estar en la notebook.

Crear un repositorio en `GitHub` con un README pertinente.

- 📄 Enunciado del proyecto específico → ver sección 2 (por definir)
- 🗂️ Base de datos **Ítems Proyecto** → ver sección 3
- 🗂️ Base de datos **Criterios** (Rúbrica) → ver sección 4

### Rúbrica (texto de la página)

Los estudiantes deberán obligatoriamente defender su proyecto. Si algún miembro del grupo no se presenta el día de la defensa, el estudiante automáticamente tendrá la nota de 0, independientemente si el trabajo está completo o no. La nota es por grupo, no por estudiante.

Las puntuaciones de cada ítem arriba solicitado serán evaluados de manera independiente bajo los siguientes criterios: *(ver sección 4)*

---

## 2. Proyecto específico

> **Tema:** _por definir_
>
> Aquí va el enunciado del proyecto concreto: descripción, objetivos, dataset, grupos y entregables.

---

## 3. Ítems Proyecto (base de datos)

### Resumen

| # | Ítem | Puntuación Máxima | Descripción | Enlace |
| --- | --- | --- | --- | --- |
| 1 | Introducción | 0.1 | Realizar una introducción donde motive el problema a resolver y su importancia. | [link](https://pointy-whistle-142.notion.site/0b329a4c1d3c46939c8d3fb136cfd80f) |
| 2 | EDA | 0.6 | Realizar un "Exploratory Data Analysis" de su dataset | [link](https://pointy-whistle-142.notion.site/1e505737435780f687d9e846fb9b7beb) |
| 3 | Feature Extraction | 0.25 | Para datasets con data no estructurada cuando se usa transfer learning | [link](https://pointy-whistle-142.notion.site/325057374357802f8ac5ef311259e885) |
| 4 | Feature Selection | 0.25 | Implementar una técnica de feature selection (tipo filter) en su pipeline. En caso de usar transfer learning o si su problema es sobre imágenes o texto, pueden omitir este paso. | [link](https://pointy-whistle-142.notion.site/ce578c1299be4924bfba95c2350bc404) |
| 5 | Optimización de hiperpámetros | 3.5 | Optimizar hiperparámetros con repeated k-fold CV; gráfica de optimización y curva de aprendizaje. (texto completo abajo) | [link](https://pointy-whistle-142.notion.site/64b977f1fc294a9b993e43473c01b643) |
| 6 | Comparación estadística de 2 técnicas ML | 3.5 | Test de hipótesis (Wilcoxon o t-test) con repeated k-fold CV. (texto completo abajo) | [link](https://pointy-whistle-142.notion.site/3a9244cc6e284d68b5a35f6e3ad543b2) |
| 7 | Pipelines | 0.5 | Usar pipelines de sklearn (o estructura modular en redes neuronales). | [link](https://pointy-whistle-142.notion.site/65f7be5c47074222a9dd9b765c9e2dbd) |
| 8 | Visualización con TSNE | 1 | Usar TSNE para visualizar el espacio de características en 2D; color = label. | [link](https://pointy-whistle-142.notion.site/28254079a21447d2a0a704bfddb5e4f7) |
| 9 | Idioma | 0.1 | Uso del idioma (ortografía y redacción) | [link](https://pointy-whistle-142.notion.site/7f2d965ab8544b69ab7981ce35041cd7) |
| 10 | Referencias | 0.1 | Referencias en formato IEEE (conceptos y código); prompts de ChatGPT como comentarios. | [link](https://pointy-whistle-142.notion.site/e7ee716e56254cf6817fb059b44469ab) |
| 11 | Conclusiones | 0.1 | Redactar conclusiones pertinentes. Recordar que la teoría no es una conclusión | [link](https://pointy-whistle-142.notion.site/2132bef1ca084f42b02fa2454d5d2598) |
| | **Total** | **10.0** | | |

**Propiedades de la base de datos:** `Name` (título) · `Puntuación Máxima` (número) · `Descripción` (texto) · `Criterio Asignado` (relación → Criterios) · `Implementación`, `Defensa`, `Comentario` (rollups del criterio) · `Ponderación` (rollup: suma de `Porcentaje Asignado` del criterio) · `Puntuación Obtenida` (fórmula).

**Fórmula `Puntuación Obtenida`:**

```
Puntuación Obtenida = Puntuación Máxima × Ponderación / 100
```

### Detalle por ítem

#### 3.1 Introducción — 0.1

<sub>https://pointy-whistle-142.notion.site/0b329a4c1d3c46939c8d3fb136cfd80f</sub>

**Descripción:** Realizar una introducción donde motive el problema a resolver y su importancia.

1. Contexto del problema
2. Aterrizan a lo que van a hacer.
3. Qué van a hacer en resumen.

#### 3.2 EDA — 0.6

<sub>https://pointy-whistle-142.notion.site/1e505737435780f687d9e846fb9b7beb</sub>

**Descripción:** Realizar un "Exploratory Data Analysis" de su dataset

Mínimo se desea responder lo siguiente:

1. ¿Es su data balanceada?
2. ¿Está normalizada?
3. ¿Hay estacionalidad en series de tiempo?
4. ¿Necesito hacer transformaciones no lineales (skewness)?
5. Si el dominio es imágenes:
   1. Distribución labels
   2. Grid de las imágenes
   3. Histograma de la resolución (H x W)
6. Si el dominio es texto, después del preprocesamiento de lemmatization and stop words removal:
   1. Word cloud
   2. Palabras frecuentes

#### 3.3 Feature Extraction — 0.25

<sub>https://pointy-whistle-142.notion.site/325057374357802f8ac5ef311259e885</sub>

**Descripción:** Para datasets con data no estructurada cuando se usa transfer learning

No estructurada (no tabular): imágenes, text, audio, etc.

**Texto**
1. Opción 1 clásica: Text → Bag of words TF-IDF → **ML algorithm**
2. Opción 2: Text → Pretrained model → vector embedding → ML Algorithm (Transfer Learning)
   1. BERT (Transformer)
3. Otros (sugeridos por ustedes)

**Imágenes**
1. Opción 1 clásica: Image → flattening → PCA → **ML Algorithm**
2. Opción 1 clásica: Image → Image Descriptors (e.g., SURF, SIFT, HoG) + Bag of Visual Words BOVW → **ML Algorithm**
3. Opción 2: Image → Pretrained model → vector embedding (feature vector) → ML Algorithm (Transfer Learning)
   1. CNN: Resnet, Mobilenet
   2. ViTs: Visual Transformers
4. Otros (sugeridos por ustedes)

**Audio**
1. Audio → DFT Magnitude (discrete fourier Transform) → ML Algorithm
2. Audio → MFCCs → ML Algorithm
3. Audio → Spectrogram (imagen 2D) → ver arriba los pipelines de imágenes
4. Audio → Pretrained model → vector embedding (feature vector) → ML algorithm (Transfer Learning)
   1. wave2vec 2.0
   2. wavenet
5. Otros (sugeridos por ustedes)

**Time series**
1. Lags
2. ARIMA
3. Time series → Pretrained model → vector embedding (feature vector) → ML algorithm (Transfer Learning)
   1. Prophet
   2. Chronos
4. Otros (sugeridos por ustedes)

**Tabular**
1. Tabular → Transformers tabulares → embedding → ML algorithm (Transfer Learning)
   1. TabPFN
   2. TabNet

#### 3.4 Feature Selection — 0.25

<sub>https://pointy-whistle-142.notion.site/ce578c1299be4924bfba95c2350bc404</sub>

**Descripción:** Implementar una técnica de feature selection (tipo filter) en su pipeline. En caso de usar transfer learning o si su problema es sobre imágenes o texto, pueden omitir este paso.

- 🔗 [1.13. Feature selection — scikit-learn](http://scikit-learn.org/stable/modules/feature_selection.html)

#### 3.5 Optimización de hiperpámetros — 3.5

<sub>https://pointy-whistle-142.notion.site/64b977f1fc294a9b993e43473c01b643</sub>

**Descripción:** Optimizar sus hiperparámetros en un conjunto de validación adecuado con `repeated k-fold cv`. Aunque se puede usar nested k fold CV, **no se recomienda** porque la ejecución puede demorar considerablemente más. Presentar una gráfica de la optimización de algún hiperparámetro de su modelo (ver figura abajo para un ejemplo). Presentar también la curva de aprendizaje (ver figura abajo).
`Opcional (extra):` usar optuna [https://optuna.org](https://optuna.org/)

- 🖼️ Figura ejemplo: gráfica de optimización de hiperparámetro *(imagen alojada en Notion; el enlace S3 requiere sesión y no se incluye)*
- 🖼️ Figura ejemplo: curva de aprendizaje *(imagen alojada en Notion)*
- 🔗 [sklearn.model_selection.learning_curve](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.learning_curve.html)

#### 3.6 Comparación estadística de 2 técnicas ML — 3.5

<sub>https://pointy-whistle-142.notion.site/3a9244cc6e284d68b5a35f6e3ad543b2</sub>

**Descripción:** Se debe realizar una comparación estadística entre dos técnicas de aprendizaje automático. La comparación debe llevarse a cabo mediante un test de hipótesis apropiado (por ejemplo, Wilcoxon o t-test), utilizando *repeated k-fold cross-validation*. Por defecto, se recomienda realizar **10 repeticiones con 10 folds** (k=10). Sin embargo, si este proceso resulta muy costoso computacionalmente, se puede reducir la validación a **2 repeticiones con 5 folds** (k=5).

- 🔗 [sklearn.model_selection.RepeatedStratifiedKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.RepeatedStratifiedKFold.html#sklearn.model_selection.RepeatedStratifiedKFold)
- 🔗 [scipy.stats.wilcoxon — SciPy Manual](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.wilcoxon.html)

**Métricas**
1. Clasificación: presentar scores de Accuracy, AUC, curva ROC, Precision, Recall, Macro/Micro Average AUC ROC, F1 score macro avg, F1 score weighted avg.
2. Regresión: MSE, RMSE, R-squared
3. Clustering: distorsión

#### 3.7 Pipelines — 0.5

<sub>https://pointy-whistle-142.notion.site/65f7be5c47074222a9dd9b765c9e2dbd</sub>

**Descripción:** Usar `pipelines` de sklearn. Si implementa una red neuronal, use las clases y métodos adecuados para que su código sea modular (e.g., clases de Lightning o pytorch para estructurar modelos, data y entrenamientos en redes neuronales).

- 🔗 [Scikit-learn Pipeline Tutorial with Parameter Tuning and Cross-Validation](https://towardsdatascience.com/scikit-learn-pipeline-tutorial-with-parameter-tuning-and-cross-validation-e5b8280c01fb)

**Keras pipelines (tensorflow)**
- 🔗 [How to Grid Search Hyperparameters for Deep Learning Models in Python with Keras — MachineLearningMastery.com](https://machinelearningmastery.com/grid-search-hyperparameters-deep-learning-models-python-keras/)

#### 3.8 Visualización con TSNE — 1

<sub>https://pointy-whistle-142.notion.site/28254079a21447d2a0a704bfddb5e4f7</sub>

**Descripción:** Usar TSNE para visualizar su espacio de características en 2 dimensiones. Si usa una técnica supervisada, usar el label como color del punto en la visualización.

- 🖼️ Figura ejemplo de t-SNE *(imagen alojada en Notion)*

#### 3.9 Idioma — 0.1

<sub>https://pointy-whistle-142.notion.site/7f2d965ab8544b69ab7981ce35041cd7</sub>

**Descripción:** Uso del idioma (ortografía y redacción)

#### 3.10 Referencias — 0.1

<sub>https://pointy-whistle-142.notion.site/e7ee716e56254cf6817fb059b44469ab</sub>

**Descripción:** Colocar referencias en formato IEEE tanto de los conceptos teóricos como del código fuente. Si usa chat gpt para el código, colocar los prompts usados como comentarios de python en el lugar apropiado.

#### 3.11 Conclusiones — 0.1

<sub>https://pointy-whistle-142.notion.site/2132bef1ca084f42b02fa2454d5d2598</sub>

**Descripción:** Redactar conclusiones pertinentes. Recordar que la teoría no es una conclusión.

---

## 4. Rúbrica — Criterios (base de datos)

Cada ítem de la sección 3 se evalúa de forma independiente con uno de estos criterios. La nota del ítem = `Puntuación Máxima × Porcentaje Asignado / 100`.

| Criterio | Implementación | Defensa | Porcentaje Asignado | Descripción |
| --- | --- | --- | --- | --- |
| [Implementación Completa, Defensa Excelente: 100%](https://pointy-whistle-142.notion.site/3e86bdcc9032478bb0dbb5c39079bab4) | Completa | Excelente | 100 | Se ha realizado completamente lo solicitado y los estudiantes son totalmente capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Completa, Defensa Regular: 75%](https://pointy-whistle-142.notion.site/da4d96ec13bb480fb98392d03d7a1ff0) | Completa | Regular | 75 | Se ha realizado completamente lo solicitado y los estudiantes son parcialmente capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Parcial, Defensa Excelente: 50%](https://pointy-whistle-142.notion.site/ad6c348e70484392b4803ab13b631242) | Parcial | Excelente | 50 | Se ha realizado parcialmente lo solicitado y los estudiantes son totalmente capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Parcial, Defensa Regular: 25%](https://pointy-whistle-142.notion.site/fb533ad54e3b4069b75c1e705f51ba02) | Parcial | Regular | 25 | Se ha realizado parcialmente lo solicitado y los estudiantes son parcialmente capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Completa, Defensa Insuficiente: 25%](https://pointy-whistle-142.notion.site/0d275e8ee9b2477aaec0ecb66beb4a01) | Completa | Insuficiente | 25 | Se ha realizado completamente lo solicitado pero los estudiantes no son capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Parcial, Defensa Insuficiente: 0 %](https://pointy-whistle-142.notion.site/e5bb34e1ea464e3f9cd876846f84cd34) | Parcial | Insuficiente | 0 | Se ha realizado parcialmente lo solicitado pero los estudiantes no son capaces de defender, entender y justificar sus decisiones de diseño, criterios de implementación y aseveraciones en el texto. |
| [Implementación Inexistente: 0 %](https://pointy-whistle-142.notion.site/0efa57d1b47346c5aeb61eebabc7b437) | Inexistente | No aplica | 0 | No se realiza lo solicitado |

**Reglas generales de la rúbrica**
- Defensa obligatoria por todos los integrantes.
- Si un integrante no se presenta a la defensa, ese estudiante obtiene 0, independientemente del estado del trabajo.
- La nota es por grupo, no por estudiante.

---

## Anexo — Notas de la extracción

- Versión genérica: no incluye el enunciado específico del proyecto. Se toman la página principal, 11 páginas de ítems y 7 de criterios. Las páginas de criterios no tienen contenido propio; solo propiedades.
- Las páginas de criterios tienen además ~20 relaciones (`Related to Ítems Proyecto (1) …`) hacia copias de ítems de otros grupos o semestres. No se incluyen porque no aportan contenido.
- Las imágenes de ejemplo están en el almacenamiento privado de Notion (enlaces S3 firmados que caducan), así que solo se describen.
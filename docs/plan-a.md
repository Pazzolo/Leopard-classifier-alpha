# Plan A — Transfer learning + ML clásico sobre `classificador_felideos.v2i.multiclass`

Plan de trabajo para cubrir los 11 ítems de la [rúbrica](01-rubrica.md) usando **solo** el dataset v2. Si algo de este plan no funciona, se registra en la [bitácora](bitacora.md) y se arma un plan B.

## Resumen en una línea

Imagen → **ResNet50 preentrenada (ImageNet)** → embedding de 2048 dims → `Pipeline(StandardScaler → clasificador)` → **SVM (RBF) vs. Random Forest**, optimizados con `RepeatedStratifiedKFold` y comparados con **Wilcoxon** pareado.

## El dataset (v2)

| | |
| --- | --- |
| Origen | `zips/classificador_felideos.v2i.multiclass.zip` (Roboflow, junio 2024) |
| Imágenes | 850 JPG, 450×450, **color**, sin augmentation |
| Splits | train 680 · valid 85 · test 85 |
| Clases | 5: `GEOFFROYI`, `PARDALIS`, `TIGRINUS`, `WIEDII`, `ONCA` |
| Balance | train balanceado (~136/clase); valid/test entre 11 y 23 por clase |
| Etiquetas | `_classes.csv` one-hot por split → se convierte a una sola columna `label` |
| Fuga de datos | ✅ verificado: sin duplicados exactos y sin imágenes compartidas entre splits. El número del nombre de archivo (`100_jpg…`) se repite entre especies distintas, no son la misma foto. |

## Decisiones clave (y por qué)

| Decisión | Elección | Justificación para la defensa |
| --- | --- | --- |
| Uso de splits | **train + valid = conjunto de desarrollo (765)** para CV; **test (85) apartado** hasta el final | La rúbrica pide repeated k-fold CV para validar, así que no hace falta un valid fijo. Con más datos en el CV la estimación es más estable. El test solo se toca una vez. |
| Features | Transfer learning con **ResNet50** (torchvision, pesos ImageNet), salida del *global average pool* | Es una CNN estándar y bien documentada, fácil de defender. Con 765 imágenes, entrenar una red desde cero sobreajustaría. |
| Extracción | Una sola vez en Colab con GPU; embeddings guardados en `.npy` en Drive | El CV de 10×10 sobre embeddings tarda segundos o minutos. Sobre imágenes tardaría horas. |
| Feature selection | **Se omite** (permitido por la rúbrica con transfer learning e imágenes). Se justifica en una celda. | Opcional: `PCA` dentro del pipeline con `n_components` como hiperparámetro |
| Modelos a comparar | **SVM (RBF)** vs. **Random Forest** | Familias distintas (márgenes con kernel vs. ensamble de árboles), así que la comparación tiene sentido. Los dos se pueden usar con `predict_proba` para ROC/AUC. |
| CV | `RepeatedStratifiedKFold(n_splits=10, n_repeats=10, random_state=42)` | Es el valor por defecto que pide la rúbrica, y sobre embeddings es barato |
| Métrica a optimizar | **F1 macro** | Valid/test no están balanceados y todas las especies importan por igual |
| Test estadístico | **Wilcoxon signed-rank** pareado sobre los 100 scores por fold, α = 0.05 | No asume normalidad. Los scores están pareados porque ambos modelos usan los mismos folds (misma semilla). |

## Estructura de la notebook (Colab)

Una sola notebook, `notebooks/clasificador_felideos.ipynb`, con estas secciones en orden:

0. **Setup:** instalar dependencias, montar Drive, descomprimir el zip v2, fijar semillas.
1. **Introducción** → [ítem 1](items/01-introduccion.md)
2. **Carga de datos:** leer los 3 `_classes.csv`, armar un DataFrame `(path, label, split)`.
3. **EDA** → [ítem 2](items/02-eda.md)
4. **Feature extraction (ResNet50)** → [ítem 3](items/03-feature-extraction.md)
5. **Feature selection: por qué se omite** → [ítem 4](items/04-feature-selection.md)
6. **Pipelines** → [ítem 7](items/07-pipelines.md)
7. **Optimización de hiperparámetros** → [ítem 5](items/05-optimizacion-hiperparametros.md)
8. **Comparación estadística** → [ítem 6](items/06-comparacion-estadistica.md)
9. **Evaluación final en test**
10. **t-SNE** → [ítem 8](items/08-tsne.md)
11. **Conclusiones** → [ítem 11](items/11-conclusiones.md)
12. **Referencias (IEEE)** → [ítem 10](items/10-referencias.md)

El ítem 9 (idioma) se cubre con una revisión final de todo el texto.

## Fases

### Fase 0 — Setup
- [ ] Crear `notebooks/` y la notebook en Colab conectada al repo.
- [ ] Celda de setup: `unzip` del v2, semillas (`random`, `numpy`, `torch`), versiones de las librerías.
- [ ] Función `load_split(split) -> DataFrame` que convierte el one-hot a `label`.

### Fase 1 — EDA (0.6)
- [ ] Barras de distribución de clases por split; explicar que train está balanceado y valid/test no tanto.
- [ ] Grid de 5×5 (5 ejemplos por especie).
- [ ] Histograma H × W: todo es 450×450 por el resize de Roboflow. Explicar la deformación por *stretch*.
- [ ] Normalización: media/std de los píxeles por canal (RGB). Explicar que se normaliza con media/std de ImageNet antes de la ResNet.
- [ ] Explicar por qué no aplican estacionalidad ni skewness.
- [ ] Mencionar la verificación de fuga entre splits (ver tabla de arriba).

### Fase 2 — Feature extraction (0.25) + selection (0.25)
- [ ] `torchvision.models.resnet50(weights="IMAGENET1K_V2")`, reemplazar `fc` por `Identity`, `eval()`, `no_grad()`.
- [ ] Transform: `Resize(224)` → `ToTensor` → `Normalize(ImageNet)`.
- [ ] Extraer embeddings de los 850 → `X_dev (765×2048)`, `y_dev`, `X_test (85×2048)`, `y_test`; guardar en `.npy`.
- [ ] Celda markdown que justifica omitir feature selection.

### Fase 3 — Pipelines (0.5)
- [ ] `make_pipe(clf) = Pipeline([("scaler", StandardScaler()), ("clf", clf)])`.
- [ ] Mismo pipeline en las fases 4 y 5.

### Fase 4 — Optimización de hiperparámetros (3.5)
- [ ] Espacios de búsqueda:
  - SVM: `C ∈ logspace(-3, 3)`, `gamma ∈ {"scale"} ∪ logspace(-5, -1)`
  - RF: `n_estimators ∈ {100, 300, 500}`, `max_depth ∈ {None, 10, 20, 40}`, `max_features ∈ {"sqrt", "log2", 0.1}`
- [ ] `GridSearchCV(pipe, grid, cv=RepeatedStratifiedKFold(10, 10), scoring="f1_macro", n_jobs=-1)`.
- [ ] **Gráfica de optimización:** F1 macro medio ± std vs. `C` (SVM, eje log) y vs. `max_depth` (RF).
- [ ] **Curva de aprendizaje** (`learning_curve`) de cada mejor modelo, con interpretación de sesgo y varianza.
- [ ] *Extra:* repetir la búsqueda del SVM con Optuna y comparar con el grid.

### Fase 5 — Comparación estadística (3.5)
- [ ] `cross_validate` de los 2 mejores pipelines con el **mismo** `RepeatedStratifiedKFold(10, 10, random_state=42)` y `scoring` = accuracy, precision_macro, recall_macro, f1_macro, f1_weighted, roc_auc_ovr (macro).
- [ ] AUC micro: con `cross_val_predict(method="predict_proba")` + `label_binarize`.
- [ ] Tabla de media ± std de todas las métricas por modelo.
- [ ] Boxplot de los 100 F1 macro de cada modelo.
- [ ] H0: no hay diferencia en F1 macro. `scipy.stats.wilcoxon(f1_svm, f1_rf)` → estadístico, p-valor, conclusión.
- [ ] Mencionar la limitación: los folds de un repeated CV no son independientes (el test es optimista). Saber explicarlo en la defensa.

### Fase 6 — Evaluación final en test
- [ ] Reentrenar ambos mejores pipelines en los 765 de desarrollo y evaluar **una vez** en los 85 de test.
- [ ] Classification report, matriz de confusión, curvas ROC one-vs-rest con AUC macro y micro.

### Fase 7 — t-SNE (1.0)
- [ ] t-SNE 2D de los embeddings ResNet (los 850), color = especie, perplexity ∈ {15, 30, 50}.
- [ ] Contraste: t-SNE de los píxeles crudos (reducidos a 64×64 + PCA 50) para mostrar lo que aporta el transfer learning.
- [ ] Interpretar qué especies se solapan y relacionarlo con la matriz de confusión.

### Fase 8 — Texto y cierre (0.4)
- [ ] Introducción, conclusiones basadas en resultados, referencias en IEEE, prompts de LLM como comentarios.
- [ ] Revisión de ortografía.
- [ ] README del repo: descripción, cómo correr la notebook, resultados principales.
- [ ] Ensayo de defensa: **cada integrante** debe poder justificar cada fila de la tabla de decisiones.

## Riesgos y cómo mitigarlos

| Riesgo | Mitigación |
| --- | --- |
| Test pequeño (85, algunas clases con 11 imágenes) → métricas ruidosas | Basar las conclusiones en el CV (100 scores) y usar el test solo como confirmación |
| Especies moteadas muy parecidas (TIGRINUS / WIEDII / GEOFFROYI) | Esperado; analizarlo en la matriz de confusión y en t-SNE. Si el rendimiento es bajo, probar otro backbone (plan B) |
| Grid de RF lento con 10×10 | Reducir el grid o usar `RandomizedSearchCV`; en el peor caso, CV de 2×5 como permite la rúbrica (y justificarlo) |
| Colab se desconecta | Embeddings y resultados de CV guardados en Drive después de cada fase |

## Plan B (si hace falta)

- Cambiar el backbone (EfficientNet-B0 o DINOv2 ViT-S/14) manteniendo el resto del pipeline.
- Usar el dataset segmentado (`classificador_imagem_segmentada.v1i`) para ver si quitar el fondo ayuda.

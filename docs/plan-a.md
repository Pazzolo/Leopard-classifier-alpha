# Plan A — Transfer learning + ML clásico sobre `classificador_felideos.v2i.multiclass`

Plan de trabajo para cubrir los 11 ítems de la [rúbrica](01-rubrica.md) usando **solo** el dataset v2. Si algo de este plan no funciona, se registra en la [bitácora](bitacora.md) y se arma un plan B.

## Resumen en una línea

Imagen → **ResNet50 preentrenada (ImageNet)** → embedding de 2048 dims → `Pipeline(StandardScaler → clasificador)` → **SVM (RBF) vs. Random Forest**, optimizados con `RepeatedStratifiedKFold` y comparados con **Wilcoxon** pareado.

## Plan A sobre el diagrama de la pizarra

El plan sigue paso a paso el diagrama visto en clase ([diagrama-pipeline.html](diagrama-pipeline.html)):

| # | Paso del diagrama | Qué hacemos | Fase | Ítem de la rúbrica |
| --- | --- | --- | --- | --- |
| 1 | **Datos** (cilindro) | 850 fotos de v2, a color, 5 especies. Antes, EDA. | 0, 1 | 2 (EDA) |
| 2 | **Feature extraction**: a) clásica / **b) transfer learning** | b) ResNet50 preentrenada (plan B: CLIP). La opción a) (píxeles → PCA) solo como contraste en el t-SNE. | 2 | 3 |
| 3 | **emb / vector** | Un vector de 2048 números por foto, guardado en `.npy` | 2 | 3 |
| 4 | **Feature selection\*** (solo tabular) | Se omite y se justifica: son imágenes con transfer learning | 2 | 4 |
| 5 | **Modelos** + curva de optimización | `Pipeline(StandardScaler → SVM / RF)`; grid search y gráfica de score vs. hiperparámetro | 3, 4 | 7, 5 |
| 5 | Barras "→ Target" (folds) | `RepeatedStratifiedKFold` 10×10; la target es la especie | 4, 5 | 5, 6 |

**Fuera del diagrama, pero exigido por la rúbrica:** comparación estadística (fase 5, ítem 6), evaluación final en test (fase 6), t-SNE (fase 7, ítem 8) y texto/cierre (fase 8, ítems 1, 9–11). El experimento color vs. gris (fase 7b) es extra.

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
10b. **Experimento extra: ¿importa el color?** (color vs. escala de grises)
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

### Fase 7b — Experimento extra: ¿importa el color?

**Pregunta:** ¿el color ayuda a distinguir las especies, o basta con la forma y el patrón de manchas?

**Diseño (comparación controlada: solo cambia el color):**
- [ ] **Versión gris = v2 convertida en código** (`transforms.Grayscale(num_output_channels=3)` antes de la ResNet). Mismas 850 fotos, mismos splits y mismo pipeline que la versión a color.
- [ ] Extraer embeddings ResNet50 de la versión gris → `X_dev_gray`, `X_test_gray` (~1 min en GPU).
- [ ] Reoptimizar el SVM sobre los embeddings grises con el mismo grid (es barato), para no favorecer al color con hiperparámetros ajustados para él.
- [ ] `cross_validate` de ambos (color y gris) con el **mismo** `RepeatedStratifiedKFold(10, 10, random_state=42)` → 100 F1 macro pareados.
- [ ] Wilcoxon pareado, α = 0.05. H0: el color no cambia el F1 macro.
- [ ] Boxplot color vs. gris y F1 por especie (¿a qué especies afecta más quitar el color?).
- [ ] Evaluación en test (85) de ambos, solo como confirmación.
- [ ] *Chequeo secundario:* repetir con la exportación gris de Roboflow (`classificador_imagem_segmentada`, 800 fotos en común, emparejadas por número + especie) para ver si coincide.
- [ ] Opcional: t-SNE de los embeddings grises al lado del de color.

**Notas:**
- No reemplaza al ítem 6 (que compara dos *técnicas de ML*); es una sección extra.
- No llamarlo "segmentado": esa exportación conserva el fondo (ver [00-proyecto.md](00-proyecto.md)).

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

Si ResNet50 rinde mal, se **cambia el modelo preentrenado** y se mantiene el resto del pipeline (mismo dataset v2, mismos folds, mismos clasificadores):

1. **CLIP** (ViT-B/32, embedding de 512 dims): es el que aparece en la pizarra de clase.
2. EfficientNet-B0 o DINOv2 ViT-S/14 como siguientes opciones.

Ya **no** se considera el dataset "segmentado" como plan B: conserva el fondo y en la práctica es v2 en escala de grises con 50 fotos menos (ver [00-proyecto.md](00-proyecto.md)). La pregunta del gris ya la cubre la fase 7b.

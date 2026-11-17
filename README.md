# Clasificador de félidos neotropicales

Clasificación de fotos de 5 especies de félidos de Latinoamérica con **transfer learning** (ResNet50) y modelos de machine learning clásicos. Proyecto del curso de Aprendizaje Supervisado, Maestría en Inteligencia Artificial, USFQ.

| | |
| --- | --- |
| Especies | *Leopardus geoffroyi*, *L. pardalis* (ocelote), *L. tigrinus* (tigrillo), *L. wiedii* (margay), *Panthera onca* (jaguar) |
| Dataset | [classificador_felideos v2](https://universe.roboflow.com/willian-colombo/classificador_felideos) (Roboflow Universe, CC BY 4.0): 850 fotos a color, 450×450 |
| Notebook | [`notebooks/clasificador_felideos.ipynb`](notebooks/clasificador_felideos.ipynb) |

## Pipeline

```
foto → ResNet50 preentrenada (ImageNet) → vector de 2048 características → StandardScaler → SVM / Random Forest
```

- Optimización de hiperparámetros con *repeated stratified k-fold* 10×10 (F1 macro).
- Comparación estadística SVM vs. Random Forest con el test de Wilcoxon y el t-test corregido de Nadeau–Bengio.
- Evaluación final en un test apartado (85 fotos), t-SNE y un experimento extra: ¿importa el color?

## Resultados principales

| Métrica (CV 10×10) | SVM | Random Forest |
| --- | --- | --- |
| F1 macro | **0.775 ± 0.040** | 0.710 ± 0.047 |
| Accuracy | **0.775** | 0.712 |
| AUC ROC macro | **0.949** | 0.928 |

- **El SVM es mejor**: gana en 93 de 100 folds (Wilcoxon p ≈ 10⁻¹⁷; t corregido p ≈ 3 × 10⁻⁵).
- **En test** (85 fotos): SVM 65 aciertos, RF 68; la diferencia no es significativa (McNemar p = 0.55).
- **Fotos nocturnas:** el 32 % del dataset son infrarrojas y se aciertan mucho menos (67.5 % vs. 82.7 % en fotos a color).
- **El color aporta poco en total** (+0.008 de F1): ayuda en fotos diurnas y empeora las nocturnas.

## Cómo ejecutar

### En Google Colab

Abrir la notebook en Colab y ejecutar todas las celdas. La primera celda clona este repositorio (que incluye el zip del dataset) e instala lo que falte.

### En local

```bash
uv venv --python 3.12 .venv
uv pip install --python .venv/bin/python -r requirements.txt
.venv/bin/jupyter lab notebooks/clasificador_felideos.ipynb
```

La notebook detecta la GPU disponible (CUDA, Apple MPS o CPU). Los resultados de la validación cruzada están guardados en `outputs/cv/`, así que una nueva ejecución tarda ~1 min. Para recalcular todo desde cero (~1.5 h en un Mac M3 Pro), hay que borrar `outputs/cv/` y `outputs/embeddings/`.

## Estructura

```
notebooks/clasificador_felideos.ipynb   Notebook del proyecto (entregable)
zips/                                   Exportaciones del dataset de Roboflow (se usa v2)
outputs/figures/                        Figuras generadas por la notebook (para la presentación)
outputs/cv/                             Resultados de la validación cruzada (caché)
docs/                                   Plan, decisiones, rúbrica, bitácora y diagrama del pipeline
requirements.txt                        Dependencias
```

En [`docs/`](docs/README.md) están el [plan de trabajo](docs/plan-a.md), las [decisiones justificadas](docs/decisiones-plan-a.md) (base de la presentación), el [diagrama del pipeline](docs/diagrama-pipeline.html) y la [bitácora](docs/bitacora.md).

## Uso de IA

El código se desarrolló con asistencia de Claude Code (Anthropic) y fue revisado por el grupo. Los prompts usados están en la sección 13 de la notebook.

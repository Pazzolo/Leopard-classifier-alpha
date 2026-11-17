# 7. Pipelines — 0.5 pts

> [Notion](https://pointy-whistle-142.notion.site/65f7be5c47074222a9dd9b765c9e2dbd)

## Requisito

Usar `Pipeline` de sklearn. Si se implementa una red neuronal, estructurar el código de forma modular (p. ej. clases de Lightning o PyTorch para modelos, datos y entrenamiento).

- [Scikit-learn Pipeline Tutorial with Parameter Tuning and Cross-Validation](https://towardsdatascience.com/scikit-learn-pipeline-tutorial-with-parameter-tuning-and-cross-validation-e5b8280c01fb)
- [Grid Search Hyperparameters for Deep Learning Models with Keras](https://machinelearningmastery.com/grid-search-hyperparameters-deep-learning-models-python-keras/)

## Checklist

- [ ] `Pipeline([("scaler", StandardScaler()), ("clf", ...)])` sobre los embeddings.
- [ ] Todo paso que aprenda de los datos (scaler, PCA, selección) va **dentro** del pipeline para que se ajuste por fold.
- [ ] Usar el mismo pipeline en el ítem 5 (búsqueda) y en el ítem 6 (comparación).
- [ ] Opcional: envolver el extractor de features en un transformer propio (`BaseEstimator, TransformerMixin`).

## Notas


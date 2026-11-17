# 5. Optimización de hiperparámetros — 3.5 pts

> [Notion](https://pointy-whistle-142.notion.site/64b977f1fc294a9b993e43473c01b643)

## Requisito

- Optimizar los hiperparámetros en un conjunto de validación adecuado con **repeated k-fold CV**.
- Se puede usar nested k-fold CV, pero **no se recomienda** porque tarda mucho más.
- Presentar una **gráfica de optimización** de algún hiperparámetro (score vs. valor).
- Presentar la **curva de aprendizaje**.
- *Opcional (extra):* [Optuna](https://optuna.org/).

Referencias:

- [sklearn.model_selection.learning_curve](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.learning_curve.html)

## Checklist

- [ ] Definir el esquema de CV: `RepeatedStratifiedKFold` (k y número de repeticiones justificados).
- [ ] Si hay copias de la misma imagen, usar CV agrupado (`StratifiedGroupKFold`) para evitar fuga.
- [ ] Definir el espacio de búsqueda por modelo (rangos justificados).
- [ ] Búsqueda: `GridSearchCV` / `RandomizedSearchCV` u Optuna (extra).
- [ ] Métrica de optimización justificada (p. ej. F1 macro).
- [ ] Gráfica de optimización: score medio ± std vs. hiperparámetro (p. ej. `C` en escala log).
- [ ] Curva de aprendizaje (train vs. validación) del mejor modelo, con interpretación (sesgo/varianza).
- [ ] Hacerlo para los dos modelos que se comparan en el ítem 6.
- [ ] Evaluación final en el test set apartado.

## Notas


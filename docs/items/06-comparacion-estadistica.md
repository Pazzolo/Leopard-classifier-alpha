# 6. Comparación estadística de 2 técnicas ML — 3.5 pts

> [Notion](https://pointy-whistle-142.notion.site/3a9244cc6e284d68b5a35f6e3ad543b2)

## Requisito

- Comparación estadística entre **dos técnicas** de aprendizaje automático.
- Test de hipótesis apropiado (p. ej. **Wilcoxon** o **t-test**) sobre los scores de **repeated k-fold CV**.
- Por defecto **10 repeticiones × 10 folds**; si es muy costoso, **2 repeticiones × 5 folds**.

Métricas de **clasificación** a presentar:

- Accuracy
- AUC y curva ROC
- Precision, Recall
- AUC ROC macro y micro average
- F1 macro avg, F1 weighted avg

Referencias:

- [RepeatedStratifiedKFold](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.RepeatedStratifiedKFold.html)
- [scipy.stats.wilcoxon](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.wilcoxon.html)

## Checklist

- [ ] Elegir las 2 técnicas (p. ej. SVM vs. Random Forest o Logistic Regression sobre los mismos embeddings).
- [ ] Mismo `RepeatedStratifiedKFold` con la **misma semilla** para ambas, así los scores quedan pareados por fold.
- [ ] Plantear H0/H1 y el nivel de significancia α.
- [ ] Justificar el test (Wilcoxon pareado: no asume normalidad; t-test: verificar normalidad de las diferencias).
- [ ] Reportar estadístico, p-valor y conclusión.
- [ ] Tabla con todas las métricas (media ± std) por modelo.
- [ ] Curvas ROC multiclase (one-vs-rest) con AUC macro y micro.
- [ ] Matriz de confusión de cada modelo.
- [ ] Boxplot de los scores por fold de ambos modelos.

## Notas


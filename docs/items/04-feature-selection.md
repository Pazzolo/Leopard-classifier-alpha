# 4. Feature Selection — 0.25 pts

> [Notion](https://pointy-whistle-142.notion.site/ce578c1299be4924bfba95c2350bc404)

## Requisito

Implementar una técnica de feature selection (tipo *filter*) en el pipeline. **Si se usa transfer learning o el problema es de imágenes o texto, se puede omitir.**

- [1.13. Feature selection — scikit-learn](http://scikit-learn.org/stable/modules/feature_selection.html)

## Checklist

- [ ] Decidir: omitir (justificado, porque es un problema de imágenes con transfer learning) o incluir igual.
- [ ] Si se omite: dejar una celda markdown que lo explique.
- [ ] Si se incluye: `SelectKBest(f_classif)` o `VarianceThreshold` **dentro** del pipeline (no antes del CV) y `k` como hiperparámetro.

## Notas


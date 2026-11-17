# Documentación del proyecto — Clasificador de félidos

Índice de trabajo derivado de [Plantilla.md](../Plantilla.md) (extracción original de Notion, se mantiene como fuente).

## Archivos

| Archivo | Contenido |
| --- | --- |
| [00-proyecto.md](00-proyecto.md) | Enunciado general, reglas de entrega y defensa, datasets disponibles |
| [01-rubrica.md](01-rubrica.md) | Criterios de evaluación y fórmula de nota |
| [plan-a.md](plan-a.md) | **Plan A:** transfer learning (ResNet50) + SVM vs. Random Forest sobre el dataset v2 |
| [decisiones-plan-a.md](decisiones-plan-a.md) | Decisiones del Plan A explicadas (qué, por qué, alternativas): base para la presentación |
| [diagrama-pipeline.html](diagrama-pipeline.html) | Diagrama del pipeline (réplica de la pizarra de clase) para la presentación |
| [bitacora.md](bitacora.md) | Bitácora del capitán: qué se hizo, qué salió bien o mal, decisiones |
| [items/](items/) | Un archivo por ítem evaluado (requisito + checklist + notas) |

## Ítems y estado

Puntos totales: **10.0**. Los ítems 5 y 6 valen 7.0 de 10, así que la mayor parte del esfuerzo va ahí.

| # | Ítem | Pts | Archivo | Estado |
| --- | --- | --- | --- | --- |
| 1 | Introducción | 0.1 | [01-introduccion.md](items/01-introduccion.md) | ✅ |
| 2 | EDA | 0.6 | [02-eda.md](items/02-eda.md) | ✅ |
| 3 | Feature Extraction | 0.25 | [03-feature-extraction.md](items/03-feature-extraction.md) | ✅ |
| 4 | Feature Selection | 0.25 | [04-feature-selection.md](items/04-feature-selection.md) | ✅ |
| 5 | Optimización de hiperparámetros | **3.5** | [05-optimizacion-hiperparametros.md](items/05-optimizacion-hiperparametros.md) | ✅ |
| 6 | Comparación estadística de 2 técnicas | **3.5** | [06-comparacion-estadistica.md](items/06-comparacion-estadistica.md) | ✅ |
| 7 | Pipelines | 0.5 | [07-pipelines.md](items/07-pipelines.md) | ✅ |
| 8 | Visualización con t-SNE | 1.0 | [08-tsne.md](items/08-tsne.md) | ✅ |
| 9 | Idioma | 0.1 | [09-idioma.md](items/09-idioma.md) | 🟨 |
| 10 | Referencias | 0.1 | [10-referencias.md](items/10-referencias.md) | ✅ |
| 11 | Conclusiones | 0.1 | [11-conclusiones.md](items/11-conclusiones.md) | ✅ |

Leyenda: ⬜ pendiente · 🟨 en progreso · ✅ listo

## Flujo sugerido

1. Seguir el [Plan A](plan-a.md) (dataset v2).
2. EDA (ítem 2) → extracción de features por transfer learning (ítem 3).
3. Pipeline (ítem 7) → optimización de hiperparámetros (ítem 5) → comparación estadística (ítem 6).
4. t-SNE (ítem 8) sobre los embeddings.
5. Introducción, conclusiones, referencias y revisión de idioma al final (ítems 1, 9–11).

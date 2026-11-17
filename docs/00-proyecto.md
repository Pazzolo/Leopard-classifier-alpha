# Proyecto ML — Enunciado general

> Fuente: [Plantilla.md §1–2](../Plantilla.md) · [Notion](https://pointy-whistle-142.notion.site/Proyecto-ML-734cbbc3faae41c9a3f2425105baab07)

## Objetivo

Usar machine learning para resolver un problema con la metodología correcta aprendida en el curso.

## Reglas de entrega

- Todo se implementa en **una notebook de Google Colab**.
- **No hay informe aparte**: todo va en la notebook.
- Repositorio en **GitHub** con un README pertinente.

## Defensa

- Obligatoria para todos los integrantes.
- Si un integrante no se presenta, ese estudiante obtiene 0 aunque el trabajo esté completo.
- La nota es por grupo.

## Proyecto específico

**Tema:** clasificación de imágenes de félidos neotropicales (5 especies).

Clases:

- `LEOPARDUS_GEOFFROYI`
- `LEOPARDUS_PARDALIS`
- `LEOPARDUS_TIGRINUS`
- `LEOPARDUS_WIEDII`
- `PANTHERA_ONCA`

_Por completar: motivación, objetivos, integrantes del grupo._

## Datasets disponibles

Exportaciones de Roboflow en formato *multiclass* (carpetas `train/ valid/ test/`, cada una con `_classes.csv` one-hot). Las carpetas están en `.gitignore`; los zips originales están en `zips/`.

| Carpeta | Preprocesamiento | train | valid | test | Observaciones |
| --- | --- | --- | --- | --- | --- |
| `classificador_felideos.v2i.multiclass` | 450×450, color | 680 | 85 | 85 | 5 clases en todos los splits; train balanceado (~136/clase) |
| `classificador_felideos.v3i.multiclass` | 150×150, escala de grises | 767 | 41 | 42 | ⚠️ valid/test **sin** `WIEDII` ni `ONCA`; columna extra `Unlabeled` (1 imagen) |
| `classificador_felideos.v4i.multiclass` | 450×450, escala de grises | 767 | 41 | 42 | Mismos problemas que v3 |
| `classificador_imagem_segmentada.v1i.multiclass` | 640×640, escala de grises, imágenes segmentadas | 640 | 80 | 80 | 5 clases en todos los splits |

Ninguna versión trae data augmentation de Roboflow. Todas usan resize con *stretch* (se deforma la relación de aspecto).

**Decisión pendiente:** qué versión usar. v2 (color) o la segmentada parecen las únicas viables; v3/v4 no permiten evaluar 2 de las 5 clases.

**Por verificar:** los nombres de archivo repiten el mismo prefijo de origen (p. ej. `100_jpg.rf.<hash>`) varias veces. Hay que confirmar si son imágenes distintas de clases distintas con el mismo número o copias de la misma foto. Si son copias, hay que agruparlas para que no caigan en folds distintos del CV (fuga de datos; ver ítems [5](items/05-optimizacion-hiperparametros.md) y [6](items/06-comparacion-estadistica.md)).

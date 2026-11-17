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
| `classificador_felideos.v3i.multiclass` | 150×150, escala de grises | 767 | 41 | 42 | ⚠️ valid/test **sin** `WIEDII` ni `ONCA`; columna extra `Unlabeled`: la foto #159 de *L. geoffroyi* perdió su etiqueta |
| `classificador_felideos.v4i.multiclass` | 450×450, escala de grises | 767 | 41 | 42 | **Idéntica a v3** (misma partición, 850/850) pero a 450×450 |
| `classificador_imagem_segmentada.v1i.multiclass` | 640×640, escala de grises | 640 | 80 | 80 | 5 clases en todos los splits. 800 fotos: faltan 50 (sobre todo *wiedii*, *geoffroyi*, *tigrinus*). ⚠️ Pese al nombre, en las muestras revisadas el fondo **no** está removido |

Ninguna versión trae data augmentation de Roboflow. Todas usan resize con *stretch* (se deforma la relación de aspecto).

**Decisión:** se usa **v2** (color) — ver [plan-a.md](plan-a.md). v3/v4 no permiten evaluar 2 de las 5 clases. La segmentada no se usa como dataset principal: conserva el fondo y es, en la práctica, v2 en grises con 50 fotos menos. El plan B es cambiar el modelo preentrenado (CLIP).

**Verificado (v2):** el número del nombre de archivo (`100_jpg.rf.<hash>`) se repite entre especies distintas, pero cada par (número, especie) aparece una sola vez. No hay duplicados exactos ni imágenes compartidas entre train/valid/test, así que no hay fuga de datos. Detalle en la [bitácora](bitacora.md).

**⚠️ Las 4 versiones son las mismas fotos:** son re-exportaciones del mismo conjunto (~850 fotos) con distinto procesamiento. 849/850 pares (número, especie) coinciden entre versiones, La inspección visual de la foto #100 (ocelote y tigrillo) muestra la misma imagen en v2, v4 y la segmentada. Una comparación de píxeles de **todos** los pares (miniaturas 32×32 en grises) da una correlación media de 0.95–0.99 entre versiones, contra ~0.05 entre fotos al azar. Cada versión reparte las fotos en splits distintos: 43/85 fotos del test de v2 están en el train de v3/v4, y 66/85 en el train de la segmentada. Fotos en el mismo split que v2: v3/v4 762/849, segmentada 518/800. **Mezclar versiones sin agrupar por foto produce fuga de datos.**

**Plan de trabajo:** ver [plan-a.md](plan-a.md) (usa v2).

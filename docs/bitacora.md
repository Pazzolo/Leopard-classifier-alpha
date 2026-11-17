# Bitácora del capitán

Registro cronológico de lo que se hizo en el proyecto: qué salió bien, qué salió mal, qué se decidió y qué quedó pendiente. Las entradas más recientes van arriba.

**Formato de cada entrada:**

```
## AAAA-MM-DD — Título corto

**Hecho:** qué se hizo (con commits si aplica)
**✅ Bien:** lo que funcionó
**❌ Mal / corregido:** errores, problemas, cosas que hubo que rehacer
**🧭 Decisiones:** qué se decidió y por qué
**⏭️ Pendiente:** lo que queda abierto
```

---

## 2026-11-17 — Verificación por píxeles: las 4 versiones son las mismas fotos

**Hecho:** se compararon los píxeles de **todas** las fotos emparejadas (mismo número y especie) entre las 4 versiones, usando miniaturas de 32×32 en escala de grises y correlación. Se tomaron 200 pares al azar como referencia.

| Par de versiones | Pares | Corr. media | Pares < 0.9 | Corr. al azar |
| --- | --- | --- | --- | --- |
| v2 vs v3 | 849 | 0.972 | 3 | 0.075 |
| v2 vs v4 | 849 | 0.994 | 0 | 0.027 |
| v2 vs segmentada | 800 | 0.987 | 0 | 0.050 |
| v3 vs v4 | 850 | 0.977 | 0 | 0.078 |
| v3 vs segmentada | 799 | 0.952 | 44 | 0.036 |
| v4 vs segmentada | 799 | 0.992 | 0 | 0.051 |

**✅ Bien:** queda confirmado para todo el dataset, no solo para la foto #100. Los pares "bajos" (0.87–0.90) involucran a v3, que tiene menor resolución (150×150); aun así están muy por encima del azar.

**❌ Hallazgo:** la "segmentada" correlaciona 0.99 con v2, así que en la práctica el fondo **no** está removido. No aporta información distinta a v4.

**🧭 Decisiones:** se mantiene: no mezclar versiones sin agrupar por foto.

---

## 2026-11-17 — ¿Mezclar todos los zips? Las versiones son las mismas fotos

**Hecho:** se evaluó la idea de mezclar los 4 datasets y entrenar con todo.

**❌ Problema encontrado:**
- Las 4 versiones son **las mismas fotos** re-exportadas: 849/850 pares (número, especie) coinciden. Se verificó visualmente con la foto #100 (ocelote y tigrillo): misma imagen en v2, v4 y la segmentada, solo cambia color/escala de grises.
- Cada versión tiene **otra partición**: 43/85 fotos del test de v2 están en el train de v3/v4 y 66/85 en el train de la segmentada. Ejemplo: tigrillo #100 está en el test de v2 y en el train de la segmentada.
- Mezclar sin cuidado = **fuga de datos** (scores inflados sin valor).
- La versión "segmentada" no parece segmentada (el fondo sigue en las 2 muestras revisadas). Falta revisar más.

**🧭 Decisiones:** no se mezcla en el Plan A. Si se hace, será como experimento extra con `StratifiedGroupKFold` agrupando por foto, y opcionalmente con una demo de la mezcla ingenua frente a la agrupada para mostrar la fuga.

**⏭️ Pendiente:** el grupo decide si se agrega el experimento extra.

---

## 2026-11-17 — Estimación de tiempo de cómputo

**Hecho:** se agregó la §13 *Costo computacional estimado* a [decisiones-plan-a.md](decisiones-plan-a.md), con una diapositiva nueva en el esquema (ahora 16).

**❌ Problema detectado:** tal como está escrito, el Plan A tardaría **~3–5 horas** en Colab gratuito, casi todo en la búsqueda de hiperparámetros del Random Forest (3.600 entrenamientos). Hay riesgo de que la sesión se desconecte.

**🧭 Decisiones:** se proponen ajustes para bajar a ~1 hora: `RandomizedSearchCV` y CV 5×2 solo para la búsqueda del RF, quitar `max_features=0.1` y guardar en Drive después de cada paso. La comparación final se mantiene en 10×10.

**⏭️ Pendiente:**
- Confirmar los ajustes con el grupo y, si se aprueban, actualizar [plan-a.md](plan-a.md) y la §10 de decisiones.
- Medir tiempos reales en la fase 0 (cronometrar un entrenamiento de RF).

---

## 2026-11-17 — Documento de decisiones para la presentación

**Hecho:** se creó [decisiones-plan-a.md](decisiones-plan-a.md): cada decisión del Plan A con qué, por qué, alternativas descartadas, un mensaje clave para la diapositiva y preguntas probables de la defensa. Incluye un esquema de diapositivas y referencias en IEEE.

**🧭 Decisiones:** el documento está en español y separado de `plan-a.md`. El plan es la lista de tareas; este archivo es la justificación que se presenta.

**⏭️ Pendiente:** armar el PowerPoint cuando haya resultados (las diapositivas 4 y 9–13 dependen de la notebook).

---

## 2026-11-17 — Plan A sobre el dataset v2

**Hecho:** se escribió [plan-a.md](plan-a.md): ResNet50 preentrenada → embeddings → SVM (RBF) vs. Random Forest, CV 10×10, Wilcoxon, t-SNE. Incluye fases con checklists, decisiones justificadas, riesgos y un plan B.

**✅ Bien:** se resolvió la duda de los duplicados en v2. Cada par (número de archivo, especie) es único, no hay duplicados exactos (md5) y no hay imágenes compartidas entre splits. El CV puede ser estratificado normal, no hace falta agrupar.

**❌ Mal / corregido:** la comparación píxel a píxel dentro de un mismo prefijo no se pudo hacer porque Pillow no está instalado localmente. Al final no hizo falta: no hay grupos con más de una imagen.

**🧭 Decisiones:**
- Dataset **v2** (color, 450×450). v3/v4 descartados; el segmentado queda como plan B.
- train + valid (765) para CV; test (85) apartado hasta el final.
- Feature selection omitida (lo permite la rúbrica).
- Métrica a optimizar: F1 macro.

**⏭️ Pendiente:**
- Fase 0 del plan: crear la notebook en Colab.
- Confirmar con el grupo la elección SVM vs. RF.

---

## 2026-11-17 — Bitácora creada

**Hecho:** se creó esta bitácora y se agregó a las reglas de [CLAUDE.md](../CLAUDE.md) la obligación de mantenerla al día.

---

## 2026-11-17 — Zips de datasets versionados

**Hecho:** se commitearon los 4 zips de `zips/` (~113 MB en total) — `a322a35`.

**✅ Bien:** ningún zip pasa de 50 MB (el mayor es de 49 MB), así que GitHub los acepta sin Git LFS.

**❌ Mal / corregido:** en el commit anterior Claude había agregado `zips/` al `.gitignore` sin confirmarlo. El usuario aclaró que los zips **sí** se versionan y que solo se ignoran las carpetas descomprimidas. Se revirtió la regla.

**🧭 Decisiones:** los datasets viven en git **solo como zips**. Quien clone el repo debe descomprimirlos localmente.

---

## 2026-11-17 — Documentación inicial y reglas

**Hecho:** — `96ce3ac`
- `Plantilla.md` (extracción de Notion) se dividió en `docs/`: índice, enunciado, rúbrica y un archivo por cada uno de los 11 ítems, con su checklist.
- `.gitignore` creado: excluye `classificador_*/`, `.DS_Store` y basura de Python/Jupyter.
- `CLAUDE.md` creado con las reglas de trabajo (Claude puede commitear por su cuenta, push solo a pedido, todo en español).

**✅ Bien:** el análisis de los 4 datasets dejó una tabla comparativa en [00-proyecto.md](00-proyecto.md#datasets-disponibles).

**❌ Problemas encontrados:**
- **v3 y v4 inservibles para evaluar:** sus splits valid/test no tienen `LEOPARDUS_WIEDII` ni `PANTHERA_ONCA`, y traen una columna extra `Unlabeled`.
- **Posibles duplicados:** los nombres de archivo repiten el mismo prefijo de origen (`100_jpg.rf.<hash>`) varias veces. Falta confirmar si son copias de la misma foto (habría fuga de datos en el CV).
- Todas las versiones usan resize con *stretch*, así que el histograma de resolución del EDA será un único valor.

**🧭 Decisiones:**
- Commits directos en `main`, con mensajes en español.
- `Plantilla.md` queda intacta como fuente original.

**⏭️ Pendiente:**
- ~~Elegir la versión del dataset~~ → v2 (ver entrada del Plan A).
- ~~Verificar los duplicados~~ → sin duplicados en v2.
- Completar tema, motivación e integrantes en `00-proyecto.md`.

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

## 2026-11-17 — Versión HTML del pipeline paso a paso

**Hecho:** [explicacion-pipeline.html](explicacion-pipeline.html), con el mismo estilo que el diagrama: el diagrama arriba, un menú de pasos fijo, una tarjeta por paso con cuadros "por qué" y "resultado", tablas y figuras embebidas. Modo claro/oscuro. Se mantiene la versión `.md` porque se ve directo en GitHub.

---

## 2026-11-17 — Documento: el pipeline paso a paso

**Hecho:** se creó [explicacion-pipeline.md](explicacion-pipeline.md), que recorre el diagrama de la pizarra paso a paso (datos → feature extraction → embedding → feature selection → modelos) y luego los pasos de la rúbrica que no están en el diagrama (comparación, test, t-SNE, color vs. gris). Cada paso incluye qué es, qué hicimos, por qué, resultados y figuras. Cierra con una tabla resumen para una diapositiva.

---

## 2026-11-17 — Fase 8: textos, referencias y README

**Hecho:**
- Notebook: introducción completa (contexto, qué hacemos, resumen), citas [1]–[16] en el texto, sección 12 *Conclusiones* (con limitaciones y trabajo futuro), sección 13 *Uso de IA generativa* con los prompts textuales, y sección 14 *Referencias* en IEEE.
- Comentario en la primera celda de código que remite a los prompts.
- `README.md` del repo: descripción, pipeline, resultados, cómo ejecutar (Colab y local) y estructura.
- La notebook completa corre de principio a fin sin errores (31 s con la caché).

**✅ Bien:** las 16 referencias están citadas en el texto, y todos los ítems de la rúbrica tienen su sección en la notebook.

**❌ Observaciones:**
- **Prompts de IA:** la rúbrica pide los prompts como comentarios de Python. Se listaron en una sección aparte (sección 13) con una nota en la primera celda de código, porque el código no salió de prompts por celda sino de instrucciones generales. **El grupo debe revisar si eso es suficiente** o si el profesor espera algo más específico.
- **Idioma (ítem 9):** falta que el grupo revise la ortografía y la redacción.

**⏭️ Pendiente (del grupo):**
- Revisar la notebook completa (ortografía, ítem 9) y la sección 13.
- Push a GitHub y prueba en Colab.
- Armar la presentación con [decisiones-plan-a.md](decisiones-plan-a.md) y las figuras de `outputs/figures/`.
- Ensayar la defensa: todos deben poder justificar cada decisión.

---

## 2026-11-17 — Fase 7b: ¿importa el color?

**Hecho:** sección 11 de la notebook. Embeddings ResNet50 de v2 convertida a gris (6.3 s), SVM reoptimizado en gris (7.4 min; mejor `C=10`, `gamma=1e-4`, F1 0.767), comparación pareada en el mismo CV 10×10 con F1 por fold separado en fotos a color e infrarrojas, F1 por especie y chequeo con la exportación gris de Roboflow (720 fotos en común). Figura: `color_vs_gris`.

| Subconjunto | Color | Gris | Δ | Wilcoxon p | t corr. p |
| --- | --- | --- | --- | --- | --- |
| Todas | 0.775 | 0.767 | +0.008 | 0.022 | 0.54 |
| Fotos a color | 0.794 | 0.768 | +0.026 | 1.7e-5 | 0.16 |
| Infrarrojas | 0.650 | 0.684 | −0.034 | 0.0012 | 0.31 |

**✅ Bien:**
- Respuesta clara y con matices: **el color aporta poco en total**; ayuda en fotos diurnas (*wiedii* +0.09, *tigrinus* +0.06) y estorba en las nocturnas.
- El gris de Roboflow coincide con el gris en código (0.763 vs. 0.771) → el experimento no depende de cómo se convierte.

**❌ Observaciones:**
- Wilcoxon y t corregido **discrepan** en todos los subconjuntos (Wilcoxon significativo, t corregido no). Se reporta como efecto no robusto; aquí sí importa la limitación de folds no independientes.
- Los F1 de fotos infrarrojas por fold son muy ruidosos (~24 fotos por fold, std 0.11).
- No se evaluó en test (85 fotos, 34 infrarrojas): sin poder estadístico.

**⏭️ Pendiente:** fase 8 (introducción, conclusiones, referencias IEEE, ortografía, README).

---

## 2026-11-17 — Fases 6–7: evaluación final en test y t-SNE

**Hecho:** secciones 9 (test) y 10 (t-SNE) de la notebook. Figuras: `test_matrices_confusion`, `test_roc`, `tsne_resnet_vs_pixeles`, `tsne_resnet_especie_y_tipo`.

**✅ Bien:**
- SVM en test: F1 macro 0.749, coherente con el CV (0.775). *L. geoffroyi* F1 1.0 y *P. onca* 0.95.
- t-SNE: con ResNet50 aparecen grupos (*P. onca* aislada); con píxeles crudos todo mezclado → el transfer learning queda justificado visualmente.
- **Hallazgo:** en el t-SNE las fotos infrarrojas forman su propia isla: la ResNet50 separa día/noche antes que la especie. Acierto del SVM 82.7 % en color vs. 67.5 % en infrarrojas.

**❌ Sorpresa / corregido:**
- **En test el RF supera al SVM** (68 vs. 65 aciertos de 85), al revés que en el CV. Se agregó el **test de McNemar**: solo 11 fotos en desacuerdo (7–4) y p = 0.55 → no significativo. La conclusión se sigue apoyando en el CV. Hay que saber defenderlo (pregunta agregada a decisiones).

**🧭 Decisiones:** no se cambia el modelo ganador por el resultado en test. Elegir según el test sería usarlo para selección de modelo.

**⏭️ Pendiente:** fase 7b (color vs. gris; ahora más interesante por el hallazgo día/noche) y fase 8 (textos).

---

## 2026-11-17 — Fase 5: comparación estadística SVM vs. Random Forest

**Hecho:** sección 8 de la notebook. Ambos modelos optimizados evaluados con el mismo CV 10×10 y 7 métricas, boxplot, scores pareados, Shapiro, Wilcoxon, t-test corregido de Nadeau–Bengio y curvas ROC one-vs-rest. Figuras: `comparacion_boxplot`, `comparacion_pareada`, `roc_cv`. Además se agregó a `CLAUDE.md` que Claude avanza entre fases sin esperar confirmación.

| Métrica | SVM | RF |
| --- | --- | --- |
| F1 macro | **0.775 ± 0.040** | 0.710 ± 0.047 |
| Accuracy | **0.775** | 0.712 |
| AUC macro / micro | **0.949 / 0.956** | 0.928 / 0.934 |

**✅ Bien:**
- SVM gana en todas las métricas y en 93/100 folds. Wilcoxon p = 2.2e-17; t corregido p = 3.4e-5 → significativo con ambos.
- Rápido: 0.6 min (SVM) + 1.8 min (RF).
- Especie más fácil: *P. onca* (AUC 0.995); más difíciles: *wiedii*, *pardalis*, *tigrinus*.

**❌ Observaciones:**
- Shapiro (p = 0.20) no rechaza normalidad, así que el t-test pareado también habría sido válido. Se dejó documentado por qué se mantiene Wilcoxon (elegido a priori).
- La brecha entre AUC (~0.95) y F1 (~0.78) indica que el modelo ordena bien pero la clase top no siempre es la correcta.
- Las especies difíciles (*pardalis*, *wiedii*) son las que tienen ~50 % de fotos infrarrojas. Revisarlo en la matriz de confusión de la fase 6.

**⏭️ Pendiente:** fase 6 (evaluación final en test), fase 7 (t-SNE), fase 7b (color vs. gris), fase 8 (textos).

---

## 2026-11-17 — Fases 3–4: pipelines y optimización de hiperparámetros

**Hecho:** secciones 6 (pipelines) y 7 (optimización) de la notebook, ejecutadas en local con CV 10×10 y F1 macro. Resultados guardados en `outputs/cv/` (caché) y figuras en `outputs/figures/` (`opt_svm_C_gamma`, `opt_rf_depth_features`, `curvas_aprendizaje`, `opt_svm_optuna`).

| Modelo | Mejores hiperparámetros | F1 macro | Tiempo |
| --- | --- | --- | --- |
| SVM (grilla, 42 comb.) | `C=100`, `gamma=1e-4` | **0.776 ± 0.040** | 7.8 min |
| SVM (Optuna, 60 pruebas) | `C≈21.6`, `gamma≈1.4e-4` | 0.780 | 9.7 min |
| RF (grilla, 36 comb.) | 500 árboles, `max_depth=20`, `max_features=0.1` | **0.710 ± 0.047** | 36.5 min |

**✅ Bien:**
- La optimización subió el SVM de 0.684 (por defecto) a 0.776.
- Optuna confirma la región óptima de la grilla (mejora de 0.004, dentro del ruido).
- Mantener la grilla completa del RF fue acertado: gana `max_features=0.1`, la opción que la propuesta de aligerar habría quitado.

**❌ Mal / corregido:**
- **Estimación de tiempo del RF errada:** 36.5 min reales contra 15–25 estimados. Medir un entrenamiento suelto no refleja que 7 procesos en paralelo compiten por la CPU.
- **Empates resueltos al azar:** `C=100` y `C=1000` (y `max_depth` 20/40/None) dan el mismo F1; GridSearch elegía uno arbitrario (`C=1000`, `max_depth=40`). Se agregó un desempate explícito por **modelo más simple** y se recalcularon las curvas de aprendizaje.
- Bug menor: la caché de la curva del RF se guardaba como `lc_random` (por `"Random Forest".split()[0]`). Ahora la caché lleva un hash de los hiperparámetros, así que se invalida sola si cambian.
- Se limpiaron un aviso de tqdm (Optuna) y uno de pandas (`fillna` con texto).

**🧭 Decisiones:**
- Desempate por parsimonia (menor `C`; menor `max_depth`, luego menos árboles).
- En la búsqueda del SVM se usa `probability=False` (5× más rápido); las probabilidades se activan para la comparación (ROC/AUC).
- Los CSV de `outputs/cv/` se versionan: permiten regenerar las gráficas sin repetir los ~70 min de cómputo.

**⏭️ Pendiente:** fase 5, comparación estadística SVM vs. RF (Wilcoxon + todas las métricas).

---

## 2026-11-17 — Entorno local y fases 0–2 de la notebook

**Hecho:**
- Entorno local: `uv` (Homebrew) + `.venv` con Python 3.12.16, torch 2.15 (GPU del Mac vía MPS), scikit-learn 1.9.1, optuna, open_clip. Dependencias en `requirements.txt`; kernel de Jupyter `leopard`.
- [notebooks/clasificador_felideos.ipynb](../notebooks/clasificador_felideos.ipynb): setup (corre en local y en Colab), carga de datos, EDA completo, extracción ResNet50 y justificación de omitir feature selection. Ejecutada entera en local.
- Figuras del EDA en `outputs/figures/` (para la presentación). Embeddings en `outputs/embeddings/` (ignorados por git, se regeneran).

**✅ Bien:**
- Notebook completa en **35 s**; embeddings ResNet50 de 850 fotos en **8.8 s** (GPU MPS).
- Chequeo rápido sin optimizar (5-fold sobre desarrollo): SVM F1 macro **0.684 ± 0.049**, RF(300) **0.694 ± 0.043**. El azar sería 0.20, así que los features sirven y hay margen para optimizar.
- Tiempos reales: SVM 0.2 s/fit; RF 500 árboles 2.3 s (`sqrt`) y 8.7 s (`0.1`) por núcleo. En local el grid completo del RF cabe en ~15–25 min → **no hace falta aligerarlo** (la propuesta de la §13 solo aplica a Colab).

**❌ Hallazgos del EDA:**
- **32 % de las fotos (274/850) son infrarrojas** (grises aunque estén en RGB), repartidas de forma desigual: ~51 % de *pardalis*, ~48 % de *wiedii*, ~16 % de *onca*. Posible atajo del modelo ("nocturna → pardalis/wiedii") y factor de confusión para el experimento de color → se agregó a la fase 7b reportar también solo fotos a color.
- Dataset heterogéneo: cámaras trampa, zoológico, animales atropellados, marcas de agua.

**🧭 Decisiones:**
- Desarrollo local con `.venv`; la notebook detecta Colab y clona el repo (necesita que el repo sea público, o autenticarse).
- La salida de la notebook se commitea (figuras visibles en GitHub/Colab).

**⏭️ Pendiente:**
- Fase 3–4: pipelines y optimización de hiperparámetros (grid completo en local).
- Confirmar si el repo en GitHub es público (para Colab).

---

## 2026-11-17 — Plan B redefinido y Plan A mapeado al diagrama

**Hecho:**
- Se agregó a [plan-a.md](plan-a.md) la sección *Plan A sobre el diagrama de la pizarra*: cada paso del diagrama con su fase y su ítem de la rúbrica.
- Plan B redefinido en plan-a, decisiones y 00-proyecto.

**🧭 Decisiones:**
- **Plan B = cambiar el modelo preentrenado** (primero CLIP, el de la pizarra; luego EfficientNet o DINOv2), manteniendo el resto del pipeline.
- El dataset "segmentado" deja de ser plan B: conserva el fondo y es v2 en grises con 50 fotos menos. El tema del gris lo cubre la fase 7b.

---

## 2026-11-17 — Experimento extra: color vs. escala de grises

**Hecho:** se agregó la Fase 7b a [plan-a.md](plan-a.md) y la §15 a [decisiones-plan-a.md](decisiones-plan-a.md) (más fila de costo y diapositiva 13).

**🧭 Decisiones:**
- La pregunta es **color vs. escala de grises**, no "segmentado": la carpeta segmentada conserva el fondo.
- La versión gris se genera **en código desde v2**, para que solo cambie el color. La exportación gris de Roboflow queda como chequeo secundario.
- Mismos folds (10×10, seed 42), SVM reoptimizado para cada versión, Wilcoxon pareado.
- Es una sección extra; no reemplaza al ítem 6.

**⏭️ Pendiente:** implementarlo después de las fases principales del Plan A.

---

## 2026-11-17 — Diferencias entre las 4 carpetas

**Hecho:** se compararon las particiones de las 4 versiones y se actualizó la tabla de [00-proyecto.md](00-proyecto.md#datasets-disponibles).

**Hallazgos:**
- **v4 = v3 a mayor resolución:** misma partición exacta (850/850).
- v3/v4 mantienen 762/849 fotos en el mismo split que v2; la segmentada solo 518/800.
- En v3/v4 la foto #159 de *L. geoffroyi* quedó como `Unlabeled`.
- A la segmentada le faltan 50 fotos (sobre todo *wiedii*, *geoffroyi* y *tigrinus*).

**🧭 Decisiones:** se confirma v2 como el dataset más limpio.

---

## 2026-11-17 — Diagrama del pipeline (pizarra de clase)

**Hecho:** se replicó en HTML el diagrama de la pizarra: [diagrama-pipeline.html](diagrama-pipeline.html). Flujo: datos → feature extraction (a) clásica / (b) transfer learning → embedding → feature selection* (solo tabular) → modelos (curva de optimización + k-fold). Lo del Plan A está marcado en verde y hay una tabla pizarra vs. Plan A.

**✅ Bien:** la pizarra coincide con el Plan A: transfer learning, feature selection omitida para imágenes y optimización con k-fold.

**❌ Dudas de lectura:**
- La pizarra dice **CLIP** como modelo preentrenado; el Plan A usa **ResNet50**.
- Las barras verticales con "→ Target" se interpretaron como los folds del CV (no es seguro).

**⏭️ Pendiente:** decidir si se cambia ResNet50 por CLIP (o se usan ambos y se comparan).

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

# Decisiones del Plan A — base para la presentación

Este archivo explica **qué se decidió, por qué y qué se descartó** en el [Plan A](plan-a.md). Cada sección corresponde más o menos a una diapositiva y termina con un **mensaje clave** (una frase lista para la diapositiva) y una **pregunta probable en la defensa** con su respuesta.

Al final hay un esquema de diapositivas sugerido.

---

## 1. El problema

**Qué:** clasificar fotos de 5 especies de félidos neotropicales: *Leopardus geoffroyi*, *L. pardalis* (ocelote), *L. tigrinus* (tigrillo), *L. wiedii* (margay) y *Panthera onca* (jaguar).

**Por qué es interesante:**
- Son especies que se monitorean para su conservación (p. ej. con cámaras trampa), y clasificar las fotos a mano es lento.
- Varias tienen pelajes moteados muy parecidos (sobre todo *tigrinus*, *wiedii* y *pardalis*), así que no es un problema trivial.

> **Mensaje clave:** clasificación multiclase de imágenes (5 especies) con pelajes visualmente muy similares.

---

## 2. Elección del dataset: v2

**Qué:** de las 4 exportaciones de Roboflow disponibles, se usa `classificador_felideos.v2i.multiclass`.

| Versión | Formato | ¿Por qué sí / no? |
| --- | --- | --- |
| **v2** ✅ | 450×450, **color** | Las 5 clases están en train, valid y test; train balanceado |
| v3 ❌ | 150×150, grises | Valid/test **no tienen** *wiedii* ni *onca* → no se puede evaluar el 40 % de las clases |
| v4 ❌ | 450×450, grises | Mismo problema que v3 |
| Segmentada ❌ | 640×640, grises | Pese al nombre, **conserva el fondo**: es v2 en grises con 50 fotos menos y otra partición |

**Por qué color:** el color del pelaje es una de las pistas para distinguir especies, y la ResNet50 se entrenó con imágenes a color.

**Datos de v2:** 850 imágenes (train 680 · valid 85 · test 85), ~136 por clase en train.

**Verificación de calidad:** no hay imágenes duplicadas ni compartidas entre splits (se revisó con hash md5), así que no hay **fuga de datos**.

**Las 4 versiones son las mismas ~850 fotos** con distinto procesamiento y distinta partición. Por eso **no se mezclan**: la misma foto podría quedar en train y en test (fuga de datos).

> **Mensaje clave:** se eligió v2 porque es la única versión a color con las 5 especies en todos los splits y sin fuga de datos.

**Pregunta probable:** *¿Por qué no usaron la versión segmentada, que quita el fondo?*
→ La revisamos y no quita el fondo: comparada píxel a píxel con v2 da una correlación de 0.99. Es v2 en escala de grises con 50 fotos menos. El efecto del gris lo medimos aparte, de forma controlada (§15).

---

## 3. Cómo se dividen los datos

**Qué:**
- **train + valid = 765 imágenes** → conjunto de desarrollo para la validación cruzada.
- **test = 85 imágenes** → apartado, se usa **una sola vez** al final.

**Por qué:**
- La rúbrica pide validar con *repeated k-fold CV*, que ya reemplaza al conjunto de validación fijo.
- Con 765 imágenes en el CV en vez de 680, la estimación es más estable.
- No tocar el test hasta el final evita elegir el modelo "mirando" el test (sesgo optimista).

> **Mensaje clave:** el CV se hace sobre 765 imágenes; el test (85) se reserva para una única evaluación final.

---

## 4. Extracción de características: transfer learning con ResNet50

**Qué es ResNet50:** una red neuronal convolucional de 50 capas (He et al., 2015), preentrenada en ImageNet (~1,2 millones de fotos, 1000 categorías). Su innovación son las **conexiones residuales** (atajos), que permiten entrenar redes muy profundas.

**Cómo se usa:** se quita su última capa y se usa como **extractor fijo de características**. No se reentrena.

```
foto (224×224, normalizada con media/std de ImageNet) → ResNet50 → vector de 2048 números → clasificador ML
```

**Por qué:**
- **Pocos datos:** entrenar una red profunda desde cero con 765 imágenes sobreajustaría.
- **Ya sabe "ver":** aprendió bordes, texturas y patrones (manchas, rayas) útiles para distinguir pelajes.
- **Eficiencia:** los embeddings se calculan **una vez** (GPU en Colab) y se guardan. Todo el CV posterior corre en segundos sobre CPU.
- **Estándar y defendible:** modelo muy citado y documentado.

**Alternativas descartadas:**

| Alternativa | Por qué no (por ahora) |
| --- | --- |
| Píxeles → PCA → ML | Los píxeles crudos no capturan bien las texturas; se usa solo como contraste en el t-SNE |
| SIFT/HoG + Bag of Visual Words | Más trabajo manual y, en general, peor que las CNN preentrenadas |
| Fine-tuning de la red completa | Más costoso, riesgo de sobreajuste con pocos datos y más difícil de evaluar con CV 10×10 |
| CLIP (el de la pizarra), EfficientNet, DINOv2 | Buenas opciones; quedan como **plan B** si ResNet50 no rinde (CLIP primero) |

> **Mensaje clave:** transfer learning: ResNet50 preentrenada convierte cada foto en un vector de 2048 características; no se entrena ninguna red.

**Pregunta probable:** *¿Por qué no hacer fine-tuning?*
→ Con 765 imágenes el riesgo de sobreajuste es alto, y repetir el entrenamiento de la red en 100 folds sería computacionalmente inviable. Usar la red fija permite hacer un CV riguroso.

---

## 5. Feature selection: se omite

**Qué:** no se aplica selección de características.

**Por qué:**
- La rúbrica lo permite explícitamente cuando se usa transfer learning o el problema es de imágenes.
- Las 2048 características de la ResNet son representaciones densas y aprendidas; no son variables originales con significado individual que convenga filtrar.

> **Mensaje clave:** feature selection omitida (lo permite la rúbrica): las características vienen de una red preentrenada.

---

## 6. Pipeline

**Qué:** `Pipeline([StandardScaler → clasificador])` de scikit-learn.

**Por qué:**
- El escalador se ajusta **dentro de cada fold**, solo con los datos de entrenamiento de ese fold → no hay fuga de información del fold de validación.
- El mismo objeto se usa en la búsqueda de hiperparámetros, la comparación estadística y la evaluación final → código modular y reproducible.
- El SVM es sensible a la escala de las características; el Random Forest no, pero se mantiene el mismo pipeline para que la comparación sea justa.

> **Mensaje clave:** todo el preprocesamiento vive dentro de un pipeline, así que se reajusta por fold y no hay fuga de datos.

---

## 7. Los dos modelos a comparar: SVM vs. Random Forest

| | SVM (kernel RBF) | Random Forest |
| --- | --- | --- |
| Idea | Busca la frontera con **máximo margen** entre clases, en un espacio transformado por el kernel | **Ensamble de árboles** de decisión entrenados sobre muestras aleatorias; vota la mayoría |
| Fuerte en | Alta dimensión (2048) con pocos ejemplos (765) | Robusto, poco ajuste necesario, no le importa la escala |
| Hiperparámetros | `C` (penalización), `gamma` (alcance del kernel) | `n_estimators`, `max_depth`, `max_features` |

**Por qué estos dos:**
- Son **familias distintas** (márgenes con kernel vs. árboles), así que la comparación responde una pregunta real.
- Ambos dan probabilidades → se pueden calcular ROC y AUC, que la rúbrica exige.
- Hipótesis previa: el SVM debería rendir mejor en este escenario de alta dimensión y pocas muestras. El test estadístico lo confirmará o no.

> **Mensaje clave:** comparamos dos familias de modelos distintas sobre las mismas características: SVM (márgenes) vs. Random Forest (árboles).

**Pregunta probable:** *¿Por qué no Logistic Regression o KNN?*
→ Eran opciones válidas. Elegimos dos modelos con supuestos muy diferentes para que la comparación sea más informativa.

---

## 8. Validación: Repeated Stratified K-Fold 10×10

**Qué:** `RepeatedStratifiedKFold(n_splits=10, n_repeats=10, random_state=42)` → **100 evaluaciones** por modelo.

**Por qué:**
- Es el valor por defecto que recomienda la rúbrica.
- **Estratificado:** cada fold conserva la proporción de especies.
- **Repetido:** reduce la varianza de la estimación (un solo k-fold depende mucho de cómo cayó la partición).
- Es viable porque se trabaja sobre los vectores precalculados, no sobre las imágenes.
- **Misma semilla para ambos modelos** → los scores quedan **pareados** fold a fold, que es lo que necesita el test de Wilcoxon.

**Descartado:** nested CV → la rúbrica no lo recomienda por el costo computacional.

> **Mensaje clave:** CV estratificado 10×10 = 100 scores por modelo, pareados entre modelos.

---

## 9. Métrica principal: F1 macro

**Qué:** los hiperparámetros se eligen maximizando **F1 macro**.

**Por qué:**
- **F1** combina precision y recall.
- **Macro** promedia por clase sin ponderar → todas las especies pesan igual. Equivocarse con el tigrillo cuenta lo mismo que con el jaguar.
- Valid y test no están balanceados (entre 11 y 23 imágenes por clase), así que la accuracy sola podría engañar.

**Se reportan además** (lo pide la rúbrica): accuracy, precision, recall, F1 weighted, AUC ROC macro y micro, y curvas ROC.

> **Mensaje clave:** optimizamos F1 macro para que todas las especies importen por igual.

---

## 10. Optimización de hiperparámetros

**Qué:** `GridSearchCV` con el CV 10×10 y `scoring="f1_macro"`.

| Modelo | Espacio de búsqueda |
| --- | --- |
| SVM | `C` ∈ 10⁻³ … 10³ (escala log); `gamma` ∈ {"scale", 10⁻⁵ … 10⁻¹} |
| RF | `n_estimators` ∈ {100, 300, 500}; `max_depth` ∈ {None, 10, 20, 40}; `max_features` ∈ {sqrt, log2, 0.1} |

**Por qué estos rangos:** escala logarítmica para `C` y `gamma` porque su efecto es multiplicativo; los valores del RF cubren desde árboles poco profundos hasta sin límite.

**Resultados:**

| Modelo | Mejores hiperparámetros | F1 macro (CV 10×10) |
| --- | --- | --- |
| SVM | `C=100`, `gamma=1e-4` | **0.776 ± 0.040** |
| SVM con Optuna (extra) | `C≈21.6`, `gamma≈1.4e-4` | 0.780 (mejora dentro del ruido) |
| Random Forest | 500 árboles, `max_depth=20`, `max_features=0.1` | **0.710 ± 0.047** |

- **SVM:** con `gamma` grande (0.01–0.1) el modelo colapsa (F1 ≈ 0.07–0.22: predice casi siempre una clase). Con `gamma ≈ 1e-4` y `C ≥ 10` se llega a una meseta. El mejor `C` está en el borde de la grilla, pero es una meseta: `C=100` y `C=1000` dan exactamente lo mismo.
- **RF:** la profundidad casi no importa a partir de 20 (los árboles no llegan más hondo); `max_features=0.1` es algo mejor que `sqrt` y `log2`.
- **Empates:** se elige el modelo más simple (menor `C`, menor profundidad). Es una decisión explícita, defendible por parsimonia.
- **Curvas de aprendizaje:** F1 de entrenamiento = 1.0 en ambos (memorizan el train) y la validación sigue subiendo al agregar imágenes → **varianza alta**: más datos mejorarían el modelo. El SVM está por encima del RF en todos los tamaños.

**Entregables:**
- **Gráfica de optimización:** F1 macro (media ± std) vs. `C` y vs. `max_depth`.
- **Curva de aprendizaje:** score de entrenamiento vs. validación según el tamaño del train → diagnóstico de sesgo/varianza.
- *Extra:* Optuna para el SVM, comparado con el grid.

> **Mensaje clave:** búsqueda en grilla con el mismo CV 10×10; se presentan la curva de optimización y la curva de aprendizaje.

---

## 11. Comparación estadística: test de Wilcoxon

**Qué:** test de **Wilcoxon de rangos con signo** pareado sobre los 100 F1 macro de cada modelo, con α = 0.05.

- **H₀:** no hay diferencia en F1 macro entre SVM y RF.
- **H₁:** sí hay diferencia.

**Por qué Wilcoxon y no t-test:**
- **No asume normalidad** de las diferencias (los scores de CV suelen estar acotados y sesgados).
- Es **pareado**, coherente con usar los mismos folds en ambos modelos.

**Limitación que hay que saber explicar:** los 100 folds no son totalmente independientes (comparten datos de entrenamiento), así que el p-valor tiende a ser optimista. Es una limitación conocida del enfoque que pide la rúbrica.

**Resultados (F1 macro, CV 10×10):** SVM 0.775 ± 0.040 vs. RF 0.710 ± 0.047. Diferencia media **+0.066**, el SVM gana en **93/100 folds**.

| Test | Resultado | Conclusión (α = 0.05) |
| --- | --- | --- |
| Shapiro–Wilk (normalidad de las diferencias) | p = 0.20 | No se rechaza la normalidad |
| **Wilcoxon pareado** | W = 58, **p = 2.2 × 10⁻¹⁷** | Se rechaza H₀ |
| t-test corregido (Nadeau–Bengio) | p = 3.4 × 10⁻⁵ | Se rechaza H₀ |

El SVM gana en todas las métricas, incluidas AUC macro (0.949 vs. 0.928) y micro (0.956 vs. 0.934). Las especies más difíciles son *wiedii*, *pardalis* y *tigrinus*; la más fácil, *P. onca*.

**Pregunta probable:** *Si Shapiro no rechaza la normalidad, ¿por qué no usaron un t-test?*
→ El Wilcoxon se eligió de antemano porque no depende de ese supuesto. Con normalidad, ambos son válidos, y ambos llevan a la misma conclusión.

**Pregunta probable:** *¿Cómo manejan que los folds no sean independientes?*
→ Agregamos el t-test corregido de Nadeau y Bengio, que infla la varianza según la proporción test/train de cada fold. El p-valor sube de 10⁻¹⁷ a 10⁻⁵, pero la diferencia sigue siendo significativa.

> **Mensaje clave:** Wilcoxon pareado sobre 100 scores: test no paramétrico, coherente con folds compartidos.

---

## 12. Evaluación final y t-SNE

**Evaluación final:** ambos modelos se reentrenan con las 765 imágenes y se evalúan **una vez** en las 85 de test (classification report, matriz de confusión, ROC). Como el test es pequeño, las conclusiones se apoyan principalmente en el CV.

**t-SNE:** proyección 2D de los vectores de la ResNet, coloreada por especie, comparada con la de los píxeles crudos.
- **Para qué:** ver si las especies forman grupos separables en el espacio de características y cuáles se solapan.
- Se conecta con la matriz de confusión: las especies que se solapan en el t-SNE deberían ser las que más se confunden.

> **Mensaje clave:** el t-SNE muestra visualmente qué aporta el transfer learning frente a los píxeles crudos.

**Resultados en test (85 imágenes):**

| | SVM | Random Forest |
| --- | --- | --- |
| Aciertos | 65/85 (0.765) | 68/85 (0.800) |
| F1 macro | 0.749 | 0.786 |
| AUC macro | 0.940 | 0.943 |

- En test gana el RF por **3 imágenes**, pero **McNemar p = 0.55 → no significativo**. La conclusión se basa en el CV (100 folds), no en 85 fotos.
- Mejor reconocidas: *L. geoffroyi* y *P. onca*. Más confundidas: *wiedii* ↔ *tigrinus* ↔ *pardalis*.
- Acierto del SVM: **82.7 % en fotos a color vs. 67.5 % en infrarrojas**.

**Resultados del t-SNE:**
- Con embeddings ResNet50 hay grupos claros (*P. onca* aislada); con píxeles crudos todo se mezcla → el transfer learning aporta.
- **Hallazgo clave:** el eje principal del espacio es **día/noche**: las fotos infrarrojas forman su propia isla y dentro de ella las especies se mezclan.

> **Mensaje clave (hallazgo):** la ResNet50 separa primero las fotos nocturnas de las diurnas y después la especie; por eso las fotos infrarrojas son las que más se equivocan.

**Pregunta probable:** *En test gana el Random Forest. ¿Entonces no es mejor el SVM?*
→ En test la diferencia son 3 imágenes de 85 y McNemar da p = 0.55: no es significativa. El CV evalúa 100 particiones y ahí el SVM gana en 93 de 100 (Wilcoxon p ≈ 10⁻¹⁷). Un test de 85 imágenes no tiene poder para distinguir modelos tan cercanos.

---

## 13. Costo computacional estimado

> **Tiempos reales en local (Mac M3 Pro, 11 núcleos):** extracción ResNet50 en 8.8 s; búsqueda SVM **7.8 min**; búsqueda RF **36.5 min** (más que los 15–25 min estimados: 7 búsquedas en paralelo se frenan entre sí); Optuna (60 pruebas × 100 folds) **9.7 min**; curvas de aprendizaje ~15 min. Total de la notebook hasta aquí: **~70 min**. El mejor RF usa `max_features=0.1`, justo la opción que la propuesta de aligerar habría quitado, así que se mantuvo la grilla completa.

Estimación previa para **Colab gratuito** (2 núcleos de CPU + GPU T4). Son números aproximados, sin medir todavía; pueden variar ×2 en cualquier dirección. Se reemplazarán por tiempos reales al correr la notebook.

| Paso | Trabajo | Tiempo estimado |
| --- | --- | --- |
| Descomprimir y cargar 850 imágenes | — | < 1 min |
| Extracción de features con ResNet50 | 850 imágenes, una sola vez, en GPU | ~1 min (5–10 min solo con CPU) |
| **Búsqueda de hiperparámetros SVM** | 42 combinaciones × 100 folds = 4.200 entrenamientos | **~5–15 min** |
| **Búsqueda de hiperparámetros RF** | 36 combinaciones × 100 folds = 3.600 entrenamientos | **~2–4 horas** ⚠️ |
| Curvas de aprendizaje (ambos modelos) | ~500 entrenamientos c/u | ~15–30 min (sobre todo RF) |
| Comparación estadística | 100 folds por modelo + probabilidades | ~10–15 min |
| Evaluación final en test | 2 entrenamientos | < 1 min |
| t-SNE (3 perplexities + píxeles crudos) | 850 puntos | ~1–2 min |
| Optuna (extra) | Límite de tiempo fijado por nosotros | ~10 min |
| Experimento color vs. gris (extra) | Embeddings grises + búsqueda y CV del SVM | ~10–20 min |
| **Total con el plan tal como está** | | **≈ 3–5 horas** |

**El cuello de botella es el Random Forest:** cada entrenamiento construye cientos de árboles sobre 2048 características, y la búsqueda lo repite 3.600 veces. Además, una sesión tan larga en Colab gratuito corre el riesgo de desconectarse.

**Ajustes propuestos para bajar el total a ~1 hora:**
- **Búsqueda aleatoria para RF:** `RandomizedSearchCV` con ~12 combinaciones en vez de las 36 (~3× más rápido).
- **CV más liviano solo para la búsqueda del RF:** 5×2 (lo permite la rúbrica, justificándolo). Se mantiene el **10×10 completo para la comparación final**, que es donde el test de Wilcoxon lo necesita.
- **Quitar `max_features=0.1`**, la opción más lenta.
- **Guardar resultados en Drive después de cada paso**, para no perder trabajo si Colab se desconecta.
- La búsqueda del SVM queda igual (es barata).

> **Estado:** propuesta pendiente de confirmar con el grupo. Si se adopta, actualizar §10 y [plan-a.md](plan-a.md).

> **Mensaje clave:** gracias a extraer los features una sola vez, todo el experimento cabe en ~1 hora de Colab. El único paso caro es la búsqueda del Random Forest, y se aligera sin tocar la comparación estadística.

**Pregunta probable:** *¿Por qué el mejor C está en el borde de la grilla? ¿No habría que ampliarla?*
→ La curva ya está plana desde C = 10: C = 100 y C = 1000 dan el mismo F1. Optuna, que explora valores continuos, tampoco encontró nada mejor fuera de ese rango (0.780 vs. 0.776).

**Pregunta probable:** *¿Usar un CV distinto para la búsqueda del RF no hace injusta la comparación?*
→ No. La búsqueda solo elige los hiperparámetros. La comparación entre SVM y RF se hace después, con el **mismo** CV 10×10 para ambos modelos.

---

## 14. Riesgos y plan B

| Riesgo | Respuesta |
| --- | --- |
| Test pequeño → métricas ruidosas | Conclusiones basadas en el CV (100 scores) |
| Especies muy parecidas | Esperado; se analiza con la matriz de confusión y el t-SNE |
| Rendimiento bajo | Plan B: cambiar el modelo preentrenado, primero **CLIP** y luego EfficientNet o DINOv2 |
| Costo computacional (~3–5 h con el plan original) | Ajustes de la §13 → ~1 h |

---

## 15. Experimento extra: ¿importa el color?

**Qué:** comparar el mismo pipeline (ResNet50 → SVM) con las fotos **a color** y **en escala de grises**.

**Por qué es interesante:** varias especies se distinguen en parte por el tono del pelaje (p. ej. jaguar y ocelote), pero otras casi solo por el patrón de manchas. Responder si el color aporta da una conclusión concreta sobre el problema, no solo sobre los modelos.

**Cómo se controla para que sea justo:**

| Factor | Control |
| --- | --- |
| Fotos | Las mismas 850 de v2; la versión gris se genera en código desde v2 |
| Partición y folds | Mismo `RepeatedStratifiedKFold(10, 10, random_state=42)` → scores pareados |
| Modelo | Mismo pipeline; el SVM se reoptimiza para cada versión |
| Resolución | Irrelevante: ambas entran a la ResNet a 224×224 |
| Test | Wilcoxon pareado sobre los 100 F1 macro, α = 0.05 |

**Por qué no usar la carpeta "segmentada" directamente:** tiene otras 800 fotos, otra partición y, pese al nombre, conserva el fondo. Con ella, la diferencia de score podría deberse a las fotos y no al color. Se usa solo como chequeo secundario.

**Costo:** ~1 min de GPU (embeddings grises) + ~5–15 min de CPU (SVM).

> **Mensaje clave:** experimento controlado donde solo cambia el color: mismas fotos, mismos folds, mismo modelo, test de Wilcoxon pareado.

**Resultados (SVM, CV 10×10, F1 macro):**

| Subconjunto | Color | Gris | Diferencia | Wilcoxon p | t corregido p |
| --- | --- | --- | --- | --- | --- |
| Todas las fotos | 0.775 | 0.767 | +0.008 | 0.022 | 0.54 |
| Solo fotos a color | 0.794 | 0.768 | +0.026 | 1.7 × 10⁻⁵ | 0.16 |
| Solo fotos infrarrojas | 0.650 | 0.684 | −0.034 | 0.0012 | 0.31 |

- **En total el color casi no importa** (+0.008, no robusto según el t corregido): manda el patrón de manchas.
- **De día ayuda, de noche estorba:** en fotos a color gana el color (sobre todo *wiedii* +0.09 y *tigrinus* +0.06); en infrarrojas gana el gris, porque al pasar todo a gris las fotos nocturnas dejan de ser un grupo aparte.
- El gris de Roboflow da lo mismo que el gris hecho en código (0.763 vs. 0.771).

> **Mensaje clave (resultado):** el color aporta poco en total. Ayuda en fotos diurnas y estorba en las nocturnas: la información principal es el patrón de manchas.

**Pregunta probable:** *El Wilcoxon da significativo en total (p = 0.02). ¿Por qué dicen que el color no importa?*
→ Porque el t corregido, que tiene en cuenta el solapamiento entre folds, da p = 0.54. La diferencia es de 0.008, menos de un punto de F1. Es estadísticamente frágil y prácticamente irrelevante.

**Pregunta probable:** *¿Por qué replicar el gris a 3 canales?*
→ La ResNet50 espera 3 canales (RGB). Copiar el canal gris en los 3 mantiene la arquitectura y los pesos preentrenados sin cambios.

**Pregunta probable:** *¿Esto no es lo mismo que el ítem 6?*
→ No. El ítem 6 compara dos técnicas de ML con los mismos datos; esto compara dos versiones de los datos con la misma técnica.

---

## Esquema de diapositivas sugerido

1. **Título:** clasificador de félidos neotropicales
2. **Problema y motivación** → §1
3. **Dataset:** por qué v2 + distribución de clases → §2
4. **EDA:** grid de imágenes, balance → (resultados de la notebook)
5. **Metodología general:** diagrama del pipeline completo → [diagrama-pipeline.html](diagrama-pipeline.html)
   `foto → ResNet50 → 2048 features → StandardScaler → SVM / RF`
6. **Transfer learning con ResNet50** → §4
7. **Partición de datos y validación 10×10** → §3, §8
8. **Modelos y métrica** → §7, §9
9. **Optimización de hiperparámetros:** gráficas → §10
10. **Comparación estadística:** boxplot + Wilcoxon → §11
11. **Resultados en test:** matriz de confusión, ROC → §12
12. **t-SNE** → §12
13. **Experimento extra: ¿importa el color?** boxplot + Wilcoxon → §15
14. **Costo computacional:** tabla de tiempos (reales, una vez medidos) → §13
15. **Conclusiones**
16. **Limitaciones y trabajo futuro** → §11 (limitación), §14
17. **Referencias**

---

## Referencias para las diapositivas

- [1] K. He, X. Zhang, S. Ren y J. Sun, "Deep residual learning for image recognition," en *Proc. IEEE CVPR*, 2016, pp. 770–778.
- [2] L. van der Maaten y G. Hinton, "Visualizing data using t-SNE," *J. Mach. Learn. Res.*, vol. 9, pp. 2579–2605, 2008.
- [3] F. Pedregosa *et al.*, "Scikit-learn: Machine learning in Python," *J. Mach. Learn. Res.*, vol. 12, pp. 2825–2830, 2011.
- [4] F. Wilcoxon, "Individual comparisons by ranking methods," *Biometrics Bulletin*, vol. 1, no. 6, pp. 80–83, 1945.
- [5] L. Breiman, "Random forests," *Machine Learning*, vol. 45, no. 1, pp. 5–32, 2001.
- [6] C. Cortes y V. Vapnik, "Support-vector networks," *Machine Learning*, vol. 20, no. 3, pp. 273–297, 1995.

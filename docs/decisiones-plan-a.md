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
| Segmentada | 640×640, grises | Válida, pero queda como **plan B** (y en las muestras revisadas el fondo no está removido) |

**Por qué color:** el color del pelaje es una de las pistas para distinguir especies, y la ResNet50 se entrenó con imágenes a color.

**Datos de v2:** 850 imágenes (train 680 · valid 85 · test 85), ~136 por clase en train.

**Verificación de calidad:** no hay imágenes duplicadas ni compartidas entre splits (se revisó con hash md5), así que no hay **fuga de datos**.

**Las 4 versiones son las mismas ~850 fotos** con distinto procesamiento y distinta partición. Por eso **no se mezclan**: la misma foto podría quedar en train y en test (fuga de datos).

> **Mensaje clave:** se eligió v2 porque es la única versión a color con las 5 especies en todos los splits y sin fuga de datos.

**Pregunta probable:** *¿Por qué no usaron la versión segmentada, que quita el fondo?*
→ Queda como plan B para comparar. Empezamos con v2 porque conserva el color y no depende de la calidad de la segmentación.

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
| EfficientNet / ViT (DINOv2) | Buenas opciones; quedan como plan B si ResNet50 no rinde |

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

> **Mensaje clave:** Wilcoxon pareado sobre 100 scores: test no paramétrico, coherente con folds compartidos.

---

## 12. Evaluación final y t-SNE

**Evaluación final:** ambos modelos se reentrenan con las 765 imágenes y se evalúan **una vez** en las 85 de test (classification report, matriz de confusión, ROC). Como el test es pequeño, las conclusiones se apoyan principalmente en el CV.

**t-SNE:** proyección 2D de los vectores de la ResNet, coloreada por especie, comparada con la de los píxeles crudos.
- **Para qué:** ver si las especies forman grupos separables en el espacio de características y cuáles se solapan.
- Se conecta con la matriz de confusión: las especies que se solapan en el t-SNE deberían ser las que más se confunden.

> **Mensaje clave:** el t-SNE muestra visualmente qué aporta el transfer learning frente a los píxeles crudos.

---

## 13. Costo computacional estimado

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

**Pregunta probable:** *¿Usar un CV distinto para la búsqueda del RF no hace injusta la comparación?*
→ No. La búsqueda solo elige los hiperparámetros. La comparación entre SVM y RF se hace después, con el **mismo** CV 10×10 para ambos modelos.

---

## 14. Riesgos y plan B

| Riesgo | Respuesta |
| --- | --- |
| Test pequeño → métricas ruidosas | Conclusiones basadas en el CV (100 scores) |
| Especies muy parecidas | Esperado; se analiza con la matriz de confusión y el t-SNE |
| Rendimiento bajo | Plan B: otro backbone (EfficientNet, DINOv2) o el dataset segmentado |
| Costo computacional (~3–5 h con el plan original) | Ajustes de la §13 → ~1 h |

---

## Esquema de diapositivas sugerido

1. **Título:** clasificador de félidos neotropicales
2. **Problema y motivación** → §1
3. **Dataset:** por qué v2 + distribución de clases → §2
4. **EDA:** grid de imágenes, balance → (resultados de la notebook)
5. **Metodología general:** diagrama del pipeline completo
   `foto → ResNet50 → 2048 features → StandardScaler → SVM / RF`
6. **Transfer learning con ResNet50** → §4
7. **Partición de datos y validación 10×10** → §3, §8
8. **Modelos y métrica** → §7, §9
9. **Optimización de hiperparámetros:** gráficas → §10
10. **Comparación estadística:** boxplot + Wilcoxon → §11
11. **Resultados en test:** matriz de confusión, ROC → §12
12. **t-SNE** → §12
13. **Costo computacional:** tabla de tiempos (reales, una vez medidos) → §13
14. **Conclusiones**
15. **Limitaciones y trabajo futuro** → §11 (limitación), §14
16. **Referencias**

---

## Referencias para las diapositivas

- [1] K. He, X. Zhang, S. Ren y J. Sun, "Deep residual learning for image recognition," en *Proc. IEEE CVPR*, 2016, pp. 770–778.
- [2] L. van der Maaten y G. Hinton, "Visualizing data using t-SNE," *J. Mach. Learn. Res.*, vol. 9, pp. 2579–2605, 2008.
- [3] F. Pedregosa *et al.*, "Scikit-learn: Machine learning in Python," *J. Mach. Learn. Res.*, vol. 12, pp. 2825–2830, 2011.
- [4] F. Wilcoxon, "Individual comparisons by ranking methods," *Biometrics Bulletin*, vol. 1, no. 6, pp. 80–83, 1945.
- [5] L. Breiman, "Random forests," *Machine Learning*, vol. 45, no. 1, pp. 5–32, 2001.
- [6] C. Cortes y V. Vapnik, "Support-vector networks," *Machine Learning*, vol. 20, no. 3, pp. 273–297, 1995.

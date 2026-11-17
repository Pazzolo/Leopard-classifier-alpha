# 2. EDA — 0.6 pts

> [Notion](https://pointy-whistle-142.notion.site/1e505737435780f687d9e846fb9b7beb)

## Requisito

Realizar un *Exploratory Data Analysis* del dataset. Como mínimo responder:

1. ¿Es la data balanceada?
2. ¿Está normalizada?
3. ¿Hay estacionalidad en series de tiempo? *(no aplica: imágenes)*
4. ¿Hacen falta transformaciones no lineales (skewness)? *(no aplica directamente a imágenes)*
5. Dominio imágenes:
   1. Distribución de labels
   2. Grid de imágenes
   3. Histograma de la resolución (H × W)

## Checklist (dominio imágenes)

- [ ] Distribución de labels por split (train / valid / test), gráfico de barras.
- [ ] Responder explícitamente si está balanceada.
- [ ] Grid de imágenes de ejemplo por clase.
- [ ] Histograma de resolución H × W. Las exportaciones de Roboflow están redimensionadas a un tamaño fijo, así que el histograma será un único valor. Explicarlo y, si se puede, mostrar la resolución de las imágenes originales.
- [ ] Normalización: rango y distribución de los píxeles (media y desviación por canal) y cómo se normaliza antes del modelo preentrenado (media/std de ImageNet).
- [ ] Indicar por qué no aplican estacionalidad y skewness.
- [ ] Revisar duplicados o casi duplicados entre splits (ver nota en [00-proyecto.md](../00-proyecto.md#datasets-disponibles)).

## Notas


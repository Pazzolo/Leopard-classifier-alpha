# 3. Feature Extraction — 0.25 pts

> [Notion](https://pointy-whistle-142.notion.site/325057374357802f8ac5ef311259e885)

## Requisito

Para datos no estructurados (imágenes, texto, audio) cuando se usa transfer learning.

Opciones para **imágenes**:

1. Clásica: imagen → flattening → PCA → algoritmo ML
2. Clásica: imagen → descriptores (SURF, SIFT, HoG) + Bag of Visual Words → algoritmo ML
3. Transfer learning: imagen → modelo preentrenado → embedding → algoritmo ML
   1. CNN: ResNet, MobileNet
   2. ViTs (Vision Transformers)
4. Otros

*(Las opciones de texto, audio, series de tiempo y tabular están en [Plantilla.md §3.3](../../Plantilla.md).)*

## Checklist

- [ ] Elegir el backbone preentrenado (p. ej. ResNet50, EfficientNet, DINOv2/ViT) y justificar la elección.
- [ ] Extraer los embeddings una vez y guardarlos (`.npy`/`.parquet`) para no recalcularlos en cada CV.
- [ ] Documentar la dimensión del vector y el preprocesamiento (resize, normalización ImageNet).
- [ ] Si el dataset es en escala de grises, replicar a 3 canales y comentar el efecto.
- [ ] Opcional: una opción clásica (HoG o PCA) como baseline o como segunda técnica para el ítem 6.

## Notas


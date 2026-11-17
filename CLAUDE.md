# Reglas del proyecto

Instrucciones para Claude Code (y referencia para el grupo) al trabajar en este repositorio.

## Git

- **Claude puede hacer commits por su cuenta**, sin pedir confirmación, cada vez que termine una unidad de trabajo coherente (un ítem, un documento, una sección de la notebook, un arreglo).
- Commits pequeños y enfocados, con mensajes descriptivos en español.
- Se trabaja directamente sobre `main`.
- **Push:** solo cuando el usuario lo pida.
- Los datasets se versionan **solo como zips** en `zips/`. Las carpetas descomprimidas `classificador_*/` quedan fuera de git (ver `.gitignore`).

## Documentación

- La documentación de trabajo vive en `docs/` (índice en `docs/README.md`).
- `Plantilla.md` es la extracción original de Notion: no se modifica.
- Al completar o avanzar un ítem, actualizar su checklist en `docs/items/` y el estado en la tabla de `docs/README.md`.
- Idioma de documentos y notebook: español (el ítem 9 de la rúbrica evalúa ortografía y redacción).

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
- Elegir la versión del dataset (candidatas: v2 color o la segmentada).
- Verificar los duplicados.
- Completar tema, motivación e integrantes en `00-proyecto.md`.

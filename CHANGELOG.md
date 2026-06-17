# Changelog

Registro de ramas, PRs y merges para trazabilidad del proyecto.
Formato de commits: [Conventional Commits](https://www.conventionalcommits.org/).

## [En progreso] — Punto 5: Optimización avanzada

### Rama `feature/05-optimizacion-avanzada` (PR pendiente → `main`)
Objetivo: recuperar el notebook de optimización de la rama `machine-learning`, sanearlo y dejarlo
autocontenido como `05_optimizacion_avanzada.ipynb`.

- `feat(05)`: optimización avanzada autocontenida (GridSearchCV + SHAP + diagnóstico de residuales),
  cargando el parquet del 02; figuras a `data/figuras/` (no versionada).

Pendiente: PR contra `main`, verificación en Codespaces y merge `--no-ff`.

---

## [Merged] — Punto 4: Validación metodológica

### Rama `feature/04-validacion-metodologica` → `main` (PR #4, merge `--no-ff` en `2462946`)
- `feat(04)`: notebook de validación metodológica con verificación de integración del pipeline
  `00→01→02→03` y los tres problemas metodológicos (P1 Gini, P2 ground truth, P3 outliers por tipo).

---

## [Merged] — Punto 3: resolución de conflictos del modelado

### Rama `fix/03-resolver-conflictos-modeling` → `main` (PR #3, merge `--no-ff` en `9b2fca9`)
Objetivo: dejar el punto 3 (modelado) **funcional y consistente**, resolviendo los conflictos
documentados en `AUDITORIA.md`.

Secuencia de commits (de más antiguo a más reciente):

1. `docs`: auditoría de notebooks 00–04 y plan de ramas (`AUDITORIA.md`, `CHANGELOG.md`).
2. `fix(00)`: incluir `wbgapi` y `pyarrow` en el `requirements.txt` del setup (C1).
3. `fix(01)`: filtrar correctamente los agregados regionales del Banco Mundial (C2).
4. `fix(02)`: filtrar agregados regionales por región vacía/nula (C2).
5. `feat(02)`: añadir flag `is_pandemic` para 2020–2021.
6. `fix(02)`: persistir el dataset procesado en `data/processed` — puente 02→03 (C3).
7. `fix(03)`: reconstrucción consistente con el 02 (flags pandemia/Gini, Parquet) (C1, C4).
8. `chore(03)`: asignar ids estables a las celdas del notebook.
9. `feat(03)`: test de robustez de pandemia, interpretación de R² y esquema consistente.
10. `docs`: actualizar CHANGELOG con la secuencia real de commits de la rama.

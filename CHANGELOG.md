# Changelog

Registro de ramas, PRs y merges para trazabilidad del proyecto.
Formato de commits: [Conventional Commits](https://www.conventionalcommits.org/).

## [En progreso] — Auditoría y resolución del punto 3

### Rama `fix/03-resolver-conflictos-modeling` (PR pendiente → `main`)
Objetivo: dejar el punto 3 (modelado) **funcional y consistente**, resolviendo los conflictos
documentados en `AUDITORIA.md`.

- `docs`: auditoría completa de notebooks 00–04 y plan de ramas (`AUDITORIA.md`, `CHANGELOG.md`).
- `fix(00)`: incluir `wbgapi` y `pyarrow` en el `requirements.txt` generado por el setup (C1).
- `fix(01)`: filtrar correctamente los agregados regionales del Banco Mundial (C2).
- `fix(02)`: filtrar agregados, añadir flag `is_pandemic` y persistir el dataset procesado (C2, C3).
- `fix(03)`: reconstrucción funcional (Parquet + flags pandemia/Gini) consistente con el 02 (C1, C4).
- `feat(03)`: test de robustez de pandemia (error con vs. sin 2020–2021).
- `docs(03)`: interpretación cuidadosa de R².

# Changelog

Registro de ramas, PRs y merges para trazabilidad del proyecto.
Formato de commits: [Conventional Commits](https://www.conventionalcommits.org/).

## [En progreso] — Auditoría y resolución del punto 3

### Rama `fix/03-resolver-conflictos-modeling` (PR pendiente → `main`)
Objetivo: dejar el punto 3 (modelado) **funcional y consistente**, resolviendo los conflictos
documentados en `AUDITORIA.md`.

Secuencia real de commits (de más antiguo a más reciente):

1. `docs`: auditoría de notebooks 00–04 y plan de ramas (`AUDITORIA.md`, `CHANGELOG.md`).
2. `fix(00)`: incluir `wbgapi` y `pyarrow` en el `requirements.txt` del setup (C1).
3. `fix(01)`: filtrar correctamente los agregados regionales del Banco Mundial (C2).
4. `fix(02)`: filtrar agregados regionales por región vacía/nula (C2).
5. `feat(02)`: añadir flag `is_pandemic` para 2020–2021.
6. `fix(02)`: persistir el dataset procesado en `data/processed` — puente 02→03 (C3).
7. `fix(03)`: reconstrucción consistente con el 02 (flags pandemia/Gini, Parquet) (C1, C4).
8. `chore(03)`: asignar ids estables a las celdas del notebook.
9. `feat(03)`: test de robustez de pandemia, interpretación de R² y esquema consistente.

Pendiente: PR contra `main` (gh), verificación de integración en Codespaces y merge `--no-ff`.

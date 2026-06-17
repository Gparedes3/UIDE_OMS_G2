# UIDE_OMS_G2 — Predicción de mortalidad y natalidad

Proyecto integrador (UIDE) que modela las **tasas brutas de mortalidad y natalidad** de los países
del mundo a partir de indicadores socioeconómicos del **Banco Mundial** (vía `wbgapi`), 2000–2022.

## Pipeline (notebooks)

Ejecutar **en orden** (`notebooks/`):

| # | Notebook | Contenido |
|---|----------|-----------|
| 00 | `00_setup` | Entorno reproducible: devcontainer, dependencias, PyTorch CPU. |
| 01 | `01_eda` | Adquisición vía `wbgapi`, calidad de datos, EDA y formulación del problema. |
| 02 | `02_preprocesamiento` | Imputación jerárquica, estructura etaria, flags `gini_imputed`/`is_pandemic`, persistencia a Parquet. |
| 03 | `03_modelado` | Ridge/RF/XGBoost/LightGBM con `GroupKFold` por país, robustez de pandemia, interpretación de R². |
| 04 | `04_validacion_metodologica` | Verificación de integración del pipeline + tres problemas metodológicos. |
| 05 | `05_optimizacion_avanzada` | GridSearchCV + SHAP + diagnóstico de residuales. |
| 06 | `06_reporte_final` | Model card, tabla final de métricas, limitaciones y checklist de reproducibilidad. |

## Reproducir

1. Abrir el repo en **GitHub Codespaces** (devcontainer Python 3.13). Tras cambiar dependencias:
   *Rebuild Container*.
2. Ejecutar los notebooks `00 → 06` (Run All en cada uno). El notebook 01 y 02 regeneran `data/`.

> La carpeta `data/` **no se versiona** (`.gitignore`): cada quien la regenera ejecutando los
> notebooks. Los datos se guardan en **Parquet** (`pyarrow`).

## Trazabilidad

- `AUDITORIA.md` — auditoría inicial de notebooks, conflictos y plan de ramas.
- `CHANGELOG.md` — secuencia de ramas, PRs y merges.

## Equipo

Guillermo Paredes · Donato Oña · Mateo Villacreses

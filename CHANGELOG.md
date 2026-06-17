# Changelog

Registro general del avance del proyecto.

## Modelado — resolución de conflictos y pipeline funcional (punto 3)

- Auditoría inicial de los notebooks 00–04 y los conflictos detectados (`AUDITORIA.md`).
- `00`: el `requirements.txt` generado ahora incluye `wbgapi` y `pyarrow` (y `shap`); se corrige el
  `ImportError: pyarrow` que rompía el pipeline.
- `01` y `02`: filtrado correcto de los agregados regionales del Banco Mundial (región vacía, no `'NA'`).
- `02`: flag `is_pandemic` (2020–2021) y persistencia del dataset procesado a `data/processed`.
- `03`: reconstrucción consistente con el `02`, uso de flags pandemia/Gini como features, test de
  robustez de pandemia (con vs. sin 2020–2021) e interpretación cuidadosa de R².

## Validación metodológica (punto 4)

- `04_validacion_metodologica.ipynb`: celda de verificación de integración del pipeline
  `00→01→02→03` y análisis de los tres problemas metodológicos (alcance del Gini, *ground truth*,
  outliers por tipo).

## Optimización avanzada (punto 5)

- `05_optimizacion_avanzada.ipynb`: GridSearchCV + SHAP + diagnóstico de residuales, autocontenido
  (carga el parquet del `02`); figuras a `data/figuras/` (no versionada).

## Reporte final (punto 6)

- `06_reporte_final.ipynb`: resumen ejecutivo, tabla final de métricas, model card, limitaciones y
  checklist de reproducibilidad.
- `README` con la descripción del proyecto y el orden de ejecución.

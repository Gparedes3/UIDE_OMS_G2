# Auditoría del pipeline — Proyecto Integrador (mortalidad / natalidad)

**Fecha:** 2026-06-16
**Rama de auditoría:** `fix/03-resolver-conflictos-modeling`
**Alcance:** notebooks `00`–`04`, configuración del entorno, topología de ramas y conflictos pendientes.

> Objetivo de esta auditoría: dejar terminados y consistentes los puntos hasta el **3**, resolver
> los conflictos existentes y planificar el avance hacia **4, 5 y 6**.

---

## 0. Topología de ramas y "conflictos" reales

No hay marcadores de conflicto de merge (`<<<<<<<`) en el árbol de trabajo: `main` está limpio.
Los "conflictos" del proyecto son de otra naturaleza:

| Rama | Estado | Observación |
|---|---|---|
| `main` | Funcional pero **incompleto** | Tiene `00`–`03`. El `03` **no corre** (ver C1). No tiene `04`. |
| `origin/machine-learning` | = `main` + `04_opt_eval_avanzada.ipynb` | El `04` nunca se integró a `main`. Es de *optimización + SHAP*, **no** de validación metodológica. |
| `origin/dev` | **Obsoleta** | Solo contiene `README.md`; está muy por detrás de `main`. |

**Conflicto de identidad del notebook 04.** El enunciado pide que `04` sea
`04_validacion_metodologica.ipynb` (verificación de integración del pipeline + tres problemas
metodológicos). Lo que existe en `origin/machine-learning` es `04_opt_eval_avanzada.ipynb`
(GridSearchCV + SHAP + residuales). Son **dos cosas distintas**: hay que decidir cómo reconciliarlas
(propuesta en §6).

---

## 1. Punto 0 — Setup (`00_setup.ipynb`)

**Implementado y correcto:**
- Devcontainer (imagen `python:3.13-bookworm`), extensiones de VS Code, `remoteUser`.
- PyTorch instalado **desde el índice CPU** vía `postCreateCommand` (evita binarios CUDA).
- `.gitignore` excluye `data/`, modelos y artefactos; conserva `data/.gitkeep`.
- Celda de validación del entorno que reporta versiones de las librerías clave.

**🔴 Conflicto C1 (bloqueante) — `requirements.txt` inconsistente.**
La celda `%%writefile requirements.txt` del `00` **NO incluye `wbgapi` ni `pyarrow`**, pero el
`requirements.txt` versionado en disco **sí** los tiene. Es decir: *correr el notebook 00 sobrescribe
el archivo y elimina ambas dependencias*. Esto rompe todo lo que viene después:
- `wbgapi` → 01/02/03 no pueden descargar datos del Banco Mundial.
- `pyarrow` → no se puede leer/escribir Parquet. **Es la causa directa del `ImportError: pyarrow`
  que aborta la ejecución del notebook 03** (visible en sus outputs).

---

## 2. Punto 1 — EDA (`01_eda.ipynb`)

**Implementado:**
- Pull del World Bank vía `wbgapi`, una sola llamada, rango 2000–2022.
- Diccionario `INDICATORS` con los **códigos de indicadores documentados**.
- Persistencia del crudo en `data/raw/...parquet`.
- Análisis de faltantes, cobertura por año, univariado de targets, correlaciones de Spearman,
  bivariados y conclusiones con formulación del problema (regresión, estructura panel).

**🟠 Conflicto C2 (consistencia/bug) — el filtrado de agregados regionales no funciona.**
El código filtra `df[df['region'] != 'NA']`, pero `wbgapi` marca los agregados (WLD, EUU, LAC…) con
región **vacía (`''`)**, no con el string `'NA'`. Resultado: el filtro no elimina nada y el dataset
queda con **266 "países"** (incluye ~49 agregados). El propio output lo delata:
`Agregados (sin región): 0`. El `02` arrastra el mismo bug (aparece la columna one-hot `region_`
vacía). Solo el `03` lo parchea localmente → **inconsistencia entre notebooks**.

**🟡 Faltantes vs. requisitos del punto 1:**
- **Alcance del Gini sin aclarar.** `SI.POV.GINI` se etiqueta solo como "Coeficiente de Gini"; no se
  documenta que mide **únicamente desigualdad de ingresos** (no de salud, género ni acceso). Es uno
  de los tres problemas metodológicos centrales del proyecto.
- **Sin detección de outliers por dominio-rango.** No hay verificación de que cada indicador caiga en
  su rango plausible (p. ej. tasas ≥ 0, porcentajes en [0,100], Gini en [0,100]).
- **Estructura etaria fuera de lugar.** `pop_65plus_pct` / `pop_0to14_pct` se introducen recién en el
  `02`; el requisito los ubica como parte del EDA (punto 1).

---

## 3. Punto 2 — Preprocesamiento (`02_preprocesamiento.ipynb`)

**Implementado y correcto:**
- **Imputación jerárquica** exactamente como se pide: forward-fill por país → backward-fill por país
  → mediana por (región × ingreso) → mediana regional → mediana global.
- **`gini_imputed` se crea ANTES de imputar** (dentro del loop, antes del ffill). ✔
- Enriquecimiento con estructura etaria (`pop_65plus_pct`, `pop_0to14_pct`) y verificación por
  correlaciones.
- Transformaciones log (`log_gdp_per_capita`, `log_population`), one-hot de región, ordinal de ingreso,
  `year_norm`; estrategia de partición (GroupKFold por país) bien justificada.

**🟠 Conflicto C2 (heredado):** mismo bug de agregados que el `01` (filtro `!= 'NA'`).

**🔴 Conflicto C3 (consistencia) — falta la persistencia del dataset procesado.**
La descripción del notebook anuncia "8. Persistencia del dataset procesado", pero **no existe ninguna
celda que guarde** `data/processed/dataset_modelado.parquet`. Sin embargo el `03` intenta cargar ese
archivo. Resultado: el `03` **siempre** cae a la rama de reconstrucción (fallback), nunca usa la salida
real del `02`. El puente `02 → 03` está roto.

**🟡 Faltantes vs. requisitos del punto 2:**
- **Sin flag `is_pandemic` para 2020–2021.** No existe en `02` (ni en `03`).
- **Sin tratamiento de outliers por tipo.** No se tratan outliers (ni uniforme ni por tipo). El
  requisito pide tratamiento *diferenciado por tipo de variable* (no un IQR uniforme).

---

## 4. Punto 3 — Modelado (`03_modelado.ipynb`) — **donde estamos**

**Implementado y correcto:**
- Comparación **Ridge / Random Forest / XGBoost / LightGBM** con **GroupKFold por país**.
- Métricas RMSE/MAE/R², predicciones OOF, *permutation importance*, análisis de residuales,
  top-errores y un **estudio de ablación de `pop_0to14_pct`** (identidad demográfica).
- `reconstruir_dataset()` ya filtra agregados correctamente y elimina la columna `region_`.

**🔴 Conflicto C1 (heredado, bloqueante):** el notebook **aborta** en la celda de carga con
`ImportError: pyarrow` (consecuencia de C1). **El punto 3 hoy no es funcional.**

**🟠 Conflicto C4 (consistencia) — `reconstruir_dataset()` diverge del `02`.**
La reconstrucción de fallback **no crea `gini_imputed`** ni `is_pandemic`, mientras que el parquet del
`02` sí tendría `gini_imputed`. Según qué rama tome el `03`, el set de features cambia → resultados no
reproducibles.

**🟡 Faltantes vs. requisitos del punto 3:**
- **Sin flag de pandemia como feature** (consecuencia de C-pandemia en `02`).
- **Sin test de robustez de pandemia** (error con vs. sin 2020–2021).
- **Interpretación de R² a medias.** La sección de conclusiones es un esqueleto con marcadores
  `[completar…]`; falta la lectura cuidadosa (R² alto en natalidad inflado por identidad demográfica
  vs. techo real en mortalidad).

---

## 5. Punto 4 — Validación metodológica

**Estado: no existe en `main`.** En `origin/machine-learning` hay un `04` *distinto*
(`04_opt_eval_avanzada.ipynb`: GridSearchCV + SHAP + residuales). Problemas de ese notebook:
- **No es autocontenido:** usa `construir_modelos()`, `cv`, `X`, `y_death`, `groups`, `df`, `FEATURES`
  sin definirlos ni cargar datos → solo corre si antes se ejecutó el `03` en el mismo kernel.
- Usa **`shap`**, que **no está en `requirements.txt`**.
- No contiene lo que el punto 4 exige: **celda de verificación de integración del pipeline** ni el
  tratamiento explícito de los **tres problemas metodológicos** (alcance del Gini, *ground truth*,
  outliers por tipo).

---

## 6. Plan de resolución y propuesta para 4–6

### 6.1 Qué se resuelve YA (rama `fix/03-resolver-conflictos-modeling`, este checkpoint)
Objetivo: **punto 3 funcional y consistente con el pipeline del que depende.**
1. **C1** — `00`: incluir `wbgapi` + `pyarrow` en el `requirements.txt` generado (y alinear el de disco).
2. **C2** — `01` y `02`: filtrar agregados por región vacía/NaN (consistente con el `03`).
3. **C3** — `02`: añadir la celda de **persistencia** de `data/processed/dataset_modelado.parquet`.
4. **C4** — `03`: `reconstruir_dataset()` replica al `02` (crea `gini_imputed` e `is_pandemic`).
5. **Pandemia** — `02`: crear flag `is_pandemic` (2020–2021) como feature; `03` lo usa.
6. **Punto 3** — `03`: **test de robustez de pandemia** (con vs. sin 2020–2021) + **interpretación de R²**.

### 6.2 Ramas posteriores (tras tu visto bueno)
- `fix/01-02-eda-preproc` — completar puntos 1–2: aclarar **alcance del Gini**, **detección de
  outliers por dominio-rango** (01) y **tratamiento de outliers por tipo** (02).
- `feature/04-validacion-metodologica` — crear `04_validacion_metodologica.ipynb`:
  **celda de verificación de integración del pipeline** + los **tres problemas metodológicos**
  (alcance del Gini, *ground truth* de las tasas del Banco Mundial, outliers por tipo).

### 6.3 Propuesta para los puntos 5 y 6 (a confirmar)
No hay backlog versionado que los especifique (el `04` existente está rotulado "TASK-10"). Propongo:

- **Punto 5 → `05_optimizacion_avanzada.ipynb`:** *recuperar y sanear* el notebook que hoy vive en
  `origin/machine-learning` (`04_opt_eval_avanzada`): GridSearchCV con GroupKFold + **SHAP** +
  diagnóstico estadístico de residuales. Hacerlo **autocontenido** (cargar el parquet del `02`) y
  **añadir `shap` a `requirements.txt`**.
- **Punto 6 → `06_reporte_final.ipynb`:** consolidación ejecutiva: *model card* del modelo elegido,
  tabla final de métricas, limitaciones, checklist de reproducibilidad y conclusiones del proyecto.

> **Pendiente de tu confirmación:** (a) reasignar el `04` existente de optimización al **punto 5**;
> (b) que el **punto 6** sea el reporte final consolidado. Si tienes un backlog/definición distinta
> para 5 y 6, lo ajusto antes de implementar.

---

## 7. Resumen de severidades

| ID | Severidad | Dónde | Descripción | Estado |
|----|-----------|-------|-------------|--------|
| C1 | 🔴 Bloqueante | 00 → 03 | `requirements.txt` sin `wbgapi`/`pyarrow`; rompe Parquet | En esta rama |
| C2 | 🟠 Bug/consistencia | 01, 02 | Filtro de agregados no funciona (`!= 'NA'`) | En esta rama |
| C3 | 🔴 Consistencia | 02 → 03 | Falta persistencia del parquet procesado | En esta rama |
| C4 | 🟠 Consistencia | 03 | `reconstruir_dataset()` diverge del `02` (sin flags) | En esta rama |
| P-pand | 🟡 Requisito | 02, 03 | Falta flag `is_pandemic` + test de robustez | En esta rama |
| P-R² | 🟡 Requisito | 03 | Interpretación de R² incompleta | En esta rama |
| P-gini | 🟡 Requisito | 01 | Alcance del Gini sin aclarar | `fix/01-02` |
| P-out | 🟡 Requisito | 01, 02 | Outliers: detección por rango + tratamiento por tipo | `fix/01-02` |
| P-04 | 🟠 Identidad | 04 | `04` existente ≠ validación metodológica pedida | `feature/04` + §6.3 |

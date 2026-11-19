# Cafetería del Molino – Módulo de Minería de Datos (Avance Proyecto Integrador)

**Autor:** Alejandro Amaya Sosa · Universidad Santo Tomás – Tunja · Facultad de Ingeniería de Datos e IA
**Módulos:** (1) Minería de Datos: limpieza, EDA, fuentes externas, clustering y reglas de asociación · (2) Computación Paralela y Distribuida: multiprocessing/threading, speedup y eficiencia

> ⚠️ **Aviso de procedencia de datos.** `data/raw/ventas_raw.csv` es un dataset **sintético** que emula la base
> de datos de la cafetería (la exportación real no estaba disponible). El clima del run entregado también es
> simulado (la API estaba bloqueada). El reporte lo declara explícitamente. Para usar datos reales, ver abajo.

## Estructura
```
src/        config.py · generate_synthetic_data.py · external_sources.py · clean_data.py
            eda.py · clustering.py · rules_mining.py · mining_utils.py
data/raw/        datos crudos (ventas_raw.csv, catalogo_productos.csv)
data/external/   clima Tunja (Open-Meteo) y festivos de Colombia
data/processed/  sales_clean.csv · orders_clean.csv · daily_demand.csv · daily_clusters.csv
reports/         report/report.pdf (IEEE) · figures/ · tables/
docs/            data_dictionary.md · external_sources_log.txt
run_all.py       ejecuta todo el pipeline (semilla 42)
```

## Reproducir
```bash
pip install -r requirements.txt
python run_all.py                  # genera datos sintéticos si no hay crudo y corre todo
```

## Usar los datos REALES
1. Copia tu exportación a `data/raw/ventas_raw.csv` (una fila por línea de pedido) y tu menú a `data/raw/catalogo_productos.csv`
   (`producto, categoria, precio_base_feb2025, ratio_costo`).
2. Si tus columnas se llaman distinto, edita **solo** `COLUMN_MAP` en `src/config.py`.
3. Ejecuta `python run_all.py --skip-generate` (con Internet, para traer el clima real de Open-Meteo).
4. Revisa `reports/tables/*.json` y actualiza las cifras del reporte (`reports/report/report.tex`) si cambian.

## Qué hace cada paso
| Script | Salida principal |
|---|---|
| `external_sources.py` | Clima diario de Tunja (API Open-Meteo, con *fallback*) y festivos CO; bitácora en `docs/` |
| `clean_data.py` | Auditoría + limpieza registrada paso a paso (`cleaning_log.csv`), datasets limpios |
| `eda.py` | Estadísticos, pruebas (Kruskal–Wallis, Mann–Whitney, χ², Spearman) y figuras |
| `clustering.py` | K-Means/Ward de productos (k=4) y de días; silhouette, DB, CH, ARI, bootstrap |
| `rules_mining.py` | FP-Growth, reglas (soporte, confianza, lift…), sensibilidad y reglas por franja |

## Módulo de Computación Paralela y Distribuida (`src/parallel/`)
| Archivo | Función |
|---|---|
| `tasks.py` | 3 cargas: A) bootstrap de lift de reglas (CPU), B) map-reduce de coocurrencias (CPU), C) consultas API concurrentes (I/O, latencia simulada) |
| `benchmark.py` | Mide serial vs procesos vs hilos; speedup, eficiencia, Karp–Flatt, proyección de Amdahl; verifica que paralelo == serial |
| `make_report_assets.py` | Genera tabla y macros LaTeX para el documento IEEE (`reports/report_parallel/`) |

```bash
python src/parallel/benchmark.py            # ~6 min (usa --quick para prueba de 30 s)
cd reports/report_parallel && pdflatex report_parallel.tex && pdflatex report_parallel.tex
```
> ⚠️ Los números entregados se midieron en una máquina de **1 solo núcleo**: los procesos no pueden acelerar ahí.
> Corre el benchmark en tu computador multinúcleo y recompila: tabla, figura y texto se actualizan solos.

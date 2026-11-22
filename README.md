# Vehículos eléctricos y calidad del aire · Calidad y preparación de datos

Aprendiz: Juan Camilo Sarrazola Agudelo · Ficha 3169901 · SENA

| Actividad | Entregable |
|---|---|
| **GA1-220501094-01-AA1-EV05** – Diagnosticar la calidad de datos | `notebooks/data_quality.ipynb` y `reports/data_quality.md` |
| **GA1-220501094-01-AA1-EV07** – Preparación, limpieza y estructuración de datos con Pandas | `notebooks/data_preparation.ipynb` y `data/processed/` |

## Descripción y propósito

Proyecto de ciencia de datos que revisa con datos abiertos si el crecimiento de los vehículos eléctricos se
relaciona con una baja del material particulado fino (PM2.5). En la EV05 se hizo el **diagnóstico de calidad** de los
datasets. En la EV07 se **limpian, analizan y combinan** para obtener datos confiables y listos para las siguientes
etapas.

**Pregunta analítica:** ¿El aumento de vehículos eléctricos en las ciudades se relaciona con una baja del PM2.5 (µg/m³) entre 2021 y 2025?

## Estructura

```
ev-pm25-diagnostico-calidad/
├── README.md
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── raw/                          ← CSV originales sin modificar (no se suben a GitHub)
│   └── processed/                    ← datasets limpios y tablas estructuradas (EV07)
│
├── notebooks/
│   ├── data_quality.ipynb            ← diagnóstico de calidad (EV05)
│   └── data_preparation.ipynb        ← limpieza, análisis y estructuración (EV07)
│
└── reports/
    └── data_quality.md               ← informe técnico del diagnóstico (lo genera data_quality.ipynb)
```

## Datasets originales (`data/raw/`)

| DataFrame | Archivo en `data/raw/` | Fuente | Descarga |
|---|---|---|---|
| `ev_registro` | `Electric_Vehicle_Population_Data.csv` | [Data.gov · Electric Vehicle Population Data](https://catalog.data.gov/dataset/electric-vehicle-population-data) | [CSV](https://data.wa.gov/api/views/f6w7-q2d2/rows.csv?accessType=DOWNLOAD) |
| `ev_historial` | `Electric_Vehicle_Population_Size_History_By_County.csv` | [Data.gov · EV Population Size History By County](https://catalog.data.gov/dataset/electric-vehicle-population-size-history-by-county) | [CSV](https://data.wa.gov/api/views/3d5d-sdqb/rows.csv?accessType=DOWNLOAD) |
| `ev_ventas` | `electric-car-sales.csv` | [Our World in Data · Electric car sales (IEA)](https://ourworldindata.org/grapher/electric-car-sales) | [CSV](https://ourworldindata.org/grapher/electric-car-sales.csv?v=1&csvType=full&useColumnShortNames=false) |
| `pm25` | `API_EN.ATM.PM25.MC.M3_DS2_en_csv_v2_….csv` y `Metadata_Country_API_EN.ATM.PM25…csv` | [Banco Mundial · EN.ATM.PM25.MC.M3](https://data.worldbank.org/indicator/EN.ATM.PM25.MC.M3) | [ZIP](https://api.worldbank.org/v2/en/indicator/EN.ATM.PM25.MC.M3?downloadformat=csv) |

Los CSV originales **no se publican** porque el registro de vehículos pesa más de 70 MB y se actualiza con
frecuencia. Para reproducir el trabajo:

1. Descargue los cuatro enlaces de la columna *Descarga* y guarde los archivos en `data/raw/`.
2. El portal de Washington los guarda como `export.csv` y `export (1).csv`. Renómbrelos así: el que tiene la columna
   `VIN (1-10)` es `Electric_Vehicle_Population_Data.csv` y el que tiene la columna `Date` es
   `Electric_Vehicle_Population_Size_History_By_County.csv`.
3. Del ZIP del Banco Mundial se usan solo `API_EN.ATM.PM25…csv` y `Metadata_Country_API_EN.ATM.PM25…csv`.

## Transformaciones realizadas (EV07)

### 1. Limpieza y estandarización

Cada paso es una función reutilizable (`renombrar`, `quitar_duplicados`, `filtrar`, `estandarizar_texto`,
`codigo_a_texto`, `cero_a_nan`, `convertir_fecha`, `a_categoria`) encadenada con `pipe()`. Cada función anota lo que
hizo en una bitácora y cada pipeline termina con validaciones `assert`.

| Dataset | Transformaciones |
|---|---|
| `ev_registro` | Columnas a `snake_case` en inglés · duplicados verificados (`dol_vehicle_id` único) · excluidos 15 vehículos sin condado y los registros fuera de WA · `county`/`city` en tipo título (unifica *McCleary/Mccleary*) · `make`/`model` en mayúscula · códigos postales, distritos y tractos censales a texto con ceros a la izquierda · `electric_range = 0` → `NaN` (coincide 100 % con CAFV *"range not researched"*) · `vehicle_location` → `longitude`, `latitude` · `ev_type` → BEV/PHEV y CAFV abreviado como `category` |
| `ev_historial` | `Date` texto → `datetime64` · excluidas filas sin condado/estado y condados fuera de WA · clave (`date`, `county`, `primary_use`) única · columnas `year`, `month` · se verifica que BEV + PHEV = total eléctrico y que cada mes tenga los 39 condados |
| `ev_ventas` | Excluidos los agregados *World*, *Europe*, *EU (27)* y *Rest of World* (allí estaban las 4 ventas negativas) · `Electric cars sold` → `ev_sales` |
| `pm25` | Eliminadas la columna `Unnamed`, las columnas de indicador (valor constante) y los años vacíos · `merge` con los metadatos para conservar solo países · `melt()` a formato largo (`iso3`, `year`, `pm25`) · excluidos país-año sin medición |

Los 14 problemas medidos en el diagnóstico de la EV05 quedan en 0 (sección 4 del notebook).

### 2. Filtrado, agrupación e indicadores

Filtros con condiciones booleanas, `between()`, `isin()` y `query()`; resúmenes con `groupby().agg()`; tablas
dinámicas con `pivot_table()`; categorías con `pd.cut()` y `pd.qcut()`.

| # | Indicador | Resultado |
|---|---|---|
| I1 | % de eléctricos en el parque de pasajeros de Washington (dic. 2021 → dic. 2025) | 1,75 % → 5,38 % (+3,63 p. p.) |
| I2 | Crecimiento mediano de las ventas de eléctricos por país (2021 → 2023, 47 países) | +100 % |
| I3 | Variación mediana del PM2.5 por país (2021 → 2023, 200 países) | −0,26 µg/m³; bajó en el 64 % de los países |
| I4 | % de eléctricos de batería (BEV) entre modelos 2021 → 2025 en WA | 78,67 % → 79,50 % |
| I5 | Correlación entre crecimiento de ventas EV y cambio de PM2.5 (47 países) | Spearman 0,39: débil y positiva, sin evidencia de que más eléctricos impliquen más baja del PM2.5 |

### 3. Combinación y estructuración

| Unión | Clave | Validación |
|---|---|---|
| `ev_ventas` + `pm25` → panel país-año | `iso3`, `year` | `validate="one_to_one"`, conteo con `indicator=True`, suma de ventas conservada |
| `ev_registro` (agregado) + `ev_historial` (último mes) → perfil por condado | `county` | 39 de 39 condados con pareja; diferencia total entre fuentes 2,7 % |
| `ev_historial` (WA anual) + `pm25` (EE. UU.) → serie de contexto | `year` | `left join`; 2024–2025 quedan sin PM2.5 nacional |

No se combina `pm25` con los condados: es un dato nacional y repetirlo en los 39 condados sería una combinación
artificial. El paso siguiente es incorporar PM2.5 por condado (EPA AirData).

## Datasets procesados (`data/processed/`)

| Archivo | Contenido | Filas |
|---|---|---|
| `ev_registry_clean.csv.gz` | Registro de eléctricos de WA limpio (comprimido; se lee con `pd.read_csv`) | 299.353 |
| `ev_history_wa_clean.csv` | Historial mensual por condado de WA, 2017-01 a 2026-09 | 9.126 |
| `ev_sales_clean.csv` | Ventas anuales de eléctricos, solo países | 759 |
| `pm25_clean.csv` | PM2.5 anual por país en formato largo | 6.800 |
| `country_year_ev_pm25.csv` | **Panel país-año** con ventas, ventas acumuladas, PM2.5 y variaciones | 641 |
| `wa_county_ev_profile.csv` | **Perfil de los 39 condados** de WA (registro + historial) | 39 |
| `wa_year_ev_us_pm25.csv` | **Serie anual de WA** con el PM2.5 de EE. UU. | 9 |
| `indicators.csv` | Indicadores I1–I5 | 5 |

## Cómo ejecutar

```bash
git clone https://github.com/juan-camilo-sarrazola-agudelo/ev-pm25-diagnostico-calidad.git
cd ev-pm25-diagnostico-calidad
python -m venv .venv
source .venv/Scripts/activate        # PowerShell: .\.venv\Scripts\Activate.ps1 · macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
# copie los CSV en data/raw/ (ver Datasets originales)
jupyter nbconvert --to notebook --execute --inplace notebooks/data_quality.ipynb       # EV05: diagnóstico e informe
jupyter nbconvert --to notebook --execute --inplace notebooks/data_preparation.ipynb   # EV07: genera data/processed/
```

También puede abrir los notebooks con `jupyter lab` y usar **Run All**. `data_preparation.ipynb` se detiene con un
`AssertionError` si alguna validación no se cumple.

## Ramas y versiones

| Rama | Uso |
|---|---|
| `main` | Solo la versión entregada |
| `develop` | Integración de todo el trabajo (cada rama entra por Pull Request) |
| `feat/entorno` | Entorno virtual y `requirements.txt` |
| `feat/datos` | Datasets en `data/raw/` y su documentación |
| `feat/notebook-diagnostico` | Notebook de diagnóstico |
| `docs/informe-readme` | Informe técnico y README |
| `release/v1.0` | Paso de `develop` a `main` (EV05) |
| `feature/project-data-preparation` | Limpieza, análisis y estructuración de datos (EV07) |

| Tag | Contenido |
|---|---|
| `v0.1` | Estructura y `.gitignore` |
| `v0.2` | Entorno y dependencias |
| `v0.3` | Datos documentados |
| `v0.4` | Notebook de diagnóstico ejecutado |
| `v0.5` | Informe técnico y README |
| `v1.0` | Entrega EV05 en `main` |

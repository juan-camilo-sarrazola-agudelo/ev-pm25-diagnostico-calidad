# Vehículos eléctricos y calidad del aire · Diagnóstico de calidad de datos


Actividad **GA1-220501094-01-AA1-EV05 – Diagnosticar la calidad de datos: preparación, exploración y documentación**

Aprendiz: Juan Camilo Sarrazola Agudelo · Ficha 3169901 · SENA

## Descripción y propósito

Proyecto de ciencia de datos que revisa con datos abiertos si el crecimiento de los vehículos eléctricos se
relaciona con una baja del material particulado fino (PM2.5). Esta etapa hace el **diagnóstico inicial de calidad**
de los datasets con Python y Pandas: estructura, tipos de datos, valores faltantes, inconsistencias y acciones de
limpieza propuestas. Los datos todavía no se limpian.

**Pregunta analítica:** ¿El aumento de vehículos eléctricos en las ciudades se relaciona con una baja del PM2.5 (µg/m³) entre 2021 y 2025?

## Estructura

```
ev-pm25-diagnostico-calidad/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                        ← CSV del proyecto (no se suben a GitHub)
├── notebooks/
│   └── diagnostico_calidad.ipynb   ← carga, inspección y diagnóstico
└── reports/
    └── informe_diagnostico.md      ← informe técnico (lo genera el notebook)
```

## Datasets

| DataFrame | Archivo en `data/raw/` | Fuente | Descarga |
|---|---|---|---|
| `ev_registro` | `Electric_Vehicle_Population_Data.csv` | [Data.gov · Electric Vehicle Population Data](https://catalog.data.gov/dataset/electric-vehicle-population-data) | [CSV](https://data.wa.gov/api/views/f6w7-q2d2/rows.csv?accessType=DOWNLOAD) |
| `ev_historial` | `Electric_Vehicle_Population_Size_History_By_County.csv` | [Data.gov · EV Population Size History By County](https://catalog.data.gov/dataset/electric-vehicle-population-size-history-by-county) | [CSV](https://data.wa.gov/api/views/3d5d-sdqb/rows.csv?accessType=DOWNLOAD) |
| `ev_ventas` | `electric-car-sales.csv` | [Our World in Data · Electric car sales (IEA)](https://ourworldindata.org/grapher/electric-car-sales) | [CSV](https://ourworldindata.org/grapher/electric-car-sales.csv?v=1&csvType=full&useColumnShortNames=false) |
| `pm25` | `API_EN.ATM.PM25.MC.M3_DS2_en_csv_v2_….csv` y `Metadata_Country_API_EN.ATM.PM25…csv` | [Banco Mundial · EN.ATM.PM25.MC.M3](https://data.worldbank.org/indicator/EN.ATM.PM25.MC.M3) | [ZIP](https://api.worldbank.org/v2/en/indicator/EN.ATM.PM25.MC.M3?downloadformat=csv) |

Los CSV **no se publican** en el repositorio porque el registro de vehículos pesa más de 80 MB y se actualiza con
frecuencia. Para reproducir el trabajo:

1. Descargue los cuatro enlaces de la columna *Descarga* y guarde los archivos en `data/raw/`.
2. El portal de Washington los guarda como `export.csv` y `export (1).csv`. Renómbrelos así: el que tiene la columna
   `VIN (1-10)` es `Electric_Vehicle_Population_Data.csv` y el que tiene la columna `Date` es
   `Electric_Vehicle_Population_Size_History_By_County.csv`.
3. Del ZIP del Banco Mundial se usan solo `API_EN.ATM.PM25…csv` y `Metadata_Country_API_EN.ATM.PM25…csv`.

## Cómo reproducir el diagnóstico

```bash
git clone https://github.com/juan-camilo-sarrazola-agudelo/ev-pm25-diagnostico-calidad.git
cd ev-pm25-diagnostico-calidad
python -m venv .venv
source .venv/Scripts/activate        # PowerShell: .\.venv\Scripts\Activate.ps1 · macOS/Linux: source .venv/bin/activate
python -m pip install -r requirements.txt
# copie los CSV en data/raw/ (ver Datasets)
jupyter nbconvert --to notebook --execute --inplace notebooks/diagnostico_calidad.ipynb
```

El último comando ejecuta el notebook y escribe `reports/informe_diagnostico.md` con las cifras obtenidas.
También puede abrirlo con `jupyter lab` y usar **Run All**.

## Ramas y versiones

| Rama | Uso |
|---|---|
| `main` | Solo la versión entregada |
| `develop` | Integración de todo el trabajo (cada rama entra por Pull Request) |
| `feat/entorno` | Entorno virtual y `requirements.txt` |
| `feat/datos` | Datasets en `data/raw/` y su documentación |
| `feat/notebook-diagnostico` | Notebook de diagnóstico |
| `docs/informe-readme` | Informe técnico y README |
| `release/v1.0` | Paso de `develop` a `main` |

| Tag | Contenido |
|---|---|
| `v0.1` | Estructura y `.gitignore` |
| `v0.2` | Entorno y dependencias |
| `v0.3` | Datos documentados |
| `v0.4` | Notebook de diagnóstico ejecutado |
| `v0.5` | Informe técnico y README |
| `v1.0` | Entrega EV05 en `main` |




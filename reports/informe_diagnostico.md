# Informe técnico · Diagnóstico inicial de calidad de datos

**Actividad:** GA1-220501094-01-AA1-EV05
**Aprendiz:** Juan Camilo Sarrazola Agudelo · Ficha 3169901 · SENA
**Fecha de ejecución del diagnóstico:** 2026-11-04
**Generado por:** `notebooks/diagnostico_calidad.ipynb` (todas las cifras salen de Pandas)

## 1. Proyecto y pregunta analítica
**Proyecto:** Vehículos eléctricos y calidad del aire: ¿el aumento de carros eléctricos ayuda a reducir el PM2.5 en las ciudades?

**Pregunta analítica:** ¿El aumento de vehículos eléctricos en las ciudades se relaciona con una baja del PM2.5 (µg/m³) entre 2021 y 2025?

## 2. Datasets utilizados
| DataFrame | Archivo | Fuente | Cada fila es |
|---|---|---|---|
| ev_registro | Electric_Vehicle_Population_Data.csv | Data.gov · Departamento de Licencias de Washington | un vehículo eléctrico registrado actualmente en el Departamento de Licencias de Washington |
| ev_historial | Electric_Vehicle_Population_Size_History_By_County.csv | Data.gov · Departamento de Licencias de Washington | un mes, un condado y un uso del vehículo (pasajeros o camión) |
| ev_ventas | electric-car-sales.csv | Our World in Data (IEA) | un país o región en un año |
| pm25 | API_EN.ATM.PM25.MC.M3_DS2_en_csv_v2_648.csv | Banco Mundial (EN.ATM.PM25.MC.M3) | un país o agregado regional; cada año es una columna (formato ancho) |

## 3. Resultados de la inspección estructural
Se aplicaron `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')` a cada DataFrame.

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| ev_registro | 300.142 | 16 | str: 10, float64: 4, int64: 2 | 8 columnas con faltantes; 0 filas duplicadas. Foto actual del parque (no es una serie de tiempo). |
| ev_historial | 36.913 | 10 | int64: 5, str: 4, float64: 1 | 2 columnas con faltantes; 0 filas duplicadas. Serie mensual; la fecha viene como texto. |
| ev_ventas | 823 | 4 | str: 2, int64: 2 | 1 columnas con faltantes; 0 filas duplicadas. Mezcla países y agregados (mundo, regiones). |
| pm25 | 266 | 71 | float64: 67, str: 4 | 67 columnas con faltantes; 0 filas duplicadas. Formato ancho: un año por columna; incluye agregados regionales. |

## 4. Valores faltantes, tipos de datos e inconsistencias
### 4.1 Valores faltantes (sin contar las columnas de año de pm25)
| Dataset | columna | faltantes | % faltantes |
|---|---|---|---|
| ev_registro | County | 15 | 0,005 % |
| ev_registro | City | 15 | 0,005 % |
| ev_registro | Postal Code | 15 | 0,005 % |
| ev_registro | Electric Range | 35 | 0,012 % |
| ev_registro | Legislative District | 790 | 0,263 % |
| ev_registro | Vehicle Location | 24 | 0,008 % |
| ev_registro | Electric Utility | 15 | 0,005 % |
| ev_registro | 2020 GEOID | 15 | 0,005 % |
| ev_historial | County | 117 | 0,317 % |
| ev_historial | State | 117 | 0,317 % |
| ev_ventas | Code | 16 | 1,944 % |
| pm25 | Unnamed: 70 | 266 | 100,000 % |

En `pm25` hay 66 columnas de año; 32 están vacías y solo hay datos entre 1990 y 2023.

### 4.2 Tipos de datos
| Dataset | Columna | Tipo actual | Tipo esperado | ¿Coherente? | Motivo |
|---|---|---|---|---|---|
| ev_registro | Postal Code | float64 | texto (código) | no | Es un código: no se suma ni promedia y un número pierde ceros a la izquierda. |
| ev_registro | Legislative District | float64 | texto o entero (categoría) | no | Un distrito no puede ser 12.0; queda decimal porque hay faltantes. |
| ev_registro | 2020 GEOID | float64 | texto (código de 11 dígitos) | no | Código geográfico; como número se muestra en notación científica. |
| ev_registro | Model Year | int64 | entero | sí | Año del modelo. |
| ev_registro | Vehicle Location | str | dos números (longitud, latitud) | no | Viene como texto 'POINT (lon lat)'. |
| ev_historial | Date | str | fecha (datetime64) | no | Sin convertir no se puede ordenar ni agrupar por año. |
| ev_historial | Electric Vehicle (EV) Total | int64 | entero | sí | Conteo de vehículos; si es texto, trae separador de miles. |
| ev_historial | Total Vehicles | int64 | entero | sí | Conteo de vehículos; si es texto, trae separador de miles. |
| ev_historial | Percent Electric Vehicles | float64 | decimal | sí | Porcentaje. |
| ev_ventas | Year | int64 | entero | sí | Año. |
| pm25 | 2023 | float64 | decimal en filas, no en columnas | no | Los años son nombres de columna (texto); para analizar por año hay que pasar a formato largo. |

### 4.3 Comprobaciones de consistencia
**ev_registro**

| comprobación | resultado |
|---|---|
| filas duplicadas | 0 |
| DOL Vehicle ID repetido | 0 |
| Electric Range = 0 | 197.085 |
| % con Electric Range = 0 | 65.66 |
| State distinto de WA | 789 |
| Model Year mínimo | 1999 |
| Model Year máximo | 2027 |
| Model Year > 2027 | 0 |

**ev_historial**

| comprobación | resultado |
|---|---|
| fechas no convertibles | 0 |
| fecha mínima | 2017-01-31 |
| fecha máxima | 2026-09-30 |
| meses distintos | 117 |
| claves repetidas (fecha, condado, estado, uso) | 0 |
| BEV + PHEV ≠ total eléctrico | 0 |
| eléctricos + no eléctricos ≠ total | 0 |
| % eléctrico no coincide al recalcular (>0,01) | 0 |
| filas con estado distinto de WA | 27.787 |
| filas con Total Vehicles = 0 | 0 |

**ev_ventas**

| comprobación | resultado |
|---|---|
| columna de valor | Electric cars sold |
| entidades distintas | 64 |
| entidades que son agregados (sin código ISO) | 4 |
| año mínimo | 2010 |
| año máximo | 2025 |
| valores negativos | 4 |
| valores iguales a 0 | 0 |
| pares (Entity, Year) repetidos | 0 |

**pm25**

| comprobación | resultado |
|---|---|
| columnas sin nombre (Unnamed) | 1 |
| columnas de año vacías | 32 |
| primer año con datos | 1990 |
| último año con datos | 2023 |
| filas que son agregados (no países) | 49 |
| filas sin ningún valor de PM2.5 | 18 |
| valores negativos | 0 |
| valor máximo (µg/m³) | 124.66 |
| códigos de país repetidos | 0 |

## 5. Principales hallazgos de calidad por dataset
- **ev_registro:** Vehículos sin ubicación; Faltantes en Electric Range; Faltantes en Legislative District; Faltantes en Vehicle Location; Autonomía en 0 usada como 'sin dato'; Vehículos registrados fuera de Washington; Códigos guardados como números; Ubicación como texto; Es una foto actual, no un historial.
- **ev_historial:** Fecha guardada como texto; Filas sin condado ni estado; Condados de otros estados; Dos tipos de uso en el mismo archivo.
- **ev_ventas:** Mezcla de países y agregados; Ventas negativas; Faltantes en Code; Nombre de columna largo.
- **pm25:** Formato ancho (un año por columna); Columnas de año vacías; Columna sin nombre; Mezcla de países y agregados; Periodo incompleto; Escala nacional.

## 6. Acciones de limpieza y preparación propuestas
| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| ev_registro | Vehículos sin ubicación | 15 filas con County, City, Postal Code, Electric Utility, 2020 GEOID vacíos a la vez; `State`: AE (6), BC (5), ON (2), NH (1) | No se pueden asignar a ningún condado de Washington. | Excluirlos del análisis por condado (son registros con dirección fuera de EE. UU. o militar). |
| ev_registro | Faltantes en `Electric Range` | `isna().sum()` = 35 (0,012 %); años de modelo: 2027 (24), 2025 (5), 2024 (3) | Sin autonomía para esos modelos. | Dejar como NaN; no imputar. |
| ev_registro | Faltantes en `Legislative District` | `isna().sum()` = 790 (0,263 %) | No se pueden agrupar por distrito. | No afecta el análisis por condado; dejar como NaN. |
| ev_registro | Faltantes en `Vehicle Location` | `isna().sum()` = 24 (0,008 %) | No se pueden ubicar en un mapa. | Usar `County` para el análisis; dejar como NaN. |
| ev_registro | Autonomía en 0 usada como 'sin dato' | `Electric Range == 0`: 197.085 filas (65,66 %) | El promedio de autonomía queda subestimado. | Reemplazar 0 por NaN antes de calcular estadísticas. |
| ev_registro | Vehículos registrados fuera de Washington | `State != 'WA'`: 789 filas | Inflan el conteo si se analiza solo Washington. | Filtrar `State == 'WA'`. |
| ev_registro | Códigos guardados como números | `dtypes`: Postal Code = float64, Legislative District = float64, 2020 GEOID = float64 | Se pierden ceros a la izquierda y aparecen decimales (12.0) en códigos. | Convertir a texto con `astype('Int64').astype(str)` o leer con `dtype=str`. |
| ev_registro | Ubicación como texto | `Vehicle Location` contiene 'POINT (lon lat)' | No se puede usar directamente en mapas. | Extraer longitud y latitud con `str.extract()`. |
| ev_registro | Es una foto actual, no un historial | No tiene columna de fecha de registro; `Model Year` es el año del modelo | No sirve para medir el crecimiento año a año. | Usar `ev_historial` para la evolución temporal. |
| ev_historial | Fecha guardada como texto | `dtype` de `Date` = str | No se puede ordenar ni agrupar por año o mes. | `pd.to_datetime(ev_historial['Date'])` y crear columna `anio`. |
| ev_historial | Filas sin condado ni estado | `County`: 117 (0,317 %); `State`: 117 (0,317 %); 117 filas tienen ambos vacíos | Vehículos que no se pueden asignar a un condado. | Excluirlas de los totales por condado. |
| ev_historial | Condados de otros estados | `State != 'WA'`: 27.787 filas | Solo 24,72 % de las filas son de Washington; el resto son condados donde viven dueños de vehículos registrados en WA. | Filtrar `State == 'WA'`. |
| ev_historial | Dos tipos de uso en el mismo archivo | `Vehicle Primary Use`: Passenger, Truck | Sumar todo mezcla pasajeros con camiones. | Analizar `Passenger` por separado o sumar de forma explícita. |
| ev_ventas | Mezcla de países y agregados | 4 entidades sin código ISO de país | Promedios o sumas por país contarían dos veces. | Filtrar filas con código ISO de 3 letras (excluir vacíos y `OWID_`). |
| ev_ventas | Ventas negativas | Rest of World 2013: -4.126; Rest of World 2015: -2.059; Rest of World 2019: -49.752; Rest of World 2022: -10.334 | Una venta no puede ser negativa; es un residuo de calcular 'resto del mundo' por diferencia. | Excluir el agregado 'Rest of World' o marcar esos valores como NaN. |
| ev_ventas | Faltantes en `Code` | `isna().sum()` = 16 (1,94 %) | Filas sin identificar como país. | Tratarlas como agregados regionales. |
| ev_ventas | Nombre de columna largo | Columna de valor: 'Electric cars sold' | Código más difícil de leer. | Renombrar a `ventas_ev`. |
| pm25 | Formato ancho (un año por columna) | 66 columnas de año | No se puede unir por año con los otros datasets. | Pasar a formato largo con `melt()` (país, año, pm25). |
| pm25 | Columnas de año vacías | 32 años sin ningún dato | Ruido en la tabla. | Eliminar esas columnas. |
| pm25 | Columna sin nombre | 1 columna(s) `Unnamed` | Columna vacía creada por la coma final del CSV. | Eliminarla. |
| pm25 | Mezcla de países y agregados | 49 filas no son países (según `Metadata_Country`) | Doble conteo en comparaciones. | Unir con los metadatos y conservar solo filas con `Region`. |
| pm25 | Periodo incompleto | Último año con dato: 2023 | No cubre todo 2021–2025. | Complementar con estaciones (EPA AirData u OpenAQ) para Washington. |
| pm25 | Escala nacional | Un valor por país, no por condado | No se puede relacionar directamente con los condados de Washington. | Usar PM2.5 por condado (EPA AirData) para el análisis local; dejar este dataset como contexto. |

## 7. Conclusiones
1. Los cuatro datasets cargan sin errores y tienen estructura tabular clara, pero **ninguno está listo para el análisis tal como viene**:
   hay tipos incoherentes (fechas y códigos), ceros que significan "sin dato" y agregados mezclados con países.
   Los problemas de mayor peso son: 65,66 % de autonomías registradas en 0 en `ev_registro`,
   75,28 % de filas de `ev_historial` que no son de Washington, los conteos del historial leídos como texto
   y 4 ventas negativas en `ev_ventas`.
2. La limitación más importante no es de limpieza sino de **cobertura**: los vehículos eléctricos están por condado de Washington
   y el PM2.5 por país, y al PM2.5 le faltan 2 año(s) del periodo 2021–2025 (2024, 2025).
3. Para responder la pregunta a nivel local se propone agregar PM2.5 por condado (EPA AirData u OpenAQ) y datos de clima (NOAA).
   `ev_ventas` y `pm25` quedan como contexto internacional.
4. Con las acciones propuestas (conversión de tipos, filtro a Washington, exclusión de agregados, formato largo para pm25)
   los datos quedan en condiciones de pasar a la fase de limpieza y unión.

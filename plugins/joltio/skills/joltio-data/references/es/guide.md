# Joltio Data para agentes

API: https://api.joltio.app/data/ · MCP: https://api.joltio.app/mcp

Con MCP, llama a `data_status()` para conocer tu plan y sus ventanas de fechas, y después a `data_guide(topic='...')` para más detalles. Desde Python, empieza por el tema `python` más abajo. Esta guía estática describe el plan público; la guía MCP filtra por tu plan.

# Joltio Data

Empieza con `data_status()` para conocer tu plan, las tablas visibles, las ventanas históricas y los límites de consulta. Usa `data_guide(topic='tables')` para ver ejemplos de las tablas de tu plan, `data_search()` o `data_describe()` para consultar columnas y `data_query()` para ejecutar SQL o GraphQL de solo lectura. Usa `data_recipes_list()` para encontrar análisis revisados; `data_recipes_run()` solo está disponible si tu plan lo muestra. Comprueba la frescura antes de citar un resultado.

ESIOS cubre indicadores eléctricos y programación I90; OMIE cubre subastas y negociación continua; MIBGAS cubre los mercados de gas. Las tablas concretas disponibles figuran en `data_status()` y `data_guide(topic='tables')`.

# Python en VS Code

Instala `pip install joltio python-dotenv ipykernel` en el entorno de Python elegido como kernel de VS Code. Escribe `JOLTIO_API_KEY=...` en el `.env` de la carpeta de trabajo. Nunca muestres la clave, la copies al código ni la subas al repositorio. Carga ese fichero antes de crear el cliente:

```python
from dotenv import find_dotenv, load_dotenv
from joltio import Client

load_dotenv(find_dotenv(usecwd=True))
client = Client()
```

`client.data.query(sql)` devuelve un DataFrame de Polars por defecto. `client.get("/data/describe")` lista las tablas de tu plan y `client.get("/data/describe", params={"domain": "archives_i90"})` devuelve las columnas, comentarios y cobertura de una tabla; toma el nombre del dominio de esa lista. `client.data.search("texto")` busca indicadores, unidades, empresas y tecnologías por nombre. `client.data.metadata()` solo cubre la tabla del I90 (`esios.archives_i90`): sus columnas y programas. Son métodos del cliente `joltio` publicado, no herramientas MCP. En Python, usa `client.get("/data/tier-info")` en lugar de `data_status()` de MCP para consultar `restrictions.data_windows` y los límites, y `/data/describe` y `client.data.search(texto)` en lugar de `data_describe()` y `data_search()` de MCP. Lee este `llms.txt` como guía estática en lugar de llamar a `data_guide()` de MCP; no existe `client.data.guide()`. La guía REST autenticada en `/data/guide` ofrece los temas actualizados para tu plan si los necesitas.

Ejemplo breve de OMIE que funciona en el plan público: una agregación con filtro de fechas reciente.

```python
df = client.data.query("SELECT toDate(datetime) AS day, round(avg(price), 2) AS eur_mwh FROM omie.prices WHERE country = 'ES' AND program = 'PDBF' AND datetime >= today() - 7 AND datetime < today() GROUP BY day ORDER BY day")
print(df)
```

El I90 se publica después de la fecha de entrega. Ancla la agregación a la última fecha de I90 permitida por tu plan en vez de dar por supuesto que corresponde a hoy:

```python
window = client.get("/data/tier-info")["restrictions"]["data_windows"]["esios.archives_i90"]
day = window["latest_data"]
df = client.data.query(f"SELECT toStartOfHour(datetime) AS hour, count() AS rows FROM esios.archives_i90 WHERE program = 'PDBF' AND datetime >= toDateTime('{day} 00:00:00', 'Europe/Madrid') AND datetime < toDateTime('{day} 01:00:00', 'Europe/Madrid') GROUP BY hour")
print(df)
```

Antes de otras consultas, comprueba la tabla, las columnas y la ventana de fechas de tu plan; no inventes nombres. Si aparece `table_not_allowed`, elige una tabla de tu plan. Para `history_no_date_filter` o `history_restricted`, usa `your_window_start` y `latest_data` de `/data/tier-info`. Para `query_too_expensive`, acota las fechas, agrega y selecciona menos columnas. Las respuestas de errores de restricción proporcionan `fix_text` y a menudo `fix_sql_example`; aplica esas indicaciones antes de reintentar. `QueryError` del cliente Python abrevia el cuerpo de la respuesta: consulta el error completo de la API si la indicación aparece cortada. Ante HTTP 429, respeta `Retry-After` (`RateLimitError.retry_after`) y reduce la frecuencia de peticiones. En una red corporativa con inspección HTTPS, fija `SSL_CERT_FILE` y `REQUESTS_CA_BUNDLE` en `.env` al certificado raíz de confianza en formato PEM, como muestra `.env.example`.

# Tablas disponibles en tu plan

Usa `data_status()` para conocer las ventanas de fechas y `data_describe()` para consultar el esquema actual. Llama a `data_guide(topic='tables')` por MCP para ver ejemplos basados en el esquema y la ventana vigentes.

## archives_liquicomun.coef_k

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.compodem

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.components

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.components_ssaa

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.petar

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.settlements

Tabla de ClickHouse de solo lectura.

## archives_liquicomun.timeseries

Tabla de ClickHouse de solo lectura.

## eia.crude_prices

Tabla de ClickHouse de solo lectura.

## eia.electricity_retail_sales

Tabla de ClickHouse de solo lectura.

## eia.electricity_retail_sales_detail

Tabla de ClickHouse de solo lectura.

## eia.international_annual

Tabla de ClickHouse de solo lectura.

## eia.lng_exports

Tabla de ClickHouse de solo lectura.

## eia.lng_exports_by_destination

Tabla de ClickHouse de solo lectura.

## eia.natural_gas_imports

Tabla de ClickHouse de solo lectura.

## eia.natural_gas_prices_daily

Tabla de ClickHouse de solo lectura.

## eia.natural_gas_prices_monthly

Tabla de ClickHouse de solo lectura.

## eia.rto_demand

Tabla de ClickHouse de solo lectura.

## eia.rto_generation_by_fuel

Tabla de ClickHouse de solo lectura.

## eia.rto_interchange

Tabla de ClickHouse de solo lectura.

## eia.steo_forecasts

Tabla de ClickHouse de solo lectura.

## eia.total_energy

Tabla de ClickHouse de solo lectura.

## entsoe.cross_border

Tabla de ClickHouse de solo lectura.

## entsoe.cross_border_latest

Tabla de ClickHouse de solo lectura.

## entsoe.dim_area

Tabla de ClickHouse de solo lectura.

## entsoe.dim_market

Tabla de ClickHouse de solo lectura.

## entsoe.dim_process

Tabla de ClickHouse de solo lectura.

## entsoe.dim_psr_type

Tabla de ClickHouse de solo lectura.

## entsoe.generation

Tabla de ClickHouse de solo lectura.

## entsoe.generation_latest

Tabla de ClickHouse de solo lectura.

## entsoe.generation_mix_hourly

Tabla de ClickHouse de solo lectura.

## entsoe.installed_capacity

Tabla de ClickHouse de solo lectura.

## entsoe.installed_capacity_latest

Tabla de ClickHouse de solo lectura.

## entsoe.interconnection_hourly

Tabla de ClickHouse de solo lectura.

## entsoe.known_gaps

Tabla de ClickHouse de solo lectura.

## entsoe.load

Tabla de ClickHouse de solo lectura.

## entsoe.load_forecast_error_hourly

Tabla de ClickHouse de solo lectura.

## entsoe.load_latest

Tabla de ClickHouse de solo lectura.

## entsoe.price_spread_hourly

Tabla de ClickHouse de solo lectura.

## entsoe.prices

Tabla de ClickHouse de solo lectura.

## entsoe.prices_daily

Tabla de ClickHouse de solo lectura.

## entsoe.prices_hourly

Tabla de ClickHouse de solo lectura.

## entsoe.prices_latest

Tabla de ClickHouse de solo lectura.

## esios.archives_i3

Tabla de ClickHouse de solo lectura.

## esios.archives_i90

Tabla de ClickHouse de solo lectura.

## esios.archives_i90_offers

Tabla de ClickHouse de solo lectura.

## esios.archives_i90_qh

Tabla de ClickHouse de solo lectura.

## esios.indicator_vintages_gates

Tabla de ClickHouse de solo lectura.

## esios.indicator_vintages_raw

Tabla de ClickHouse de solo lectura.

## esios.indicators

Tabla de ClickHouse de solo lectura.

## mibgas.auctions

Tabla de ClickHouse de solo lectura.

## mibgas.imbalances_pvb_vtp

Tabla de ClickHouse de solo lectura.

## mibgas.imbalances_tvb_avb

Tabla de ClickHouse de solo lectura.

## mibgas.indexes

Tabla de ClickHouse de solo lectura.

## mibgas.prices

Tabla de ClickHouse de solo lectura.

## mibgas.regulated_gas

Tabla de ClickHouse de solo lectura.

## mibgas.trading_data

Tabla de ClickHouse de solo lectura.

## omie.continuous_orders

Tabla de ClickHouse de solo lectura.

## omie.continuous_prices

Tabla de ClickHouse de solo lectura.

## omie.continuous_schedule

Tabla de ClickHouse de solo lectura.

## omie.continuous_trades

Tabla de ClickHouse de solo lectura.

## omie.continuous_volume

Tabla de ClickHouse de solo lectura.

## omie.interconnection_capacities

Tabla de ClickHouse de solo lectura.

## omie.offer_complex

Tabla de ClickHouse de solo lectura.

## omie.offers

Tabla de ClickHouse de solo lectura.

## omie.prices

Tabla de ClickHouse de solo lectura.

## omie.unavailabilities

Tabla de ClickHouse de solo lectura.

# Dialectos de consulta

Usa SQL de ClickHouse para agregaciones y series temporales. Usa GraphQL para navegar entre unidades, empresas, instalaciones y nodos. En SQL, nombra solo las tablas y columnas que indiquen `data_status()`, `data_guide(topic='tables')` y `data_describe()`; no consultes catálogos del sistema ni uses `SELECT *`.

Los indicadores de ESIOS incluyen varias geografías. Filtra por `geo_id` antes de comparar series: los precios suelen usar 3 (España), la generación en tiempo real usa 8741 (Península) y la generación medida se publica por provincia, así que suma las provincias de las mismas horas. Comprueba las geografías, la frecuencia y la magnitud de cada indicador con `data_search()` antes de hacer un JOIN. Las tablas de ClickHouse siguen la política de deduplicación del servidor; no añadas ajustes a un perfil de producción de solo lectura.

# Tiempo y unidades

Las marcas de tiempo se almacenan en UTC y señalan el inicio de cada periodo. Los días del mercado español siguen Europe/Madrid: agrupa con `toDate(datetime, 'Europe/Madrid')`, nunca por la fecha UTC, y filtra con la ventana de fechas de `data_status()`. Los días de cambio de hora tienen 23 o 25 horas (92 o 100 cuartos de hora), así que cuenta los periodos en vez de suponer 24 o 96. Los mercados españoles pasaron de periodos horarios a cuartohorarios el 2025-10-01; cuando una tabla tiene `source_interval`, indica cuál usa cada fila. Suma la energía a través de ese cambio, pero no compares a través de él el número de filas ni las medias por periodo. Indica la zona horaria en la respuesta.

Los precios suelen expresarse en EUR/MWh (EUR/MW en las bandas de capacidad); la energía, en MWh; y la potencia y la capacidad, en MW. Una potencia solo es energía después de multiplicarla por la duración del intervalo en horas: 0,25 en un cuarto de hora y 1/12 en una lectura de cinco minutos. Toma cada unidad de los comentarios de columna de `data_describe()` y, en los indicadores, de la magnitud (energía o potencia) y la frecuencia que devuelve `data_search()`; nunca tomes una unidad, un código de unidad ni una geografía de un ejemplo.

ESIOS publica la generación dos veces. Las series en tiempo real, como 1295 (solar fotovoltaica) y 551 (eólica), son potencia peninsular en MW cada cinco minutos (`geo_id` 8741) y se publican al momento; 1295 incluye el autoconsumo fotovoltaico. Las series medidas, como 1161 (solar fotovoltaica) y 10037 (eólica), son energía horaria en MWh por provincia y se publican a partir del mes siguiente. Si la pregunta pide energía medida o generada, usa la serie medida en su último día completo e indica qué día es. Usa las series en tiempo real solo para días recientes, intégralas a MWh y presenta el resultado como una estimación a partir de la potencia en tiempo real.

Algunas columnas guardan marcas en lugar de valores. En las ofertas de restricciones técnicas del I90 (`OFFER_RTR`, `OFFER_RTR_UPDATE`), un precio de 9999 o más significa sin oferta o sin límite, y también una energía de 9999,9 o 99999,99: exclúyelas con `abs(energy) < 9999 AND (price IS NULL OR price < 9999)` antes de sumar o promediar; las ofertas de compra (`OFFER_RTR_BUY`) no tienen precio, así que un `price < 9999` sin más las eliminaría todas. Consulta los comentarios de columna y `data_guide(topic='caveats')` para las marcas de otras tablas. Las unidades del I90 son unidades de programación (UP) y OMIE usa unidades de oferta (UOF) con otros códigos: crúzalas con `uof_code` o con la tabla de correspondencia UP-UOF, nunca por código igual.

La cobertura dice si los datos están al día. Cada fila de `/coverage` y cada entrada `coverage` de las respuestas de consulta tiene `status`, `status_reason` y una `explanation` en lenguaje llano. Algunas fuentes publican tarde por diseño: el I90 y el libro de órdenes del continuo de OMIE llegan unos 90 días después de la entrega, así que unos datos que acaban hacia hoy menos 90 días están `on_time`. `retired` marca una serie que terminó (por ejemplo, las sesiones 4 a 7 de las subastas intradiarias el 2024-06-13), `irregular` una fuente sin cadencia fija y `not_applicable` una tabla de referencia sin fechas; la marca `source_stopped` de un indicador significa que el propio ESIOS no ha publicado nada más reciente. Nada de esto es un hueco: menciónalo en una frase solo si limita la respuesta. Avisa de `late` y `stale`.

# Tu plan: Incluido

Ventana histórica: 1 meses. Filas brutas por consulta: 50. Filas agregadas por consulta: 5000. Llama a `data_status()` para conocer las fechas por tabla y la cuota actual.

# Errores

`table_not_allowed`: usa `data_status()` y `data_guide(topic='tables')` para ver las tablas de tu plan. `history_no_date_filter` y `history_restricted`: usa la ventana de fechas de `data_status()`. Para restricciones de agregación o identidad I90, sigue `fix_text` y `fix_sql_example`. Ante un error de sintaxis de ClickHouse, consulta `data_describe()` y `data_guide(topic='dialects')`.

# Recetas

Llama a `data_recipes_list(view='full')` para ver parámetros y costes actuales; usa `data_recipes_get(recipe_id=...)` para una receta. Ejecuta `data_recipes_run(recipe_id=..., params={...})` solo si tu plan muestra esa herramienta.

# Avisos

Los avisos conocidos sobre fuentes y mercados aparecen abajo cuando están disponibles. Comprueba la cobertura y frescura antes de citar datos.

# Comentarios

La herramienta MCP `feedback_submit(category, message, target, context)` está disponible para cualquier miembro del espacio. Ofrece enviar comentarios, sin esperar a que te lo pidan, cuando una skill, un tema de la guía, una receta o una función te confunda o falle, cuando una herramienta falla por permisos o acceso, cuando un resultado parece incompleto o el usuario lo cuestiona, cuando no has podido resolver la petición o has tenido que asumir algo, o cuando el usuario muestra frustración.

Pregunta brevemente al usuario y confirma siempre antes de enviar. Después llama a `feedback_submit` con:

- `category`: una de `bug`, `access`, `inaccuracy`, `suggestion`, `praise`, `other`.
- `message`: el problema con las palabras del propio usuario, sin interpretarlo. Si el fallo está en lo que has leído, cita la línea que te confundió.
- `target`: a qué se refiere, `{kind, id, version?, path?}`. `kind` es `skill`, `guide_topic`, `recipe`, `function`, `instructions` o `tool`; `id` es su nombre (`joltio-artifacts`, `time-and-units`, el id de la receta, el slug de la función, el nombre de la herramienta). En una skill, pasa su `version` del listado de skills y el `path` del fichero que leíste.
- `context`: la consulta, los parámetros o el error que lo provocó (opcional).

Un valor erróneo de una unidad, empresa, grupo o instalación no es un comentario: envíalo con `data_corrections_submit`. La identidad de quien llama se registra automáticamente: no pongas nombres ni correos en `message`. Tras enviarlo, confírmaselo al usuario; si falla, dilo con honestidad.

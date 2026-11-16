---
name: joltio-data
description: "Consult Joltio Data through its MCP or REST API for Spanish electricity and gas market analysis. Spanish reference: @references/es/guide.md."
---

# Joltio Data for agents

API: https://api.joltio.app/data/ · MCP: https://api.joltio.app/mcp

With MCP, call `data_status()` for your plan and date windows, then `data_guide(topic='...')` for details. From Python, start with the `python` topic below. This static guide describes the public plan; the MCP guide is filtered to your plan.

# Joltio Data

Start with `data_status()` to read your plan, visible tables, history windows and query limits. Use `data_guide(topic='tables')` for plan-specific table examples, `data_search()` or `data_describe()` for columns, and `data_query()` for read-only SQL or GraphQL. Use `data_recipes_list()` to find vetted analyses; `data_recipes_run()` is available only when your plan exposes it. Check freshness before citing a result.

ESIOS covers electricity indicators and I90 dispatch; OMIE covers power market auctions and continuous trading; MIBGAS covers gas markets. The exact tables available to you come from `data_status()` and `data_guide(topic='tables')`.

# Python in VS Code

Install `pip install joltio python-dotenv ipykernel` in the Python environment selected as your VS Code kernel. Put `JOLTIO_API_KEY=...` in `.env` in your working folder. Never print, copy into code, or commit the key. Load that file before creating the client:

```python
from dotenv import find_dotenv, load_dotenv
from joltio import Client

load_dotenv(find_dotenv(usecwd=True))
client = Client()
```

`client.data.query(sql)` returns a Polars DataFrame by default. `client.get("/data/describe")` lists the tables on your plan, and `client.get("/data/describe", params={"domain": "archives_i90"})` returns one table's columns, comments and coverage; take the domain name from the list. `client.data.search("text")` finds indicators, units, companies and technologies by name. `client.data.metadata()` covers only the I90 table (`esios.archives_i90`): its columns and programs. These are methods of the published `joltio` client, not MCP tools. In Python, use `client.get("/data/tier-info")` instead of MCP `data_status()` to inspect `restrictions.data_windows` and limits, and `/data/describe` and `client.data.search(text)` instead of MCP `data_describe()` and `data_search()`. Read this `llms.txt` for static guidance instead of calling MCP `data_guide()`; there is no `client.data.guide()` method. The authenticated REST guide at `/data/guide` has current plan-specific topics if needed.

For a short OMIE example on the public plan, use an aggregate with a recent date filter:

```python
df = client.data.query("SELECT toDate(datetime) AS day, round(avg(price), 2) AS eur_mwh FROM omie.prices WHERE country = 'ES' AND program = 'PDBF' AND datetime >= today() - 7 AND datetime < today() GROUP BY day ORDER BY day")
print(df)
```

I90 publishes later than the delivery date. Anchor an aggregate to the latest I90 date allowed by your plan, rather than assuming it is current:

```python
window = client.get("/data/tier-info")["restrictions"]["data_windows"]["esios.archives_i90"]
day = window["latest_data"]
df = client.data.query(f"SELECT toStartOfHour(datetime) AS hour, count() AS rows FROM esios.archives_i90 WHERE program = 'PDBF' AND datetime >= toDateTime('{day} 00:00:00', 'Europe/Madrid') AND datetime < toDateTime('{day} 01:00:00', 'Europe/Madrid') GROUP BY hour")
print(df)
```

Before other queries, check the table, columns and date window in your plan; never guess names. For `table_not_allowed`, choose a table exposed by your plan. For `history_no_date_filter` or `history_restricted`, use `your_window_start` and `latest_data` from `/data/tier-info`. For `query_too_expensive`, narrow the date range, aggregate and select fewer columns. Restriction error responses provide `fix_text` and often `fix_sql_example`; apply that guidance before retrying. The Python client's `QueryError` abbreviates the response body, so inspect the full API error when a hint is cut off. For HTTP 429, honor `Retry-After` (`RateLimitError.retry_after`) and reduce request frequency. On a corporate network with HTTPS inspection, set `SSL_CERT_FILE` and `REQUESTS_CA_BUNDLE` in `.env` to the trusted root certificate PEM, as shown in `.env.example`.

# Tables available on your plan

Use `data_status()` for current date windows and `data_describe()` for the live schema. Call `data_guide(topic='tables')` through MCP for examples built from the current schema and date window.

## archives_liquicomun.coef_k

Read-only ClickHouse table.

## archives_liquicomun.compodem

Read-only ClickHouse table.

## archives_liquicomun.components

Read-only ClickHouse table.

## archives_liquicomun.components_ssaa

Read-only ClickHouse table.

## archives_liquicomun.petar

Read-only ClickHouse table.

## archives_liquicomun.settlements

Read-only ClickHouse table.

## archives_liquicomun.timeseries

Read-only ClickHouse table.

## eia.crude_prices

Read-only ClickHouse table.

## eia.electricity_retail_sales

Read-only ClickHouse table.

## eia.electricity_retail_sales_detail

Read-only ClickHouse table.

## eia.international_annual

Read-only ClickHouse table.

## eia.lng_exports

Read-only ClickHouse table.

## eia.lng_exports_by_destination

Read-only ClickHouse table.

## eia.natural_gas_imports

Read-only ClickHouse table.

## eia.natural_gas_prices_daily

Read-only ClickHouse table.

## eia.natural_gas_prices_monthly

Read-only ClickHouse table.

## eia.rto_demand

Read-only ClickHouse table.

## eia.rto_generation_by_fuel

Read-only ClickHouse table.

## eia.rto_interchange

Read-only ClickHouse table.

## eia.steo_forecasts

Read-only ClickHouse table.

## eia.total_energy

Read-only ClickHouse table.

## entsoe.cross_border

Read-only ClickHouse table.

## entsoe.cross_border_latest

Read-only ClickHouse table.

## entsoe.dim_area

Read-only ClickHouse table.

## entsoe.dim_market

Read-only ClickHouse table.

## entsoe.dim_process

Read-only ClickHouse table.

## entsoe.dim_psr_type

Read-only ClickHouse table.

## entsoe.generation

Read-only ClickHouse table.

## entsoe.generation_latest

Read-only ClickHouse table.

## entsoe.generation_mix_hourly

Read-only ClickHouse table.

## entsoe.installed_capacity

Read-only ClickHouse table.

## entsoe.installed_capacity_latest

Read-only ClickHouse table.

## entsoe.interconnection_hourly

Read-only ClickHouse table.

## entsoe.known_gaps

Read-only ClickHouse table.

## entsoe.load

Read-only ClickHouse table.

## entsoe.load_forecast_error_hourly

Read-only ClickHouse table.

## entsoe.load_latest

Read-only ClickHouse table.

## entsoe.price_spread_hourly

Read-only ClickHouse table.

## entsoe.prices

Read-only ClickHouse table.

## entsoe.prices_daily

Read-only ClickHouse table.

## entsoe.prices_hourly

Read-only ClickHouse table.

## entsoe.prices_latest

Read-only ClickHouse table.

## esios.archives_i3

Read-only ClickHouse table.

## esios.archives_i90

Read-only ClickHouse table.

## esios.archives_i90_offers

Read-only ClickHouse table.

## esios.archives_i90_qh

Read-only ClickHouse table.

## esios.indicator_vintages_gates

Read-only ClickHouse table.

## esios.indicator_vintages_raw

Read-only ClickHouse table.

## esios.indicators

Read-only ClickHouse table.

## mibgas.auctions

Read-only ClickHouse table.

## mibgas.imbalances_pvb_vtp

Read-only ClickHouse table.

## mibgas.imbalances_tvb_avb

Read-only ClickHouse table.

## mibgas.indexes

Read-only ClickHouse table.

## mibgas.prices

Read-only ClickHouse table.

## mibgas.regulated_gas

Read-only ClickHouse table.

## mibgas.trading_data

Read-only ClickHouse table.

## omie.continuous_orders

Read-only ClickHouse table.

## omie.continuous_prices

Read-only ClickHouse table.

## omie.continuous_schedule

Read-only ClickHouse table.

## omie.continuous_trades

Read-only ClickHouse table.

## omie.continuous_volume

Read-only ClickHouse table.

## omie.interconnection_capacities

Read-only ClickHouse table.

## omie.offer_complex

Read-only ClickHouse table.

## omie.offers

Read-only ClickHouse table.

## omie.prices

Read-only ClickHouse table.

## omie.unavailabilities

Read-only ClickHouse table.

# Query dialects

Use ClickHouse SQL for aggregations and time series. Use GraphQL for typed navigation among units, companies, installations and nodes. In SQL, name only tables and columns reported by `data_status()`, `data_guide(topic='tables')` and `data_describe()`; never query system catalogs or use `SELECT *`.

ESIOS indicators contain multiple geographies. Filter by `geo_id` before comparing series: prices commonly use 3 (España), real-time generation uses 8741 (Península) and measured generation is published by province, so sum the provinces of the same hours. Verify each indicator's geographies, frequency and magnitude with `data_search()` before joining. ClickHouse tables are read with the server's deduplication policy; do not add settings to a read-only production profile.

# Time and units

Stored timestamps are UTC and label the start of each period. Spanish market days use Europe/Madrid: group by `toDate(datetime, 'Europe/Madrid')`, never by the UTC date, and filter with the date window from `data_status()`. Clock-change days have 23 or 25 hours (92 or 100 quarter-hours), so count periods instead of assuming 24 or 96. Spanish markets moved from hourly to 15-minute periods on 2025-10-01; where a table has `source_interval`, it says which one a row uses. Sum energy across that change, but never compare row counts or period averages across it. State the timezone in answers.

Prices are usually EUR/MWh (EUR/MW for capacity bands), energy is MWh, and power and capacity are MW. A power value becomes energy only after multiplying by the interval in hours: 0.25 for a quarter-hour, 1/12 for a five-minute reading. Take each unit from the column comments in `data_describe()` and, for indicators, from the magnitude (energy or power) and frequency reported by `data_search()`; never take a unit, unit code or geography from an example.

ESIOS publishes generation twice. Real-time series, such as 1295 (solar PV) and 551 (wind), are five-minute peninsular power in MW (`geo_id` 8741), published within minutes; 1295 includes PV self-consumption. Measured series, such as 1161 (solar PV) and 10037 (wind), are hourly energy in MWh by province, published from the following month. When the question asks for measured or generated energy, use the measured series on its latest complete day and say which day that is. Use real-time series only for recent days, integrate them to MWh and label the result as an estimate from real-time power.

Some columns hold markers instead of values. In I90 technical-restriction offers (`OFFER_RTR`, `OFFER_RTR_UPDATE`), prices of 9999 or more mean no offer or no limit, and so do energies of 9999.9 or 99999.99: exclude them with `abs(energy) < 9999 AND (price IS NULL OR price < 9999)` before summing or averaging; buy offers (`OFFER_RTR_BUY`) have no price, so a bare `price < 9999` would drop them all. Read the column comments and `data_guide(topic='caveats')` for markers in other tables. I90 units are programming units (UP) and OMIE uses offer units (UOF) with different codes: join them through `uof_code` or the UP-UOF crosswalk, never by equal code.

Coverage says whether data is current. Every `/coverage` row and every `coverage` entry in query responses has a `status`, a `status_reason` and a plain-language `explanation`. Some sources publish late by design: the I90 and OMIE's continuous order book arrive about 90 days after delivery, so data ending around today minus 90 days is `on_time`. `retired` marks a series that ended (for example intraday auction sessions 4-7 on 2024-06-13), `irregular` a source without a fixed cadence and `not_applicable` a reference table without dates; an indicator badge `source_stopped` means ESIOS itself published nothing newer. None of these is a gap: mention them in one sentence only when they limit the answer. Report `late` and `stale`.

# Your plan: Incluido

History window: 1 months. Raw rows per query: 50. Aggregate rows per query: 5000. Call `data_status()` for per-table dates and live quota.

# Errors

`table_not_allowed`: use `data_status()` and `data_guide(topic='tables')` for tables your plan can read. `history_no_date_filter` and `history_restricted`: use the date window in `data_status()`. I90 aggregation or identity restrictions: follow `fix_text` and `fix_sql_example`. A ClickHouse syntax error: inspect `data_describe()` and `data_guide(topic='dialects')`.

# Recipes

Call `data_recipes_list(view='full')` for current parameter schemas and cost; use `data_recipes_get(recipe_id=...)` for one recipe. Execute with `data_recipes_run(recipe_id=..., params={...})` only if that tool is visible on your plan.

# Caveats

Known source and market caveats are listed below when available. Check live coverage and freshness before citing data.

# Feedback

The `feedback_submit(category, message, target, context)` MCP tool is available to every workspace member. Offer to send feedback, without waiting to be asked, when a skill, guide topic, recipe or function confuses you or fails, when a tool fails with a permission or access error, when a result looks incomplete or the user questions it, when you could not resolve the request or had to assume something, or when the user shows frustration.

Ask the user briefly and always confirm before sending. Then call `feedback_submit` with:

- `category`: one of `bug`, `access`, `inaccuracy`, `suggestion`, `praise`, `other`.
- `message`: the issue in the user's own words, without editorializing. When the problem is in what you read, quote the line that misled you.
- `target`: what it refers to, `{kind, id, version?, path?}`. `kind` is `skill`, `guide_topic`, `recipe`, `function`, `instructions` or `tool`; `id` is its name (`joltio-artifacts`, `time-and-units`, the recipe id, the function slug, the tool name). For a skill, pass its `version` from the skill listing and the `path` of the file you read.
- `context`: the query, parameters or error that prompted it (optional).

A wrong value in a unit, company, group or installation is not feedback: submit it with `data_corrections_submit`. The caller's identity is recorded automatically: do not put names or emails in `message`. After sending, confirm it to the user; if it fails, say so honestly.

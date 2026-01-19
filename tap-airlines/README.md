# tap-airlines

Singer tap (Meltano Singer SDK) for blueprintdata that pulls flight information from `/all-flights`.

## What it does
- Builds contexts for every airport/movement (A/D) for each day from today back to `days_back`.
- Adds metadata fields (`x_fetched_at`, `x_airport_iata`, `x_movtp`, `x_date`) to every record and keeps all source fields (`additionalProperties: true`).
- Uses header `Key` authentication plus `Origin`, `User-Agent`, and optional `Accept-Language`.

## Quickstart

```bash
uv sync
cp .env.example .env
# Edit .env with your API key and airports JSON array, e.g. ["AEP","EZE"]

# Direct run (uses .env when --config=ENV)
uv run tap-airlines --config ENV --about
uv run tap-airlines --config ENV --discover
uv run tap-airlines --config ENV --test=schema
uv run tap-airlines --config ENV --test=records
```

### Date window and airports
- `days_back`: number of days backwards from today (UTC). `0` = only today; `3` = today + previous 3 days.
- `airports`: JSON array of IATA codes. Strings like `"AEP,EZE"` are normalized, but prefer `["AEP","EZE"]`.

## Configuration

- `api_url` (string, required): Base URL without `/all-flights`. Default `https://webaa-api-h4d5amdfcze7hthn.a02.azurefd.net/web-prod/v1/api-aa`.
- `api_key` (string, required, secret): Value for header `Key`.
- `origin` (string, optional): `Origin` header. Default `https://www.aeropuertosargentina.com`.
- `airports` (array[string], required): IATA codes, e.g. `["AEP","EZE"]`. Strings are normalized but prefer JSON array.
- `days_back` (int, optional): Days back from today (inclusive). Default `1`.
- `user_agent` (string, optional): `User-Agent` header. Default `Mozilla/5.0`.
- `language` (string, optional): `Accept-Language` header. Default `es-AR`.

Notes:
- The tap reads `.env` when you pass `--config=ENV`. Environment variables are `TAP_AIRLINES_<SETTING>`.
- Do not commit secrets. Prefer `meltano config set tap-airlines api_key <...>` or environment variables.

## Meltano (dev)

```bash
# Discover catalog and validate schema/records
uv run meltano --environment dev invoke tap-airlines --discover
uv run meltano --environment dev invoke tap-airlines --test=schema
uv run meltano --environment dev invoke tap-airlines --test=records

# Full pipeline to target-jsonl
uv run meltano --environment dev run tap-airlines target-jsonl
```

`meltano.yml` already includes non-sensitive defaults; only set `api_key` (and optionally `airports`, `days_back`, `origin`, `language`, `user_agent`) via `meltano config set` or env vars.

## Generate/refresh schema

The schema lives in `tap_airlines/schemas/aerolineas_all_flights.json` and keeps `additionalProperties: true`. To refresh it from live data:

```bash
TAP_AIRLINES_API_KEY=xxx \
TAP_AIRLINES_API_URL=https://webaa-api-h4d5amdfcze7hthn.a02.azurefd.net/web-prod/v1/api-aa \
TAP_AIRLINES_AIRPORTS='["AEP"]' \
uv run python scripts/generate_schema.py
```

The script writes a normalized schema and ensures metadata fields are present.

## Troubleshooting

- **Invalid `airports`**: must be a JSON array (e.g. `["AEP","EZE"]`). Empty values fail validation.
- **Missing fields / warnings**: schema allows extras, but if new fields appear, rerun `scripts/generate_schema.py`.
- **Stale catalog**: if Meltano ignores schema updates, remove `.meltano/run/tap-airlines/tap.properties.json`.
- **Structured logging**: requests log `airport/movtp/date` for traceability.

## Development and tests

```bash
uv run pytest
uv run tap-airlines --help
uv run tap-airlines --discover --config ENV
```

Python 3.11+ is supported. Follow `AGENTS.md` for contribution guidelines.

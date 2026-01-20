# tap-airlines-arg

Singer tap (Meltano Singer SDK) for blueprintdata that pulls flight information from `/all-flights`.

## What it does
- Builds contexts for every airport/movement (A/D) for each day from today back to `days_back`.
- Adds metadata fields (`x_fetched_at`, `x_airport_iata`, `x_movtp`, `x_date`) to every record and keeps all source fields (`additionalProperties: true`).
- Uses header `Key` authentication plus `Origin`, `User-Agent`, and optional `Accept-Language`.

## Prereqs
- Python 3.10+
- [uv](https://docs.astral.sh/uv/) installed (recommended) **or** a virtualenv + pip

## Install
With uv (recommended):
```bash
uv sync
```

With pip (when published on PyPI):
```bash
pip install tap-airlines-arg
```

From the repo (adjust URL if it changes):
```bash
pip install git+https://github.com/blueprintdata/tap-airlines-arg.git
# or editable local
pip install -e .
```

## Quickstart (CLI tap)

```bash
uv sync
# Optional: cp .env.example .env to override defaults.

# Use public defaults or your overrides (passing --config=ENV reads .env)
uv run tap-airlines-arg --config ENV --about
uv run tap-airlines-arg --config ENV --discover
uv run tap-airlines-arg --config ENV --test=schema
uv run tap-airlines-arg --config ENV --test=records

# Quick E2E (emits records with public defaults)
uv run tap-airlines-arg --config ENV --test=records > output/sample.jsonl
```

### Date window and airports
- `days_back`: number of days backwards from today (UTC). `0` = only today; `3` = today + previous 3 days.
- `airports`: JSON array of IATA codes. Strings like `"AEP,EZE"` are normalized, but prefer `["AEP","EZE"]`.

## Configuration

- `api_url` (string, required): Base URL without `/all-flights`. Default `https://webaa-api-h4d5amdfcze7hthn.a02.azurefd.net/web-prod/v1/api-aa`.
- `api_key` (string, required): Header `Key`. Public default `HieGcY2nFreIsNLuo5EbXCwE7g0aRzTN`.
- `origin` (string, optional): `Origin` header. Default `https://www.aeropuertosargentina.com`.
- `airports` (array[string], required): IATA codes, e.g. `["AEP","EZE"]`. Default `["AEP","EZE"]`.
- `days_back` (int, optional): Days back from today (inclusive). Default `1`. Users can change this via Meltano or env.
- `user_agent` (string, optional): `User-Agent` header. Default `Mozilla/5.0`.
- `language` (string, optional): `Accept-Language` header. Default `es-AR`.

Notes:
- `.env` is optional: the tap reads `.env` only when you pass `--config=ENV`. Variables: `TAP_AIRLINES_<SETTING>`.
- Defaults are embedded in code (public API). If you need private credentials, override `api_key`.

## Meltano (dev)

```bash
# Discover catalog and validate schema/records
uv run meltano --environment dev invoke tap-airlines-arg --discover
uv run meltano --environment dev invoke tap-airlines-arg --test=schema
uv run meltano --environment dev invoke tap-airlines-arg --test=records

# Full pipeline to target-jsonl
uv run meltano --environment dev run tap-airlines-arg target-jsonl
```

`meltano.yml` already includes public defaults (including `api_key`). Adjust `days_back`, `airports`, or other values via `meltano config set` or environment variables.

If `uv run` complains about cache permissions, use `UV_CACHE_DIR=.uv-cache uv run ...` or run Meltano from an environment with write access to `~/.cache/uv`.

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
- **Stale catalog**: if Meltano ignores schema updates, remove `.meltano/run/tap-airlines-arg/tap.properties.json`.
- **Structured logging**: requests log `airport/movtp/date` for traceability.

## Development and tests

```bash
uv run pytest
uv run tap-airlines-arg --help
uv run tap-airlines-arg --discover --config ENV

# Local runner without uv (if you already have a venv): ./.venv/bin/tap-airlines-arg --config ENV --test=records
```

Python 3.10+ is supported. Follow `AGENTS.md` for contribution guidelines.

## Release name
- Package: `tap-airlines-arg`
- Wheel: `tap_airlines_arg-<version>-py3-none-any.whl`
- CLI: `tap-airlines-arg`

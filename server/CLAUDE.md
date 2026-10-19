# CLAUDE.md - Server

This file provides guidance to Claude Code (claude.ai/code) when working with the FastAPI backend. See the root `CLAUDE.md` for the overall architecture.

## Commands

```bash
uv sync                        # Python >= 3.11; installs pytest etc. as dev deps
uv run python main.py          # http://localhost:8001, docs at /docs
uv run pytest ../tests         # backend tests (must run from server/)
uv run pytest ../tests/backend/test_dashboard.py::TestDashboardEndpoints
```

## Structure
- Everything is in `main.py`: the Pydantic models, the `QUARTER_MAP`, the `apply_filters()` / `filter_by_month()` helpers, and every route. There is no separate models or services module.
- `mock_data.py` loads `data/*.json` into module-level lists at import. Data stays in memory, and changes are lost on restart. `generate_data.py` regenerates the JSON.
- CORS allows all origins so the Vite dev server on port 3000 can call the API.

## Conventions
- Optional filter query params; skip a filter when it's missing or `'all'`. Reuse `apply_filters()` and `filter_by_month()` rather than writing new loops.
- `filter_by_month()` filters on `order_date`, so it only applies to data that has that field. Inventory has no time dimension.
- Use `response_model=` on routes that return model data, and update the model whenever a JSON file's shape changes.
- Single-item routes raise `HTTPException(status_code=404, ...)` when not found.
- Don't mutate the global lists. Copy items first (as `/api/backlog` does with `dict(item)` before adding `has_purchase_order`).
- Add tests in `tests/backend/` (see the `backend-api-test` skill).

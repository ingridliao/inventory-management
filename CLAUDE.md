# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System Demo with GitHub integration - Full-stack application with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

> ⚠️ **This repository and any fork you create are PUBLIC.** Do not commit credentials, internal hostnames, or private registry URLs. `client/.npmrc` pins the public npm registry and `client/package-lock.json` is gitignored to prevent locally-configured registries from leaking into commits — leave both in place.

## Critical Tool Usage Rules

### Subagents
Use the Task tool with these specialized subagents for appropriate tasks:

- **vue-expert**: Use for Vue 3 frontend features, UI components, styling, and client-side functionality
  - Examples: Creating components, fixing reactivity issues, performance optimization, complex state management
  - **MANDATORY RULE: ANY time you need to create or significantly modify a .vue file, you MUST delegate to vue-expert**
- **code-reviewer**: Use after writing significant code to review quality and best practices
- **Explore**: Use for understanding codebase structure, searching for patterns, or answering questions about how components work
- **general-purpose**: Use for complex multi-step tasks or when other agents don't fit

### Skills
- **backend-api-test** skill: Use when writing or modifying tests in `tests/backend` directory with pytest and FastAPI TestClient

### MCP Tools
- **ALWAYS use GitHub MCP tools** (`mcp__github__*`) for ALL GitHub operations
  - Exception: Local branches only - use `git checkout -b` instead of `mcp__github__create_branch`
- **ALWAYS use Playwright MCP tools** (`mcp__playwright__*`) for browser testing
  - Test against: `http://localhost:3000` (frontend), `http://localhost:8001` (API)

## Release Standards
- All changes need test evidence before merging.
- No direct commits to `main`. Work on a branch and merge through a PR.
- Every PR must include a one-line business impact summary.

## Stack
- **Frontend**: Vue 3 + Vite + vue-router + axios (port 3000, set in `client/vite.config.js`)
- **Backend**: Python FastAPI, Python >= 3.11, managed with uv (port 8001, set at the bottom of `server/main.py`)
- **Data**: JSON files in `server/data/` loaded via `server/mock_data.py`

## Commands

```bash
# Backend
cd server && uv sync                    # install deps (includes pytest dev deps)
cd server && uv run python main.py      # http://localhost:8001, docs at /docs

# Frontend
cd client && npm install
cd client && npm run dev                # http://localhost:3000
cd client && npm run build

# Both at once (logs in /tmp/inventory-*.log)
./scripts/start.sh

# Tests — run from server/ so uv uses its venv; tests/ has no pyproject of its own
cd server && uv run pytest ../tests
cd server && uv run pytest ../tests/backend/test_inventory.py
cd server && uv run pytest ../tests/backend/test_inventory.py::TestInventoryEndpoints::test_get_all_inventory
cd server && uv run pytest ../tests --cov=. --cov-report=html
```

There is no linter, type checker, or frontend test suite configured. Backend tests import `main.app` directly via `tests/backend/conftest.py`, so they don't need a running server.

## Architecture

**Data flow**: `useFilters()` state → view calls `api.getX(getCurrentFilters())` in `client/src/api.js` → FastAPI route → `apply_filters()` / `filter_by_month()` over in-memory lists → `response_model` Pydantic validation → view refs → computed properties.

**Filters**:
- The four filters (Time Period, Warehouse, Category, Order Status) are module-level refs in `client/src/composables/useFilters.js`, so every view and `FilterBar.vue` share one instance. Views `watch` them and refetch.
- `'all'` means "no filter" on both sides: `api.js` omits the param, and the server helpers skip it.
- Time Period is sent as `month`: either `YYYY-MM` (substring match on `order_date`) or `Q1-2025`…`Q4-2025` via the hard-coded `QUARTER_MAP` in `server/main.py`.
- Category and status match case-insensitively; warehouse is exact.

**Mock data**: `mock_data.py` loads every JSON file once at import into module globals that `main.py` imports. Anything mutated at runtime lives until restart. `server/generate_data.py` regenerates the JSON.

**i18n and currency**:
- All UI strings go through `t('section.key')` from `client/src/composables/useI18n.js`, backed by `client/src/locales/en.js` and `ja.js`. New UI text needs keys in both files.
- The locale is persisted in `localStorage` (`app-locale`). Japanese also switches currency to JPY: format money with `formatCurrency()` from `client/src/utils/currency.js` (fixed USD→JPY rate), not `toLocaleString` directly.
- `useAuth.js` is a mock user whose fields change with the locale.

**Routing**: Routes are registered in `client/src/main.js`; the nav links are in `client/src/App.vue`. Update both when adding a page.

## Known Gaps
- Purchase orders are only half built:
  - `api.js` has `createPurchaseOrder` (`POST /api/purchase-orders`) and `getPurchaseOrderByBacklogItem` (`GET /api/purchase-orders/{id}`), but the server has no such routes. Only the `PurchaseOrder` / `CreatePurchaseOrderRequest` models and the `purchase_orders` data exist.
  - `Dashboard.vue` renders `<PurchaseOrderModal>`, which is neither imported nor present in `components/`.
- `client/src/views/Backlog.vue` exists but is not registered as a route.

## API Endpoints
- `GET /api/inventory` - Filters: warehouse, category
- `GET /api/inventory/{item_id}`
- `GET /api/orders` - Filters: warehouse, category, status, month
- `GET /api/orders/{order_id}`
- `GET /api/dashboard/summary` - All filters
- `GET /api/demand`, `/api/backlog` - No filters (backlog adds `has_purchase_order`)
- `GET /api/spending/summary`, `/monthly`, `/categories`, `/transactions` - No filters
- `GET /api/reports/quarterly`, `/api/reports/monthly-trends` - No filters

## Common Issues
1. Use unique keys in v-for (not `index`) - use `sku`, `month`, etc.
2. Validate dates before `.getMonth()` calls
3. Update Pydantic models when changing JSON data structure
4. Inventory filters don't support month (no time dimension)
5. Revenue goals: $800K/month single, $9.6M YTD all months

## File Locations
- Views: `client/src/views/*.vue`
- Shared state: `client/src/composables/` (filters, i18n, auth)
- API Client: `client/src/api.js`
- Backend: `server/main.py` (models, filter helpers, and all routes), `server/mock_data.py`
- Data: `server/data/*.json`
- Global styles: `client/src/App.vue`

## Design System
- Colors: Slate/gray (#0f172a, #64748b, #e2e8f0)
- Status: green/blue/yellow/red
- Charts: Custom SVG, CSS Grid for layouts
- No emojis in UI

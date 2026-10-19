# CLAUDE.md - Client

This file provides guidance to Claude Code (claude.ai/code) when working with the Vue 3 frontend. See the root `CLAUDE.md` for the overall architecture.

## Commands

```bash
npm run dev     # http://localhost:3000 (expects the API on http://localhost:8001)
npm run build
```

No linter or test runner is configured.

## Conventions
- Views (`src/views/`) and `FilterBar.vue`/`TasksModal.vue` use the Options-style `setup()` function that returns bindings. The other components use `<script setup>`. Match the style of the file you're editing.
- Raw API data goes in refs; anything derived from it goes in computed properties. Views reload when they `watch` the `useFilters()` refs.
- Most views track `loading` and `error` refs and render `v-if="loading"` / `v-else-if="error"` states.
- All API calls live in `src/api.js`, which uses a hard-coded `API_BASE_URL` of `http://localhost:8001/api`. Don't call axios from components.
- Text: always use `t('...')` from `useI18n()`, and add the key to both `src/locales/en.js` and `src/locales/ja.js`.
- Money: use `formatCurrency()` / `formatCurrencyWithDecimals()` from `src/utils/currency.js` with the current currency from `useI18n()`, so JPY mode works.
- v-for keys: use a stable id (`sku`, `id`, `month`), never the index.
- Validate dates (`!isNaN(date.getTime())`) before calling `.getMonth()` etc.
- Charts are hand-built SVG with a `viewBox`; there is no chart library.

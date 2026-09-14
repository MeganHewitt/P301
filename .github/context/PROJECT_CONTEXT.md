# ShelfPulse Project Context

## Purpose

ShelfPulse is a single-page operational dashboard for a retail category manager. It summarizes weekly revenue, sales versus plan, inventory turns, foot traffic, promotions, and SKU health.

## Stack

- Vue 3 with `<script setup>`
- TypeScript
- Vite
- Chart.js with `vue-chartjs`
- Local JSON data; no API layer

## Key locations

| Location | Responsibility |
| --- | --- |
| `src/main.ts` | Creates and mounts the Vue app |
| `src/App.vue` | Dashboard state, filtering, KPI calculations, charts, table interactions |
| `src/style.css` | Global layout, colors, typography, and responsive behavior |
| `src/data/metrics.json` | Twelve weeks of aggregate metrics |
| `src/data/skus.json` | SKU records and weekly sales trends |
| `src/data/promotions.json` | Promotion performance records |
| `BRIEF.MD` | Product requirements and acceptance criteria |
| `README.md` | Setup, commands, and repository map |
| `.github/` | AI instructions and project context |

## Important behavior

- Category navigation filters SKU rows and category-sensitive KPI/chart values.
- The date range is constrained to the twelve-week local dataset and controls visible weekly revenue, KPI comparison, and promotion overlap.
- Clicking sortable table headers changes ordering; clicking a SKU expands its detail row.
- The alerts rail is static dashboard guidance, not an API-backed notification system.

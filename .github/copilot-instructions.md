# ShelfPulse Contributor Instructions

## Project shape

- This is a Vue 3 + TypeScript + Vite single-page dashboard.
- The application entry point is `src/main.ts`; the primary UI lives in `src/App.vue`.
- Global styling lives in `src/style.css`.
- Local dashboard data lives in `src/data/` as JSON files.
- Keep generated output in `dist/` and dependencies in `node_modules/`; do not edit either directory by hand.

## Development workflow

- Install dependencies with `npm install`.
- Start the app with `npm run dev`.
- Validate production changes with `npm run build`.
- Use the existing Vue and Chart.js patterns before introducing new dependencies.

## UI and data conventions

- Preserve the operational dashboard layout and responsive desktop/mobile behavior.
- Keep status colors semantic: green for healthy, amber for attention, and red for risk.
- Keep the navy accent for navigation and primary dashboard emphasis.
- Treat `metrics.json`, `skus.json`, and `promotions.json` as the local data contracts. Update the consuming TypeScript types when those shapes change.
- Keep filtering and sorting behavior in `App.vue` unless a new component boundary clearly reduces complexity.

## Change discipline

- Make focused edits and preserve unrelated user changes.
- Do not commit generated files or credentials.
- After UI or data changes, run `npm run build` and report any remaining issues.

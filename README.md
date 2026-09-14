# ShelfPulse

ShelfPulse is a Vue dashboard for retail category operations. It gives a category manager a fast view of weekly revenue, sales versus plan, inventory turns, promotions, and SKU health.

## Requirements

- Node.js 20 or newer
- npm

## Run locally

```bash
npm install
npm run dev
```

Open the local URL printed by Vite, usually `http://localhost:5173/`.

## Available commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the Vite development server |
| `npm run build` | Type-check and create a production build |
| `npm run preview` | Preview the production build locally |

## Repository map

```text
.
├── .github/
│   ├── context/PROJECT_CONTEXT.md
│   └── copilot-instructions.md
├── public/                 # Static public assets
├── src/
│   ├── data/               # Local JSON dashboard data
│   ├── App.vue             # Dashboard UI and interactions
│   ├── main.ts             # Vue application entry point
│   └── style.css           # Global styles and responsive layout
├── BRIEF.MD               # Product requirements
├── LICENSE                # MIT license
├── package.json           # Scripts and dependencies
└── vite.config.ts         # Vite configuration
```

Generated `dist/` output and installed `node_modules/` are intentionally omitted from the source map.

## Data

The dashboard uses local data only:

- `src/data/metrics.json` contains twelve weeks of aggregate metrics.
- `src/data/skus.json` contains SKU status, promotion, and sales trend data.
- `src/data/promotions.json` contains promotion lift records.

The category navigation and date range filter update the dashboard from these records without API calls.

## AI-assisted development

AI guidance is centralized in `.github/`:

- [`copilot-instructions.md`](.github/copilot-instructions.md) contains contribution and coding conventions.
- [`context/PROJECT_CONTEXT.md`](.github/context/PROJECT_CONTEXT.md) contains the architecture map and behavior contracts.

Keeping project context in one location makes it discoverable for contributors and AI coding tools without scattering instruction files through `src/`.

## License

This project is released under the MIT License. See [`LICENSE`](LICENSE).

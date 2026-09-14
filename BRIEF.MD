# What is this?
An operational dashboard for a retail category manager to monitor daily performance across SKUs, inventory, and promotions. Think a simplified Tableau or Looker view — dense but scannable, designed for a working professional at the start of their day.

## Data
- Generate a fake dataset as a JSON file (src/data/metrics.json) Weekly data for the last 12 weeks, each week containing:
    - week (label, e.g. "Wk 1", "Wk 2", etc.)
    - totalSales (dollar amount per week, trending upward with some variation)
    - salesVsPlan (percentage, e.g. 0.94 = 94% of plan, fluctuates between 85–110%)
    - inventoryTurnRate (number, e.g. 4.2 turns, relatively stable with small variation)
    - footTrafficIndex (number, normalized to 100 baseline, seasonal dip in middle weeks)
- Also include a src/data/skus.json file containing 12 SKU records, each with:
    - id, name, category (one of: "Snacks", "Household", "Personal Care")
    - weeklySalesTrend (array of 12 numbers — sparkline data)
    - stockStatus ("In Stock", "Low", "At Risk")
    - promoActive (boolean)
    - salesVsPlan (percentage, e.g. 0.91)
- And a src/data/promotions.json file containing 4 promotions, each with:
    - name, category, startWeek, endWeek
    - baselineSales (dollar amount per week before promo)
    - promoSales (dollar amount per week during promo)
    - liftPercent (calculated, e.g. 18.4)

## Layout
- App bar at top: dashboard title ("ShelfPulse"), category filter dropdown (All / Snacks / Household / Personal Care)
- Below app bar: a row of 4 summary KPI cards showing   
    — Total Sales, Sales vs. Plan (%), Inventory Turn Rate, Foot Traffic Index
    - Each card shows the current week's value, a small up/down indicator vs. the previous week, and a subtle color state (green / amber / red)
- Below cards: two-column row
    - Left: Bar chart — Weekly revenue for the last 12 weeks
    - Right: Horizontal bar chart — Promo lift % per active promotion
- Below charts: full-width SKU table with columns: SKU Name, Category, Stock Status (badge), Promo Active (icon), Sales vs. Plan (%), Sparkline (inline mini chart)
- Use a fixed left sidebar (200px) for category nav on desktop; collapse to top filter on mobile
- Use CSS Grid or Flexbox for layout — no UI framework required, but Tailwind is fine if preferred

## Interactions
- Category filter in app bar filters the SKU table and updates the KPI cards to reflect only that category's data
- "All" selected shows aggregate across all categories
- Stock Status badges are color-coded: green = In Stock, amber = Low, red = At Risk
- SKU table rows are sortable by clicking column headers (Sales vs. Plan, Stock Status)
- Clicking a SKU row expands an inline detail panel showing the sparkline larger and a short notes field (static text, no editing needed)
- Alerts rail on the right side (desktop only, 240px): shows static list of 3–4 flagged items (e.g. "SKU #04 approaching stockout", "Promo lift below target for Household")

## - Style
Light theme with a neutral gray base (
#F8F9FA background)
- Semantic status colors only: 
#22C55E (green), 
#F59E0B (amber), 
#EF4444 (red) — used exclusively for status, not decoration
- Primary accent: 
#1E40AF (navy blue) for headers, active states, and the app bar
- Clean, high-density layout — this is a work tool, not a marketing page
- Font: Inter or system-ui; 13px base for table rows, 15px for body, 24px+ for KPI values
- Mobile responsive: KPI cards stack 2x2, sidebar collapses, table becomes horizontally scrollable

## Tech
- Vue 3 + Typescript + Vuetify 3
- Chart.js via vue-chartjs for all charts
- Fake data from local JSON file (no API calls)
- Single page - no routing needed for this App

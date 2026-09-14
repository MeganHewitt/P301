<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar as VueChartBar } from 'vue-chartjs'
import { BarElement, CategoryScale, Chart as ChartJS, LinearScale, Tooltip } from 'chart.js'
import metricsData from './data/metrics.json'
import promotionsData from './data/promotions.json'
import skusData from './data/skus.json'

ChartJS.register(CategoryScale, LinearScale, BarElement, Tooltip)

interface Metric { week: string; totalSales: number; salesVsPlan: number; inventoryTurnRate: number; footTrafficIndex: number }
interface SKU { id: string; name: string; category: string; weeklySalesTrend: number[]; stockStatus: 'In Stock' | 'Low' | 'At Risk'; promoActive: boolean; salesVsPlan: number }
interface Promotion { name: string; category: string; startWeek: number; endWeek: number; baselineSales: number; promoSales: number; liftPercent: number }

const metrics = metricsData as Metric[]
const skus = skusData as SKU[]
const promotions = promotionsData as Promotion[]
const categories = ['All', 'Snacks', 'Household', 'Personal Care']
const selectedCategory = ref('All')
const datasetStartDate = '2026-07-01'
const datasetEndDate = '2026-09-16'
const startDate = ref('2026-07-01')
const endDate = ref('2026-09-16')
const sortBy = ref<'name' | 'salesVsPlan' | 'stockStatus'>('name')
const expandedSKU = ref<string | null>(null)
const sortDescending = ref(false)

const weekDates = metrics.map((_, index) => {
  const date = new Date(`${datasetStartDate}T00:00:00Z`)
  date.setUTCDate(date.getUTCDate() + index * 7)
  return date.toISOString().slice(0, 10)
})
const visibleWeekIndexes = computed(() => metrics
  .map((_, index) => index)
  .filter((index) => weekDates[index] >= startDate.value && weekDates[index] <= endDate.value))

const filteredSKUs = computed(() => {
  const filtered = selectedCategory.value === 'All' ? [...skus] : skus.filter((sku) => sku.category === selectedCategory.value)
  return filtered.sort((a, b) => {
    if (sortBy.value === 'salesVsPlan') return (b.salesVsPlan - a.salesVsPlan) * (sortDescending.value ? -1 : 1)
    if (sortBy.value === 'stockStatus') { const order = { 'At Risk': 0, Low: 1, 'In Stock': 2 }; return (order[a.stockStatus] - order[b.stockStatus]) * (sortDescending.value ? -1 : 1) }
    return a.name.localeCompare(b.name) * (sortDescending.value ? -1 : 1)
  })
})
const categorySKUs = computed(() => selectedCategory.value === 'All' ? skus : skus.filter((sku) => sku.category === selectedCategory.value))
const weeklySales = computed(() => visibleWeekIndexes.value.map((weekIndex) => selectedCategory.value === 'All' ? metrics[weekIndex].totalSales : categorySKUs.value.reduce((sum, sku) => sum + sku.weeklySalesTrend[weekIndex], 0)))
const currentKPIs = computed(() => {
  const lastWeek = visibleWeekIndexes.value.at(-1) ?? metrics.length - 1
  if (selectedCategory.value === 'All') return { totalSales: weeklySales.value.at(-1) ?? 0, salesVsPlan: metrics[lastWeek].salesVsPlan, inventoryTurnRate: metrics[lastWeek].inventoryTurnRate, footTrafficIndex: metrics[lastWeek].footTrafficIndex }
  return { totalSales: weeklySales.value.at(-1) ?? 0, salesVsPlan: categorySKUs.value.reduce((sum, sku) => sum + sku.salesVsPlan, 0) / categorySKUs.value.length, inventoryTurnRate: metrics[lastWeek].inventoryTurnRate, footTrafficIndex: metrics[lastWeek].footTrafficIndex }
})
const previousKPIs = computed(() => {
  const previousWeek = visibleWeekIndexes.value.at(-2) ?? Math.max((visibleWeekIndexes.value.at(-1) ?? metrics.length - 1) - 1, 0)
  if (selectedCategory.value === 'All') return { totalSales: weeklySales.value.at(-2) ?? 0, salesVsPlan: metrics[previousWeek].salesVsPlan, inventoryTurnRate: metrics[previousWeek].inventoryTurnRate, footTrafficIndex: metrics[previousWeek].footTrafficIndex }
  return { totalSales: weeklySales.value.at(-2) ?? 0, salesVsPlan: currentKPIs.value.salesVsPlan, inventoryTurnRate: metrics[previousWeek].inventoryTurnRate, footTrafficIndex: metrics[previousWeek].footTrafficIndex }
})
const kpiCards = computed(() => [
  { key: 'totalSales', label: 'Total sales', value: currentKPIs.value.totalSales, format: 'currency' },
  { key: 'salesVsPlan', label: 'Sales vs plan', value: currentKPIs.value.salesVsPlan, format: 'percent' },
  { key: 'inventoryTurnRate', label: 'Inventory turn rate', value: currentKPIs.value.inventoryTurnRate, format: 'turns' },
  { key: 'footTrafficIndex', label: 'Foot traffic index', value: currentKPIs.value.footTrafficIndex, format: 'index' },
])
const chartData = computed(() => ({ labels: visibleWeekIndexes.value.map((index) => metrics[index].week), datasets: [{ data: weeklySales.value, backgroundColor: '#1E40AF', borderRadius: 3, barPercentage: 0.72 }] }))
const chartOptions = { responsive: true, maintainAspectRatio: false, plugins: { legend: { display: false }, tooltip: { callbacks: { label: (context: { parsed: { y: number | null } }) => `$${(context.parsed.y ?? 0).toLocaleString()}` } } }, scales: { x: { grid: { display: false }, ticks: { color: '#64748B' } }, y: { beginAtZero: true, grid: { color: '#E2E8F0' }, ticks: { color: '#64748B', callback: (value: string | number) => `$${Number(value) / 1000}k` } } } }
const activePromos = computed(() => promotions.filter((promo) => {
  const overlapsRange = visibleWeekIndexes.value.some((weekIndex) => weekIndex + 1 >= promo.startWeek && weekIndex + 1 <= promo.endWeek)
  return overlapsRange && (selectedCategory.value === 'All' || promo.category === selectedCategory.value || promo.category === 'All')
}))
const alerts = [
  { level: 'critical', text: 'SKU #04 approaching stockout', detail: 'Laundry Detergent 64oz' },
  { level: 'warning', text: 'Promo lift below target', detail: 'Household Essentials Clearance' },
  { level: 'warning', text: 'Foot traffic index declining', detail: 'Down 3.8% week over week' },
  { level: 'info', text: 'Inventory turn rate below 4.0', detail: 'Review replenishment plan' },
]
function formatValue(value: number, format: string) { if (format === 'currency') return `$${value.toLocaleString()}`; if (format === 'percent') return `${(value * 100).toFixed(0)}%`; if (format === 'turns') return `${value.toFixed(1)}x`; return value.toFixed(0) }
function categoryLabel(category: string) { return category === 'Snacks' ? 'Groceries' : category }
function trend(current: number, previous: number) { const delta = ((current - previous) / Math.max(Math.abs(previous), 1)) * 100; return { direction: delta >= 0 ? 'up' : 'down', delta: `${Math.abs(delta).toFixed(1)}%` } }
function setSort(column: 'name' | 'salesVsPlan' | 'stockStatus') { if (sortBy.value === column) sortDescending.value = !sortDescending.value; else { sortBy.value = column; sortDescending.value = false } }
function sparklinePoints(values: number[], width = 120, height = 34) { const min = Math.min(...values); const range = Math.max(Math.max(...values) - min, 1); return values.map((value, index) => `${((index / (values.length - 1)) * width).toFixed(1)},${(height - ((value - min) / range) * (height - 4) - 2).toFixed(1)}`).join(' ') }
</script>

<template>
  <div class="app-shell">
    <header class="app-bar"><div class="brand-lockup"><div class="brand-mark">SP</div><div><strong>ShelfPulse</strong></div></div><div class="header-context"><span class="live-dot"></span> Live view <span class="header-divider"></span> {{ visibleWeekIndexes.length }} weeks</div><div class="date-filter"><label>From <input v-model="startDate" :min="datasetStartDate" :max="endDate" type="date" aria-label="Start date"></label><span>to</span><label>To <input v-model="endDate" :min="startDate" :max="datasetEndDate" type="date" aria-label="End date"></label></div></header>
    <div class="workspace">
      <aside class="sidebar"><p class="sidebar-label">Workspace</p><nav><button v-for="category in categories" :key="category" :class="{ active: selectedCategory === category }" @click="selectedCategory = category"><span class="nav-icon" :class="category.toLowerCase().replace(' ', '-')"></span>{{ categoryLabel(category) }}<span v-if="category !== 'All'" class="nav-count">{{ skus.filter((sku) => sku.category === category).length }}</span></button></nav><div class="sidebar-footer"><span class="status-pulse"></span><span>Data synced<br><b>Today, 08:42 AM</b></span></div></aside>
      <main class="main-content"><div class="page-heading"><div><p class="eyebrow">Overview / {{ categoryLabel(selectedCategory) }} · {{ startDate }} to {{ endDate }}</p><h1>Good morning, Megan.</h1></div><button class="export-button" type="button">↓ <span>Export report</span></button></div>
        <section class="kpi-grid" aria-label="Key performance indicators"><article v-for="card in kpiCards" :key="card.key" class="kpi-card"><div class="kpi-topline"><span>{{ card.label }}</span></div><strong>{{ formatValue(card.value, card.format) }}</strong><div class="kpi-trend" :class="trend(card.value, previousKPIs[card.key as keyof typeof previousKPIs]).direction">{{ trend(card.value, previousKPIs[card.key as keyof typeof previousKPIs]).direction === 'up' ? '↗' : '↘' }} {{ trend(card.value, previousKPIs[card.key as keyof typeof previousKPIs]).delta }} <span>vs last week</span></div></article></section>
        <section class="charts-grid"><article class="panel revenue-panel"><div class="panel-heading"><div><h2>Weekly revenue</h2><p>{{ visibleWeekIndexes.length }} weeks · {{ categoryLabel(selectedCategory) }}</p></div><span class="panel-stat">{{ formatValue(currentKPIs.totalSales, 'currency') }} <small>current</small></span></div><div class="revenue-chart"><VueChartBar :data="chartData" :options="chartOptions" /></div></article><article class="panel promo-panel"><div class="panel-heading"><div><h2>Promotion lift</h2><p>Performance by active promotion</p></div><span class="panel-stat accent-stat">{{ activePromos.length }} <small>active</small></span></div><div class="promo-list"><div v-for="promo in activePromos" :key="promo.name" class="promo-row"><div class="promo-label"><span>{{ promo.name }}</span><small>{{ categoryLabel(promo.category) }}</small></div><div class="promo-track"><i :style="{ width: `${Math.min((promo.liftPercent / 30) * 100, 100)}%` }"></i></div><b>{{ promo.liftPercent.toFixed(1) }}%</b></div></div></article></section>
        <section class="panel table-panel"><div class="panel-heading table-heading"><div><h2>SKU performance</h2><p>{{ filteredSKUs.length }} products in view · Click a row for details</p></div><span class="table-filter">Sorted by {{ sortBy === 'name' ? 'name' : sortBy === 'salesVsPlan' ? 'sales vs plan' : 'stock status' }}</span></div><div class="table-scroll"><table><thead><tr><th @click="setSort('name')">SKU name <span>↕</span></th><th>Category</th><th @click="setSort('stockStatus')">Stock status <span>↕</span></th><th>Promo</th><th @click="setSort('salesVsPlan')">Sales vs plan <span>↕</span></th><th>Trend</th></tr></thead><tbody><template v-for="sku in filteredSKUs" :key="sku.id"><tr class="sku-row" :class="{ expanded: expandedSKU === sku.id }" @click="expandedSKU = expandedSKU === sku.id ? null : sku.id"><td><strong>{{ sku.name }}</strong><small>{{ sku.id }}</small></td><td>{{ categoryLabel(sku.category) }}</td><td><span class="status-badge" :class="sku.stockStatus.toLowerCase().replace(' ', '-')">{{ sku.stockStatus }}</span></td><td><span :class="['promo-indicator', { on: sku.promoActive }]">{{ sku.promoActive ? '●' : '—' }}</span></td><td><strong :class="sku.salesVsPlan >= 1 ? 'above-plan' : 'below-plan'">{{ (sku.salesVsPlan * 100).toFixed(0) }}%</strong></td><td><svg class="sparkline" viewBox="0 0 120 34" role="img" :aria-label="`${sku.name} sales trend`"><polyline :points="sparklinePoints(sku.weeklySalesTrend)" /></svg><span class="row-chevron">{{ expandedSKU === sku.id ? '⌃' : '⌄' }}</span></td></tr><tr v-if="expandedSKU === sku.id" class="detail-row"><td colspan="6"><div class="detail-content"><div><span class="detail-label">12-week sales trend</span><svg class="large-sparkline" viewBox="0 0 360 90"><polyline :points="sparklinePoints(sku.weeklySalesTrend, 360, 90)" /></svg></div><div class="detail-copy"><span class="detail-label">Performance note</span><p>{{ sku.stockStatus === 'At Risk' ? 'Replenishment is the priority. Current demand is outpacing available stock.' : sku.salesVsPlan >= 1 ? 'Strong performer this period, with sales pacing ahead of plan.' : 'Monitor this SKU closely and review the next promotion opportunity.' }}</p><span class="detail-meta">{{ sku.promoActive ? 'Active promotion' : 'No active promotion' }} · Latest week: ${{ sku.weeklySalesTrend[11].toLocaleString() }}</span></div></div></td></tr></template></tbody></table></div></section>
      </main>
      <aside class="alerts-rail"><div class="rail-heading"><h2>Attention</h2><span>{{ alerts.length }}</span></div><p class="rail-note">Items that may need your review.</p><div v-for="alert in alerts" :key="alert.text" class="alert-item" :class="alert.level"><span class="alert-marker"></span><div><strong>{{ alert.text }}</strong><small>{{ alert.detail }}</small></div></div><div class="rail-footer">Review all alerts <span>→</span></div></aside>
    </div>
  </div>
</template>

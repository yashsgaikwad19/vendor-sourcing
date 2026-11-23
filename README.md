# Vendor Sourcing Dashboard

A single-file, browser-based decision-support tool for pharmaceutical packaging-material sourcing. It ranks suppliers with **AHP + TOPSIS**, allocates demand across the eligible ones, and stress-tests the plan against **rainfall-driven supply disruption** using live weather data and **Monte Carlo simulation**.

No backend, no build step, no install. Open `index.html` in a browser.

> **Live demo:** `https://<your-username>.github.io/<repo-name>/`

---

## What it does

The dashboard is an 8-step guided workflow (sidebar navigation):

| Step | Section | Purpose |
|---|---|---|
| 1 | **Variables** | Define evaluation criteria and whether lower or higher is better. Defaults: cost, quality (defect rate/PPM), OTIF %, lead-time consistency, financial stability, GMP/ISO compliance, capacity flexibility, responsiveness. |
| 2 | **AHP weights** | Pairwise-comparison matrix per expert (up to 6), geometric-mean aggregation, principal-eigenvector weights, and a **Consistency Ratio** check (flags CR ≥ 0.10). Includes a one-variable **weight sensitivity** slider. |
| 3 | **Suppliers** | Enter scores, location, capacity and MOQ per supplier, or bulk-import from Excel/CSV (template provided). |
| 4 | **TOPSIS ranking** | Vector-normalised, weighted closeness-to-ideal composite score per supplier. |
| 5 | **Constraints** | Hard filters (e.g. minimum composite score) that decide which suppliers are eligible. |
| 6 | **Rainfall risk** | Per-supplier disruption signal from [Open-Meteo](https://open-meteo.com/): share of the next 7 forecast days exceeding that location's own 90th-percentile daily rainfall over the past year. Optional 15-minute auto-refresh. |
| 7 | **Monte Carlo** | 1,000-iteration simulation of the fixed allocation plan under random supplier disruption; reports mean/P5/P50/P95 cost, mean and worst-case service level, and a cost histogram. |
| 8 | **Optimize** | Allocates total demand and compares **optimized** (with and without rainfall uncertainty) against three baselines: cheapest-only, highest-quality-only, and 50-50 split. |

## Method summary

- **Weights:** AHP. Multiple experts are aggregated by element-wise geometric mean; weights come from the principal eigenvector (power iteration); consistency uses Saaty's Random Index.
- **Ranking:** TOPSIS. Criteria are direction-aware (`lower` / `higher` is better); score = D⁻ / (D⁺ + D⁻).
- **Allocation:** eligible suppliers are sorted by composite score and filled in order up to capacity. Under uncertainty, capacity is derated by `(1 − rainfall risk)`. Allocations below a supplier's MOQ are dropped.
- **Disruption probability (Monte Carlo):** `p = min(0.9, 0.05 + 0.35·(1 − TOPSIS score) + 0.5·rainfall risk)`. Suppliers sharing the same location string share one random draw, so regional events hit them together (common-cause correlation).

## Usage

### Run locally
```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
# open index.html in any modern browser, or serve it:
python -m http.server 8000   # then visit http://localhost:8000
```

### Deploy on GitHub Pages
1. Rename the file to `index.html` and push it to the repo root.
2. **Settings → Pages → Build and deployment →** Source: *Deploy from a branch*, branch `main`, folder `/ (root)`.
3. Your site goes live at `https://<your-username>.github.io/<repo-name>/`.

### Excel / CSV import
- **Suppliers:** click *Download template*, fill it in, and re-import. Column headers are matched fuzzily (`Supplier/Name`, `Location`, `Capacity`, `MOQ`, plus one column per variable name).
- **AHP matrices:** one sheet per expert (max 6) in the downloaded template layout.

## Tech stack

React 18 (UMD) · Babel Standalone (in-browser JSX) · Tailwind CSS (CDN) · SheetJS (`xlsx`) · Open-Meteo geocoding, archive and forecast APIs · IBM Plex fonts.

All dependencies load from CDNs, so an internet connection is required. State lives in memory only; refreshing the page resets everything.

## Limitations

Read these before using any output for a real decision.

- **The allocator is a greedy heuristic, not a mathematical optimizer.** It fills suppliers in TOPSIS order. It does not minimise cost subject to constraints (no LP/MILP), so it can be beaten on cost by a true optimizer.
- **Disruption probabilities are assumed, not calibrated.** The coefficients (0.05 / 0.35 / 0.5) are modelling assumptions, not fitted to historical supplier-failure data.
- **Rainfall risk is a proxy.** It uses a 1-year baseline and a 7-day forecast and does not model monsoon seasonality, flooding, or logistics routes.
- **Monte Carlo tests a fixed plan.** It does not re-optimise within each iteration, and a disrupted supplier is modelled as all-or-nothing.
- **Single-product, single-period.** No multi-period planning, inventory, or price-volume discounts.
- **Location matching is string-based.** "Baddi" and "Baddi, HP" are treated as different locations for the correlation logic.
- **Browser-only:** no persistence or export of results.

## Roadmap

- [ ] Replace the greedy allocator with an LP/MILP formulation (cost minimisation with capacity, MOQ and quality constraints)
- [ ] Risk-aware objective (CVaR) in place of post-hoc Monte Carlo
- [ ] Calibrate disruption probabilities from historical OTIF/rejection data
- [ ] Export results to Excel/PDF
- [ ] Persist sessions (localStorage / JSON save-load)

## License

MIT — see `LICENSE`.

## Author

**Yash** — PGDM (Operations Management), Great Lakes Institute of Management, Chennai.

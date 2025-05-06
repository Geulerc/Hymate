# Hymate
# Hymate Energy Flow Optimization

This repository contains an hourly energy‐dispatch model (PV, battery, grid) built in Pyomo, plus post‐processing to generate results.

## Project Structure
├── tech_tasks/ ← input Excel file(s)

├── notebooks/ ← Jupyter notebook with models


├── Solutions/ ← output Excel & plot images

├── requirements.txt ← Python dependencies

├── .gitignore

└── README.md


## 📦 Setup

1. **Clone the repo**  
   ```bash
   git clone https://github.com/Geulerc/Hymate.git
   cd Hymate

2. **Create & activate a virtual environment**  
bash
python3 -m venv .venv
source .venv/bin/activate   # on macOS/Linux
.venv\Scripts\activate      # on Windows
Install dependencies

bash
pip install -r requirements.txt

 
Download CBC from COIN-OR.

Unpack and add the cbc executable to your PATH.

Verify availability:

bash
cbc --version


3. **Ensure tech_tasks/Energy Flow Optimization/test_data.xlsx is present (already in this repo).** 
Notebooks in notebooks/ refer to that file via a relative path.

# Model Overview
## Decision Variables

- pv2load, pv2batt, pv2grid: split PV output.

- grid2load, grid2batt: grid import for load & charging.

- batt2load, batt2grid: battery discharge for load & export.

- soc: state of charge (0 … B_CAP).

- b- uy_indicator: binary, enforces no simultaneous buy & sell.


## Key Constraints

- PV conservation: all PV must go to load, battery or grid.

- Load balance: supply exactly meets demand each interval.

- Grid limits: import & export ≤ G_CAP.

- Battery power: charge/discharge rates ≤ B_RATE.

- SoC dynamics: SoCₜ = SoCₜ₋₁ + η·charge − discharge.

- Mutual exclusivity: buy_indicator prevents simultaneous import & export. (Optional B)

## Objective Function

We minimize total cost over all time steps \(t \in T\), including grid import/export, LCOS, and battery capacity-expansion:

```latex
$$
\min \;\sum_{t\in T}\Bigl[
  p^{\mathrm{buy}}_t\,(grid2load_t + grid2batt_t)
  \;-\;
  p^{\mathrm{sell}}_t\,(pv2grid_t + batt2grid_t)
  \;+\;
  \mathrm{lcos}_t\,(batt2load_t + batt2grid_t)
\Bigr]
\;+\;
\mathrm{CAP\_COST}\,\delta_{\mathrm{cap}}
$$
# Public-Data-Backed AMXL-Style Heavy/Bulky Delivery Science Framework

This repository is an end-to-end, local-runnable operations science framework designed for AMXL-style heavy/bulky delivery analytics. It combines public/proxy datasets with a documented synthetic AMXL-style constraint overlay to demonstrate forecasting, routing, capacity planning, workforce scheduling, KPI monitoring, reliability gates, and stakeholder-ready reporting.

## What this project demonstrates

- Data engineering and analytics: canonical schema, schema validation, DuckDB or SQLite-compatible KPI marts, and explicit data-source usage logging.
- Public-data adapters: sample-first loaders plus adapter paths for Amazon Last Mile, Olist, and M5-style data.
- Forecasting: temporal demand forecasting baselines, ML forecasting, under-forecast and capacity-shortage metrics.
- Routing: nearest-neighbor baseline plus time-window-aware routing and automatic OR-Tools VRPTW usage when `ortools` is installed.
- Capacity planning: fixed capacity, forecast-driven capacity, reserve capacity, bias-corrected reserve, and Capacity Recall/Precision/F1.
- Workforce scheduling: synthetic two-person, installation, assembly, shift, overtime, and skill constraints.
- Network impact assessment: baseline network simulation, bottleneck detection, and policy intervention comparison.
- Monitoring and release gates: rolling KPI checks, worst-slice checks, false-greenlight detection, PASS/ITERATE/BLOCK decisions.
- Research specialty translation: operational-state timestep, operations-informed residual correction, two-stage KPI impact assessment, and ABC-style capacity coverage metrics.

## Claim boundary

This project does **not** use Amazon internal AMXL data and does **not** claim production deployment. See [`claim_boundary.md`](claim_boundary.md).

## Data sources expected

The project is sample-first and full-run-ready. If public data files are not available locally, the pipeline generates schema-compatible synthetic samples and marks them as fallback data in `data/processed/data_source_usage.csv`.

Expected raw-data folders:

```text
data/raw/amazon_last_mile/
data/raw/olist/
data/raw/m5/
```

Supported public/proxy dataset roles:

- Amazon Last Mile Routing Research Challenge: public route/stop/package-level routing benchmark. The loader looks for common challenge files such as `route_data.json`, `package_data.json`, and `actual_sequences.json`.
- Olist Brazilian E-Commerce: product weight/dimension, freight, delivery-delay, and review proxies. The loader expects `olist_products_dataset.csv`, `olist_orders_dataset.csv`, `olist_order_items_dataset.csv`, and optionally `olist_order_reviews_dataset.csv`.
- M5 Forecasting Accuracy: hierarchical demand forecasting proxy. The loader expects `sales_train_validation.csv` or `sales_train_evaluation.csv`, and optionally `calendar.csv`.

## Quick start

```bash
python -m venv .venv
source .venv/bin/activate  # Windows PowerShell: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
make all
pytest -q
```

If you are on Windows without `make`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
$env:PYTHONPATH="src"
python -m amxl_delivery_science.pipeline all
pytest -q
```

or run:

```powershell
.\scripts\run_pipeline.ps1
```

If `duckdb` is not installed, the mart builder falls back to SQLite while preserving the same CSV outputs.

## Main commands

```bash
make data        # ingest or generate sample data, canonicalize, validate
make marts       # build SQL KPI marts
make baseline    # train/evaluate forecasting, service, routing, capacity, workforce baselines
make evaluate    # network what-if, monitoring, release gate
make report      # generate markdown reports and executive readout
make all         # run the full local pipeline
make clean       # remove generated local artifacts and recreate empty artifact folders
```

## Key outputs

```text
outputs/forecast_metrics.csv
outputs/service_time_metrics.csv
outputs/routing_metrics.csv
outputs/routing_optimization_metrics.csv
outputs/routing_method_comparison.csv
outputs/capacity_metrics.csv
outputs/capacity_policy_sweep.csv
outputs/workforce_metrics.csv
outputs/network_impact_summary.csv
outputs/release_decision.json

reports/public_dataset_summary.md
reports/forecast_report.md
reports/routing_report.md
reports/capacity_report.md
reports/workforce_report.md
reports/network_what_if_report.md
reports/monitoring_report.md
reports/ablation_report.md
reports/executive_readout.md
reports/model_card.md
reports/release_packet.md
```

## Repository layout

```text
configs/                 YAML configs for data, modeling, routing, gates
warehouse/               SQL marts for route, demand, capacity, workforce, release KPIs
src/amxl_delivery_science/
  data/                  ingestion, canonical schema, overlay, validation
  forecasting/           baselines, operational-state timestep, residual correction
  service_burden/        delivery-burden and delay-risk prediction
  routing/               nearest-neighbor, time-window-aware routing, optional OR-Tools VRPTW
  capacity/              capacity planning and capacity reliability metrics
  workforce/             workforce scheduling and skill constraints
  network_impact/        what-if and two-stage KPI impact assessment
  monitoring/            drift, KPI, release gates
  reporting/             report builders
reports/                 generated markdown reports, gitignored by default
outputs/                 generated metrics and release artifacts, gitignored by default
examples/sample_outputs/ tracked sample outputs for quick review
examples/sample_reports/ tracked sample reports for quick review
tests/unit/              fast unit tests
tests/integration/       one-session full pipeline smoke and contract tests
```

## Resume-safe project description

Built a public-data-backed AMXL-style heavy/bulky delivery science framework combining public last-mile route data, e-commerce freight/product attributes, demand forecasting proxies, and synthetic service constraints for demand forecasting, route sequencing, capacity planning, workforce scheduling, KPI monitoring, and release-gated operational reporting.


## Repository hygiene

Generated `data/processed/`, `outputs/`, and `reports/` artifacts are ignored by Git. Representative sample artifacts are stored under `examples/sample_outputs/` and `examples/sample_reports/` after validation so reviewers can inspect the expected outputs without confusing them with freshly generated local results.

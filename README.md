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

## Repository hygiene

Generated `data/processed/`, `outputs/`, and `reports/` artifacts are ignored by Git. Representative sample artifacts are stored under `examples/sample_outputs/` and `examples/sample_reports/` after validation so reviewers can inspect the expected outputs without confusing them with freshly generated local results.

<!-- FINAL_VALIDATED_MODEL_SELECTION_START -->
## Model selection results

This project is an offline, public-data-backed heavy/bulky delivery science framework using public/proxy delivery, e-commerce, and demand data. The final validated result is a model-selection story, not a claim that every advanced model wins on every metric.

### Final model roles

| Component | Final role | Why |
|---|---|---|
| GPU PyTorch route-transition ranker | Main routing model | Clearly improves next-stop ranking over nearest-distance routing |
| Tree quantile residual capacity guardrail | Main balanced capacity model | Best recall/F1/over-reserve tradeoff |
| Tree quantile + ABC safe-deflation | Specialty extension | Slightly improves F1 and over-reserve while preserving 0.90+ recall |
| GPU MLP quantile capacity with calibration | Advanced ML benchmark | Useful benchmark, but not the best balanced capacity policy |
| Classifier reserve and early ABC experiments | Superseded experiments | Replaced by quantile residual capacity planning |

### Key validated results

| Area | Baseline | Final / selected method | Result |
|---|---|---|---|
| Route ranking | Nearest-distance baseline | GPU PyTorch transition ranker | Top-1 accuracy improved from 0.605 to 0.687 |
| Capacity planning | Uniform reserve | Tree quantile residual guardrail | Recall improved from 0.882 to 0.910 |
| Capacity F1 | Uniform reserve | Tree quantile residual guardrail | F1 improved from 0.626 to 0.664 |
| Capacity over-reserve | Uniform reserve | Tree quantile + ABC safe-deflation | Mean over-reserve reduced from 16.54 to 12.24 |
| Specialty extension | Tree quantile guardrail | ABC safe-deflation | Preserved 0.909 recall while slightly improving F1 and reducing over-reserve |
| Advanced ML benchmark | Tree quantile guardrail | Calibrated GPU MLP quantile | MLP was competitive but did not beat tree quantile as the best balanced policy |

### Capacity model comparison

| Policy | Recall | Precision | F1 | Shortage rate | Mean over-reserve | Role |
|---|---:|---:|---:|---:|---:|---|
| Uniform reserve | 0.8822 | 0.6177 | 0.6257 | 0.2931 | 16.5435 | Baseline |
| Tree quantile guardrail | 0.9103 | 0.6303 | 0.6645 | 0.2538 | 12.4060 | Main capacity model |
| Tree ABC safe-deflation | 0.9093 | 0.6318 | 0.6649 | 0.2562 | 12.2376 | Specialty extension |
| Calibrated MLP guardrail | 0.9041 | 0.6340 | 0.6604 | 0.2650 | 15.2125 | Advanced ML benchmark |

### Claim boundary

This repository uses offline public/proxy data and local validation. It should be described as a public-data-backed delivery science framework, not as a production Amazon AMXL deployment or an internal-data result.

Safe resume wording:

> Built and locally validated a public-data-backed heavy/bulky delivery science framework using Amazon Last Mile, Olist, and M5 data; trained GPU PyTorch route-ranking and quantile capacity models that improved next-stop Top-1 accuracy from 0.605 to 0.687, raised tree-quantile capacity recall from 0.882 to 0.910, improved F1 from 0.626 to 0.664, and reduced mean over-reserve to 12.24 with an ABC-style safe-deflation extension under offline validation.

Specialty extension wording:

> Adapted an ABC-style safe-deflation layer to quantile capacity planning, preserving 0.909 recall while reducing over-reserve from 12.41 to 12.24 and slightly improving F1 over the tree-quantile guardrail under offline public-data validation.
<!-- FINAL_VALIDATED_MODEL_SELECTION_END -->


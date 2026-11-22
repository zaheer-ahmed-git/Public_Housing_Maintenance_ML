# Public Housing Maintenance ML

CAIP capstone for **property-level maintenance cost estimation** and high-cost
prioritization. The target is a WAPDA staff-colony house over the next
12 months. Because linked operational WAPDA extracts are not available, the
implemented path uses **audited public U.S. housing proxies** (RHFS and AHS),
not validated WAPDA outcomes.

## What this repo does

- Build lineage-preserving public corpora and freeze train/validation/test splits
- Compare transparent baselines with classical regressors (including XGBoost)
- Evaluate cost error (MAE/RMSE) and high-cost triage metrics
- Expose a minimal Streamlit proof of concept on audited local artifacts only

## Current status

| Slice | Content | Role |
|---|---|---|
| `public-corpus-v0.1.0-rhfs` | 2024 RHFS; 4,425 rentals; 1,488 usable annual cost labels | Cross-sectional annual-cost proxy |
| `public-corpus-v0.2.0-ahs` | AHS 2015–2023; 36,623 units; 86,125 adjacent-wave pairs | Longitudinal future-cost proxy |

Under `ahs-selection-policy-v1`, **type median** is the primary cost estimator and
**prior cost** is the high-cost reference. Among fitted squared-error models,
validation-tuned XGBoost is the strongest balanced system on the primary test.
No run claims a final WAPDA/WASC-validated model.

Private WASC seed material (101 units) is framing evidence only. Do not treat
ledger debits as property-level labels, and do not present public-proxy labels as
observed WAPDA outcomes.

## Prediction contract

One property at one historical cutoff (recommended: 30 June). Label window:
1 July–30 June. Eligible routine/corrective/emergency maintenance only; major
renovation and reconstruction stay separate. Features must be available by the
cutoff; learned preprocessing is fitted inside the training fold only.

## Repository layout

```text
configs/                 sources, schemas, splits, experiments
src/caip_maintenance/    data, features, modeling, evaluation, app
scripts/                 thin CLI and Streamlit entry points
tests/                   pipeline, modeling, and inference checks
Documentation/           dataset cards, gates, experiment write-ups
reports/submission/      final report PDF, code annex, source file
```

Raw/processed row-level data, secrets, and trained weights stay local (gitignored).

## Quick start

Python 3.11+. From the repository root:

```bash
# data / corpus pipeline
PYTHONPATH=src python -m caip_maintenance.data register-sources
PYTHONPATH=src python -m caip_maintenance.data harmonize --release public-corpus-v0.2.0-ahs
PYTHONPATH=src python -m caip_maintenance.data assign-splits --release public-corpus-v0.2.0-ahs --split ahs-grouped-temporal-v1
PYTHONPATH=src python -m caip_maintenance.data preprocess-ahs --release public-corpus-v0.2.0-ahs --split ahs-grouped-temporal-v1 --preprocessor ahs-training-fold-v1

# modeling environment
python -m venv .venv
.venv/Scripts/python -m pip install -r requirements-modeling.txt
PYTHONPATH=src .venv/Scripts/python -m caip_maintenance.data train-ahs-experiment --release public-corpus-v0.2.0-ahs --split ahs-grouped-temporal-v1 --preprocessor ahs-training-fold-v1 --experiment ahs-baselines-models-v1

# tests and demo app
.venv/Scripts/python -m unittest discover -s tests -v
.venv/Scripts/python -m pip install -r requirements-app.txt
PYTHONPATH=src .venv/Scripts/streamlit run scripts/streamlit_app.py
```

On macOS/Linux, use `.venv/bin/python` instead of `.venv/Scripts/python`.
Equivalent corpus commands: `python scripts/build_public_corpus.py COMMAND`.

## Key documents

- `AGENTS.md` — project rules for contributors and coding agents
- `ARCHITECTURE.md` — system boundaries and module contracts
- `Documentation/DatasetPolicy.md` — approved data strategy
- `Documentation/AHSPublicCorpusDatasetCard.md` — AHS release card
- `Documentation/AHSSelectionPolicy.md` — reporting / promotion policy
- `reports/CAIPMethodsResults.md` — methods and results draft
- `reports/submission/` — CAIP PDF report, annex, and source package

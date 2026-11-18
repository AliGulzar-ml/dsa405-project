# Temperature Near Cushing and WTI Crude-Oil Price Changes

**DSA 405-002, Fall 2026 · Ali Gulzar (agulzar2)**

Cushing, Oklahoma, is the delivery hub for the West Texas Intermediate (WTI) crude-oil benchmark. This project asks whether **daily temperature near Cushing is associated with daily percentage changes in the Cushing WTI spot price** from 2015 through 2025. It joins NOAA daily weather from the Stillwater 2 W station (about 20 miles from Cushing) to the U.S. EIA daily WTI spot price by date, verifies the join, and then tests the relationship.

**Finding:** across 2,739 trading days, local temperature carries no detectable information about the same-day or next-week WTI return. The Spearman correlation is ρ = −0.005 (p = 0.78) for temperature and ρ = −0.017 (p = 0.38) for the seasonal temperature anomaly. The HAC regression slope is −0.001 percentage points per °F, and its 95% CI includes zero. The null result holds for the coldest and hottest 5% of days, for lags of 0–5 trading days, and under seven sensitivity checks. With this sample size, any linear association larger than |ρ| ≈ 0.054 would have been detected. The full write-up is in [`Project/DSA405_002_FA26_P4_Report_agulzar2.md`](Project/DSA405_002_FA26_P4_Report_agulzar2.md).

## Data Sources

| Source | Publisher | What is used | Access | License / terms |
|---|---|---|---|---|
| [Cushing, OK WTI Spot Price FOB, daily (`RWTC`)](https://www.eia.gov/dnav/pet/hist/RWTCD.htm) | U.S. Energy Information Administration (EIA) | `period`, `value` (USD per barrel), 2015–2025 | [EIA Open Data API v2](https://www.eia.gov/opendata/) with the public `DEMO_KEY`; the response is cached in `Project/data/raw/` | U.S. government data, public domain; use governed by the [EIA API Terms of Service](https://www.eia.gov/opendata/terms-of-service.php), which permit analysis and require crediting EIA |
| [GHCN-Daily, Version 3, station `USW00053926` (Stillwater 2 W, OK)](https://www.ncei.noaa.gov/access/metadata/landing-page/bin/iso?id=gov.noaa.ncdc:C00861) | NOAA National Centers for Environmental Information (NCEI) | `DATE`, `TMAX`, `TMIN` (°F), 2015–2025 | [NCEI Daily Summaries web service](https://www.ncei.noaa.gov/access/services/data/v1?dataset=daily-summaries&stations=USW00053926&startDate=2015-01-01&endDate=2025-12-31&format=csv&units=standard&includeAttributes=false), saved unmodified in `Project/data/raw/` | U.S. government data, public domain; NOAA asks users to cite Menne et al. (2012), doi:10.7289/V5D21VHZ |

Retrieval dates and SHA-256 checksums are in [`Project/data/raw/SOURCES.md`](Project/data/raw/SOURCES.md).

## How to Run

**Colab:** open any project notebook with its "Open in Colab" badge and choose *Runtime → Run all*. The first cell clones this repository and moves into `Project/`.

**Locally:**

```bash
git clone https://github.com/AliGulzar-ml/dsa405-project.git
cd dsa405-project
python -m pip install -r requirements.txt
cd Project
jupyter nbconvert --to notebook --execute --inplace DSA405_002_FA26_P3_agulzar2.ipynb   # raw data -> joined table
jupyter nbconvert --to notebook --execute --inplace DSA405_002_FA26_P4_agulzar2.ipynb   # joined table -> findings and figures
```

You can also open the notebooks in Jupyter and run them in that order after restarting the kernel.

1. **P3** reads the unmodified NOAA CSV and the cached EIA response from `data/raw/`, rebuilds the cleaned weather table, joins the two sources by date, runs the verification suite, and writes `data/processed/temperature_wti_join_2015_2025.csv`. If the EIA cache is missing, P3 fetches it once through the API (rate-limited and validated) and saves it.
2. **P4** re-verifies the joined table (row count, unique dates, no missing cells), runs the analysis, and saves four figures to `figures/`. It uses a fixed random seed (405), so its results are the same on every run.

No API key, password, or token is needed. The EIA calls use the public `DEMO_KEY`.

## Repository Structure

```
dsa405-project/
├── README.md                     this file
├── requirements.txt              Python packages for the project notebooks
├── Labs/                         weekly lab notebooks (not part of the project pipeline)
└── Project/
    ├── DSA405_002_FA26_P1_agulzar2.ipynb    P1: question, sources, access evidence, ethics, tier
    ├── DSA405_002_FA26_P2_agulzar.ipynb     P2: NOAA audit, data dictionary, cleaning log, provenance
    ├── DSA405_002_FA26_P3_agulzar2.ipynb    P3: EIA acquisition and cache, join design, verification suite
    ├── DSA405_002_FA26_P4_agulzar2.ipynb    P4: analysis, figures, findings
    ├── DSA405_002_FA26_P4_Report_agulzar2.md  P4 written report and self-scored rubric
    ├── data/
    │   ├── raw/
    │   │   ├── noaa_stillwater_daily_2015_2025.csv   unmodified NOAA download (never overwritten)
    │   │   ├── eia_wti_page_00000.json               unmodified cached EIA API response
    │   │   ├── eia_wti_page_00000_headers.json       HTTP status and rate-limit headers for that response
    │   │   └── SOURCES.md                            source URLs, retrieval dates, checksums
    │   └── processed/
    │       └── temperature_wti_join_2015_2025.csv    2,741-row joined, complete-case table (P3 output)
    └── figures/                  PNG figures written by the P4 notebook
```

## Requirements

Python 3.10+ with pandas, NumPy, SciPy, statsmodels, and matplotlib (see `requirements.txt`). P1–P3 need only pandas and NumPy plus the standard library. Google Colab already has every package installed.

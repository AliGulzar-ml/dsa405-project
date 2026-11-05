
# Temperature Near Cushing and WTI Crude-Oil Price Changes

This DSA 405 project asks whether daily temperature near Cushing, Oklahoma, is associated with daily percentage changes in the Cushing WTI crude-oil spot price from 2015 through 2025.

P1 provides an in-depth overview of the project, identifies the data sources, demonstrates that the data are accessible, and discusses the ethical and legal considerations associated with using the sources.

P2 audits and cleans the NOAA/NCEI Global Historical Climatology Network–Daily data from the Stillwater 2 W station (`USW00053926`). The unmodified NOAA download is preserved in `data/raw/`.

P3 retrieves the EIA Cushing WTI spot-price series (`RWTC`) through the documented EIA API, caches the unmodified response, and joins it to the cleaned NOAA weather data by date. The notebook documents the join design, handles unmatched dates, and uses assertions to verify that the integration does not create duplicate or missing oil-price observations.

## Files

- `DSA405_002_FA26_P1_agulzar2.ipynb`: Project scope, research question, source-access evidence, ethics, guardrails, and tier declaration.
- `DSA405_002_FA26_P2_agulzar2.ipynb`: NOAA data audit, data dictionary, cleaning log, accounting, provenance brief, and self-score.
- `DSA405_002_FA26_P3_agulzar2.ipynb`: EIA acquisition, caching, failure handling, join design, integration verification, and self-score.
- `data/raw/noaa_stillwater_daily_2015_2025.csv`: Unmodified NOAA Daily Summaries download.
- `data/raw/eia_wti_page_00000.json`: Cached, unmodified EIA API response.
- `data/raw/eia_wti_page_00000_headers.json`: HTTP status and rate-limit headers recorded with the EIA response.
- `data/processed/temperature_wti_join_2015_2025.csv`: Analysis-ready table created by joining the EIA and NOAA sources.
- `data/raw/SOURCES.md`: Source URLs, retrieval dates, coverage, and file checksums.

## How to Run

1. Open the repository in Google Colab or Jupyter and set the repository root as the working directory.
2. Restart the Python kernel before running a notebook.
3. Run all cells in the selected notebook from top to bottom.
4. To reproduce P3, run `DSA405_002_FA26_P3_agulzar2.ipynb`. It reads the preserved NOAA file from `data/raw/` and reconstructs the cleaned weather table.
5. If an EIA cache is not already present, P3 retrieves the WTI data through the EIA API, waits between requests, checks the HTTP status and rate-limit headers, validates the response, and saves the unmodified response in `data/raw/`. Later runs reuse the cache.
6. P3 saves the final complete-case table to `data/processed/temperature_wti_join_2015_2025.csv`.

## Requirements

The notebooks require:

- Python 3
- pandas
- NumPy

P3 also uses Python standard-library modules for API requests, timing, JSON handling, and file paths. No additional third-party package is required. The preserved NOAA file is never overwritten.

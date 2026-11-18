# Raw Data Sources

## NOAA/NCEI Daily Summaries

- **File:** `noaa_stillwater_daily_2015_2025.csv`
- **Publisher:** NOAA National Centers for Environmental Information (NCEI)
- **Dataset:** Global Historical Climatology Network–Daily, Version 3
- **Station:** Stillwater 2 W, Oklahoma (`USW00053926`)
- **Coverage requested:** 2015-01-01 through 2025-12-31
- **Units requested:** U.S. standard units
- **Download URL:** <https://www.ncei.noaa.gov/access/services/data/v1?dataset=daily-summaries&stations=USW00053926&startDate=2015-01-01&endDate=2025-12-31&format=csv&units=standard&includeAttributes=false>
- **Downloaded:** 2026-09-28
- **SHA-256:** `73490f5e091b16c1d65446b2c0d89c49c73bccd6b3307ece3393e02603148bbb`

The CSV is preserved exactly as downloaded. Project code may read this file but must not modify or overwrite it.

## EIA Cushing WTI Spot Price

- **Files:** `eia_wti_page_00000.json` (unmodified API response) and `eia_wti_page_00000_headers.json` (HTTP status and rate-limit headers)
- **Publisher:** U.S. Energy Information Administration (EIA)
- **Series:** Cushing, OK WTI Spot Price FOB, daily (`RWTC`), $/BBL
- **Coverage requested:** 2015-01-01 through 2025-12-31 (2,760 trading-day rows)
- **API endpoint:** <https://api.eia.gov/v2/petroleum/pri/spt/data/> with `frequency=daily`, `facets[series][]=RWTC`, and the public `DEMO_KEY`
- **Retrieved:** 2026-11-18
- **SHA-256:** `12910ee679c8809904f1e7bbc552283a3796483020354745fcf5c919932272ec`

The response is saved exactly as returned by the API. P3 reads this cache on later runs instead of calling the API again.

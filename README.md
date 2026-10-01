# Temperature Near Cushing and WTI Crude-Oil Price Changes

This DSA 405 project asks whether daily temperature near Cushing, Oklahoma, is associated with daily percentage changes in the Cushing WTI crude-oil spot price from 2015 through 2025.

P1 gives a more in depth overview of the project, along with the sources used for this project and the ethics and usability behind the sources. 

P2 audits and cleans one source: NOAA/NCEI Global Historical Climatology Network–Daily data from the Stillwater 2 W station (`USW00053926`). The unmodified NOAA download is stored in `data/raw/`. The EIA oil-price source and the date join will be added in P3.

## Files
- `DSA405_002_FA26_P1_agulzar2`: P1: The scope of the project, along with proof of being able to obtain the data
- `DSA405_002_FA26_P2_agulzar2`: P2 audit, data dictionary, cleaning log, accounting, provenance brief, and self-score
- `data/raw/noaa_stillwater_daily_2015_2025.csv`: unmodified NOAA Daily Summaries download
- `data/raw/SOURCES.md`: source URL, download date, and file checksum

## How to run

1. Open the notebook in Google Colab or Jupyter.
2. Run all cells from top to bottom on a restarted Python kernel.
3. The notebook reads the same date-and-station request from NOAA's public web service with `pandas.read_csv()`. The unmodified snapshot remains preserved at `data/raw/noaa_stillwater_daily_2015_2025.csv` for auditing and recovery.
4. The notebook requires Python 3, pandas, and NumPy. It does not write to `data/raw/`.

Before submission, replace the Unity ID placeholder in the notebook and filename, upload these files to the course project repository, and submit both the notebook and repository link to Moodle.

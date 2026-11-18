# P4: Project Synthesis Report

**Temperature near Cushing and WTI crude-oil price changes**

**Name:** Ali Gulzar · **Unity ID:** agulzar2 · **Course:** DSA 405-002, Fall 2026
**Repository:** <https://github.com/AliGulzar-ml/dsa405-project>
**Analysis notebook:** `Project/DSA405_002_FA26_P4_agulzar2.ipynb`. Every number in this report is printed by that notebook or by the P3 notebook. Section numbers like "nb §6" refer to sections of the P4 notebook.

---

## 1. Data Motivation

**Question:** Is there an association between daily average temperature near Cushing, Oklahoma, and daily percentage changes in the Cushing WTI crude-oil spot price from 2015 through 2025?

**Why this question.** Weather data appear in energy-market forecasting. Temperature drives heating and cooling demand, and deep cold can freeze wellheads and pipeline equipment. Cushing is the physical delivery point for WTI, the main U.S. oil benchmark, so it is the place where local weather and the benchmark price meet. If temperature near the hub moves with the price, it could be one input to a broader oil model. If it does not, that is also useful to know: local weather by itself is not a trading signal.

**How the question changed once I had the real data.** The question's wording held, but three things about it changed:

1. **"Near Cushing" became "20 miles from Cushing."** The NOAA station in Cushing (`USC00342318`) stopped reporting before 2015. I switched to Stillwater 2 W (`USW00053926`), the nearest station with complete 2015–2025 coverage (P1, P2).
2. **"Daily percentage change" turned out not to be defined on every day.** On April 20, 2020, WTI closed at −$36.98. A percentage change into or out of a negative price is meaningless, and the next day's computed return (−124%) even has the wrong sign. The question now applies to trading days with a positive price on both ends (nb §3).
3. **"Is there an association" became "how small would an association have to be to go unseen?"** Once the simple correlation came back near zero, the more useful question was whether the data could have detected a small effect at all (nb §11), and whether an effect hides in extreme weather or arrives with a delay (nb §7–8).

## 2. Sources, Provenance & Terms

| | EIA WTI spot price | NOAA GHCN-Daily |
|---|---|---|
| **Who produced it** | U.S. Energy Information Administration, the statistical agency of the U.S. Department of Energy | NOAA National Centers for Environmental Information (NCEI) |
| **How it was collected** | EIA republishes daily Cushing WTI spot prices (FOB) that it obtains from a commercial price-reporting service (EIA credits Thomson Reuters). It is a price assessment from market trading, not a physical measurement. | Automated surface observing equipment at Stillwater 2 W records daily max and min air temperature. NCEI runs automated quality-control checks and archives the results in GHCN-Daily Version 3. |
| **What it covers** | Series `RWTC`, USD per barrel, trading days only (no weekends or market holidays). 2,760 observations from 2015-01-02 to 2025-12-31. | One station, every calendar day. 4,013 recorded dates from 2015-01-01 to 2025-12-31. Five calendar dates are absent and 22 dates are missing a temperature. |
| **License / terms** | U.S. government work, public domain. The [EIA API Terms of Service](https://www.eia.gov/opendata/terms-of-service.php) permit using the API to "search, display, analyze, retrieve, view" EIA data and ask users to credit EIA. The project uses the public `DEMO_KEY` within its rate limits for a single cached pull, which fits these terms. | U.S. government work, public domain. NOAA asks users to cite Menne et al. (2012), *GHCN-Daily, Version 3*, doi:10.7289/V5D21VHZ, and disclaims any warranty of accuracy or completeness. |
| **How I accessed it** | Documented EIA Open Data API v2 (`/v2/petroleum/pri/spt/data/`). The request uses an honest User-Agent, a 1-second delay, and a timeout, and checks the status code and rate-limit headers. The unmodified JSON and its headers are cached in `data/raw/` (P3). | NCEI Daily Summaries web service, CSV, no key (P1/P2). The unmodified file is saved as `data/raw/noaa_stillwater_daily_2015_2025.csv` and never overwritten. |

Retrieval dates and SHA-256 checksums for both raw files are recorded in `Project/data/raw/SOURCES.md`.

## 3. Data Dictionary & Metadata (final data)

**File:** `Project/data/processed/temperature_wti_join_2015_2025.csv`. One row per WTI trading day that has a defined return and a complete temperature record: **2,741 rows × 8 columns**, 2015-01-05 to 2025-12-31. The P4 analysis sample drops 2 more rows (April 20–21, 2020; see §4), leaving **2,739 rows**.

| Variable | Type | Units | Source | Valid range (validation bound) | Observed range | Missing | Notes |
|---|---|---|---|---|---|---|---|
| `date` | date | calendar day | both (join key) | 2015-01-01 to 2025-12-31 | 2015-01-05 to 2025-12-31 | 0 | Unique. WTI trading days only. |
| `wti_price_usd_per_barrel` | float | USD per barrel | EIA `value` | −100 to 200 | −36.98 to 123.64 | 0 | Negative only on 2020-04-20. |
| `daily_return_pct` | float | percent | derived | finite | −301.97 to 53.09 | 0 | `pct_change` from the previous EIA trading day, computed before the join. **Not meaningful on 2020-04-20 and 2020-04-21** (negative base or end price). P4 excludes those two rows. Without them, the range is −24.5 to 53.1. |
| `station_id` | string | identifier | NOAA `STATION` | `USW00053926` | `USW00053926` | 0 | Kept as text. |
| `temp_min_f` | float | °F | NOAA `TMIN` | −100 to 140 | −15 to 82 | 0 | |
| `temp_max_f` | float | °F | NOAA `TMAX` | −100 to 140 | 13 to 110 | 0 | Always ≥ `temp_min_f`. |
| `temp_avg_f` | float | °F | derived | −100 to 140 | −1 to 94 | 0 | (`temp_min_f` + `temp_max_f`) / 2. This is not NOAA's `TAVG`, which was empty. |
| `temp_range_f` | float | °F difference | derived | 0 to 240 | 2 to 55 | 0 | `temp_max_f` − `temp_min_f`. |

**Variables created in P4 (in memory only, not saved):** `month`; `temp_anomaly_f`, which is `temp_avg_f` minus the 2015–2025 mean for that calendar month; `abs_return_pct`; `temp_band`, either cold extreme (≤ 31.5°F, the 5th percentile, 142 days), typical (2,447 days), or hot extreme (≥ 86.0°F, the 95th percentile, 150 days); and `return_defined`.

**Metadata:** the joined file's SHA-256 is printed in nb §2 (`054c3827…19c43`). The weather comes from one station, so all temperatures describe one point about 20 miles northwest of Cushing, not a regional average.

## 4. Data Cleaning Narrative

The path from raw to final data has four stages. Each one keeps the raw files untouched and makes every exclusion visible.

1. **NOAA audit and cleaning (P2).** The raw CSV has 151 columns. 146 are empty for this station, and `PRCP` is outside the question, so I kept `STATION`, `DATE`, `TMAX`, `TMIN`, renamed them, and derived `temp_avg_f` and `temp_range_f`. 22 dates were missing at least one temperature. I considered imputing them from neighboring days and decided against it, because that would create observations NOAA never made. Instead, I kept the rows with a `temperature_complete` flag so later steps could filter only what a specific calculation needs. Every value was checked against plausibility bounds, and no value was out of range. The same rules are re-run inside P3 so that notebook stands alone.
2. **EIA acquisition and preparation (P3).** EIA returned 2,760 rows in one page. I parsed dates and prices strictly (`errors="raise"`), confirmed that every row is series `RWTC`, and computed `daily_return_pct` on the full EIA series *before* joining. This choice matters. If returns were computed after the join, a gap left by a missing weather day would turn a two-day price change into what looks like a one-day change.
3. **Join and complete-case filter (P3).** A one-to-one left join from oil to weather kept all 2,760 trading days (details in §5). From those, I removed 3 days with no NOAA row, 15 days with an incomplete temperature, and the first day of the series (no prior price for a return). That leaves 2,741 rows.
4. **Undefined returns (P4).** The P4 analysis found that two rows passed every P3 check but carry meaningless returns. On 2020-04-20 the price fell from about $18 to −$36.98, giving −302%. On 2020-04-21 it rose from −$36.98 to $8.91, giving −124%, so the sign is wrong. I considered three options: keeping them, recomputing a "return" in dollars, or excluding them. I chose to exclude them from the main analysis, because a percentage change across zero has no meaning. A sensitivity row puts them back (nb §9), and the conclusion does not change (ρ = −0.017 either way).

**Accounting:** 4,013 NOAA dates and 2,760 EIA dates → 2,760 joined rows → 2,741 complete-case rows → 2,739 analysis rows.

## 5. Integration & Verification

**How the sources were joined.** The join key is `date`. The left table (EIA) has one row per trading day, and the right table (NOAA) has one row per calendar day. The design was written down before the merge (P3 §4): a **left join from oil to weather** with `validate="one_to_one"` and `indicator=True`, expecting exactly 2,760 rows.

- My P1 plan proposed an *inner* join. I changed it in P3 because an inner join would silently drop oil dates that have no weather, the "wreck" failure. The left join made those dates visible: 2020-12-10, 2022-10-17, and 2024-10-14 have no NOAA row at all.
- An outer join was rejected because it would add 1,256 weather-only rows (4,013 NOAA dates minus the 2,757 matched dates), almost all of them weekends and market holidays with no price change.

**How I verified the result.** The P3 verification suite has nine assertions, and each states the failure it protects against: unique keys on both sides (against fan-out), 2,760 rows preserved, joined dates unique, every oil date present, exactly 3 unmatched rows, prices and temperatures within bounds, and finite returns. To show the suite catches a real mistake, P3 also runs it on a copy of the joined table with one date row duplicated, and the uniqueness assertion fails with a printed `AssertionError`. P4 re-checks the processed file before using it (nb §2): 8 expected columns, 2,741 rows, unique and sorted dates, no missing cells, one station, `temp_avg_f` equal to the mean of min and max, and max ≥ min.

**What verification missed, and what that taught me.** All nine P3 assertions passed on the April 2020 rows. They checked that returns are *finite* and that prices are *within bounds*, and a −124% return on a day the price rose $46 satisfies both. Assertions only catch the failures you write them for. The P4 check that caught it reconstructs each day's previous price from the stored return and requires both prices to be positive (nb §3). This is a check on *meaning*, not on *shape*.

## 6. Findings

**What the data support:**

1. **No detectable same-day association.** Over 2,739 trading days, the Spearman correlation between temperature and WTI return is ρ = −0.005 (p = 0.78). For the temperature anomaly, it is ρ = −0.017 (p = 0.38). The Pearson values are similar (nb §6). A regression with Newey–West standard errors gives −0.0013 percentage points per °F (95% CI −0.0064 to +0.0038, p = 0.62). In practical terms, a 30°F swing changes the expected return by −0.04 points, while a typical day's move is 1.35 points. Within each calendar month (month fixed effects), the anomaly is also unrelated to the return (p = 0.58).
2. **The null is robust.** A permutation test that circularly shifts the temperature series, preserving each series' own clustering, gives p = 0.37 (nb §6). The anomaly correlation stays between −0.015 and −0.043, and is never significant, when the undefined April 2020 rows are kept, when March–June 2020 is removed, before COVID, after COVID, in winter only, and in summer only (nb §9).
3. **The test had the power to see a small effect.** With n = 2,739, a correlation of |ρ| ≥ 0.054 would be detected 80% of the time (nb §11). Whatever same-day linear association exists explains less than about 0.3% of the variation in daily returns.
4. **No hidden effect at the extremes or with a delay.** The coldest 5% of days (≤ 31.5°F) and the hottest 5% (≥ 86°F) show no reliable difference from typical days in either the direction or the size of the move. The smallest Bonferroni-corrected p-value is 0.16 (nb §7, Figure 3). No lag from 0 to 5 trading days is significant after correction (nb §8, Figure 4).

**What is suspected but not shown:**

- Cold-extreme days averaged +0.35% compared with +0.06% on typical days (medians +0.42% and +0.07%). The uncorrected Mann–Whitney p is 0.04, but after correcting for four comparisons it is 0.16, and it rests on 142 days. This could be a small real cold-weather effect or noise. This data set cannot tell which.
- Lag 3 has ρ = −0.038 (raw p = 0.045). It does not survive correction for testing six lags, and it has no obvious mechanism, so I treat it as chance.
- One of ten temperature deciles (around 60°F) has a mean-return interval that excludes zero (Figure 2). With ten intervals, about one is expected to do this by chance, and it is a mild-weather decile, not an extreme one.
- **Winter Storm Uri (February 2021)** is the strongest test of the motivating story. WTI rose 7.6% from February 5 to its storm-week peak (nb §10). However, it had already risen 19.7% in the month before the storm (January 4 to February 5, nb §10). Context from outside this data set, not tested here, is that Uri also forced oil production shut-ins in *Texas*. The storm is consistent with cold mattering through supply, but one event cannot separate local temperature from everything else that happened at the same time.

**Conclusion:** temperature at a single station near Cushing gives no usable information about daily WTI price changes in 2015–2025. WTI is a global benchmark priced by worldwide supply, demand, and storage. The data support "no meaningful daily linear association." They do not support "weather never matters to oil."

## 7. What Is Missing

| Who or what is excluded | Why | How it affects the conclusions |
|---|---|---|
| **Weekends and market holidays**, including the coldest day in the data set | WTI has no price on non-trading days, so the left join keeps only trading days. Of 4,013 recorded weather days, 1,274 are not in the analysis, and 1,146 of those are Saturdays and Sundays (nb §12). | This matters most for the extremes. Only **5 of the 10 coldest** calendar days are in the analysis, compared with 8 of the 10 hottest and 68% of all days (nb §12). The coldest day, **2021-02-15 (−3°F average), was Presidents' Day**, and Feb 14 was a Sunday. In January 2024, the cold snap on Jan 14 (0°F) and Jan 15 (3°F) also fell on a Sunday and on Martin Luther King Jr. Day. The market's reaction can show up only on the next trading day, so the same-day extreme-cold tests miss several of the most severe events. The "cold extreme" band is therefore milder than the weather record. |
| **Everywhere except one point in Oklahoma** | The project uses a single station, Stillwater 2 W, 20 miles from Cushing. | Oil supply shocks from weather mostly happen in production regions (the Permian Basin, Gulf Coast refineries, the Gulf of Mexico during hurricanes), not at the storage hub. A null result for Stillwater says nothing about weather in Texas or the Gulf. This is the biggest limit on how far the conclusion reaches. |
| **3 trading days with no NOAA record** (2020-12-10, 2022-10-17, 2024-10-14) and **15 with an incomplete temperature** | The station did not report. | These are 18 of 2,760 days (0.7%). They are spread across years and seasons, so they are unlikely to change the result. However, if a station fails *because* of extreme weather (for example, icing), the missing days could be the extreme ones. |
| **April 20–21, 2020** | The returns are undefined (negative price). | This removes the single most extreme oil event in the period. Results with those rows included are the same (nb §9). Because it falls in April, a mild month, it is unlikely to carry a temperature signal. |
| **Intraday timing** | Both sources are daily. | Weather happens over the day, and the spot price is an end-of-day assessment. An effect that markets price in *before* the weather arrives, using forecasts, would not appear as a same-day or lagged association with *observed* temperature. |
| **Years outside 2015–2025** | Project scope. | The conclusion describes this decade's market structure, including the shale era, the 2015–16 collapse, COVID, and 2022. It may not hold for other periods. |
| **The Cushing station itself** (`USC00342318`) | It stopped reporting before 2015. | Stillwater is close and in similar terrain, so day-to-day temperature should track Cushing closely, but not exactly. |

**One question this data cannot answer:** *Did Winter Storm Uri move WTI prices because of freezing at Cushing, or because of production shut-ins in Texas?* Answering it would need regional production data (for example, EIA's weekly or monthly field production by state) and weather from the producing regions. A price series and one Oklahoma station cannot separate those two channels.

## 8. AI Use Log & Reflection

This log is assembled from the AI-use notes in my P1–P3 notebooks, plus P4.

| Milestone | Tool | What I used it for | How I checked the output |
|---|---|---|---|
| P1 | AI assistant | Finding candidate sources, checking whether they fit the question, writing the code that proves each source is reachable, and formatting the notebook | I ran the access code myself and confirmed the row counts and previews against the EIA and NOAA pages. |
| P2 | AI assistant | Help writing the cleaning code and formatting the notebook | I double-checked the code and made sure I understood what each step does. The plausibility-bound checks and row accounting confirmed the cleaned table. |
| P3 | AI assistant | Help writing the acquisition and join code and formatting the notebook | The verification suite, the cache re-read check, and the deliberate failure demonstrations. |
| P4 | Claude (Anthropic) | Drafting the analysis notebook, the README, and this report from my P1–P3 work | I re-ran the notebook top to bottom and checked every number in this report against the notebook output. |

**Something a model got wrong, and how it was caught.** The P3 return calculation was AI-assisted code: `pct_change()` on the EIA price series. It is correct on 2,739 of 2,741 rows and wrong on two. On 2020-04-21 the price rose from −$36.98 to $8.91, but the code reports −124%, a fall, because the starting price was negative. None of the nine P3 assertions caught it. They test that returns are finite and prices are within bounds, and a wrong-sign return passes both. I did not catch it myself in P3. The catch also came from AI: while Claude was helping me build the P4 analysis, it audited the extreme values before running any tests and flagged that the two largest losses in the table were −302% and −124% on back-to-back days, and that a 124% *loss* the day after a negative price could not be right. I checked the flag against the raw prices (−$36.98 on April 20, $8.91 on April 21) and confirmed the price had actually risen. So one AI tool's mistake was found by another AI tool, and confirmed by me looking at the data. The fix was a check on meaning (nb §3): rebuild the previous day's price and require both prices to be positive. Then the main result was rerun with and without those rows.

**Reflection.** What I take from the April 2020 issue is that AI-written code tends to be correct for the typical case and to pass the tests it was written alongside. The edge case only showed up when I looked at the most extreme values in the data rather than at whether the code ran. Next time I would add a check on the *meaning* of every derived column (here, that a price can't rise while its return is negative) alongside the checks on its shape.

## 9. Reproducibility

**Repository:** <https://github.com/AliGulzar-ml/dsa405-project>

1. `git clone https://github.com/AliGulzar-ml/dsa405-project.git && cd dsa405-project`
2. `python -m pip install -r requirements.txt` (Colab already has these packages.)
3. `cd Project`
4. Run `DSA405_002_FA26_P3_agulzar2.ipynb` top to bottom on a fresh kernel. It reads the unmodified NOAA CSV and the cached EIA JSON from `data/raw/` and writes `data/processed/temperature_wti_join_2015_2025.csv`.
5. Run `DSA405_002_FA26_P4_agulzar2.ipynb` top to bottom on a fresh kernel. It re-verifies that file, prints every statistic in this report, and writes four figures to `figures/`.

Each notebook can also be opened from its "Open in Colab" badge and run with *Runtime → Run all*. The first cell clones the repository. No API key or password is committed or needed. The EIA cache means a fresh clone makes no network request to EIA, and the P4 random seed (405) makes the permutation and bootstrap results identical on every run. The repository structure is described in `README.md`.

---

## Self-Scored Rubric

*Scale: 4 = excellent, 3 = proficient, 2 = developing, 1 = beginning.*

| Criterion | Weight | Self-score | Evidence |
|---|---:|---:|---|
| Findings & interpretation | ×2 | **4** | Effect sizes with CIs. A power analysis shows what the null rules out. Known findings and suspected-but-not-shown findings are kept separate (§6). |
| Narrative coherence | ×2 | **4** | One line of argument: question → how the data changed it → join → verification → null result → why it is robust → what it cannot say. |
| Who is missing | ×2 | **4** | Specific dates and places excluded, the consequence of each, and the weekend/holiday exclusion of the coldest day (§7). One unanswerable question is stated. |
| AI use log & reflection | ×2 | **4** | Tools and tasks per milestone, plus a specific AI-code error (wrong-sign return) and how it was caught (§8). |
| Technical ambition (Tier 1) | ×2 | **4** | Beyond a single correlation: HAC regression, month fixed effects, a seasonal anomaly, a time-preserving permutation test, bootstrap CIs, extremes with multiple-testing correction, lags, seven sensitivity checks, and a power analysis. |
| Repository & reproducibility | ×2 | **4** | README with summary, sources and licenses, run steps, and structure. Raw data and the EIA cache are committed. Both notebooks run top to bottom, and the seed is fixed. |
| Documentation carried forward | ×1 | **4** | The data dictionary is updated to the final 2,741/2,739-row table, the cleaning log is extended with the P4 exclusion, and provenance and terms cover both sources. |

**Weighted self-score:** (4×2) × 6 + (4×1) = **52 / 52**

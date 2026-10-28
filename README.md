# Blog 04 — The Price Tag Behind America's Income Rankings

## Overview

This repository reproduces a comparison of **2024 nominal and regional-price-adjusted per capita personal income (PCPI)** across the 50 U.S. states. Using data from the U.S. Bureau of Economic Analysis (BEA), the analysis calculates income rankings before and after adjusting for regional price parities (RPPs), examines changes in rank, and produces five figures.

**Research question:** Do states with the highest nominal per capita incomes remain at the top after accounting for differences in local price levels?

The United States aggregate and the District of Columbia are excluded. All paths below are relative to the **`RodgersYe_6400_Blog04/` repository root**.

## Repository Structure

```text
RodgersYe_6400_Blog04/
├── README.md
├── RodgersYe_6400_Blog04.Rproj
├── .gitignore
├── code/
│   ├── 01_get_raw_data.R
│   ├── 02_clean_data.R
│   ├── 03_analyze_data.R
│   └── 04_make_figures.R
├── data_raw/
│   ├── bea_state_annual_income_2025-09-26.zip
│   ├── bea_nominal_pcpi_2024.csv
│   ├── bea_all_items_rpp_2024.csv
│   ├── bea_official_real_pcpi_2024.csv
│   ├── bea_2024_metadata.rds
│   └── .gitkeep
├── data_processed/
│   ├── state_income_prices_2024.csv
│   ├── state_income_analysis_2024.csv
│   └── rank_summary_2024.csv
└── figures/
    ├── 01_nominal_pcpi_map_2024.png
    ├── 01_nominal_pcpi_ranking_2024.png
    ├── 02_rpp_adjusted_pcpi_map_2024.png
    ├── 02_rpp_adjusted_pcpi_ranking_2024.png
    └── 03_rank_changes_2024.png
```

Local RStudio files such as `.Rproj.user/` and a private `.Renviron` may also exist; they are not needed in a shared checkout. **Never commit a `.Renviron` containing a BEA API key.**

## Reproduce the Research in RStudio

### 1. Get the repository and install requirements

Clone this repository using its GitHub **Code → Clone** URL (or download its ZIP), then open `RodgersYe_6400_Blog04.Rproj` in RStudio. This establishes the project root for `here::here()`. Run all commands below in the **RStudio Console**.

Install R and the packages required by the four scripts:

```r
install.packages(c("here", "httr2", "ggplot2", "usmap", "usmapdata", "sf"))
```

`httr2` is needed **only** to download fresh data using script 01. The other packages support path resolution and figure generation. `sf` may require operating-system-level dependencies. Exact R and package versions are not pinned in this project.

### 2. Choose a reproduction method

There are **two ways to run this project**. Both use the same cleaning, analysis, and visualization scripts, but they differ in the source of the three BEA API datasets.

#### Method A — Reproduce the original study using the uploaded data (recommended)

Use the data already provided in this repository's `data_raw/` directory. Confirm that it contains the archived income ZIP, three BEA CSV files, and the metadata RDS listed under [Data Sources](#data-sources). **Do not run `01_get_raw_data.R`.** This preserves the source snapshots used for the original 2024 comparison.

With `RodgersYe_6400_Blog04.Rproj` open, run the following commands in the **RStudio Console**, in this order:

```r
source(here::here("code", "02_clean_data.R"))
source(here::here("code", "03_analyze_data.R"))
source(here::here("code", "04_make_figures.R"))
```

No BEA API key or new data download is required. This is the recommended method for reproducing the original study's processed datasets, rankings, and five figures.

#### Method B — Download new data through the BEA API and rerun the study

Use this method if you want to retrieve **currently available BEA estimates for the reference year 2024**, instead of relying on the repository's saved API CSV snapshots. You need Internet access and your own [BEA API UserID](https://apps.bea.gov/api/signup/). Review and comply with the [BEA API Terms of Service](https://apps.bea.gov/API/_pdf/bea_api_tos.pdf) before making requests.

1. In the repository root, create a **private, untracked** `.Renviron` file containing:

   ```text
   BEA_API_KEY=your_personal_api_key
   ```

   Ensure `.Renviron` is excluded by `.gitignore`, never commit the API key, and restart your R session.

2. With `RodgersYe_6400_Blog04.Rproj` open and the R session restarted, run these scripts **in order** in the **RStudio Console**:

   ```r
   source(here::here("code", "01_get_raw_data.R"))
   source(here::here("code", "02_clean_data.R"))
   source(here::here("code", "03_analyze_data.R"))
   source(here::here("code", "04_make_figures.R"))
   ```

Script 01 requests three 2024 BEA Regional API series (SAINC1 line 3, SARPP line 1, SARPI line 2). It **overwrites** `bea_nominal_pcpi_2024.csv`, `bea_all_items_rpp_2024.csv`, `bea_official_real_pcpi_2024.csv`, and `bea_2024_metadata.rds` in `data_raw/`. Back up the original snapshots first if you want to preserve the exact original-study inputs.

**Important limitation:** Script 01 does **not** download or replace `bea_state_annual_income_2025-09-26.zip`. Script 02 continues to use that fixed September 26, 2025 archived SAINC1 release as the **main nominal-PCPI input**; the freshly downloaded SAINC1 series is used only to audit revisions. As a result, Method B refreshes the API inputs but is **not an all-new-vintage analysis**. BEA may revise the 2024 RPP and official real-PCPI series, and their updated versions may no longer match the archived income release. Script 03 may then stop at its vintage-consistency checks. Do **not** disable those checks simply to force output; restore the original snapshots for exact replication, or investigate the source-vintage mismatch. Even if all checks pass, results from a fresh download are not guaranteed to match the originally uploaded outputs.

### 3. Outputs and checks (both methods)

Both methods use `source(here::here())` in the RStudio Console. Method A begins with `02_clean_data.R`; Method B begins with `01_get_raw_data.R`. Keep the RStudio Project open so that paths resolve to the correct repository folders.

| Script | Inputs | Outputs |
|---|---|---|
| `01_get_raw_data.R` (Method B only) | BEA API, personal API key | Three API CSV snapshots and `bea_2024_metadata.rds` in `data_raw/` |
| `02_clean_data.R` | Archived ZIP and three BEA API CSVs in `data_raw/` | `data_processed/state_income_prices_2024.csv` |
| `03_analyze_data.R` | `state_income_prices_2024.csv` | `state_income_analysis_2024.csv`, `rank_summary_2024.csv` |
| `04_make_figures.R` | `state_income_analysis_2024.csv` | **All five PNG files** in `figures/` |

**Validation checks:** Script 02 must produce one observation per state (50 rows). Script 03 checks the adjusted ranking against BEA's official real-PCPI ranking and verifies consistency of the source vintages. Script 04 validates 50-state coverage and ranking variables before generating the figures.

If all scripts complete, they regenerate the **three processed CSVs and five PNG figures** named in the repository tree. Existing output files with the same names may be overwritten. If a validation fails, check the input files and their source versions rather than removing the checks.

## Data Sources

All quantitative source data are from the [U.S. Bureau of Economic Analysis (BEA)](https://www.bea.gov/). The snapshots are included in `data_raw/` to support reproduction; **their inclusion does not remove users' obligation to follow BEA's data-use policies and API terms** (see [BEA Data Use, Redistribution, and Attribution](#bea-data-use-redistribution-and-attribution)).

| File in `data_raw/` | BEA source | Role in the analysis |
|---|---|---|
| `bea_state_annual_income_2025-09-26.zip` | [Regional Accounts historical archive](https://apps.bea.gov/histdata/RegionalAccounts.html); SAINC1 line 3, archived September 26, 2025 release | **Primary nominal PCPI** used in the rankings; selects 2024 from the ZIP's `SAINC1__ALL_AREAS_1929_2024.csv` member |
| `bea_nominal_pcpi_2024.csv` | Regional API, SAINC1 line 3 | Later nominal-PCPI version used to **audit revisions**, not as the main nominal income input |
| `bea_all_items_rpp_2024.csv` | Regional API, SARPP line 1 | All-items RPP, used to adjust nominal PCPI for regional prices |
| `bea_official_real_pcpi_2024.csv` | Regional API, SARPI line 2 | BEA official real PCPI, used for validation |
| `bea_2024_metadata.rds` | Acquisition metadata created by script 01 | Retrieval timestamp, request specifications, and available BEA notes; downstream scripts do not read this file |

The archive's **release date** is not necessarily its **download date**. For API data, script 01 records the retrieval timestamp in `bea_2024_metadata.rds` as `retrieved_at_utc`. Avoid assuming a new API download reproduces the exact original vintage.

## Methods

For each state in 2024, the project's price-adjusted per capita personal income is calculated as:

\[
\text{RPP-adjusted PCPI} = \frac{\text{Nominal PCPI}}{\text{All-items RPP}/100}.
\]

An RPP of 100 corresponds to the national average price level. **Rank 1** indicates the highest income, and ties use R's `ties.method = "min"`. The rank-change variable is:

\[
\text{Rank change} = \text{Nominal rank} - \text{Adjusted rank}.
\]

A **positive** change means a state moves toward rank 1 after the price adjustment. The analysis also calculates counts of states moving up, down, or not moving; absolute rank changes; top-ten overlap; and the correlation between nominal income and RPP.

The adjusted PCPI above is **calculated by the author**, not a separately published BEA series. BEA's official real PCPI (in constant 2017 dollars) is used to validate the ordering, not substituted for the project's own 2024 price adjustment.

## Generated Figures

`code/04_make_figures.R` creates all five plots from `data_processed/state_income_analysis_2024.csv`:

| Output file in `figures/` | Description |
|---|---|
| `01_nominal_pcpi_map_2024.png` | Map of nominal-PCPI ranks |
| `01_nominal_pcpi_ranking_2024.png` | Nominal-PCPI values for all 50 states, arranged in two ranked columns |
| `02_rpp_adjusted_pcpi_map_2024.png` | Map of RPP-adjusted PCPI ranks |
| `02_rpp_adjusted_pcpi_ranking_2024.png` | RPP-adjusted PCPI values for all 50 states, arranged in two ranked columns |
| `03_rank_changes_2024.png` | Changes in state rank following price adjustment |

The ranking charts highlight the top 10 states. Mapping uses `usmap`, `usmapdata`, and `sf`; no separate shapefile needs to be downloaded into the repository.

## Interpretation and Reproducibility Limits

- Results describe **state averages**, not a person's wage offer, household purchasing power, or conditions within cities and neighborhoods.
- PCPI includes income beyond wages. All-items RPP does not capture individual consumption patterns or taxes.
- Rankings express **relative order**; changes in rank are not equivalent to changes in the dollar income gap.
- BEA may revise historical observations. Use the saved raw snapshots for the study's fixed-vintage comparison; rerunning the API retrieval is a **data refresh**, not an exact reproduction.
- This repository documents the scripts and saved data but does not pin the exact R or package versions. Software-version differences can affect map rendering or other output details.

## BEA Data Use, Redistribution, and Attribution

**Please read this section before using, copying, or redistributing any files in `data_raw/`.**

BEA states that, **unless otherwise specified**, information published on its website is generally in the public domain and can be used or reproduced without specific permission. BEA appreciates attribution. The inclusion of BEA data in this GitHub repository does **not** waive any BEA-specific conditions or authorize uses that conflict with BEA policies. Before reusing or redistributing these files, consult:

- [BEA: Is permission needed to reproduce information from the BEA website?](https://www.bea.gov/help/faq/145)
- [BEA: Are information and data on its website copyright-protected?](https://www.bea.gov/help/faq/147)
- [BEA API Terms of Service Agreement](https://apps.bea.gov/API/_pdf/bea_api_tos.pdf)
- [BEA Guidelines for Citing BEA](https://www.bea.gov/help/guidelines-for-citing-bea)

If you use the BEA API, comply with its terms and any applicable access limits. In particular:

- **Cite BEA as the source** of the original observations, including relevant table identifiers, reference year, and data vintage.
- **Do not represent adjusted, cleaned, or otherwise modified data as unmodified BEA-published results.** Identify the project's adjustments, ranks, summaries, and figures as the author's calculations.
- **Do not imply BEA sponsorship, endorsement, or certification** of this project or downstream products.
- **Do not share API credentials.** Use your own BEA API UserID when requesting new data.

The BEA API agreement specifies this notice for services using the API:

> This product uses the Bureau of Economic Analysis (BEA) Data API but is not endorsed or certified by BEA.

**Suggested attribution:** Data: U.S. Bureau of Economic Analysis (BEA), Regional Economic Accounts, SAINC1, SARPP, and SARPI, 2024. Nominal PCPI for the analysis uses the archived SAINC1 release dated September 26, 2025. Price-adjusted income, state rankings, statistics, and figures are the author's calculations.

The BEA public-data policy and API terms are separate from any license covering the author's code or documentation. Users are responsible for reviewing and complying with the applicable terms when redistributing BEA source data.

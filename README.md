# HyAQMet — Hybrid Air Quality & Meteorology Dataset
### Dhaka, Bangladesh · 2017–2022 · Hourly Resolution

---

## Overview

**HyAQMet** is a six-year (2017–2022) hourly dataset integrating four independent reanalysis and atmospheric composition products for **Dhaka, Bangladesh**. It provides a single, analysis-ready flat file combining surface meteorology, boundary layer dynamics, near-surface pollutant concentrations, and speciated aerosol composition — with zero missing values across the complete record.

| Property | Value |
|---|---|
| **Period** | 2017-01-01 00:00 BST → 2022-12-31 23:00 BST |
| **Temporal resolution** | Hourly |
| **Rows** | 52,584 |
| **Columns** | 42 |
| **Missing values** | 0 |
| **Time zone** | Bangladesh Standard Time (BST = UTC+6) |

---

## Repository Structure

```
hyaqmet/
│
├── data/
│   ├── CAMS_3hourly_2017-2022_bst.csv       # CAMS EAC4 source extract (3-hourly, pre-merge)
│   ├── ERA5_hourly_2017-2022_bst.csv        # ERA5-Land + Single Levels source extract
│   └── MERRA-2_hourly_2017-2022_bst.csv     # MERRA-2 source extract (26 raw variables)
│
├── notebooks/
│   ├── hyaqmet-data-extraction.ipynb        # Download pipelines: CAMS, ERA5
│   └── hyaqmet-creation-analysis.ipynb      # Merging, processing & validation pipeline
│
└── README.md
```

> **The final merged dataset (HyAQMet.csv / .parquet / .json) is hosted on Zenodo:**
> **[https://doi.org/10.5281/zenodo.20705735](https://doi.org/10.5281/zenodo.20705735)**

---

## Data Sources

| Source | Provider | Resolution | Temporal res. | Portal |
|---|---|---|---|---|
| ERA5-Land | ECMWF / Copernicus | 0.1° × 0.1° | Hourly | [CDS](https://cds.climate.copernicus.eu) |
| ERA5 Single Levels | ECMWF / Copernicus | 0.25° × 0.25° | Hourly | [CDS](https://cds.climate.copernicus.eu) |
| CAMS EAC4 | ECMWF / Copernicus | 0.75° × 0.75° | 3-hourly | [ADS](https://ads.atmosphere.copernicus.eu) |
| MERRA-2 | NASA GES DISC | 0.5° × 0.625° | Hourly | [GES DISC](https://disc.gsfc.nasa.gov) / [Giovanni] (https://giovanni.gsfc.nasa.gov/giovanni/)|

---

## Source CSV Files (in `/data/`)

These are the extracted, unit-converted source files **before** merging into HyAQMet. They are provided for reproducibility — if you want to re-run the merge or modify the processing, start here.

### `CAMS_3hourly_2017-2022_bst.csv`
> 17,528 rows × 8 columns · 3-hourly · BST

| Column | Description | Units |
|---|---|---|
| `timestamp_bst` | Timestamp in BST | YYYY-MM-DD HH:MM:SS |
| `pm2_5_ugm3` | PM2.5 near-surface | µg/m³ |
| `pm10_ugm3` | PM10 near-surface | µg/m³ |
| `no2_ugm3` | NO2 near-surface | µg/m³ |
| `o3_ugm3` | O3 near-surface | µg/m³ |
| `co_mgm3` | CO near-surface | mg/m³ |
| `so2_ugm3` | SO2 near-surface | µg/m³ |
| `dust_aod_550nm` | Dust aerosol optical depth at 550 nm | dimensionless |

### `ERA5_hourly_2017-2022_bst.csv`
> 52,584 rows × 12 columns · hourly · BST

| Column | Description | Units |
|---|---|---|
| `timestamp_bst` | Timestamp in BST | YYYY-MM-DD HH:MM:SS |
| `t2m_celsius` | 2 m air temperature | °C |
| `dewpoint_celsius` | 2 m dewpoint temperature | °C |
| `relative_humidity_pct` | Relative humidity (Magnus-derived) | % |
| `wind_speed_ms` | Wind speed magnitude √(u²+v²) | m/s |
| `wind_u_ms` | 10 m zonal wind component | m/s |
| `wind_v_ms` | 10 m meridional wind component | m/s |
| `precip_mm` | Total precipitation | mm/hr |
| `surface_pressure_hpa` | Surface pressure | hPa |
| `boundary_layer_height_m` | Boundary layer height | m |
| `solar_radiation_wm2` | Surface solar radiation downwards (corrected) | W/m² |
| `total_cloud_cover_fraction` | Total cloud cover fraction | 0–1 |

### `MERRA-2_hourly_2017-2022_bst.csv`
> 52,584 rows × 27 columns · hourly · BST (26 raw variables + timestamp)

Contains all 26 raw MERRA-2 variables before VIF screening. Key variables include speciated aerosol surface mass concentrations (BC, OC, SO4, Dust), aerosol optical properties (AOT, Ångström exponent), column mass densities, surface meteorology, SO2, and total column ozone. See the column headers in the file for full variable names and units, which are written out in full (e.g. `Black Carbon Surface Mass Concentration (µg/m³)`).

---

## Final Dataset — Download from Zenodo

The final merged and processed **HyAQMet** dataset (52,584 × 42) is available at:

> 📦 **[https://doi.org/10.5281/zenodo.20705735](https://doi.org/10.5281/zenodo.20705735)**

The Zenodo ZIP contains:

| File | Format | Description |
|---|---|---|
| `HyAQMet.csv` | CSV (UTF-8) | Primary dataset, human-readable |
| `HyAQMet.parquet` | Apache Parquet | Fast-loading binary format |
| `HyAQMet.json` | JSON | For web/API applications |
| `HyAQMet_README.md` | Markdown | Variable descriptions and usage notes |

### Quick load (Python)

```python
import pandas as pd

# Option 1 — CSV
df = pd.read_csv("HyAQMet.csv", parse_dates=["ts"])

# Option 2 — Parquet (faster for large workflows)
df = pd.read_parquet("HyAQMet.parquet")

print(df.shape)        # (52584, 42)
print(df.isnull().sum().sum())  # 0
print(df["season"].unique())
# ['Winter' 'Spring' 'Summer' 'Monsoon' 'Autumn' 'Late Autumn']
```

### Quick load (R)

```r
library(arrow)
df <- read_parquet("HyAQMet.parquet")

# or CSV
df <- read.csv("HyAQMet.csv")
df$ts <- as.POSIXct(df$ts, tz = "Asia/Dhaka")
```

---

## Notebooks

### `hyaqmet-data-extraction.ipynb`
Downloads raw data from all three APIs. Run this first if you want to reproduce the source CSVs from scratch.

**Sections:**
- **CAMS EAC4** — ADS API download, unit conversion (kg/kg → µg/m³ via ideal gas law), BST buffer-day fix, annual CSV merge
- **ERA5-Land + ERA5 Single Levels** — CDS API download, solar radiation accumulation correction, derived wind speed and relative humidity
- **MERRA-2** — NASA GES DISC download, individual variables can also be dowloaded from Giovanni portal https://giovanni.gsfc.nasa.gov/giovanni/

**Requirements:**
```bash
pip install cdsapi xarray netcdf4 pandas
```
You will also need:
- A [Copernicus CDS API key](https://cds.climate.copernicus.eu/user/register) (for ERA5)
- A [Copernicus ADS API key](https://ads.atmosphere.copernicus.eu/user/register) (for CAMS)
- A [NASA Earthdata account](https://urs.earthdata.nasa.gov/) (for MERRA-2)

Replace `YOUR_ADS_API_KEY` and `YOUR_CDS_API_KEY` with your credentials in the first cell of the notebook.

---

### `hyaqmet-creation-analysis.ipynb`
Merges the three source CSVs into the final HyAQMet dataset and runs validation checks.

**Sections:**
- Temporal alignment: UTC → BST, CAMS 3-hourly → hourly linear interpolation, MERRA-2 half-hourly offset → on-hour interpolation
- MERRA-2 VIF screening (26 → 13 variables)
- Cyclical feature engineering (hour_sin/cos, doy_sin/cos)
- Bangladesh six-season label assignment
- Merge on timestamp → HyAQMet
- Technical validation: integrity checks, BLH–PM2.5 inverse relationship, inter-source wind and PM2.5 consistency

**Requirements:**
```bash
pip install pandas numpy scipy matplotlib seaborn statsmodels
```

---

## Column Reference (HyAQMet — 42 columns)

| Group | Columns | Count |
|---|---|---|
| Temporal index | ts, hour, dayofyear, month, year, hour_sin, hour_cos, doy_sin, doy_cos | 9 |
| Season | season, season_code | 2 |
| ERA5-Land + Single Levels | t2m_celsius, dewpoint_celsius, relative_humidity_pct, wind_speed_ms, wind_u_ms, wind_v_ms, precip_mm, surface_pressure_hpa, solar_radiation_wm2, boundary_layer_height_m, total_cloud_cover_fraction | 11 |
| CAMS EAC4 | cams_pm25_ugm3, cams_pm10_ugm3, cams_no2_ugm3, cams_o3_ugm3, cams_so2_ugm3, cams_co_mgm3, cams_dust_aod_550nm | 7 |
| MERRA-2 (post-VIF) | merra_bc_surface_ugm3, merra_oc_surface_ugm3, merra_so4_surface_ugm3, merra_dust_pm25_ugm3, merra_aot_extinction_550nm, merra_angstrom_470_870, merra_dust_ext_aot_pm25, merra_wind_speed_ms, merra_specific_humidity_gkg, merra_precip_mmhr, merra_so2_surface_ugm3, merra_so2_col_mgm2, merra_col_ozone_du | 13 |

---

## Bangladesh Six-Season Calendar

Season labels in HyAQMet follow Bangladesh's traditional six-season (Ritu) calendar:

| season_code | season | Approximate period |
|---|---|---|
| 0 | Winter | 16 Dec – 15 Feb |
| 1 | Spring | 16 Feb – 15 Apr |
| 2 | Summer | 16 Apr – 15 Jun |
| 3 | Monsoon | 16 Jun – 15 Oct |
| 4 | Autumn | 16 Oct – 15 Nov |
| 5 | Late Autumn | 16 Nov – 15 Dec |

---

## Citation

The dataset is also used in the following esearch, please cite:

> [Adity Barua ] (2026). *HyAQMet: A Hybrid Hourly Air Quality and Meteorology Reanalysis Dataset for Dhaka, Bangladesh (2017–2022)
> Reanalysis Dataset for Dhaka, Bangladesh (2017–2022)*. Zenodo.
> https://doi.org/10.5281/zenodo.20705735

A data descriptor article has been submitted to **Geoscience Data Journal** (GDJ).
This README will be updated with the full citation upon acceptance.

---

## Licence

This dataset is released under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) licence.

Source data acknowledgements:
- ERA5 and ERA5-Land: Generated using Copernicus Climate Change Service information. Neither the European Commission nor ECMWF is responsible for any use that may be made of this information.
- CAMS EAC4: Generated using Copernicus Atmosphere Monitoring Service information.
- MERRA-2: Provided by the NASA Goddard Earth Sciences Data and Information Services Center (GES DISC).

---

## Contact

[Please contact if you have any queries — Adity Barua; aditybarua07@gmail.com]

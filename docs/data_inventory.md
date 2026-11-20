# Data inventory

## Whale occurrences (OBIS)
- Source: OBIS API v3 (https://api.obis.org/v3/occurrence), species Megaptera novaeangliae. OBIS aggregates many providers, see data_sources.md and dataset_notes.md
- Snapshot / download date: 2026-11-06 12:07 UTC (data/raw/obis_humpback/download_meta.json)
- Records: 100,185 from 322 datasets, spanning 1901-2026 (main analysis: 2010 onwards)
- Region: longitude -85 to 40, latitude 10 to 80
- Used in the model table: 9,481 presence cell-months (one record per 0.25 degree cell per month; human observations with precise coordinates; 2010 onwards) and 47,335 matched background cell-months (5 per presence, same month)
- License and citation: mixed at record level (about 40% CC BY-NC, 32% CC0, 23% without a license, 2% CC BY, 2% CC BY-SA). Raw records are not redistributed. Every dataset is cited in data_sources.md and dataset_notes.md
- Known limitations: opportunistic sampling with strongly increasing effort; month-level placeholder dates in older records (flagged, outside the main period); coordinate uncertainty missing in about 25% of records (inferred from decimals); possible duplicates across datasets (merged by thinning); presence-only data, so scores are relative suitability, not true probability; effort rarely recorded
- Status: [x] researched [x] downloaded [x] verified

## Sea surface temperature (NOAA OISST v2.1, final)
- Source: NOAA CoastWatch ERDDAP, dataset ncdcOisst21Agg_LonPM180 (https://coastwatch.pfeg.noaa.gov/erddap/griddap/ncdcOisst21Agg_LonPM180)
- Download date: [date you ran the full download; every file also stores it in the attribute downloaded_utc]
- Period: 2010-01-01 to 2026-10-23 (202 monthly files, 6,121 days; 19 days missing = 0.3%, largest gap December 2023 with 11 days)
- Region: same box as above
- Variables: sst, anom (the built-in anomaly is not used; anomalies will be recomputed from a 2010-2019 climatology)
- Resolution: 0.25 degree, daily; the model uses monthly means
- Size on disk: 2.37 GB (data/raw/sst_oisst/)
- License and citation: [copy the recommended citation from the dataset page]
- Known limitations: 40.1% of the box is masked (land); 1,136 of 9,591 presence cells (11.8%) fell on the mask, mostly fjords and coastal whale-watching areas (Iceland, northern Norway, Newfoundland, Svalbard); 1,026 were filled from an adjacent ocean cell and flagged, 110 were excluded; the built-in anomaly shows implausible values near some coasts; the last two weeks of the record are not yet included
- Status: [x] researched [x] downloaded [x] verified

## Bathymetry (GEBCO)
- Source: GEBCO download app (https://download.gebco.net)
- Version: GEBCO_2026 Grid (published April 2026), under-ice topography/bathymetry version
- File: gebco_2026_sub_ice_n80.0_s10.0_w-85.0_e40.0.nc (1.01 GB), downloaded [date]
- Region: N 80, S 10, W -85, E 40 | 16,800 x 30,000 cells
- Resolution: 15 arc-seconds; values are elevation in metres (negative = depth), pixel-centre registered
- Processing: aggregated to 280 x 500 cells of 0.25 degrees (60 x 60 GEBCO cells each): mean and standard deviation of depth over ocean pixels, and ocean fraction (data/processed/gebco_cells_025.csv)
- License: public domain, free of charge, attribution required; not to be used for navigation or safety at sea
- Citation: GEBCO Compilation Group (2026) GEBCO 2026 Grid (doi: 10.5285/4f68d5c7-45eb-f999-e063-7086abc036fa)
- Known limitations: in unsurveyed areas depth is predicted from satellite gravity; the grid is an interpolated information product and its accuracy is not guaranteed; 47,117 of 140,000 cells (33.7%) contain no ocean; cell means smooth out small-scale topography
- Status: [x] researched [x] downloaded [x] verified

## Vessel density (EMODnet Human Activities)
- Status: coverage of Iceland, Norway and Svalbard not yet checked; postponed beyond the first release
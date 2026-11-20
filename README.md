# Humpback Horizon — monthly habitat-suitability maps for North Atlantic humpback whales

> **Status: research prototype (v1.0-application).** Scores are *relative suitability* (a ranking of ocean cells),
> **not** the probability of seeing a whale. Maps in this repository are hindcasts made with observed sea-surface temperature.

**Author:** Merve Nur İREGÖZ · iregozmerve@gmail.com · https://www.linkedin.com/in/merve-nur-iregoz/ · Version: v1.0-application (20 November 2026)

## 1. Question

Where, and in which month, is the North Atlantic more or less suitable for humpback whales, and how much do
environmental variables add to what past sightings alone already tell us?

The same model output is meant to support three decisions (see *Roadmap* for what is implemented):
reducing ship-strike risk, setting conservation priorities, and planning surveys where the model is uncertain.

## 2. Project history

This project grew out of an earlier graduation project on whale route prediction that used an older dataset
(first exploration: `notebooks/01_explore_old_data.ipynb`). A systematic audit of that data and code
showed several problems that made the earlier analysis unreliable:
(1) the identifier used to follow individual whales was the encounter URL, not the individual's ID, so every "whale" consisted of a single record and no tracks existed;
(2) the observation date was read from a column that holds only the time of day, so the date silently defaulted to the day the code was run;
(3) sea-surface temperature was never matched to individual sightings (only global averages were plotted);
(4) the temperature files covered only four days per year (June to September), missing most of the seasonal cycle;
(5) some code cells were adapted from unrelated tutorials and did not fit the data.
The audit directed the redesign: a new multi-dataset OBIS download, explicit handling of dates and coordinate precision,
spatial-temporal thinning, background points, and time-based validation. The old data are **not** used in any result reported here.

## 3. Repository structure

```text
notebooks/   numbered notebooks (01 exploration, 02 data download and modelling)
docs/        data inventory, data sources, dataset notes, references, figures/
requirements.txt
```
Raw and processed data are not stored in the repository (third-party licences, size). They are rebuilt by the notebooks.
**Not yet tested from a clean environment.**

## 4. Data

| Source | Use | Notes |
|---|---|---|
| OBIS (API v3), humpback whale occurrences | Presences | 100,185 raw records from 322 datasets; licences mixed (CC0, CC BY, CC BY-NC, CC BY-SA, 23% unknown). Raw data are **not** redistributed. |
| NOAA OISST v2.1 (via ERDDAP) | Sea-surface temperature, own anomaly vs. 2010–2019 monthly climatology | Monthly files, 0.25° |
| GEBCO_2026 | Depth (mean and std per 0.25° cell), ocean fraction | Not for navigation |

Every OBIS dataset that contributes records is listed in `docs/data_sources.md`. Download dates: OBIS 6.11.2026, NOAA OISST 8.11.2026, GEBCO 12.11.2026.
Please cite the original dataset providers when using any derived product.

## 5. Method (short)

- **Presence-only modelling.** Presences are thinned to one record per 0.25° cell per month (spatial-temporal thinning) to reduce observer-effort bias. 
  Background points are random ocean cells, month-matched, five per presence. Background is **not** absence.
- **Seasonal baseline.** Gaussian kernel (σ = 100 km by default) over past presences of the same calendar month.
- **Environmental model.** Gradient boosting (scikit-learn `HistGradientBoostingClassifier`) with SST, own SST anomaly, log depth, depth variability, ocean fraction, month (sin/cos).
- **Combined model.** Environmental features plus the past-only baseline score.
- **Validation.** Rolling-origin: train on years before *Y*, test on year *Y*, for *Y* = 2020–2025. Metrics: AUC and the share of presences inside the highest-scoring 10% of the test area.
- **Harder tests.** (a) shelf-only evaluation (≤ 500 m), (b) transfer between western and eastern Atlantic, (c) "new territory" test (points > 100 km from every earlier presence), (d) sensitivity to kernel width, number of background points and coastal SST filling.
- **Uncertainty.** Disagreement between 10 bootstrap models (model uncertainty only).

## 6. Results (test years 2020–2025, averages)

| Test set | Seasonal baseline | Environment only | Baseline + environment |
|---|---|---|---|
| All depths — AUC | 0.968 | 0.966 | **0.987** |
| All depths — presences in top 10% area | 0.944 | 0.933 | 0.982 |
| Shelf ≤ 500 m — AUC | 0.929 | 0.899 | **0.964** |
| Shelf ≤ 500 m — presences in top 10% area | 0.767 | 0.638 | 0.906 |

- Depth alone gives AUC 0.943 on all depths but only 0.820 on the shelf: much of the "easy" skill is the shelf-versus-open-ocean contrast of the random background.
- The combined model beats the baseline in each of the six test years. The gain is small to moderate and stays positive in every sensitivity setting:
  +0.013 to +0.032 AUC (all depths) and +0.021 to +0.053 (shelf) for baseline kernel widths from 25 to 200 km. Against the strongest baseline tried (σ = 25 km): 0.975 → 0.988 (all), 0.947 → 0.967 (shelf).
  The kernel width was chosen after looking at the test years, so the whole range is reported.
- Changing the background ratio (1:1 to 5:1 in training) or excluding rows with filled coastal SST leaves the gain essentially unchanged.
- Transfer test (train in one region, test in the other): environment-only AUC 0.903 (west → east) and 0.914 (east → west), versus 0.625 and 0.333 for the baseline, which has no memory there.
- New-territory test (> 100 km from earlier presences): AUC 0.816 (baseline), 0.890 (environment), 0.931 (combined). **Small sample** (about 12 presences per test year); the > 300 km setting has one year with 7 presences and is not interpreted.
- Permutation importance (environment model, 2025): log depth 0.237, SST 0.097, depth variability 0.026, ocean fraction 0.015; own SST anomaly 0.003. 
  The temperature-anomaly feature adds no measurable skill at this scale, so no climate-change claim is made.
- Hindcast maps (trained on years before 2025): essentially all of the 42 recorded January presences and 98% of the 194 July presences lie in the top 10% of cells (values rounded). 
  This number is inflated by the baseline's memory of where observers went; see Limitations.

Figures: `docs/figures/`.

## 7. Limitations (please read)

- **Presence-only data are biased by where people look.** The baseline learns observer effort as well as whale habitat, so scores mix suitability and sighting intensity.
- **Random background includes much unsuitable open ocean,** which makes AUC look high; this is why shelf-only and new-territory tests are reported.
- **Relative suitability, not probability.** The ranking is only meaningful within a month.
- **Hindcast, not forecast.** Maps use observed SST. A true forecast would need forecast SST, which is not implemented.
- **Model uncertainty only.** The uncertainty layer does not capture observation bias or data errors.
- **Data quality.** Some older records carry placeholder dates (1st of month) and coordinate uncertainty inferred from decimals; 23% of records come from datasets with unknown licence.
- Cells with sea ice or missing SST get no score.
- Not for navigation or operational decisions.

## 8. Contribution to the Sustainable Development Goals

Main link: SDG 14 (Life Below Water).

Direct: target 14.2 (sustainable management and protection of marine ecosystems), through ship-strike and priority-area information.
Direct: target 14.a (scientific knowledge and research capacity), through open, reproducible methods and uncertainty maps.
Indirect: target 14.5 (protected areas), as scientific input only; this project does not make protection decisions.

Possible secondary links (not claimed as results):

SDG 17 (Partnerships for the Goals): the work depends on open data shared by many organisations (OBIS and its dataset providers, Happywhale contributors, NOAA, GEBCO) and credits them in docs/data_sources.md.
SDG 9 (Industry, Innovation and Infrastructure): the code and data pipeline are open and could be reused or extended by others; reuse has not been demonstrated yet.
SDG 13 (Climate Action): climate enters only as a sensitivity question (own SST anomaly). In this study the anomaly feature added no measurable skill, so no climate-change claim is made.

## 9. Reproduce

```text
git clone https://github.com/Mervvee/humpback-horizon.git
cd humpback-horizon
python -m venv .venv
.venv\Scripts\activate          # Windows (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
```
Run the notebooks in `notebooks/` in numerical order, and the cells in each notebook from top to bottom.
OBIS records and NOAA OISST are downloaded by the notebook (the monthly SST download is resumable).
The GEBCO grid has to be downloaded manually from the GEBCO website and placed in `data/raw/gebco/`.
Downloads take a long time. This workflow has **not yet been tested from a clean environment**.

## 10. Roadmap

Implemented: data pipeline, baseline, environment model, rolling-origin evaluation, harder tests, monthly suitability maps with model uncertainty.
Planned: ship-strike risk layer, conservation-priority stability analysis, survey-planning panel, forecast SST, accessible-area background, climate sensitivity module.

## 11. Licence and citation

Code: MIT licence (see `LICENSE`). Data: not redistributed; each source keeps its own licence (see `docs/data_sources.md`).

If you use this work, please cite:

> İregöz, M. N. (2026). *Humpback Horizon: monthly relative habitat-suitability maps for North Atlantic humpback whales* (version v1.0-application) [Software]. GitHub. https://github.com/Mervvee/humpback-horizon

## 12. Acknowledgements and references

[OBIS and dataset providers · NOAA OISST · GEBCO · references from `docs/references.md`]
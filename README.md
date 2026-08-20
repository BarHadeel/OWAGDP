# OWAGDP — Ordered Weighted Averaging of Institutional GDP Forecasts

Data, code, and results for the paper:

> H. S. Baraheem and J. M. Merigó, "Ordered Weighted Averaging of Institutional
> GDP Forecasts." (Under review, 2026.)

The paper aggregates the 2026 and 2027 real GDP growth forecasts of the IMF,
the OECD, and the European Commission for 37 economies with an ordered weighted
averaging (OWA) operator, orders the forecasts by past accuracy through an
induced operator, and propagates historical forecast error through the
aggregation by exact enumeration of an empirical bootstrap distribution.

## Repository structure

Each numbered folder is one pipeline stage, with the notebook at the top level
and its outputs in `Result/`. The raw source data is **not redistributed
here**, for size and licensing reasons; it is freely available online from the
institutions themselves — the tables below name every file and its download
location. All downstream stages (03b onward, including the verification) run
entirely from the result files included in this repository. The notebooks
locate their inputs by column signature, so a stage can be run by placing the
required files in its working directory.

| Stage | Notebook | Main outputs |
|---|---|---|
| 01_build_panel | `01_build_panel.ipynb` | `panel.csv` (74 rows: 37 economies × 2 years) |
| 02_aggregate | `02_aggregates.ipynb`, `fig1.ipynb` | `aggregates.csv` (OWAGDP at α = 0.1–0.9, spread, range), `weights.csv`, Fig. 1 |
| 03a_build_error | `03a_build_error.ipynb` | `forecast_errors.csv` (181 rows: archived forecasts and realised outcomes, five rounds Autumn 2020–Autumn 2022) |
| 03b_accuracy | `03b_accuracy.ipynb` | `mae.csv` (per-country MAE, induced order, separation), `induced.csv` (IOWAGDP vs OWAGDP per cell) |
| 03c_rescore | `03c_rescore.ipynb` | `rescore.csv` (robustness of the accuracy ranking to the outcome series) |
| 04_uncertainty | `04_uncertainty.ipynb` | `uncertainty_full.csv`, `uncertainty_excl2021.csv` (exact 2.5–97.5 percentile ranges), Fig. 2 |
| 05_verification | `05_verification.ipynb` | prints every number quoted in the paper against the result files |

Note on naming: `forecast_errors.csv` contains archived **forecasts and
realised outcomes** (columns `iso, round, target, IMF, OECD, EC, actual`);
signed and absolute errors are derived inside each notebook as
`forecast − actual`.

## Raw data files (available online, not included)

Stage 01 (current vintages):

| File | Contents | Source |
|---|---|---|
| `WEOApr2026all.xlsx` | IMF World Economic Outlook database, April 2026 (series `NGDP_RPCH`) | [IMF WEO database](https://www.imf.org/en/Publications/WEO) |
| `OECD.ECO.MAD,DSD_EO@DF_EO,+.GDPV_ANNPCT.A.csv` | OECD Economic Outlook No. 119, June 2026 (series `GDPV_ANNPCT`) | [OECD Data Explorer](https://data-explorer.oecd.org/) |
| `AMECO6.TXT` | European Commission AMECO database, Spring 2026 (chapter 6, series `OVGD`) | [AMECO](https://economy-finance.ec.europa.eu/economic-research-and-databases/economic-databases/ameco-database_en) |

Stage 03a (archived vintages for the accuracy window):

| File | Contents | Source |
|---|---|---|
| `OECD,DF_EO108_INTERNET,+..A.csv` … `OECD,DF_EO112_INTERNET,+..A.csv` | OECD Economic Outlook archived editions 108–112 | [OECD Data Explorer archive](https://data-explorer.oecd.org/) |
| `WEOhistorical.xlsx` | IMF Historical WEO Forecasts Database | [IMF](https://www.imf.org/external/pubs/ft/weo/data/WEOhistorical.xlsx) |
| `ameco_autumn2020.zip`, `ameco_spring2021.zip`, `ameco_autumn2021.zip`, `ameco_spring2022.zip`, `ameco_autumn2022.zip` | European Commission archived AMECO releases, the five scoring rounds | [AMECO archive](https://economy-finance.ec.europa.eu/economic-research-and-databases/economic-databases/ameco-database_en) |
| `WEOApr2026all.xlsx` | IMF WEO April 2026 — source of the realised outcomes | [IMF WEO database](https://www.imf.org/en/Publications/WEO) |

To rebuild from raw sources, download the files above into a `Data/` folder
inside the corresponding stage and run the stage notebook. To reproduce the
paper's results without the raw sources, start at stage 03b using the result
files already included.

## Method notes

- OWA weights are maximum-entropy vectors on a declared orness grid
  (geometric form `w_i ∝ h^(i−1)`); displayed tables are rounded, all
  computations use unrounded weights.
- The accuracy-induced ordering uses `u = 1/MAE` per country; only the rank
  of `u` enters.
- The uncertainty calculation enumerates all 5³ = 125 independent error
  combinations per economy (4³ = 64 excluding the 2021 target) and takes
  percentiles from the inverse empirical distribution function
  (`numpy.percentile(..., method='inverted_cdf')`). No random draws and no
  seed are involved; results are exactly reproducible.
- At α = 0.5 the weights are equal, so the percentile-range width is
  identical for the 2026 and 2027 forecasts of the same economy by
  construction.

## Running

The notebooks were written for Google Colab but run anywhere with Python 3,
`numpy`, `pandas`, and `matplotlib` (NumPy ≥ 1.22 for
`method='inverted_cdf'`).

Run order: 01 → 02 → 03a → 03b → 03c → 04 → 05. Stages 01–03a require the
raw data files listed above; stages 03b–05 run from the included result
files. Stage 05 should end with `86 of 86 checks passed`.

## License

Code and derived data are released under the MIT License (see `LICENSE`).

## Citation

If you use this repository, please cite the paper above. A DOI for the
archived release is available via Zenodo (see the badge at the top of this
page once the release is archived).

# FPV Systematic Review — analysis code

Analysis and figure-generation code for a systematic literature review on the
effects of floating photovoltaic (FPV) systems on freshwater bodies.

> **Status note:** This repository contains the analysis code and screening
> data for the review. It was originally developed in Google Colab (April 2023)
> and adapted to run locally. See *Known issues* below before running.

---

## What this does

`systematic_literature_review.py` reads a screening database and derived count
tables, then produces the review's figures:

- Publications per year (primary research vs. synthesis) with cumulative count
- Proportion of review vs. non-review papers, and publication types
- Evidence type (empirical vs. modelled)
- Köppen–Geiger climate zones and lake thermal regions
- Water body type, hydropower association
- FPV study-design coverage (with/without FPV; before/after deployment)
- World map of study locations and a per-country count map
- Direction-of-impact charts (physical / chemical / biological processes)

The same code is provided as a Jupyter notebook
(`systematic_literature_review.ipynb`) and as an exported script
(`systematic_literature_review.py`). They are equivalent; use whichever you
prefer.

## Repository layout

```
.
├── systematic_literature_review.py     # analysis script
├── systematic_literature_review.ipynb  # same analysis, notebook form
├── data/                               # input screening data (see below)
├── figures/                            # generated figures (created on run)
├── requirements.txt
├── LICENSE                             # PLACEHOLDER — choose before publishing
└── .gitignore
```

## Data files

| File | Used by script? | Notes |
|------|-----------------|-------|
| `data/database_papers_updated_format.xlsx` | **Yes** | Main screening database the script reads |
| `data/database_papers_format.xlsx` | No | Appears to be an earlier version — **verify / remove if stale** |
| `data/database_clean_modelling-only_noreview.xlsx` | No | Modelling-only subset — **verify whether still needed** |
| `data/impact_direction_format.csv` | (older) | Direction-of-impact counts |
| `data/impact_direction_format_updated.csv` | Yes | Updated direction-of-impact counts |
| `data/processes_frequency_count.csv` | (older) | Process frequency counts |
| `data/processes_frequency_count_updated_search.csv` | Yes | Updated process frequency counts |

> The three database spreadsheets share the same column schema (`id`,
> `authors`, `DOI`, `citations`, `year`, `publication_type`, `review`,
> `evidence_type`, ...). I could not determine from the files alone which
> non-`updated` files are still needed — please confirm and prune.

## Running

```bash
pip install -r requirements.txt
python systematic_literature_review.py
```

Figures are written to `figures/`. The script reads inputs from `data/`
(paths are now relative to the script — the original hardcoded Google Drive
paths were replaced).

## Known issues / things to check

1. **GeoPandas world-map call.** The script uses
   `gpd.datasets.get_path('naturalearth_lowres')`. This dataset accessor was
   **removed in GeoPandas 1.0 (2024)**; on newer GeoPandas this line fails.
   Either install `geopandas<1.0`, or download the Natural Earth dataset and
   load it directly. *Verify against your installed version.*

2. **Natural Earth shapefile (per-country map).** The country-count map needs
   the Natural Earth 1:10m Admin 0 Countries shapefile, which is **not included**
   (it is a large external dataset). Download it from naturalearthdata.com and
   place the files under `data/shapefiles/` so that
   `data/shapefiles/ne_10m_admin_0_countries.shp` exists.

3. **Versions are unpinned** in `requirements.txt` — I do not have a record of
   the original environment. Pin to your tested versions for reproducibility.

4. **Some manual values in the code.** At least one array (`reviews = [...]`) is
   noted in the source as manually estimated in Excel. Treat such values as
   inputs to verify, not as computed results.

## Citation / provenance — TO COMPLETE

The following could not be determined from the code and should be filled in by
the author:

- [ ] Full title of the review / thesis chapter
- [ ] Associated publication (journal, year, DOI) if any
- [ ] How to cite this repository (consider archiving a release on Zenodo for a DOI)
- [ ] Data-use statement (see LICENSE note)

## Author

Sofia Midauar G Rocha — Lancaster Environment Centre, Lancaster University.

## Licence

Not yet chosen — see `LICENSE`. Select one before making the repository public.

# Torun Reservoir — WAQ Water Quality Analysis (PhD Chapter 5)

Python notebooks for analysing Delft3D-WAQ / BLOOM water quality model output
for the Torun Reservoir, including floating photovoltaic (FPV) scenarios.

> Developed during PhD research at Lancaster University. Notebooks were run on a
> Windows workstation against model output on a network drive; absolute paths
> have been replaced with the placeholder `<MODEL_DIR>` (see *Paths* below).

> **Radiation and wind forcing:** use only `forcing_remapped\*_remapped_upper.bin`.
> The `*_fixed.bin` and `*_remapped_column.bin` versions are wrong, and FPV
> scenario results made with them should not be used. See
> [`tools/FORCING_FIX.md`](tools/FORCING_FIX.md) for what went wrong and how the
> correct files were made and checked.

---

## Repository layout

```
.
├── notebooks/        # main analysis notebooks (active)
├── tools/            # single-purpose preprocessing / conversion utilities
├── archive/          # superseded versions kept for provenance (not maintained)
├── requirements.txt
├── LICENSE           # PLACEHOLDER — choose before publishing
└── .gitignore
```

### notebooks/

| Notebook | Purpose |
|----------|---------|
| `Torun_WAQ_Analysis_netcdf_20260603.ipynb` | Main WAQ output analysis (NetCDF-aware): DO across layers, modelled vs. observed, multi-run comparison, nutrients, vertical profiles, light climate, performance statistics, phytoplankton/chlorophyll. |
| `Torun_WAQ_FPV_Diagnostic_netcdf_20260611.ipynb` | FPV scenario diagnostic: auto-detects the CAB column, unified NetCDF/`.his` accessors, DO/Secchi/nutrient/phytoplankton diagnostics, seasonal difference boxplots, performance summary. |
| `WAQ_comprehensive_analysis_netcdf_20260616.ipynb` | Perimeter-averaged comprehensive analysis: defines the FPV_02 perimeter mask, computes daily/monthly perimeter-mean time series, species composition, Hovmöller depth profiles, surface maps. |
| `WAQ_map_to_netcdf_20260602.ipynb` | Converts WAQ `.map` output to NetCDF, with validation. |
| `compare_waq_radiation.ipynb` | Compares WAQ solar-radiation binary input files. |
| `compare_baseline_his_20261008.ipynb` | Compares the `.his` output of a run with the calibrated Run42, per layer (`RadAve`, `Limit e`, phytoplankton) and for every variable on the depth-averaged locations. Used to verify the corrected forcing files; see `tools/FORCING_FIX.md`. Self-contained. |

### tools/

| Notebook | Purpose |
|----------|---------|
| `conversion_binarytogrid_20260601.ipynb` | Converts WAQ binary output to a gridded form. |
| `create_tekal.ipynb` | Reads measurement CSVs and writes Tekal (`.tek`) files for Delft3D QUICKPLOT. |
| `fix_communicationfiles_clean.ipynb` | Repairs communication files (`.vol`, `.tem`, `.vdf`) by remapping from the 27,904-segment scheme to the 10,999 active-segment scheme. **Its wind/radiation output (`*_fixed.bin`) is wrong:** those files were truncated, not remapped. See `FORCING_FIX.md`. |
| `forcing_column_to_upper_20261008.ipynb` | Makes the correct radiation and wind forcing files: converts `*_remapped_column.bin` to `forcing_remapped\*_remapped_upper.bin` (layers K ≤ 7 keep their value, deeper layers 0), with checks. See `FORCING_FIX.md`. |
| `FORCING_FIX.md` | History of the forcing files (`_fixed` → `_remapped_column` → `_remapped_upper`), what is wrong with each, how the correct files were made and how the Baseline was verified. |

### archive/

| Notebook | Why archived |
|----------|--------------|
| `Torun_WAQ_Analysis_20260531.ipynb` | Earlier version of the main analysis notebook. **Superseded by** `notebooks/Torun_WAQ_Analysis_netcdf_20260603.ipynb` (June 3 adds NetCDF integration; 28 of its 37 code cells reappear in the June-3 version). Kept for provenance — not maintained. **Verify which version produced your thesis figures before relying on either.** |

## Model conventions (as used in the code)

These constants appear in the notebooks and are recorded here for reference
(values taken directly from the code, not from memory):

- Active WAQ segments: `N_SEG = 10999`
- Original FLOW segment count: `N_SEG_IN = 27904`
- (The communication-file tool remaps `27904 → 10999`.)

> Other conventions you have used elsewhere (e.g. `SURFACE_LAYER = 0`,
> 16 layers) are **not** all explicitly set as named constants in every
> notebook — confirm against each notebook's configuration cell rather than
> assuming.

## Paths

The original notebooks read from a network drive and a mapped `L:` drive. Those
absolute paths (which included a username) have been replaced with the
placeholder **`<MODEL_DIR>`**. Before running, set `<MODEL_DIR>` (or the `BASE`
/ `BIN_DIR` variables near the top of each notebook) to the folder containing
your model run, preserving the internal subfolder names the code expects
(`com_files/`, `01scen_baseline/`, `02scen/`, `04scen/`, `results_scenarios/`, etc.).

## Data and model output — NOT included

The model output files (`.map`, `.his`, `.nc`, communication binaries `.vol` /
`.tem` / `.vdf`, wind/radiation `.bin`) are **not** in this repository. They are
large binaries far exceeding GitHub's per-file limits and are inputs to, not
products of, this code. To reproduce the analysis you need the corresponding
Delft3D-WAQ run outputs placed under `<MODEL_DIR>`. Consider archiving the model
outputs separately (e.g. on an institutional store or Zenodo) and linking them
here.

## Notebook outputs

Large embedded figure images were stripped to keep the repository small; text
and table outputs were kept. Rerun a cell to regenerate its figure.

## Running

```bash
pip install -r requirements.txt
jupyter lab     # or jupyter notebook
```

Open a notebook, set `<MODEL_DIR>` in the configuration cell, and run.

## Citation / provenance — TO COMPLETE

- [ ] Thesis chapter title and number this code supports
- [ ] Associated publication (if any) and DOI
- [ ] How to cite this repository (consider a Zenodo release for a DOI)
- [ ] Where the underlying model outputs are archived

## Author

Sofia Midauar G Rocha — Lancaster Environment Centre, Lancaster University.
Supervisor: Andrew Folkard.

## Licence

Not yet chosen — see `LICENSE`. Select one before making the repository public.

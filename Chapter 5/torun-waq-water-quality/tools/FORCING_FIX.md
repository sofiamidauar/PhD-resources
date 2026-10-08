# Radiation and wind forcing files: history of the fix (Sep–Oct 2026)

**Use only `forcing_remapped\*_remapped_upper.bin`** for WAQ radiation and wind
forcing. Every earlier version of these files is wrong in some way; this note
explains what each one is, why it is wrong, and how the correct files were made
and checked.

## Summary

| File name pattern | Made by | Status | What is wrong |
|---|---|---|---|
| `waq-…` (no suffix, 27,904 values per record) | FLOW → WAQ coupling | Not usable | Built for the 27,904-segment FLOW scheme, not the 10,999 WAQ segments |
| `*_fixed.bin` | `tools/fix_communicationfiles_clean.ipynb` (cell 4) | **Wrong** – used by the calibrated Run42 | Truncated instead of remapped (see 1) |
| `*_remapped_column.bin` | `rebuild_forcing_bins_20261002` (not in this repository) with `FILL_LAYERS = 'column'` | **Wrong – do not use** | Surface value given to every layer (see 2) |
| `*_remapped_upper.bin` | `tools/forcing_column_to_upper_20261008.ipynb` | **Correct** | – |

## 1 — `*_fixed.bin`: truncated, not remapped

`fix_communicationfiles_clean.ipynb` remapped `.vol`, `.tem` and `.vdf` from
27,904 to 10,999 segments with `raw[new_to_old]`, but trimmed the radiation and
wind files by **keeping the first 10,999 values**. According to checks recorded
in `rebuild_forcing_bins_20261002` (25 Sep 2026):

* the untrimmed files hold values only in FLOW layer 0 (segments 1–1,744);
* so compact WAQ segment *i* received the FLOW-layer-0 value *i*, which belongs
  to a different horizontal position;
* 43 of the 1,357 surface (K = 7) cells received 0 in every run, the Baseline
  included.

For the Baseline (no FPV) the radiation field is spatially uniform, so the
horizontal mix-up made no difference there; only the 43 dark cells did. For the
FPV scenarios the shading pattern ended up in the wrong place.

The values in a `_fixed` file sit in compact segments 0–1,743, i.e. all of
layers K = 0–6 plus most of K = 7, and are 0 below. This is the layer
structure the calibration was done with.

## 2 — `*_remapped_column.bin`: right place, wrong layers

`rebuild_forcing_bins_20261002` fixed the horizontal mapping: each compact
segment gets the layer-0 value of its own (M, N) column. With
`FILL_LAYERS = 'column'`, however, **every** segment of the column got that
value, down to the bottom layer.

DELWAQ uses the value given to each segment. So in runs with these files every
layer received full surface radiation. The Baseline `.his` output showed this
(CAB, raw `.his` values, means over 1 Apr – 1 Sep 2023; BIS was the same):

| | calibrated Run42 (`_fixed`) | `_remapped_column` run |
|---|---|---|
| `RadAve`, layer 8 (top active) | 233.5 | 233.5 |
| `RadAve`, layers 9–16 | 0 | 233.5 |
| `Limit e`, layers 9–16 | 0.9935 | mostly 0 |
| phytoplankton (sum of BLOOM types), layer 8 | 0.303 | 2.851 |
| phytoplankton, layer 16 | 0.012 | 2.859 |

Phytoplankton was about 10× too high at the surface and over 200× too high at
the bottom. The wind files were built the same way (`FILL_LAYERS_WIND =
'column'`) and are wrong in the same way.

## 3 — `*_remapped_upper.bin`: the correct files

`tools/forcing_column_to_upper_20261008.ipynb` reads each `_remapped_column`
file and writes a `_remapped_upper` file in which layers K = 0–7 keep their
value and deeper layers are 0. This is the calibrated layer structure, with the
horizontal mapping fixed and the 43 dark surface cells now included.

### How the notebook works

1. **Configuration.** Set `BIN_DIR` to `wq_0.1min_secondfolder`. Inputs are the
   `*_remapped_column.bin` files directly in that folder (no subfolders).
   Outputs go to `BIN_DIR\forcing_remapped`. `PATTERN` selects the files:
   `'*_remapped_column.bin'` (all scenarios) or
   `'*noFPV*_remapped_column.bin'` (Baseline only). The cell lists the files it
   found and marks those whose output already exists.
2. **Layer of every segment.** Reads `segment_lookup.npz` (`segnum`,
   `new_to_old`) to get the (M, N, K) of each of the 10,999 compact segments,
   and prints how many segments per layer are kept.
3. **Convert.** Each record is an int32 timestamp + 10,999 float32 values.
   Timestamps are copied unchanged, segments with K > 7 are set to 0. The input
   is also checked: a `_column` file must have one value per (M, N) column.
   Records where it does not are counted, and the file is marked `CHECK`.
4. **Checks**, on one record per file (at 55 % of the file, summer for daily
   radiation): conversion status `OK`, timestamps equal, values in K ≤ 7
   identical to the input, all values below K = 7 zero, and the number of
   non-zero segments per layer. For summer radiation without full shading the
   last one should be `[104, 161, 45, 37, 25, 36, 22, 1357, 0, …, 0]`; fully
   shaded scenarios (e.g. `SRfactor000`) show fewer or none.

Safety: `_remapped_column` files are only read. Existing outputs are not
replaced unless `OVERWRITE = True`; outputs of the wrong size are reported as
incomplete. If a run is interrupted, re-run the notebook: finished files are
skipped.

## 4 — Verification of the Baseline

`notebooks/compare_baseline_his_20261008.ipynb` compares the `.his` of a run
with the calibrated Run42 (`his_scenario_files`).

* Set `BASE`, and `NEW_FOLDER` / `NEW_RUNID` for the run to check. `CUTOFF`
  limits the period (default 1 Sep 2023; `None` for the full run).
* Section 3: per layer at CAB and BIS, `RadAve`, `Limit e` and total
  phytoplankton, old vs new, with the % difference.
* Section 4: every `.his` variable at `CAB_averaged` and `BIS_averaged`,
  sorted by the size of the % change.
* Section 5: time series at CAB (phytoplankton layer 8, Chl-a, DO).

All values are raw `.his` values (no division by `Continuity`), which is
appropriate for process outputs such as `RadAve` and for comparing two runs.

Result for the Baseline with `_remapped_upper` files (8 Oct 2026, same period):
`RadAve` and `Limit e` identical to Run42 in every layer at CAB and BIS;
phytoplankton 1.6–6.5 % higher (largest in layers 9–11). The difference is
attributed to the 43 surface cells that now receive light and wind, and possibly
to the rebuilt wind file; this was not separated further. Based on this and the
section 4 comparison, the calibration was judged to still hold.

The same notebook can be used to check a scenario run against the new
Baseline: change `OLD_FOLDER` / `NEW_FOLDER`.

## Next steps (as of 8 Oct 2026)

1. Conversion of all FPV scenario files to `_remapped_upper` was started on
   8 Oct 2026. Record here whether every file passed the checks.
2. Point each scenario's `.inp` at its two `forcing_remapped\*_remapped_upper.bin`
   files (radiation and wind) and re-run all scenarios. The FPV results from
   `_fixed` or `_remapped_column` files must not be used.

## Open questions

* **No light below the top active layer.** In both the calibrated run and the
  `_upper` runs, `RadAve` is 0 in layers 9–16. Physically, light should
  decrease with depth rather than stop. One possible explanation, **not
  verified**, is that DELWAQ does not pass light down from the layer above,
  e.g. because the segment attributes (surface / middle / bottom) are missing
  or all default. Check the attributes block of the `.inp` or the `.lst` file
  and the D-Water Quality process documentation.
* **Wind in deep layers.** It was not checked which processes in this setup use
  `VWind` per segment (e.g. reaeration), or how much the `_column` wind files
  affected those runs.
* `rebuild_forcing_bins_20261002` (which made the `_column` files) is not in
  this repository. Its mapping (compact segment → column's layer-0 value) is
  correct; only its `'column'` option is wrong.

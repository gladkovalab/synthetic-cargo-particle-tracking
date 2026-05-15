# Synthetic cargo particle tracking

Jupyter notebook for analysing single-particle tracks of the Miro1 synthetic cargo across TRAK isoform conditions. Reads the per-cell CSV exports produced by the Fiji [TrackMate](https://imagej.net/plugins/trackmate/) plugin, rotates each cell's tracks to a common axis so backwards-vs-forwards is well-defined, and produces per-condition speed distributions and forward/backward proportions.

Companion to the imaging-pipeline repos under [`gladkovalab`](https://github.com/gladkovalab) — see the paper for the full set.

## What the notebook does

For each `(date, condition, cell)`:

1. Loads the per-cell TrackMate output (`tracks.csv`, `edges.csv`, `spots.csv`).
2. Looks up that cell's rotation angle in `{date}_cell_orientation_coordinates.xlsx` (one row per cell, hand-annotated for the cell's microtubule polarity).
3. Rotates the track coordinates by `−angle` so the cell's "forward" axis points along `+y`.
4. Records per-track stats (max distance, mean/max/min speed, final y position).

Then across all cells:

- Plots example tracks per cell to verify the rotation is correct.
- Computes the distribution of *final y position* per track for TRAK1 vs. TRAK2 (positive = anterograde / kinesin-direction, negative = retrograde / dynein-direction).
- Compares per-edge speeds split by `final_y_is_above_zero` to test whether anterograde and retrograde edges move at different speeds within each condition.
- Exports `final_y_comparison.csv` and per-condition track-speed tables for plotting outside the notebook.

The three conditions are encoded by directory name: `no_TRAK_77`, `TRAK1_79`, `TRAK2_78`.

## Expected data layout

The notebook expects the following structure relative to the repository (one level up from the notebook):

```
../input_folder/
└── tracking_results_sub_pixel/
    └── {date}/                                      # 231027, 231102, 231103, 231117
        ├── {date}_cell_orientation_coordinates.xlsx # angles_per_cell sheet
        ├── no_TRAK_77/
        │   └── {date}_{cell_id}/                    # e.g. 231103_06/
        │       ├── tracks.csv
        │       ├── edges.csv
        │       └── spots.csv
        ├── TRAK1_79/
        │   └── ...
        └── TRAK2_78/
            └── ...
```

Output is written under `../plots/{date}/` (track diagnostics, per-cell) and `../output_folder/tracking_results_sub_pixel/` (the `final_y_comparison.csv` and any other aggregated tables).

The `cell_orientation_coordinates.xlsx` workbook needs an `angles_per_cell` sheet with columns `Condition`, `Cell`, and `angle (rad)`. Cells without a rotation angle are skipped.

## Running

This is a plain Jupyter notebook with no compiled or packaged components. Open it with:

```bash
jupyter notebook trak_mediated_miro1_cargo_tracking.ipynb
```

or

```bash
jupyter lab trak_mediated_miro1_cargo_tracking.ipynb
```

### Dependencies

- Python 3.11+
- `numpy`, `pandas`, `polars`, `matplotlib`, `seaborn`
- `fastexcel` (used by polars to read the orientation xlsx; install with `pip install fastexcel` if `polars.read_excel` complains)
- `openpyxl` is an acceptable fallback xlsx engine

No `pyproject.toml` is committed — pick whatever Python environment manager suits you (venv + pip, uv, conda, etc.).

## TrackMate inputs

The per-cell CSVs are the standard outputs of the [Fiji TrackMate](https://imagej.net/plugins/trackmate/) plugin's *Export tracks to CSV* action, with these key columns consumed by the notebook:

- From `spots.csv`: `FRAME`, `POSITION_X`, `POSITION_Y`, `POSITION_T`, `TRACK_ID`, plus per-track aggregates (`MAX_DISTANCE_TRAVELED`, `TRACK_MEAN_SPEED`, `TRACK_MAX_SPEED`, etc.) that TrackMate replicates onto every spot row.
- From `edges.csv`: `SPEED`, `DISPLACEMENT`, `DIRECTIONAL_CHANGE_RATE`, `TRACK_ID`.

The notebook does not call TrackMate; it consumes the pre-computed CSVs.

## License

MIT — see [`LICENSE`](LICENSE).

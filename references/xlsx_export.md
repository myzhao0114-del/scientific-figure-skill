# Process-data xlsx export

How to write intermediate dataframes into `process_data/` so every number in the final PDF can be traced.

## File naming convention

Number files so they sort in execution order:

```
01_raw_loaded.xlsx        # input as pandas read it
02_cleaned.xlsx           # after dedup / type fixes / missing-handling
03_<cut>.xlsx             # one per analytical cut
04_<cut>.xlsx
05_statistical_tests.xlsx # raw test outputs (Step 5)
fig_01_data.xlsx          # data sidecar for figure 1
fig_02_data.xlsx          # data sidecar for figure 2
...
99_summary_stats.xlsx     # describe() + custom aggregates
```

The `99_` prefix on summary stats always sorts last, regardless of how many cuts come before.

Figure data sidecars use the `fig_NN_data.xlsx` pattern so they pair visually with the figure files.

## Single-sheet write

```python
df.to_excel("process_data/02_cleaned.xlsx", index=False, sheet_name="cleaned")
```

Use `index=False` unless the index carries meaning (e.g. a datetime index for time series — then keep it).

## Multi-sheet write

When tables are logically grouped, one file with multiple sheets:

```python
with pd.ExcelWriter("process_data/99_summary_stats.xlsx", engine="openpyxl") as writer:
    df.describe().to_excel(writer, sheet_name="numeric")
    df.select_dtypes("object").describe().to_excel(writer, sheet_name="categorical")
    df.corr(numeric_only=True).to_excel(writer, sheet_name="correlations")
```

Sheet names: short, lowercase, snake_case, ≤ 31 characters (Excel limit).

## Statistical test outputs

`05_statistical_tests.xlsx` should have one sheet per comparison, named after the figure it supports:

```python
with pd.ExcelWriter("process_data/05_statistical_tests.xlsx", engine="openpyxl") as writer:
    # Raw test result for figure 2's pairwise comparisons
    pd.DataFrame([
        {"group_a": "A", "group_b": "B", "test": "Welch t",
         "statistic": 4.21, "p_value": 0.0012, "mark": "**"},
        {"group_a": "A", "group_b": "C", "test": "Welch t",
         "statistic": 2.87, "p_value": 0.018,  "mark": "*"},
        {"group_a": "B", "group_b": "C", "test": "Welch t",
         "statistic": 1.04, "p_value": 0.31,   "mark": "ns"},
    ]).to_excel(writer, sheet_name="fig_02_pairwise", index=False)
```

## Minimal formatting (apply to every file)

Two touches make the files readable without imposing style: freeze the header row, autosize columns.

```python
from openpyxl import load_workbook
from openpyxl.utils import get_column_letter

def polish_xlsx(path):
    wb = load_workbook(path)
    for ws in wb.worksheets:
        ws.freeze_panes = "A2"  # freeze header row
        for col_idx, col_cells in enumerate(ws.columns, start=1):
            vals = [str(c.value) if c.value is not None else "" for c in col_cells]
            width = min(max((len(v) for v in vals), default=10) + 2, 50)
            ws.column_dimensions[get_column_letter(col_idx)].width = width
    wb.save(path)
```

Call `polish_xlsx` on every file after writing. Do not add cell colors, borders, or bold headers — these are reference files, not presentation files.

## Encoding and dtypes on load

- For CSV input, try encodings in order: `utf-8` → `utf-8-sig` (BOM from Excel exports) → `gbk` → `gb18030` → `cp1252`. Wrap in try/except.
- Parse dates on load (`parse_dates=[...]`) so they survive the xlsx round-trip as real datetimes.
- For numeric columns stored as text (common from Excel exports), coerce explicitly with `pd.to_numeric(col, errors="coerce")` and note coercion failures in the cleaning summary.

## What counts as "meaningful intermediate data"

Save a file (or sheet) whenever the dataframe meaningfully differs from the previous step. The test: if a reviewer asks "where does the number 47,302 in figure 2 come from?", point at one file, one sheet, one cell.

**Worth saving:**
- Raw load (proves what came in)
- Post-cleaning (typed, deduped, missing-handled)
- Each aggregation feeding a chart
- Statistical test outputs
- Per-figure data sidecars

**Not worth saving:**
- Trivial transformations (renaming one column)
- Intermediate states not referenced downstream

# Configuration block template

Paste this block at the top of every analysis script. Users change settings here, not by hunting through code. All downstream steps read from these variables.

```python
# ============================================================
# CONFIGURATION — edit only this block
# ============================================================

from pathlib import Path

# --- Paths ---
INPUT_FILES = [
    "path/to/your_data.csv",   # one or more input files
]
OUTPUT_DIR = Path("outputs/analysis_<slug>")
# Keep outputs separate from raw inputs whenever possible.

# --- Project metadata (appears on PDF title page) ---
PROJECT_TITLE = "Untitled Analysis"

# --- Fonts (fallback chains — first available is used) ---
CN_SERIF_FALLBACK = [
    "SimSun",                 # Windows
    "Songti SC",              # macOS
    "Source Han Serif SC",    # Adobe / cross-platform
    "Noto Serif CJK SC",      # Linux
    "serif",                  # last-resort fallback
]
EN_SERIF_FALLBACK = [
    "Times New Roman",        # Windows / macOS
    "Liberation Serif",       # Linux
    "DejaVu Serif",           # matplotlib default
    "serif",
]

# --- Output formats (all three are produced) ---
OUTPUT_FORMATS = ["pdf", "png", "jpg"]
DPI = 300                     # for png and jpg

# --- Statistics ---
ALPHA = 0.05
SIGNIFICANCE_MARKS = {
    0.001: "***",
    0.01:  "**",
    0.05:  "*",
}                              # used as: pick the lowest threshold p is under
NS_LABEL = "ns"
ERROR_BAR = "SEM"             # one of: "SD", "SEM", "CI95"
NORMALITY_ALPHA = 0.05        # threshold for choosing parametric vs non-parametric

# --- Numeric precision ---
DEFAULT_DECIMALS = 3
SCIENTIFIC_LOW = 1e-3         # |x| < this → scientific notation
SCIENTIFIC_HIGH = 1e5         # |x| >= this → scientific notation
THOUSANDS_SEP = ","           # for non-scientific values >= 1000

# --- Figure sizing (inches) ---
FIG_SINGLE_COL = (3.5, 2.6)   # ~89 mm — standard journal single column
FIG_DOUBLE_COL = (7.2, 4.0)   # ~183 mm — standard journal double column

# ============================================================
# END CONFIGURATION
# ============================================================
```

## Notes on use

- **`OUTPUT_DIR` slug**: derive a short kebab-case description from the user's request (e.g. `analysis_sales-q3`, not `analysis_20260519`). Use a timestamp suffix only if the user explicitly wants to keep prior runs separate.
- **Font fallback resolution**: don't hard-code a font in `rcParams`. Resolve the first available one at runtime using `matplotlib.font_manager.fontManager.ttflist`. The pattern is in `chart_style.md`.
- **`SIGNIFICANCE_MARKS` lookup**: when annotating a p-value, walk thresholds ascending and return the mark for the smallest threshold p is below. If p ≥ 0.05, use `NS_LABEL`.
- **`ERROR_BAR`**: this is read by both the plotting code (to compute the right interval) and the caption generator (to name it in the "How" section). Changing it here changes both.
- **Sanity check at script start**: before doing anything, assert `OUTPUT_DIR.resolve() != Path(INPUT_FILES[0]).resolve().parent`. Refuse to proceed if they match.

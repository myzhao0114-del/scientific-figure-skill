# Chart style reference

Publication-grade matplotlib style for the `sci-figure` skill. Read this before drawing the first chart.

## Resolving fonts at runtime

Never hard-code a font that may not exist. Walk the fallback chains from the config block and pick the first one available.

```python
from matplotlib import font_manager
import matplotlib as mpl

def resolve_font(candidates):
    available = {f.name for f in font_manager.fontManager.ttflist}
    for name in candidates:
        if name in available or name == "serif":
            return name
    return "serif"

CN_FONT = resolve_font(CN_SERIF_FALLBACK)
EN_FONT = resolve_font(EN_SERIF_FALLBACK)
```

For mixed Chinese and English text in the same figure, list both fonts in the family so matplotlib can fall back correctly.

```python
mpl.rcParams["font.serif"] = [EN_FONT, CN_FONT, "serif"]
mpl.rcParams["axes.unicode_minus"] = False
```

## rcParams block

Apply once at the top of the analysis, after font resolution:

```python
mpl.rcParams.update({
    # Typography
    "font.family": "serif",
    "font.serif": [EN_FONT, CN_FONT, "serif"],
    "font.size": 10,
    "axes.titlesize": 11,
    "axes.labelsize": 10,
    "xtick.labelsize": 9,
    "ytick.labelsize": 9,
    "legend.fontsize": 9,
    "axes.unicode_minus": False,

    # Color
    "axes.prop_cycle": mpl.cycler(color=["#000000", "#555555", "#888888", "#bbbbbb"]),
    "axes.edgecolor": "#000000",
    "axes.labelcolor": "#000000",
    "xtick.color": "#000000",
    "ytick.color": "#000000",
    "text.color": "#000000",

    # Frame and grid
    "axes.spines.top": True,
    "axes.spines.right": True,
    "axes.spines.left": True,
    "axes.spines.bottom": True,
    "axes.grid": True,
    "axes.axisbelow": True,
    "grid.color": "#e5e5e5",
    "grid.linewidth": 0.5,
    "grid.linestyle": "-",

    # Background
    "axes.facecolor": "white",
    "figure.facecolor": "white",
    "savefig.facecolor": "white",
    "legend.frameon": False,

    # Output
    "figure.dpi": 100,
    "savefig.dpi": DPI,
    "savefig.bbox": "tight",
})
```

This skill uses a **full enclosed plotting frame** as the default house style. Keep all four spines visible unless the user explicitly asks for another convention or the target journal requires a different frame treatment.

## Full border requirement

All standard figures should show a four-sided frame around the plotting area.

```python
for side in ["top", "right", "left", "bottom"]:
    ax.spines[side].set_visible(True)
    ax.spines[side].set_linewidth(0.8)
    ax.spines[side].set_color("black")
```

Use the grid sparingly so it supports reading values without overpowering the border.

## Color palettes

### Default: grayscale

Use the grayscale cycle in the rcParams above. Distinguish series with line style and marker shape, not color.

```python
LINE_STYLES = ["-", "--", "-.", ":"]
MARKERS = ["o", "s", "^", "D"]
```

If more than four series are needed, prefer splitting the chart into small multiples rather than extending the palette.

### When color is unavoidable: Okabe-Ito

For categorical data where grayscale cannot disambiguate the groups, use the Okabe-Ito color-blind-safe palette.

```python
OKABE_ITO = [
    "#000000",
    "#E69F00",
    "#56B4E9",
    "#009E73",
    "#F0E442",
    "#0072B2",
    "#D55E00",
    "#CC79A7",
]
```

For sequential numeric data such as heatmaps, `viridis` and `cividis` are acceptable.

## Axis labels and units

Every numeric axis label that has a unit shows it in parentheses.

| Correct | Wrong |
|---|---|
| `Revenue (USD)` | `Revenue USD` |
| `Temperature (deg C)` | `Temperature in Celsius` |
| `Time (s)` | `Time seconds` |
| `Concentration (umol/L)` | `Concentration umol/L` |
| `Mass (kg)` | `kg` |

Dimensionless quantities omit the parenthetical, such as `Count`, `Ratio`, or `Proportion`.

```python
ax.set_xlabel("Time (s)")
ax.set_ylabel("Voltage (mV)")
```

## Numeric precision

Do not trust matplotlib's default tick formatting. Apply explicit rules:

```python
from matplotlib.ticker import FuncFormatter

def smart_format(x, _pos=None):
    if x == 0:
        return "0"
    ax_val = abs(x)
    if ax_val < SCIENTIFIC_LOW or ax_val >= SCIENTIFIC_HIGH:
        return f"{x:.{DEFAULT_DECIMALS}e}"
    if ax_val >= 1000:
        if x == int(x):
            return f"{int(x):{THOUSANDS_SEP}}"
        return f"{x:{THOUSANDS_SEP}.{DEFAULT_DECIMALS}f}"
    return f"{x:.{DEFAULT_DECIMALS}g}"

ax.yaxis.set_major_formatter(FuncFormatter(smart_format))
ax.xaxis.set_major_formatter(FuncFormatter(smart_format))
```

For p-values, show `p < 0.001` instead of rounding to zero.

```python
def format_p(p: float) -> str:
    if p < 0.001:
        return "p < 0.001"
    if p < 0.01:
        return f"p = {p:.3f}"
    return f"p = {p:.2f}"
```

## Error bars

When bars or points represent a summary statistic, error bars are required and their meaning must be named in the caption.

```python
def error_interval(values, kind="SEM"):
    n = len(values)
    if kind == "SD":
        return values.std(ddof=1)
    if kind == "SEM":
        return values.std(ddof=1) / np.sqrt(n)
    if kind == "CI95":
        from scipy import stats as sps
        return sps.t.ppf(0.975, df=n - 1) * values.std(ddof=1) / np.sqrt(n)
    raise ValueError(f"Unknown error bar type: {kind}")
```

Caption examples should say `mean ± SEM`, `mean ± SD`, or `95% CI` explicitly.

## Worked example: grouped bar chart with significance brackets

```python
fig, ax = plt.subplots(figsize=FIG_SINGLE_COL)

groups = ["A", "B", "C"]
means = df.groupby("group")["value"].mean().reindex(groups)
errs = df.groupby("group")["value"].apply(
    lambda s: error_interval(s, ERROR_BAR)
).reindex(groups)

x = np.arange(len(groups))
ax.bar(
    x,
    means.values,
    yerr=errs.values,
    color="#555555",
    edgecolor="black",
    linewidth=0.5,
    capsize=4,
    error_kw={"linewidth": 0.8, "ecolor": "black"},
)

ax.set_xticks(x)
ax.set_xticklabels(groups)
ax.set_xlabel("Treatment group")
ax.set_ylabel("Response (mg/L)")
ax.yaxis.set_major_formatter(FuncFormatter(smart_format))
ax.yaxis.grid(True)
ax.xaxis.grid(False)

for side in ["top", "right", "left", "bottom"]:
    ax.spines[side].set_visible(True)

y_max = (means + errs).max()
add_significance_bracket(ax, 0, 1, y_max * 1.08, "**")
add_significance_bracket(ax, 0, 2, y_max * 1.18, "*")
ax.set_ylim(top=y_max * 1.30)

save_figure(fig, "fig_02_response_by_group")
plt.close(fig)
```

## The `save_figure` helper

Use this at the end of every figure:

```python
def save_figure(fig, slug: str):
    for fmt in OUTPUT_FORMATS:
        path = OUTPUT_DIR / "figures" / fmt / f"{slug}.{fmt}"
        path.parent.mkdir(parents=True, exist_ok=True)
        fig.savefig(path, dpi=DPI if fmt != "pdf" else None, bbox_inches="tight")
```

## Common mistakes

- Do not call `ax.set_title()` for publication figures; titles belong in captions.
- Do not let seaborn defaults override the house style.
- Do not rotate labels into illegibility when a horizontal layout would work better.
- Do not use color encoding when grayscale already separates the series clearly.
- Do not forget `axes.unicode_minus = False` when Chinese fonts are involved.
- Do not keep only the left and bottom spines; this skill's default is a full boxed frame.
- Do not draw every `ns` bracket when that would clutter the figure.

# Statistical testing and significance annotation

When to test, which test to use, and how to mark significance on figures.

## When to test

Only when the planned figure is a **group comparison**: bar chart, box plot, violin plot, or any chart where the x-axis is categorical and the y-axis is numeric, and the user (implicitly or explicitly) wants to know whether groups differ.

**Do NOT test:**

- Line charts over time (no group comparison)
- Scatter plots (correlation, not group difference)
- Single-group distributions (histogram, KDE)
- Heatmaps
- Pie charts (don't make pie charts at all if avoidable)

If unsure whether the user wants statistical comparison, **ask** rather than guess.

## Test selection decision tree

Reads `findings["normality"]` from the data health report (Step 3) — don't re-run normality tests.

### Two groups

| Both groups normal? | Equal variance? | Paired? | Test |
|---|---|---|---|
| Yes | — | No | Welch's t-test (default, robust to unequal variance) |
| Yes | — | Yes | Paired t-test |
| No | — | No | Mann-Whitney U |
| No | — | Yes | Wilcoxon signed-rank |

Default to **Welch's t-test** when normality holds — it handles unequal variance gracefully and the loss of power vs Student's t is negligible.

### Three or more groups

| All normal? | Equal variance (Levene)? | Test | Post-hoc |
|---|---|---|---|
| Yes | Yes | One-way ANOVA | Tukey HSD |
| Yes | No | Welch's ANOVA | Games-Howell |
| No | — | Kruskal-Wallis | Dunn's test with Bonferroni |

### Paired vs independent

If group membership comes from the same subjects measured under different conditions (before/after, treatment A vs B on the same person), it's **paired**. Otherwise **independent**.

This is rarely obvious from the data alone. **Ask the user** when:
- Sample sizes are equal across groups (suspicious — could be paired)
- Column names suggest related conditions (`pre_*` / `post_*`, `t1` / `t2`)

## Code patterns

### Two-group, independent, normal:

```python
from scipy import stats as sps

group_a = df[df["group"] == "A"]["value"].dropna()
group_b = df[df["group"] == "B"]["value"].dropna()

stat, p = sps.ttest_ind(group_a, group_b, equal_var=False)  # Welch's
mark = significance_mark(p)
```

### Two-group, non-parametric:

```python
stat, p = sps.mannwhitneyu(group_a, group_b, alternative="two-sided")
```

### Three+ groups, ANOVA + Tukey:

```python
from scipy import stats as sps
from statsmodels.stats.multicomp import pairwise_tukeyhsd

groups = [df[df["group"] == g]["value"].dropna() for g in df["group"].unique()]
f_stat, p_omnibus = sps.f_oneway(*groups)

# Only do post-hoc if omnibus is significant
if p_omnibus < ALPHA:
    tukey = pairwise_tukeyhsd(df["value"], df["group"], alpha=ALPHA)
    # tukey.summary() gives the pairwise table
    pairwise = pd.DataFrame(
        data=tukey._results_table.data[1:],
        columns=tukey._results_table.data[0]
    )
```

### Three+ groups, non-parametric:

```python
import scikit_posthocs as sp

h_stat, p_omnibus = sps.kruskal(*groups)

if p_omnibus < ALPHA:
    pairwise = sp.posthoc_dunn(df, val_col="value", group_col="group",
                                p_adjust="bonferroni")
```

If `scikit_posthocs` is unavailable, either use a manual Dunn-style fallback or tell the user that the current environment needs that dependency before completing the post-hoc analysis.

## Significance mark lookup

```python
def significance_mark(p: float) -> str:
    """Walk thresholds ascending, return mark for smallest threshold p is below."""
    if p >= ALPHA:
        return NS_LABEL  # "ns"
    for threshold in sorted(SIGNIFICANCE_MARKS.keys()):
        if p < threshold:
            return SIGNIFICANCE_MARKS[threshold]
    return SIGNIFICANCE_MARKS[max(SIGNIFICANCE_MARKS.keys())]
```

With the default config, `p = 0.003` returns `**`, `p = 0.04` returns `*`, `p = 0.08` returns `ns`.

## Drawing significance brackets on the figure

Standard convention: horizontal bracket spanning the two compared groups, with the mark centered above.

```python
def add_significance_bracket(ax, x1, x2, y, mark, line_height=0.02, **kwargs):
    """Draw a horizontal bracket from x1 to x2 at height y, label centered above."""
    h = line_height * (ax.get_ylim()[1] - ax.get_ylim()[0])
    ax.plot([x1, x1, x2, x2], [y, y + h, y + h, y], lw=1.0, color="black")
    ax.text((x1 + x2) / 2, y + h * 1.2, mark,
            ha="center", va="bottom", fontsize=10)
```

When multiple comparisons share the figure, stack the brackets so they don't overlap. Increment `y` by ~8% of the y-range for each additional bracket.

Skip `ns` brackets when the figure already has several significant ones — drawing every non-significant comparison clutters the figure. Mention non-significant comparisons in the caption text instead.

## Documenting the test in the caption

The "How" segment of the caption (see `captions.md`) must name:

1. The test used (e.g. "Welch's t-test", "one-way ANOVA with Tukey HSD")
2. What n represents
3. What the error bars represent

Example:

> Bars represent mean ± SEM (n=12 measurements per group). Pairwise comparisons by Welch's t-test; *p<0.05, **p<0.01, ***p<0.001.

Without these, the figure is not interpretable and reviewers will flag it.

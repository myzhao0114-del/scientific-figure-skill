# Data health diagnosis (`00_data_health.md`)

Produced at Step 3, before any chart is drawn. This is itself a deliverable — a reviewer may ask for it.

## What the report covers

For each input dataframe:

1. **Shape & memory** — rows, columns, memory footprint
2. **Per-column inventory** — dtype, missing count and %, unique count
3. **Duplicate rows** — count of fully duplicated rows
4. **Numeric columns** — descriptive stats, normality test, outlier flag
5. **Categorical columns** — number of categories, top-5 frequencies, whitespace check
6. **Multicollinearity** — correlation matrix + VIF when ≥3 numeric columns

## Code template

```python
import pandas as pd
import numpy as np
from scipy import stats
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.tools.tools import add_constant

def health_report(df: pd.DataFrame, output_path: Path) -> dict:
    """Generate the data health report and save as markdown. Returns the
    findings dict so downstream code (e.g. test selection) can read normality."""

    lines = ["# Data Health Report", ""]
    findings = {"normality": {}, "vif_flags": [], "warnings": []}

    # --- 1. Shape ---
    lines += [
        "## Shape & Memory",
        f"- Rows: {len(df):,}",
        f"- Columns: {df.shape[1]}",
        f"- Memory: {df.memory_usage(deep=True).sum() / 1024**2:.2f} MB",
        "",
    ]

    # --- 2. Per-column inventory ---
    lines += ["## Per-column Inventory", ""]
    inventory = pd.DataFrame({
        "dtype": df.dtypes.astype(str),
        "missing": df.isna().sum(),
        "missing_pct": (df.isna().mean() * 100).round(2),
        "n_unique": df.nunique(),
    })
    lines.append(inventory.to_markdown())
    lines.append("")

    # Warning for high-missing columns
    high_missing = inventory[inventory["missing_pct"] > 30].index.tolist()
    if high_missing:
        findings["warnings"].append(
            f"Columns with >30% missing values: {high_missing}. "
            "Consider whether downstream analysis can rely on these."
        )

    # --- 3. Duplicates ---
    n_dup = df.duplicated().sum()
    lines += ["## Duplicate Rows", f"- Fully duplicated rows: {n_dup:,}", ""]

    # --- 4. Numeric columns ---
    numeric_cols = df.select_dtypes(include=np.number).columns.tolist()
    if numeric_cols:
        lines += ["## Numeric Columns", ""]

        stats_table = df[numeric_cols].describe().T
        stats_table["skew"] = df[numeric_cols].skew()
        lines.append(stats_table.round(3).to_markdown())
        lines.append("")

        # Normality
        lines += ["### Normality", ""]
        norm_rows = []
        for c in numeric_cols:
            s = df[c].dropna()
            if len(s) < 3:
                norm_rows.append((c, "n<3", None, "insufficient"))
                continue
            if len(s) < 5000:
                stat, p = stats.shapiro(s)
                test = "Shapiro-Wilk"
            else:
                stat, p = stats.kstest(s, "norm", args=(s.mean(), s.std()))
                test = "Kolmogorov-Smirnov"
            verdict = "normal" if p >= 0.05 else "not normal"
            findings["normality"][c] = (verdict == "normal")
            norm_rows.append((c, test, round(p, 4), verdict))
        lines.append(pd.DataFrame(norm_rows,
                     columns=["column", "test", "p_value", "verdict"]
                     ).to_markdown(index=False))
        lines.append("")

        # Outliers (IQR)
        lines += ["### Outliers (IQR rule)", ""]
        outlier_rows = []
        for c in numeric_cols:
            s = df[c].dropna()
            q1, q3 = s.quantile([0.25, 0.75])
            iqr = q3 - q1
            n_out = ((s < q1 - 1.5 * iqr) | (s > q3 + 1.5 * iqr)).sum()
            outlier_rows.append((c, n_out, round(n_out / len(s) * 100, 2)))
        lines.append(pd.DataFrame(outlier_rows,
                     columns=["column", "n_outliers", "pct"]
                     ).to_markdown(index=False))
        lines.append("")

    # --- 5. Categorical columns ---
    cat_cols = df.select_dtypes(include=["object", "category"]).columns.tolist()
    if cat_cols:
        lines += ["## Categorical Columns", ""]
        for c in cat_cols:
            top = df[c].value_counts().head(5)
            ws_issue = df[c].astype(str).str.strip().ne(df[c].astype(str)).any()
            lines += [
                f"### `{c}`",
                f"- Unique categories: {df[c].nunique()}",
                f"- Whitespace issues: {'yes' if ws_issue else 'no'}",
                "- Top 5:",
                top.to_markdown(),
                "",
            ]
            if ws_issue:
                findings["warnings"].append(
                    f"Column `{c}` has leading/trailing whitespace — consider stripping."
                )

    # --- 6. Multicollinearity ---
    if len(numeric_cols) >= 3:
        lines += ["## Multicollinearity", ""]

        corr = df[numeric_cols].corr().round(3)
        lines += ["### Correlation matrix", corr.to_markdown(), ""]

        # VIF — drop rows with any NaN for VIF calculation
        X = df[numeric_cols].dropna()
        if len(X) > len(numeric_cols) + 1:
            X_const = add_constant(X)
            vif_rows = []
            for i, c in enumerate(numeric_cols, start=1):
                try:
                    v = variance_inflation_factor(X_const.values, i)
                except Exception:
                    v = np.nan
                flag = ""
                if pd.notna(v):
                    if v > 10:
                        flag = "severe"
                        findings["vif_flags"].append((c, "severe", v))
                    elif v > 5:
                        flag = "moderate"
                        findings["vif_flags"].append((c, "moderate", v))
                vif_rows.append((c, round(v, 2) if pd.notna(v) else "n/a", flag))
            lines += [
                "### Variance Inflation Factor (VIF)",
                "VIF > 5 = moderate concern, > 10 = severe.",
                "",
                pd.DataFrame(vif_rows,
                    columns=["column", "VIF", "flag"]
                ).to_markdown(index=False),
                "",
            ]

    # --- Warnings summary ---
    if findings["warnings"]:
        lines += ["## ⚠ Warnings", ""]
        for w in findings["warnings"]:
            lines.append(f"- {w}")
        lines.append("")

    output_path.write_text("\n".join(lines), encoding="utf-8")
    return findings
```

## Reading the findings dict downstream

The `findings` dict is passed to Step 5 (statistics) so test selection can read `findings["normality"][col_name]` instead of re-running the normality test. This avoids redundant computation and inconsistent test choices.

## When to pause before continuing

After saving the report, summarize key findings to the user in 2–3 sentences. **Pause and ask** before producing charts if any of these are true:

- Any column has > 30% missing values AND that column appears in the user's analytical request
- VIF > 10 on any column AND the user mentioned regression/modeling
- The dataframe has < 10 rows after dropping NaN (statistical claims become questionable)

For non-blocking issues (a few outliers, mild skewness), note them in the chat summary but proceed.

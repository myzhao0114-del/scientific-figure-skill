---
name: sci-figure
description: Create publication-grade scientific figures and analysis deliverables from tabular data. Use when the user provides Excel, CSV, or similar structured data and wants charts, plotting, exploratory analysis, figure polishing, statistical annotations, captions, or a report suitable for a paper, thesis, dissertation, lab report, or formal submission. Trigger on requests about scientific plotting, standardized chart styling, publication formatting, statistical comparison figures, caption writing, figure export, or traceable analysis workflows.
---

# Scientific Figure Standards

Use this skill to turn spreadsheet-style data into a reproducible, publication-grade figure workflow. Prioritize rigor, consistency, and traceability over speed.

## Core promises

1. **Compliant**: use academic typography, unit labeling, export settings, and restrained color choices.
2. **Rigorous**: document statistical methods, error bars, and significance marks when group comparisons are shown.
3. **Traceable**: save the intermediate tables that support each figure so another reviewer can trace numbers back to the source data.

## When to use this workflow

Run the full workflow when both are true:

- The input is one or more structured data files such as `.xlsx`, `.xls`, or `.csv`.
- The user wants analysis, charts, figure generation, figure cleanup, or a formal report rather than a one-line answer.

For tiny one-off questions such as "what is the mean of column B", answer directly and skip the full workflow.

## Workflow

Execute these steps in order. Skip only when the user explicitly asks for a narrower deliverable.

### Step 1: Define a configuration block

Before writing analysis code, place a configuration block at the top of the script so paths, output formats, fonts, statistical thresholds, and precision rules are controlled in one place.

Read `references/config_template.md` and copy the template into the script before adding analysis logic.

### Step 2: Choose a separate output directory

Never write outputs into the raw input directory unless the user explicitly asks for that layout and the risk is acceptable. Prefer a sibling or child output directory dedicated to the run.

Default pattern:

- Derive a short kebab-case slug from the task, such as `cell-growth` or `survey-comparison`.
- Create an output folder for the run.
- Create `figures/` and `process_data/` subfolders inside it.

Recommended structure:

```text
<output_dir>/
├── 00_data_health.md
├── report.pdf
├── figures/
│   ├── pdf/
│   ├── png/
│   └── jpg/
└── process_data/
```

Tell the user where outputs will be written before proceeding.

### Step 3: Diagnose data health first

Before plotting anything, generate `00_data_health.md` for each analysis dataset. This is mandatory because plotting bad data neatly still produces bad science.

The report should cover:

- Dataset shape and memory footprint
- Column dtypes, missingness, and unique counts
- Duplicate rows
- Descriptive statistics for numeric columns
- Normality and outlier checks where relevant
- Category summaries for non-numeric columns
- Correlation and VIF checks when multiple numeric predictors are involved

Read `references/data_health.md` for the code pattern and markdown layout.

After generating the report, summarize the important findings for the user in a couple of sentences. Pause before continuing if the data shows serious issues that would undermine the analysis.

### Step 4: Clean and export process data

Clean the data based on the health report. Typical actions include:

- Drop exact duplicates
- Strip whitespace from text fields
- Parse dates
- Coerce numeric-looking text into numeric columns
- Handle missing values in a way that matches the analysis goal

Save meaningful intermediate tables into `process_data/` as numbered `.xlsx` files. A table is meaningful if it supports a figure, result, or reviewer question later.

Read `references/xlsx_export.md` for naming and formatting patterns.

### Step 5: Run statistics only when justified

Only run inferential statistics when the figure is intended to compare groups or conditions. Do not add p-values to line charts, scatter plots, or single-distribution plots unless the design truly calls for them.

Use the decision rules in `references/statistics.md` to choose the test. If paired versus independent samples is ambiguous, ask the user before committing to a test.

Save raw statistical outputs into `process_data/05_statistical_tests.xlsx` or a similarly clear file name.

### Step 6: Draw figures in publication style

Generate figures with matplotlib unless the user explicitly requests another plotting stack. Keep the visual style restrained and reviewer-friendly.

Read `references/chart_style.md` before drawing the first chart. Apply these rules consistently:

- Use serif fonts with runtime fallback for mixed Chinese and English text.
- Prefer grayscale; only use categorical color when monochrome would fail.
- Use a full boxed frame around the plotting area by default: top, right, left, and bottom spines should all be visible unless the user explicitly asks for another house style.
- Put measurement units in parentheses in axis labels.
- Apply explicit number formatting rather than matplotlib defaults.
- Use error bars only when they represent a clearly named summary interval.
- Do not place decorative titles inside the chart area.

For each figure, save:

- `figures/pdf/fig_<NN>_<slug>.pdf`
- `figures/png/fig_<NN>_<slug>.png`
- `figures/jpg/fig_<NN>_<slug>.jpg`

Also save the exact plotting dataframe for each figure in `process_data/fig_<NN>_data.xlsx`.

### Step 7: Write complete captions

Every figure needs a caption that states:

1. What is shown
2. How it was summarized or tested
3. What conclusion the reader should take away

Use the patterns in `references/captions.md`. Do not stop at description only; the caption should help interpretation.

### Step 8: Bundle the figures into a report when requested

When the user wants a deliverable report, combine the figures and captions into `report.pdf`.

Read `references/pdf_layout.md` for a reportlab-based layout template. If reportlab is unavailable, state that limitation and use a simpler fallback such as a figure-only PDF.

### Step 9: Present results clearly

When the work is done, present the main outputs in a concise order:

- `report.pdf` if one was created
- `00_data_health.md`
- The main figure files
- The key process-data files or the `process_data/` directory

Summarize what was analyzed, what the figures show, and where the outputs live.

## What to avoid

- Do not skip the data health report just because the table looks clean.
- Do not add significance marks to plots that are not genuine group comparisons.
- Do not rely on default matplotlib colors for publication figures.
- Do not drop the top or right border by default; this skill's house style is a four-sided enclosed frame.
- Do not put the figure title inside the chart if the caption already carries that information.
- Do not save JPG only; always keep a high-quality PDF or PNG version.
- Do not write captions that merely restate the axes.

## Reference files

Load these only when the matching step is active:

- `references/config_template.md`
- `references/data_health.md`
- `references/xlsx_export.md`
- `references/statistics.md`
- `references/chart_style.md`
- `references/captions.md`
- `references/pdf_layout.md`

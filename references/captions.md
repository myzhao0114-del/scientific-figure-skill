# Caption template — the three-part rule

Every figure caption follows the three-part structure. Captions without all three parts are incomplete and the figure is not ready.

## Structure

```
Figure N. [WHAT]. [HOW]. [SO WHAT].
```

Three sentences (or one sentence per part), separated by periods, continuous prose. Not bullets.

### Part 1 — What

One phrase naming what is plotted. The axes and the population.

- ✓ "Monthly revenue across three sales regions, January 2023 to December 2024."
- ✗ "A line chart showing data." (uninformative)
- ✗ "Revenue." (incomplete)

### Part 2 — How

The methodology a reviewer needs to interpret the marks on the chart:

- **n** — sample size, per group if applicable
- **Error bar meaning** — SD, SEM, or 95% CI
- **Statistical test** — only if significance is marked
- **Significance threshold meanings** — `*p<0.05, **p<0.01, ***p<0.001`

Examples:

- "Bars represent mean ± SEM (n=12 per group). Pairwise comparisons by Welch's t-test; *p<0.05, **p<0.01."
- "Points represent individual measurements; lines show the median of each group (n=24 per condition)."
- "Shaded region denotes 95% confidence interval from 1000 bootstrap iterations."

If no statistical test was performed, omit that clause — don't fabricate one.

### Part 3 — So what

One sentence stating the takeaway. What does the reader walk away knowing?

- ✓ "Revenue in Region A significantly exceeded both B and C across the entire period."
- ✓ "Response increased monotonically with dose, with no plateau observed at the highest concentration tested."
- ✗ "The chart shows three lines." (descriptive, not interpretive)
- ✗ "Region A is higher." (vague)

The "So what" is the hardest part because it requires actually reading the data. Don't write it from a template — write what the chart shows.

## Worked examples

### Bar chart with group comparisons

> **Figure 2.** Mean response across treatment groups A, B, and C. Bars represent mean ± SEM (n=12 measurements per group). Pairwise comparisons by Welch's t-test; *p<0.05, **p<0.01. Group A response was significantly higher than both B and C, while B and C did not differ.

### Line chart, time series, no statistical test

> **Figure 1.** Monthly revenue across three sales regions, January 2023 to December 2024. Lines show monthly totals; line styles distinguish regions (solid: A; dashed: B; dotted: C). Region A maintained the highest revenue throughout, with all three regions showing seasonal peaks in Q4.

### Scatter plot with regression

> **Figure 3.** Relationship between dose and response (n=48 subjects). Points represent individual measurements; the solid line shows the ordinary least-squares fit with the shaded band denoting the 95% confidence interval. Response increased approximately linearly with dose (R²=0.78, p<0.001).

### Box plot

> **Figure 4.** Distribution of measurement values across the four conditions (n=30 per condition). Boxes show the interquartile range with the median as the central line; whiskers extend to 1.5×IQR, with outliers plotted individually. Comparisons by one-way ANOVA with Tukey HSD post-hoc; ***p<0.001. Condition D showed both a higher median and wider variance than the other three conditions.

## Caption length

Captions are typically 2–4 sentences. They can be longer if the figure is information-dense, but each sentence must earn its place. A 6-sentence caption usually signals the figure is trying to do too much — consider splitting it.

## Language

- English captions: serif font, italic, 9 pt, justified.
- Chinese captions: same formatting; use 全角 punctuation (。,;) consistently, not mixed half/full width.
- Captions are in the same language as the rest of the report. Don't mix unless the user explicitly wants bilingual captions.

## What to do when the "So what" is hard

If you can't write the "So what" sentence, one of three things is true:

1. **The chart isn't worth showing.** Cut it.
2. **The chart shows the wrong thing.** Replot the right comparison.
3. **You haven't actually looked at the data.** Look.

The "So what" sentence is a forcing function — it makes you confront whether the figure has a point.

## Common mistakes

- **Writing the title inside the figure AND in the caption.** Caption only.
- **"Error bars represent error bars."** Always name them: SD/SEM/CI95.
- **Significance marks without naming the test.** Reviewers will reject.
- **Captions that only describe.** Always include interpretation.
- **Bullets in captions.** Continuous prose only — captions read like a paragraph.

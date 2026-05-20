# Sci-Figure

中文 | [English](#english)

## 中文

### 项目简介

`Sci-Figure` 是一套面向科研绘图与数据分析的 Codex Skill，用来把 Excel、CSV 等表格数据转成更加规范、可追溯、适合论文或正式报告使用的图件与分析结果。

它不只是“帮你画图”，而是把科研图件常见的规范要求一起打包进工作流里，例如：

- 先做数据体检，再开始绘图
- 统一字体、单位、数字精度和导出格式
- 默认使用上下左右全包围边框
- 需要统计比较时，补充检验方法、显著性标记和说明
- 为每张图保留中间数据，方便追溯和复核
- 支持图注、过程数据和 PDF 报告式交付

### 适用场景

这个 Skill 适合下面这些任务：

- 根据 Excel / CSV 数据生成论文图
- 对已有数据做探索性分析并输出规范图件
- 为实验结果补齐统计注释和图注
- 将原始数据、清洗数据、作图数据和最终图件组织成可复核流程
- 提高科研绘图的一致性、可读性和投稿规范性

### 核心特点

- 规范化：统一字体、单位、边框、配色、精度和导出规则
- 可追溯：保留 `process_data/` 中间表格，便于检查每张图的数据来源
- 可复用：按步骤拆分为多个参考文档，方便扩展和维护
- 可交付：支持生成图件、图注、过程数据和报告式输出

### 工作流程

`Sci-Figure` 的默认流程包括：

1. 读取并检查输入数据
2. 生成 `00_data_health.md` 数据体检报告
3. 清洗并导出中间数据
4. 根据任务需要进行统计分析
5. 按统一样式绘制图件
6. 输出 PDF / PNG / JPG 多格式文件
7. 生成图注和可选的 PDF 报告

### 项目结构

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── captions.md
│   ├── chart_style.md
│   ├── config_template.md
│   ├── data_health.md
│   ├── pdf_layout.md
│   ├── statistics.md
│   └── xlsx_export.md
└── sci-figure.zip
```

### 使用方式

将这个 Skill 放入 Codex 可发现的 skills 目录后，在涉及科研绘图、论文图件、统计注释、图注撰写等任务时调用即可。

典型请求示例：

- 用这个 skill 帮我把 Excel 数据画成适合论文投稿的图
- 对这份 CSV 做分析并输出规范图件
- 基于 `year` 和 `Chlorophyll A` 做回归并出图
- 基于 `AQI` 和 `AI` 做 GAM，并导出过程数据和结果图

### 设计目标

这个项目的目标不是追求“最快出图”，而是追求：

- 图件更规范
- 过程更透明
- 结果更容易复查
- 输出更适合科研写作和正式汇报

### 适合放在 GitHub 的一句话介绍

面向科研绘图的 Codex Skill，提供从数据体检、统计注释到论文级图件输出的一体化规范工作流。

---

## English

### Overview

`Sci-Figure` is a Codex Skill for scientific plotting and data-analysis workflows. It helps turn Excel, CSV, and other tabular datasets into cleaner, more standardized, and more traceable figures suitable for papers, theses, and formal reports.

It is not only a plotting helper. It also packages common scientific-figure requirements into one workflow, including:

- checking data quality before plotting
- standardizing fonts, units, precision, and export formats
- using a four-sided boxed frame by default
- adding statistical annotations and method descriptions when needed
- preserving intermediate data for traceability
- supporting captions, process-data exports, and report-style outputs

### Use Cases

This Skill is useful when you want to:

- generate publication-style figures from Excel or CSV files
- run exploratory analysis and produce standardized scientific charts
- add statistical labels and figure captions to research outputs
- keep raw data, cleaned data, plotting data, and final figures in one reviewable pipeline
- improve consistency and submission readiness for academic graphics

### Key Features

- Standardized styling: fonts, units, borders, color usage, precision, and export rules
- Traceable outputs: intermediate tables are saved in `process_data/`
- Reusable structure: the workflow is split into reference files for maintenance and extension
- Deliverable-ready: supports figures, captions, process data, and report-style outputs

### Workflow

The default `Sci-Figure` workflow includes:

1. reading and checking the input data
2. generating a `00_data_health.md` data-health report
3. cleaning and exporting intermediate data
4. running statistical analysis when needed
5. drawing figures with a unified publication-style layout
6. exporting PDF / PNG / JPG outputs
7. generating captions and, when requested, a PDF report

### Repository Structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── captions.md
│   ├── chart_style.md
│   ├── config_template.md
│   ├── data_health.md
│   ├── pdf_layout.md
│   ├── statistics.md
│   └── xlsx_export.md
└── sci-figure.zip
```

### How to Use

Place this Skill in a Codex-discoverable skills directory, then invoke it for tasks involving scientific plotting, paper-ready figures, statistical annotations, or caption generation.

Typical prompts:

- Use this skill to turn my Excel data into publication-grade figures
- Analyze this CSV and generate standardized scientific charts
- Run a regression on `year` and `Chlorophyll A` and make the figure
- Fit a GAM on `AQI` and `AI`, then export the figure and process data

### Design Goal

This project does not aim to produce the fastest possible chart. Its goal is to produce outputs that are:

- more standardized
- more transparent
- easier to review
- better suited for academic writing and formal reporting

### One-Line GitHub Description

A Codex Skill for scientific plotting that combines data-health checks, statistical annotation, and publication-grade figure output in one reproducible workflow.

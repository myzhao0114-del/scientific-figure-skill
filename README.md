# scientific-figure-skill

> Tell it what data to use and what figure to make, and it will give you a cleaner, more standard, more publication-ready scientific figure.

## Preview

<img width="3661" height="2480" alt="OLS" src="https://github.com/user-attachments/assets/93c2d8c1-5255-4bb0-aec8-d7448f3e4536" />
<img width="2377" height="1160" alt="柱状图" src="https://github.com/user-attachments/assets/ccc45362-6a0b-4367-94a1-70e3236bdfa9" />


中文 | [English](#english)

---

## 中文

### 项目介绍

为什么做这个项目
做这个项目,是为了把科研绘图规范化。
我把自己这些年画图踩过的坑、总结的经验,全部写进了这个 skill。
如果你也用过 AI 直接出图,你应该见过它们——线条粗细随机、字体莫名其妙、配色花里胡哨、坐标轴没单位、误差线不知道代表什么。看着花哨,放到论文里一张都不能用,改细节能改到怀疑人生。

scientific-figure-skill 想解决的就是这个问题。我把那些"应该这样做"的规范,变成了 skill 里的默认行为:

✅ 字体规范:中文宋体 + 英文 Times New Roman,跨平台 fallback 链不翻车

✅ 数据前处理:共线性(VIF)、缺失值、异常值、正态性,画图前先做一份诊断报告

✅ 配色规范:深沉不显眼、简约——默认黑白灰,必要时用色盲友好的 Okabe-Ito

✅ p 值显示规则:p < 0.001,不是 p = 0.0000;有效数字、千分位、科学计数法全套规则

✅ 统计标注:*/**/***/ns 标准标注 + 图注里点名测试方法,审稿人无话可说

✅ 过程数据全部保留:从原始数据到每张图用的那张表,全部导出 xlsx,每个数字都能追溯到源

✅ 配置区前置:所有参数(字体、颜色、精度、误差线类型)在脚本开头集中配置,改一处,全流程生效

用法很简单。你只要说一句话——「用什么数据做什么图」——就能得到一张非常标准、美观、规范的图。

### 融入到 skill 里的经验

这里面融进去的不是“AI 会不会画图”这种问题，而是“图拿去投稿、汇报、答辩时会不会被挑毛病”。

比如：

- 字体默认按中文和英文分开处理，避免混乱
- 配色默认走克制、简约、深沉路线，不追求花哨
- 默认保留所有过程数据，保证每个数字都能追溯
- 先做数据体检，再决定怎么分析、怎么画
- 图默认四边全包围边框，而不是只留左下两条轴
- p 值、显著性、误差线、图注这些细节，尽量一次到位

### 用起来到底有多简单

用这个 skill，很多时候你真的只要说一句话：

> 用什么数据，做什么图。

比如：

- 用这个 Excel 做回归图
- 用 AQI 和 AI 做广义加性模型并出图
- 用这份 CSV 画箱线图并加统计显著性
- 用 year 和 Chlorophyll A 做趋势分析

然后它会尽量把数据检查、作图规范、图注、导出格式和过程数据一起处理好，最后给你一套更标准、更美观、更规范的结果。

### 这个 skill 会替你做什么

上传一份 Excel 或 CSV，告诉它你想分析什么，它通常会：

1. 先做数据体检，输出 `00_data_health.md`
2. 检查缺失项、异常值、共线性、分布情况
3. 清洗并保存中间过程数据
4. 按统一规范绘图
5. 输出 PDF / PNG / JPG 多格式图件
6. 生成规范图注
7. 保留统计结果和作图数据，方便追溯
8. 需要时打包成报告式结果

### 为什么图会更“像论文图”

因为这个项目默认盯的是那些最容易让科研图“不像论文图”的细节：

- 字体不乱
- 单位齐全
- 配色不浮夸
- 边框完整
- 数字精度统一
- p 值写法规范
- 图注不只描述，还会交代方法和结论
- 所有过程数据都留档

### 使用方式

#### 1. 让 Codex 直接安装这个 skill

如果你希望 Codex 直接读取并安装这个 skill，可以使用下面的命令。

PowerShell:

```powershell
git clone https://github.com/myzhao0114-del/scientific-figure-skill.git "$HOME\.codex\skills\scientific-figure-skill"
```

Bash:

```bash
git clone https://github.com/myzhao0114-del/scientific-figure-skill.git "$HOME/.codex/skills/scientific-figure-skill"
```

如果目录已经存在，可以先删除旧版本或改成你自己的目标目录。

#### 2. 安装后怎么用

安装完成后，在 Codex 里直接提出任务即可，例如：

```text
用这个 skill 帮我把 Excel 数据画成适合论文投稿的图
```

或者：

```text
读取这个表，用 AQI 和 AI 做 GAM 并出图
```

你不需要先手动把所有规范都说一遍，这个 skill 会尽量按内置标准处理。

### 项目结构

```text
scientific-figure-skill/
├── SKILL.md
├── README.md
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

### 适合谁用

- 需要论文图、投稿图、答辩图的研究生
- 想把绘图流程尽量标准化的人
- 经常被字体、配色、图注、统计标注折腾的人
- 想让 AI 出图结果更像真正可交付成果的人

### Roadmap

- [ ] 增加更多期刊风格预设
- [ ] 增加更丰富的统计出图模板
- [ ] 增加更多报告式输出模板
- [ ] 继续补充科研绘图里那些真正痛的细节规范

### 支持一下

如果你觉得这个项目有用，欢迎点一个 `Star`。

**Star 是我继续维护它的唯一燃料。**

---

## English

### Overview

I built this project to make scientific plotting more standardized.

Over the years, I have spent far too much time fixing figures that were technically “done” but still not ready for a paper, a thesis, a defense slide deck, or a formal report.

A lot of AI-generated figures look acceptable at first glance, but once you try to use them seriously, the problems start showing up:

- mixed Chinese and English fonts
- unclear axis units
- noisy or overly flashy colors
- non-standard p-value formatting
- ambiguous error bars
- captions that describe but do not conclude
- no preprocessing checks for missing values, outliers, or collinearity
- no saved process data for traceability

In practice, the biggest time sink is often not making the figure, but repeatedly fixing all those details afterward.

So I turned many years of plotting, revising, reworking, and submission experience into this skill. It is not just a tool that “draws a chart”. It tries to turn figure-quality rules into default behavior.

### What problem it solves

`scientific-figure-skill` solves a very practical problem:

**AI can generate a figure, but it does not automatically make that figure publication-ready.**

This skill is designed to reduce repeated manual cleanup by standardizing things such as:

- font rules for Chinese and English text
- data preprocessing and health checks
- collinearity, missing-value, outlier, and distribution checks
- restrained color rules
- p-value display conventions
- error-bar and significance annotation rules
- caption structure
- full retention of process data
- a front-loaded configuration block

### Experience built into the skill

What is built into this skill is not just plotting logic. It is the kind of experience that comes from having figures questioned, revised, and refined many times.

For example:

- fonts are handled with separate Chinese and English fallbacks
- colors default to restrained, simple, low-noise styles
- all process data is kept so every number can be traced back
- data health checks happen before plotting
- figures default to a full four-sided boxed frame
- p-values, significance marks, error bars, and captions are pushed toward one-pass correctness

### How simple it should feel

In many cases, you only need to say one sentence:

> what data to use, and what figure to make

Examples:

- use this Excel file to make a regression figure
- fit a GAM on AQI and AI and make the plot
- use this CSV to draw a box plot with significance annotations
- analyze year and Chlorophyll A and make a trend figure

The skill then tries to handle the data checks, plotting standards, captions, export formats, and process-data outputs in one workflow.

### What the skill does for you

Given an Excel or CSV file and a plotting request, it typically:

1. generates a `00_data_health.md` report
2. checks missing values, outliers, collinearity, and distribution shape
3. cleans and saves intermediate process data
4. draws figures under a unified house style
5. exports PDF / PNG / JPG versions
6. writes structured captions
7. keeps statistical results and plotting data for traceability
8. bundles outputs into report-style deliverables when needed

### Why the figures look more like paper figures

Because the skill pays attention to the details that most often make figures look unpolished:

- consistent fonts
- complete units
- restrained colors
- enclosed borders
- unified numeric precision
- standard p-value formatting
- captions that explain both method and takeaway
- full process-data retention

### How to Use

#### 1. Let Codex install this skill directly

If you want Codex to install this skill from GitHub, use one of the following commands.

PowerShell:

```powershell
git clone https://github.com/myzhao0114-del/scientific-figure-skill.git "$HOME\.codex\skills\scientific-figure-skill"
```

Bash:

```bash
git clone https://github.com/myzhao0114-del/scientific-figure-skill.git "$HOME/.codex/skills/scientific-figure-skill"
```

If the directory already exists, remove the old copy first or clone into a different target path.

#### 2. After installation

Once installed, you can directly ask Codex to do things like:

```text
Use this skill to turn my Excel data into publication-grade figures.
```

or:

```text
Read this table, fit a GAM on AQI and AI, and make the figure.
```

### Repository Structure

```text
scientific-figure-skill/
├── SKILL.md
├── README.md
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

### Who this is for

- graduate students preparing figures for papers, theses, or defenses
- researchers who want more standardized plotting workflows
- people tired of repeatedly fixing fonts, color choices, captions, and statistics formatting
- anyone who wants AI-generated figures to feel more like deliverable scientific outputs

### Roadmap

- [ ] Add more journal-style presets
- [ ] Add more statistical figure templates
- [ ] Add more report-style output templates
- [ ] Keep encoding more real-world scientific plotting experience into the skill

### Support

If you find this project useful, please consider giving it a `Star`.

**Stars are the only fuel that keeps me maintaining it.**

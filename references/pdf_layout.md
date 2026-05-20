# PDF layout reference

How to bundle the figures into `report.pdf` using reportlab.

## When to use reportlab vs PdfPages

- **reportlab** (default) — title page, styled captions, controlled margins, mixed CN/EN typography
- **PdfPages** — only when user explicitly says "just dump the figures, no report layout"

Prefer reportlab when it is available because it gives better title-page and caption control. If it is unavailable, fall back to a simpler PDF assembly approach and state that limitation.

## Template

```python
from reportlab.lib.pagesizes import A4
from reportlab.lib.units import cm
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib.enums import TA_CENTER, TA_LEFT, TA_JUSTIFY
from reportlab.platypus import (
    SimpleDocTemplate, Image, Paragraph, Spacer, PageBreak
)
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

# Register the resolved Chinese font for reportlab (independent of matplotlib)
def register_cn_font():
    """Find the CN font's TTF file and register with reportlab."""
    from matplotlib.font_manager import findfont, FontProperties
    try:
        path = findfont(FontProperties(family=CN_FONT), fallback_to_default=False)
        pdfmetrics.registerFont(TTFont("CNSerif", path))
        return "CNSerif"
    except Exception:
        return None  # fallback: reportlab default

CN_REPORTLAB_FONT = register_cn_font()

PAGE_W, PAGE_H = A4
MARGIN = 2.5 * cm
MAX_IMG_WIDTH = PAGE_W - 2 * MARGIN

# Styles — Times-Italic for captions matches the academic convention
title_style = ParagraphStyle(
    "title", fontName="Times-Roman", fontSize=18,
    alignment=TA_CENTER, spaceAfter=12, leading=22,
)
meta_style = ParagraphStyle(
    "meta", fontName="Times-Roman", fontSize=10,
    alignment=TA_CENTER, textColor="#555555", leading=14,
)
caption_style = ParagraphStyle(
    "caption", fontName="Times-Italic", fontSize=9,
    alignment=TA_JUSTIFY, leading=12, spaceBefore=10,
)

def build_pdf(title: str, source_names: list, figures: list, out_path: Path):
    """
    figures: list of (pdf_path, caption_text) tuples in display order.
    caption_text already contains the three-part caption (What. How. So what.).
    """
    doc = SimpleDocTemplate(
        str(out_path), pagesize=A4,
        leftMargin=MARGIN, rightMargin=MARGIN,
        topMargin=MARGIN, bottomMargin=MARGIN,
    )
    story = []

    # --- Title page ---
    story.append(Spacer(1, 6 * cm))
    story.append(Paragraph(title, title_style))
    story.append(Spacer(1, 0.5 * cm))
    story.append(Paragraph(f"Source: {', '.join(source_names)}", meta_style))
    story.append(Paragraph(f"Generated: {datetime.now():%Y-%m-%d}", meta_style))
    story.append(PageBreak())

    # --- Figure pages ---
    # NOTE: embedding the PDF version is preferred (vector). If the version
    # produced by matplotlib is not embeddable directly by reportlab, fall back
    # to the PNG (which is at full DPI anyway).
    for fig_num, (fig_path, caption) in enumerate(figures, start=1):
        img_path = fig_path
        if str(fig_path).endswith(".pdf"):
            # reportlab can't directly embed PDF as an Image — use PNG sibling
            img_path = str(fig_path).replace("/pdf/", "/png/").replace(".pdf", ".png")

        img = Image(img_path)
        aspect = img.imageHeight / img.imageWidth
        img.drawWidth = MAX_IMG_WIDTH
        img.drawHeight = MAX_IMG_WIDTH * aspect

        # Cap height — if image is taller than ~75% of page, scale down further
        max_h = (PAGE_H - 2 * MARGIN) * 0.75
        if img.drawHeight > max_h:
            img.drawHeight = max_h
            img.drawWidth = max_h / aspect

        story.append(img)
        story.append(Paragraph(caption, caption_style))
        story.append(PageBreak())

    # --- Page numbers if > 6 pages ---
    page_num_callback = add_page_number if len(figures) + 1 > 6 else None

    doc.build(story, onLaterPages=page_num_callback)
```

## Page number callback

Only used when report exceeds 6 pages:

```python
def add_page_number(canvas, doc):
    canvas.saveState()
    canvas.setFont("Times-Roman", 9)
    canvas.setFillColorRGB(0.4, 0.4, 0.4)
    canvas.drawCentredString(PAGE_W / 2, 1.2 * cm, f"{doc.page}")
    canvas.restoreState()
```

## Mixed Chinese/English captions

If captions contain Chinese characters, reportlab's built-in `Times-Italic` won't render them. Solution: build a paragraph style that uses the registered CN font for CN runs.

The simplest approach — use a single font that has both Latin and CJK glyphs (e.g. Source Han Serif, Noto Serif CJK):

```python
if CN_REPORTLAB_FONT:
    caption_style = ParagraphStyle(
        "caption", fontName=CN_REPORTLAB_FONT, fontSize=9,
        alignment=TA_JUSTIFY, leading=14, spaceBefore=10,
    )
```

If the registered font supports italics, use the italic variant. Otherwise fall back to roman — Chinese typography traditionally doesn't use italic for emphasis, so this is acceptable.

## Title page rules

- Title: report subject in human language, not "Analysis Report" or "Charts" — those are filenames, not titles
- Below the title: source filenames + generation date, small gray text
- Nothing else. No logos, no author block, no abstract.

## Layout dos and don'ts

**Do:**
- One figure per page (clarity over density)
- Generous margins (2.5 cm) — the figure should breathe
- Captions immediately below the figure, never on a separate page
- Preserve aspect ratio (the template above does this correctly)

**Don't:**
- Stretch images to fill the page
- Add headers/footers beyond the optional page number
- Use sans-serif anywhere in the PDF (even when figures internally use sans)
- Repeat the figure number inside the image and again in the caption — caption only

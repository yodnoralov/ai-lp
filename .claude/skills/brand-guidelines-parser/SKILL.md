---
name: brand-guidelines-parser
description: Use when parsing brand assets and guidelines into structured brand context. Covers extracting colors, typography, logo usage, tone of voice, and visual style from PDFs, images, and text descriptions.
---

# Brand Guidelines Parser

Patterns for extracting structured, actionable brand information from raw assets.

<!-- TODO: This skill needs real examples to be fully useful. Once we have actual brand
     guideline PDFs and asset packages to test against, add:
     - Example parsing of a real brand PDF (section-by-section extraction)
     - Example handling of font file packages (mapping .woff2/.otf files to CSS declarations)
     - Example color extraction from different source formats
     - Example of handling incomplete/contradictory guidelines
     - Common brand guideline structures and where key info typically lives
     Until then, the patterns below are general guidance for the brand-context-analyst agent. -->

## Output Target

The parser produces `brand-context.json` — see the brand-context-analyst agent definition for the full schema.

## Color Extraction

### From PDFs
- Look for sections titled "Color Palette", "Brand Colors", "Color System"
- Colors may be specified as: hex (#FF6B00), RGB (255, 107, 0), CMYK, Pantone, or named
- Always normalize to hex for the output
- If only CMYK or Pantone values are given, note them in the output but flag that hex conversion is approximate

### From CSS/Websites
- Extract from existing stylesheets, CSS custom properties, or Tailwind configs
- Check for design token files (tokens.json, variables.css)

### Color Role Mapping
Map extracted colors to roles:
- **Primary**: The dominant brand color, used for CTAs and key UI elements
- **Secondary**: Supporting color for secondary elements
- **Accent**: Highlight color for emphasis, hover states
- **Neutrals**: Dark (text), medium (borders, secondary text), light (backgrounds, dividers)
- **Background**: Page background (often white or near-white)

If the guidelines don't specify roles, infer from usage context and flag as an assumption.

## Typography Extraction

### From Font Files
- Map file names to font families: `BrandSans-Bold.woff2` → family: "Brand Sans", weight: 700
- Common naming patterns: Regular(400), Medium(500), SemiBold(600), Bold(700), ExtraBold(800)
- Variable fonts: single file with weight range (e.g., 100-900)
- Preferred formats: `.woff2` (best compression), `.woff` (fallback)

### From PDFs
- Look for "Typography", "Type System", "Fonts" sections
- Extract: font family names, recommended weights, size scale, line heights
- If a type scale is provided (H1: 48px, H2: 36px, etc.), map directly
- If no scale is provided, use a reasonable default and flag as assumption

### Default Type Scale (when none provided)
```
h1: 3rem/1.1 (48px)
h2: 2.25rem/1.2 (36px)
h3: 1.5rem/1.3 (24px)
body: 1rem/1.6 (16px)
small: 0.875rem/1.5 (14px)
```

## Logo Extraction

- Identify primary logo, alternate/reversed versions, and favicon/icon mark
- Note file formats: SVG preferred for web, PNG as fallback
- Extract usage rules: minimum size, clear space, acceptable backgrounds
- If no rules are provided, apply sensible defaults:
  - Minimum width: 120px
  - Clear space: equal to the height of the logo mark
  - Don't stretch, rotate, or alter colors

## Tone of Voice

### From PDFs
- Look for "Voice", "Tone", "Messaging", "Writing Style" sections
- Extract: adjectives describing the voice, example copy, words to use/avoid
- Map to actionable style: "professional but approachable" → "use active voice, contractions OK, avoid jargon"

### When Absent
If no tone guidelines exist, infer from:
- The brand's existing website copy (if a URL is provided)
- The industry and target audience
- Flag everything as an assumption

## Handling Gaps

When information is missing, follow this priority:
1. **Colors**: Must have at least primary. If missing, STOP — can't proceed without a color palette.
2. **Typography**: Default to system fonts if no font files provided. Flag prominently.
3. **Logo**: Can proceed without, but flag that no logo was provided.
4. **Tone**: Default to "professional, clear, concise" and flag.
5. **Visual style**: Default to "clean, minimal" and flag.

Every gap goes into the `assumptions` array in brand-context.json.

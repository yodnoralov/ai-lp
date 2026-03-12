---
name: tooling-page-analyzer
description: "Tooling skill (build-time only): Analyze a live landing page URL, extract its structure into the reference library format, capture screenshots, and store the entry. Use when adding new pages to the reference library."
---

# Page Analyzer — Tooling Skill

Analyze a live landing page and produce a structured reference library entry. This is a **build-time tooling skill** — it's used to populate the reference library, not during runtime pipeline execution.

## Input

- **URL**: The landing page to analyze
- **ID** (optional): A slug for the directory name. If not provided, derive from the URL path.

## Process

### Step 1: Navigate and Snapshot

1. Open the URL in the browser using Playwright
2. Take an accessibility snapshot to get the full page structure
3. Capture full-page screenshot at **desktop viewport (1440x900)**
4. Resize to **mobile viewport (375x812)** and capture full-page screenshot

### Step 2: Analyze Page Structure

From the accessibility snapshot, identify each distinct section of the page. For each section determine:

- **order**: Position in the page (1-indexed)
- **type**: What kind of section it is. Use these canonical types:
  - `hero` — Primary above-the-fold section
  - `navigation` — Nav bar (if distinct from hero)
  - `social-proof` — Logos, testimonials, reviews, ratings
  - `trust-signals` — Stats, credentials, awards, media mentions
  - `value-propositions` — Feature/benefit grid or list
  - `feature-detail` — Deep dive on a single feature (often alternating image/text)
  - `product-comparison-list` — Ranked or side-by-side product/service comparison
  - `pricing` — Pricing table or cards
  - `testimonials` — Customer quotes, case studies
  - `how-it-works` — Step-by-step process explanation
  - `faq` — Frequently asked questions
  - `cta-block` — Standalone call-to-action section
  - `form` — Lead capture or signup form
  - `footer` — Page footer
  - `other` — Anything that doesn't fit (describe in detail)
- **variant**: A descriptive name for the specific layout approach (e.g., "split-hero-with-cards", "3-column-icon-grid", "accordion")
- **description**: 1-3 sentences describing the layout, content approach, and visual treatment. Focus on the **pattern**, not the specific copy or brand.
- **cta_present**: Whether this section contains a call-to-action
- **cta_type**: If CTA present, what kind (button, form, link, phone, app-store-badge, etc.)

### Step 3: Analyze CTA Strategy

Across the full page, determine:
- **primary_action**: The main thing the page wants visitors to do
- **secondary_action**: Any secondary conversion path
- **cta_count**: How many times the primary CTA appears across all sections
- **placement_notes**: Where CTAs appear and any patterns in their placement

### Step 4: Classify the Page

Determine:
- **page_type**: One of: `lead-gen`, `product-launch`, `event-signup`, `app-download`, `brand-awareness`, `comparison`, `other`
- **industry**: The industry/vertical
- **target_audience**: Who this page is designed for
- **desired_outcome**: What conversion the page optimizes for

### Step 5: Identify Notable Patterns

This is the highest-value part. List 3-6 things that are **specific and interesting** about how this page applies (or breaks from) common patterns. Focus on:
- Unusual section ordering and why it works
- How the page handles the transition between building trust and asking for action
- Responsive adaptations that go beyond simple stacking
- Content strategies (how headlines are framed, how objections are addressed)
- Anything the strategist agent should learn from this specific page

### Step 6: Responsive Notes

Describe how the page adapts between desktop and mobile. Reference both screenshots. Focus on structural changes, not just "things get smaller."

### Step 7: Tag the Entry

Add tags for filtering. Include:
- The page_type
- Industry keywords
- CTA pattern types (e.g., "multi-cta", "single-form", "click-through")
- Layout characteristics (e.g., "card-based", "long-form", "minimal")
- Audience type (e.g., "b2b", "b2c", "developer")

## Output

### Directory Structure

```
reference-library/
  <id>/
    metadata.json
    screenshots/
      desktop.png
      mobile.png
```

### metadata.json Schema

```json
{
  "id": "string — directory name slug",
  "source_url": "string — original URL",
  "captured_date": "string — YYYY-MM-DD",
  "page_type": "string — from classification",
  "industry": "string",
  "target_audience": "string",
  "desired_outcome": "string",

  "sections": [
    {
      "order": "number",
      "type": "string — canonical type",
      "variant": "string — descriptive variant name",
      "description": "string — 1-3 sentences about the pattern",
      "cta_present": "boolean",
      "cta_type": "string | null"
    }
  ],

  "cta_strategy": {
    "primary_action": "string",
    "secondary_action": "string | null",
    "cta_count": "number",
    "placement_notes": "string"
  },

  "responsive_notes": "string — how layout adapts between breakpoints",

  "notable_patterns": [
    "string — each entry is one specific, interesting observation"
  ],

  "tags": ["string"]
}
```

### Step 8: Update the Index

After saving metadata.json and screenshots, update `reference-library/index.json`.

Read the current index, append a new entry with only these fields:

```json
{
  "id": "string — matches directory name",
  "page_type": "string",
  "tags": ["string"],
  "desired_outcome": "string"
}
```

If an entry with the same `id` already exists, replace it. Write the updated index back.

## Quality Checklist

Before saving the entry:
- [ ] Both screenshots captured (desktop 1440px, mobile 375px)
- [ ] Every visible section is represented in the sections array
- [ ] Section descriptions focus on **patterns**, not specific copy or brand details
- [ ] notable_patterns contains at least 3 entries with genuine insights, not generic observations
- [ ] Tags are specific enough to be useful for filtering (not just "landing-page")
- [ ] metadata.json is valid JSON
- [ ] The entry would help the strategist agent recommend a similar layout for a different brand

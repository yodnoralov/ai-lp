---
name: user-story-writer
description: Use when creating, writing, or generating user stories for landing page implementation. Produces structured JSON stories with acceptance criteria, test cases, and technical notes scoped to landing page sections and cross-cutting concerns.
---

# User Story Writer — Landing Page Format

Generate user stories in structured JSON format. Each story is a standalone `.json` file scoped to a single page section or cross-cutting concern.

## JSON Schema

Every story is a single JSON object with these fields:

### Required Fields

| Field                 | Type   | Description                                                |
| --------------------- | ------ | ---------------------------------------------------------- |
| `id`                  | string | `"US-<seq>"` e.g. `"US-3"`                                 |
| `title`               | string | Brief descriptive title                                    |
| `priority`            | string | `"high"`, `"medium"`, or `"low"`                           |
| `story`               | object | `{ "as_a", "i_want_to", "so_that" }`                      |
| `acceptance_criteria` | array  | See format below                                           |
| `technical_notes`     | array  | Array of strings with implementation guidance              |
| `dependencies`        | array  | Array of story IDs this depends on, e.g. `["US-1"]`       |
| `estimated_hours`     | number | Effort estimate                                            |

### Optional Fields

| Field           | Type   | Description                                        |
| --------------- | ------ | -------------------------------------------------- |
| `test_cases`    | array  | See format below                                   |
| `responsive`    | object | Breakpoint-specific behavior notes                 |
| `brand_refs`    | array  | Specific brand-context.json fields this story uses |

### Acceptance Criteria Format

```json
{
  "id": "AC-<story_seq>.<seq>",
  "description": "Specific, testable behavior description",
  "status": "pending"
}
```

**Guidelines:**
- Each criterion describes ONE testable behavior
- Reference specific brand-context.json values: "Primary CTA uses `colors.primary.hex` background"
- Include responsive requirements: "On mobile (< 768px), the grid collapses to a single column"
- Include accessibility: "Section has an `aria-labelledby` pointing to the section heading"
- Typical count: 5-10 criteria per story

### Test Cases Format

```json
{
  "id": "TC-<story_seq>.<seq>",
  "name": "Descriptive test name",
  "type": "build | visual | accessibility | responsive",
  "steps": ["What to check or do"],
  "expected_results": ["What should be true"],
  "status": "pending"
}
```

**Test types:**
- `build`: `npm run build` and `npm run lint` pass with the new code
- `visual`: Component renders correctly, matches brand context
- `accessibility`: axe-core passes, keyboard navigation works, screen reader flow correct
- `responsive`: Layout is correct at specified breakpoints

### Technical Notes Guidelines

Array of strings. Each note is a standalone implementation hint:
- Component file paths: `"Component: src/components/sections/Hero.tsx"`
- Tailwind classes: `"Use brand tokens: bg-brand-primary, text-brand-secondary"`
- Image handling: `"Hero image: use next/image with priority, 1200x630, WebP format"`
- Font usage: `"Heading uses font-heading (var(--font-heading)), body uses font-body"`
- Accessibility: `"Add aria-labelledby='hero-heading' to the section element"`
- Responsive: `"Mobile: stack vertically. Desktop (lg:): side-by-side with image right"`
- Brand context refs: `"CTA color: brand-context.json → colors.primary.hex"`

## Example

```json
{
  "id": "US-3",
  "title": "Hero Section",
  "priority": "high",
  "story": {
    "as_a": "visitor landing on the page",
    "i_want_to": "immediately understand the product's value proposition and have a clear next action",
    "so_that": "I know what this page is about and can convert if interested"
  },
  "acceptance_criteria": [
    {
      "id": "AC-3.1",
      "description": "Hero section displays a headline using font-heading at text-5xl (desktop) / text-3xl (mobile) with brand primary color",
      "status": "pending"
    },
    {
      "id": "AC-3.2",
      "description": "Subheadline displays below the headline using font-body at text-xl with neutral-dark color",
      "status": "pending"
    },
    {
      "id": "AC-3.3",
      "description": "Primary CTA button uses bg-brand-primary with white text, rounded-lg, and hover state using brand-primary/90",
      "status": "pending"
    },
    {
      "id": "AC-3.4",
      "description": "Hero image uses next/image with priority attribute, explicit width/height, and WebP format",
      "status": "pending"
    },
    {
      "id": "AC-3.5",
      "description": "On mobile (< 768px), layout stacks vertically: headline, subheadline, CTA, then image. On desktop (>= 1024px), split layout with text left and image right.",
      "status": "pending"
    },
    {
      "id": "AC-3.6",
      "description": "Section uses semantic <section> element with aria-labelledby pointing to the headline's id",
      "status": "pending"
    }
  ],
  "test_cases": [
    {
      "id": "TC-3.1",
      "name": "Hero section builds without errors",
      "type": "build",
      "steps": ["Run npm run build"],
      "expected_results": ["Build succeeds with no errors or warnings related to Hero component"],
      "status": "pending"
    },
    {
      "id": "TC-3.2",
      "name": "Hero is accessible",
      "type": "accessibility",
      "steps": ["Check semantic HTML", "Verify heading hierarchy", "Check image alt text", "Verify color contrast"],
      "expected_results": [
        "Section has aria-labelledby",
        "H1 is the first heading on the page",
        "Hero image has descriptive alt text",
        "Text on background meets 4.5:1 contrast ratio"
      ],
      "status": "pending"
    }
  ],
  "technical_notes": [
    "Component: src/components/sections/Hero.tsx",
    "Uses Container wrapper for consistent padding and max-width",
    "Hero image: next/image with priority, 1200x630, WebP",
    "CTA links to the primary conversion section via anchor: href='#signup'",
    "Mobile: grid-cols-1, Desktop: lg:grid-cols-2 with gap-12",
    "Brand refs: colors.primary for CTA, typography.heading_font for headline"
  ],
  "dependencies": ["US-1", "US-2"],
  "estimated_hours": 3,
  "responsive": {
    "mobile": "Single column stack: heading → subheading → CTA → image",
    "tablet": "Same as mobile with larger text sizes",
    "desktop": "Two-column split: text left (55%), image right (45%)"
  },
  "brand_refs": ["colors.primary", "colors.neutral.dark", "typography.heading_font", "typography.body_font"]
}
```

## Quality Checklist

Before saving each story:
- [ ] ID follows `US-<seq>` pattern and is unique
- [ ] Acceptance criteria are specific and testable (not "make it look good")
- [ ] Acceptance criteria reference brand-context.json values by path
- [ ] Responsive behavior is specified with actual breakpoints
- [ ] Accessibility requirements are included
- [ ] Technical notes include component file paths and Tailwind class references
- [ ] Dependencies list actual story IDs
- [ ] JSON is valid (no trailing commas, proper quoting)

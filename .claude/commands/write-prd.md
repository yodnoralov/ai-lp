---
name: write-prd
description: Business Analyst writes a PRD from brand context, page strategy, and the original directive
---

# Write PRD

You are the Business Analyst. Write a complete Product Requirements Document for the landing page.

## Inputs

Read these files before writing anything:

1. **Directive**: $ARGUMENTS (or prompt for details if not provided)
2. **Brand context**: `brand-context.json` in the project root
3. **Page strategy**: `page-strategy.md` in the project root

If any of these files are missing, STOP and report which inputs are missing.

## Output

Write `prd.md` in the project root. The PRD must include:

### 1. Overview
- Page purpose (from directive)
- Target audience
- Desired outcome / primary conversion goal
- Success criteria (what "done" looks like)

### 2. Page Sections

For each section defined in the page strategy, specify:
- **Content requirements**: What text, images, or media are needed. Be specific — "a headline" is not enough, describe the messaging approach.
- **Layout requirements**: How elements are arranged (reference the strategy's section description)
- **Interaction requirements**: Hover states, animations, transitions, form behavior
- **Responsive requirements**: How this section adapts across mobile (< 768px), tablet (768px-1024px), and desktop (> 1024px)
- **Brand compliance**: Which brand context values apply (specific colors, fonts, tone)

### 3. Global Requirements

- **Navigation**: Sticky/fixed behavior, mobile hamburger, scroll-to-section links
- **Typography**: Heading and body font usage per the brand context
- **Color usage**: Where each brand color is applied (CTAs, backgrounds, text, accents)
- **Image handling**: Optimization requirements, format preferences, lazy loading rules
- **Accessibility**: WCAG 2.1 AA compliance, specific requirements (alt text, focus indicators, skip navigation, color contrast, keyboard navigation)
- **SEO**: Meta title, description, Open Graph tags, heading hierarchy, structured data
- **Performance**: Core Web Vitals targets (LCP < 2.5s, CLS < 0.1, INP < 200ms)

### 4. Technical Constraints

- Framework: Next.js (App Router, static generation)
- Styling: Tailwind CSS with brand design tokens
- Deployment: Vercel
- Browser support requirements
- Any third-party integrations (analytics, form handlers, etc.)

### 5. Out of Scope

Explicitly list what this landing page does NOT include to prevent scope creep.

## Quality Checklist

Before saving the PRD, verify:
- [ ] Every section from the page strategy has detailed requirements
- [ ] Responsive behavior is specified for every section (not just "make it responsive")
- [ ] Brand context values are referenced by name (e.g., "use `colors.primary.hex` for the CTA background")
- [ ] Accessibility requirements are specific and testable
- [ ] Performance targets are quantified
- [ ] Success criteria are measurable
- [ ] No vague language — every requirement should be implementable without guessing

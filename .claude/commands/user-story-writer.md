---
name: user-story-writer
description: Write structured JSON user stories for landing page implementation following the established format
---

# User Story Writer

Write structured JSON user stories for landing page implementation.

**Arguments:** `$ARGUMENTS`

## Instructions

Follow the `user-story-writer` skill for the complete JSON schema, field conventions, and quality checklist.

### Step 1: Parse Arguments

If arguments were provided (`$ARGUMENTS`), extract:
- **Epic** — epic number or name
- **Title or description** — what the story is about

If no arguments were provided, read the PRD (`prd.md`) and break it into stories covering all sections and cross-cutting concerns.

### Step 2: Research Context

1. **Read the PRD** — understand the full scope of requirements
2. **Read the brand context** — `brand-context.json` for design constraints
3. **Read the page strategy** — `page-strategy.md` for layout and section decisions
4. **Check existing stories** — if other stories already exist, determine the next sequence number and check for consistency

### Step 3: Story Scoping

Each story should be scoped to ONE of:
- A single page section (e.g., "Hero Section", "Testimonials Section")
- A cross-cutting concern (e.g., "Global Typography & Font Loading", "SEO Meta Configuration", "Responsive Navigation")
- A shared component (e.g., "CTA Button Component", "Section Wrapper Layout")

**Ordering guidance:**
1. Project setup / global styles / Tailwind config with brand tokens → FIRST
2. Shared components (buttons, section wrappers, typography) → SECOND
3. Page sections in visual order (hero → value props → features → social proof → CTA → footer)
4. Cross-cutting concerns (SEO, accessibility audit, performance optimization) → LAST

### Step 4: Draft and Save Stories

Write each story as a JSON file to `docs/stories/us_<ID>.json`.

If writing a full set from a PRD, present the story list (IDs, titles, dependencies) as a summary before writing files.

### Story Dependencies

- Global styles/config stories have no dependencies
- Shared component stories depend on global styles
- Section stories depend on shared components they use
- SEO/accessibility/performance stories depend on all section stories

## Quality Checklist

Before saving each story:
- [ ] ID follows `US-<seq>` pattern and is unique
- [ ] Acceptance criteria reference specific brand context values, responsive breakpoints, and accessibility requirements
- [ ] Technical notes reference Next.js file paths, Tailwind classes, and component names
- [ ] Dependencies list actual story IDs that must be completed first
- [ ] Story is implementable without guessing — a developer can read it and start coding
- [ ] JSON is valid

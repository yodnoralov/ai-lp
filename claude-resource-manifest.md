# Claude Resource Manifest

Resources needed to implement the autonomous landing page builder pipeline. Each resource is either new, a rewrite of an existing boilerplate, or kept as-is.

## Agents (5)

### 1. `brand-context-analyst` — NEW

**Purpose**: Ingests brand assets (logos, fonts, colors, PDFs of brand guidelines) and produces a structured brand context document that all downstream agents consume.

**Read-only**: Yes — analyzes input, produces structured output, does not write code.

**Input**: Brand assets and guidelines from the directive (files, PDFs, images, text descriptions).

**Output**: Structured brand context (JSON or markdown) covering:
- Color palette (primary, secondary, accent) with exact hex/RGB values
- Typography rules (font families, weights, sizes, heading/body hierarchy)
- Logo usage rules (spacing, minimum sizes, acceptable backgrounds, file paths)
- Tone of voice (formal vs. casual, key phrases, words to avoid)
- Visual style notes (photography style, illustration approach, iconography)
- Do's and don'ts extracted from guidelines

**Key behavior**: Must handle incomplete or ambiguous guidelines gracefully — make reasonable defaults and flag assumptions explicitly for downstream agents.

---

### 2. `landing-page-strategist` — NEW

**Purpose**: Reviews the landing page reference library and the directive's goals to recommend a page type, structure, and strategy.

**Read-only**: Yes — analyzes references and directive, produces a strategy recommendation.

**Input**: Brand context (from agent 1) + directive (page type, desired outcome) + landing page reference library (screenshots, JSON metadata describing previous pages and known patterns).

**Output**: A page strategy document describing:
- Recommended page type and layout structure
- Section order and hierarchy (hero, value props, social proof, CTA, etc.)
- CTA strategy (placement, copy approach, urgency/scarcity considerations)
- Content strategy (what messaging to emphasize based on the desired outcome)
- Rationale for each choice grounded in the directive's goals
- Any elements borrowed from specific reference pages (with attribution)

**Key behavior**: Must tie every recommendation back to the directive's desired outcome and the brand context. No generic "best practices" without justification for this specific page.

---

### 3. `business-analyst` — REWRITE from existing

**Purpose**: Writes the PRD and user stories for the landing page. Participates in review checkpoints as the requirements reviewer.

**Read-only**: Yes — produces documents (PRD, stories), does not write code.

**Rewrite notes**:
- Remove all mighty-munchkins references (Firestore, gamification, kids app, TypeBox, monorepo)
- Remove memory-keeper MCP integration
- Focus on landing page requirements: sections, content, responsive behavior, interactions, accessibility, brand compliance
- PRD writing should consume brand context + page strategy as inputs
- Story writing should use the `user-story-writer` skill for JSON format
- Review participation should focus on: requirements coverage, acceptance criteria completeness, alignment with directive and brand

---

### 4. `frontend-developer` — REWRITE from `typescript-engineer`

**Purpose**: Implements landing page stories. The only agent with write access to code.

**Read-only**: No — has full read/write/bash tools for implementation.

**Rewrite notes**:
- Remove all mighty-munchkins references (Fastify, Firestore, React Native, Expo, monorepo)
- Remove memory-keeper MCP integration
- Focus on Next.js + Tailwind CSS + landing page implementation
- Should consult `nextjs-landing-page` skill before writing code
- Must follow brand context for all visual decisions (colors, fonts, spacing)
- Quality checks: `npm run build`, `npm run lint` (not the old `npm run check`)
- Review participation should focus on: code quality, implementation correctness, performance, responsive design, accessibility

---

### 5. `systems-architect` — REWRITE from `typescript-systems-architect`

**Purpose**: Reviews technical architecture and design decisions. Participates in review checkpoints as the architecture reviewer.

**Read-only**: Yes — analyzes and recommends, does not write code.

**Rewrite notes**:
- Remove all mighty-munchkins references (Firestore, GCP, mobile architecture, gamification)
- Remove memory-keeper MCP integration
- Focus on Next.js project structure, component architecture, performance (Core Web Vitals), SEO, image optimization, font loading strategy
- Review participation should focus on: technical feasibility, component structure, performance implications, accessibility architecture, deployment readiness

---

## Commands (6)

### 1. `run-pipeline` — NEW

**Purpose**: The master orchestrator. Runs the full sequential pipeline from directive to deployed landing page.

**This is the top-level entry point for the entire system.**

**Flow**:
1. Parse the directive (validate required inputs: page type, desired outcome, brand assets)
2. Run Stage 1: Brand Context Agent → produces brand context
3. Run Stage 2: Landing Page Strategist → produces page strategy
4. Run Stage 3: BA writes PRD (using `write-prd` command)
5. Run Review Checkpoint: team review of PRD (using `review/team-review`)
   - Review loop: min 1, max 3 iterations, 3/4 majority vote to pass
   - BA fixes between rounds
6. Run Stage 4: BA writes stories (using `user-story-writer` command)
7. Run Review Checkpoint: team review of stories
   - Same loop rules
   - BA fixes between rounds
8. Run Stage 5: Developer implements epic (using `run-epic` command)
   - Per-story: implement → team review → fix → commit → next
9. Run Stage 6: Final review of completed landing page
10. Deploy via Vercel CLI, return live URL

**Key behavior**: Each stage produces artifacts (files in the project directory) that downstream stages consume. Context is passed via files, not in-memory state.

---

### 2. `write-prd` — NEW

**Purpose**: BA agent writes a PRD from brand context + page strategy.

**Input**: Brand context document + page strategy document + original directive.

**Output**: A PRD (markdown) defining:
- Page purpose and goals (from directive)
- Target audience
- Page sections with content requirements for each
- Interaction requirements (animations, hover states, form behavior)
- Responsive behavior requirements (mobile, tablet, desktop)
- Accessibility requirements
- Performance requirements (load time targets, Core Web Vitals)
- Brand compliance requirements (referencing the brand context)
- Success criteria / definition of done

---

### 3. `user-story-writer` — REWRITE from existing

**Purpose**: BA breaks the PRD into implementable JSON stories.

**Rewrite notes**:
- Remove mighty-munchkins references from the JSON schema (Firestore collections, TypeBox schemas, factory functions)
- Adapt fields for landing page work:
  - Technical notes should reference Next.js components, Tailwind classes, responsive breakpoints, image assets
  - Acceptance criteria should cover visual appearance, responsive behavior, accessibility, brand compliance
  - Test cases should cover visual regression, responsive layouts, interaction behavior
- Keep the JSON schema structure (id, title, story, acceptance_criteria, test_cases, technical_notes, dependencies) — it's solid
- Stories should be scoped to individual page sections or cross-cutting concerns (e.g., "Hero Section", "Navigation Bar", "Responsive Grid Layout", "SEO Meta Tags")

---

### 4. `run-epic` — REWRITE from existing

**Purpose**: Developer agent implements all stories in sequence with review gates.

**Rewrite notes**:
- Remove mighty-munchkins references (Firestore, TypeBox, contracts package, React Native)
- Remove memory-keeper integration — use file-based progress tracking
- Remove the separate adversarial review round (now merged into team review with 4th adversarial subagent)
- Adapt implementation order for Next.js landing pages:
  1. Project setup / global styles / font loading
  2. Shared components (buttons, typography, layout containers)
  3. Page sections (hero, features, testimonials, CTA, footer) in story order
  4. Responsive adjustments
  5. Interactions / animations
  6. SEO / meta / performance optimization
- Quality gate: `npm run build` + `npm run lint` (not `npm run check`)
- Per-story flow: implement → team review (4 reviewers) → fix → commit → next story
- Brand context must be available to the developer agent throughout implementation

---

### 5. `review/team-review` — REWRITE from `review/ts-team-review`

**Purpose**: Dispatches 4 parallel subagents to review an artifact (PRD, stories, or implementation).

**Reviewers**:

1. **Business Analyst** (subagent: `business-analyst`)
   - Requirements coverage: are all acceptance criteria met?
   - Alignment with directive goals and desired outcome
   - Content completeness: is anything missing for the page to achieve its purpose?
   - Brand compliance: does everything align with the brand context?

2. **Systems Architect** (subagent: `systems-architect`)
   - Technical feasibility and component structure
   - Performance implications (image sizes, font loading, bundle size, Core Web Vitals)
   - Accessibility architecture (semantic HTML, ARIA, keyboard navigation)
   - SEO considerations (meta tags, heading hierarchy, structured data)
   - Responsive design approach

3. **Frontend Developer** (subagent: `frontend-developer`)
   - Code quality and Next.js/React best practices
   - Tailwind CSS usage (no conflicting classes, responsive prefixes, design token consistency)
   - Component composition and reusability
   - Build passing, lint clean
   - Visual correctness against brand context

4. **Adversarial Reviewer** (subagent spawned with adversarial prompt — not a separate agent definition)
   - What's missing that nobody mentioned?
   - What breaks on mobile? On slow connections? On screen readers?
   - What looks wrong when real content replaces placeholder text?
   - What happens with edge-case content (very long headlines, missing images, no testimonials)?
   - What would a real user hate about this page?
   - Challenge the other reviewers' assumptions

**Voting**: Each reviewer votes pass/fail with specific feedback. **3 of 4 must pass** for the review to pass. All findings rated Critical or High must be fixed regardless of vote outcome.

**Review loop** (managed by the calling command — `run-pipeline` or `run-epic`):
- Minimum 1 iteration (always runs at least once)
- Maximum 3 iterations (hard cap — advance even if not all pass)
- Early exit on 3/4 majority pass with no Critical findings

---

### 6. `review/fix-review-findings` — KEEP AS-IS

**Purpose**: Address all issues found by reviews regardless of priority.

**No changes needed** — already generic enough. The only acceptable reason to skip a finding is a genuine architectural constraint requiring separate design effort.

---

## Skills (4)

### 1. `brand-guidelines-parser` — NEW

**Purpose**: Teaches the brand context analyst agent how to extract structured data from various brand asset formats.

**Contents**:
- How to parse PDF brand guidelines (extract colors, fonts, tone, rules)
- How to handle logo files (identify formats, note file paths, extract usage rules if provided)
- How to structure the output brand context document (JSON schema)
- How to handle incomplete guidelines (default assumptions, what to flag)
- How to extract color values from various formats (hex, RGB, HSL, Pantone references)
- How to map font file names to font family declarations
- Common brand guideline structures and where to find key information in them

---

### 2. `landing-page-patterns` — NEW

**Purpose**: Reference library of landing page types, section patterns, and strategies. This is what the strategist agent and BA consult.

**Contents**:
- Landing page type taxonomy:
  - Lead generation (form-focused, gated content)
  - Product launch (feature showcase, waitlist)
  - Event/webinar signup
  - App download
  - Brand awareness / storytelling
  - Comparison / competitive
- Section pattern library:
  - Hero variants (centered, split, video background, animated)
  - Value proposition layouts (icon grid, alternating image/text, cards)
  - Social proof patterns (testimonials, logos, stats, case studies)
  - CTA patterns (inline, sticky, exit-intent, multi-step)
  - Footer patterns (minimal, mega-footer, newsletter)
- Strategy guidance:
  - Which patterns work for which page types
  - CTA placement strategies by desired outcome
  - Content hierarchy principles
  - Above-the-fold priorities by page type
- Evaluation criteria for choosing between strategies

---

### 3. `nextjs-landing-page` — NEW

**Purpose**: Next.js + Tailwind CSS patterns specific to landing page development. This is what the frontend developer agent consults before writing code.

**Contents**:
- Next.js App Router file structure for a landing page
- Tailwind CSS design token setup (extending theme with brand colors/fonts)
- Component patterns:
  - Section wrapper components (consistent padding, max-width, responsive)
  - Responsive image handling (next/image, srcSet, art direction)
  - Font loading strategy (next/font, variable fonts, fallbacks)
  - Animation patterns (CSS transitions, Framer Motion, scroll-triggered)
- Performance patterns:
  - Static generation for landing pages
  - Image optimization (WebP/AVIF, lazy loading, priority hints)
  - Font subsetting and display strategies
  - Bundle size management
- Accessibility patterns:
  - Semantic HTML for landing page sections
  - Skip navigation, focus management
  - Color contrast verification against brand palette
  - Screen reader considerations for decorative elements
- Responsive design patterns:
  - Mobile-first Tailwind breakpoints
  - Container queries for component-level responsiveness
  - Touch targets and mobile interaction patterns
- SEO patterns:
  - Meta tags, Open Graph, Twitter cards
  - Heading hierarchy
  - Structured data for landing pages

---

### 4. `user-story-writer` — REWRITE from existing

**Purpose**: Defines the JSON schema and conventions for landing page user stories.

**Rewrite notes**:
- Keep the core JSON structure (id, title, epic, story, acceptance_criteria, test_cases, technical_notes, dependencies)
- Remove mighty-munchkins specifics (Firestore, TypeBox, factory functions, gamification)
- Adapt for landing page stories:
  - Acceptance criteria should reference visual appearance, responsive behavior, accessibility, brand compliance, content
  - Technical notes should reference Next.js file paths, Tailwind classes, component names, image assets, font declarations
  - Test cases should cover: build passes, visual correctness, responsive layouts, accessibility (axe-core), performance (Lighthouse thresholds)
  - Story scoping examples: "Hero Section", "Mobile Navigation Drawer", "Testimonial Carousel", "Contact Form with Validation", "SEO Meta Configuration"
- Keep the quality checklist pattern — adapt checks for landing page context

---

## TODOs — What's Blocking Full Readiness

All 15 resources have been created in `.claude/` but the following have TODO gaps inline:

1. **Landing page reference library** — The `landing-page-strategist` agent and `landing-page-patterns` skill need a curated collection of previous landing pages (screenshots + JSON metadata describing layout, strategy, and performance). Without this, the strategist relies on general expertise only. We need to decide: where does this library live, what format, and how do we populate it initially?

2. **Real brand guideline examples** — The `brand-guidelines-parser` skill needs at least one real brand PDF and asset package to build concrete parsing examples. The current patterns are general guidance. Testing against actual assets will reveal what parsing logic is missing.

3. **Vercel deploy step** — The `run-pipeline` command's Stage 10 (deploy) is stubbed out. Needs the actual Vercel CLI commands and environment setup once we're ready to test end-to-end.

4. **Adversarial reviewer prompt tuning** — The 4th reviewer in `team-review` uses a general adversarial prompt (not a dedicated agent). May need refinement once we see real review output — the prompt might be too broad or not adversarial enough.

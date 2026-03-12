---
name: run-pipeline
description: Run the full autonomous landing page pipeline — from directive to deployed page
---

# Autonomous Landing Page Pipeline

You are the master orchestrator. Run the full pipeline from directive to finished landing page. Each stage produces artifacts that the next stage consumes.

**DO NOT ask the user questions during execution.** Make reasonable decisions and keep moving.

**This command is designed to run in `--dangerously-skip-permissions` mode.**

## Arguments

`$ARGUMENTS` — Path to the directive file, or inline directive text.

---

## Stage 0: Parse Directive

Read the directive from `$ARGUMENTS`. It must include:

- **Page type**: What kind of landing page (lead gen, product launch, event signup, etc.)
- **Desired outcome**: What we're optimizing for (conversions, signups, awareness, etc.)
- **Brand assets**: Logo files, font files, color specs, or paths to them
- **Brand guidelines**: PDF or document, or path to it
- **Additional context**: Target audience, messaging, competitive references (optional)

If critical information is missing (no page type, no desired outcome), STOP and report what's missing.

Create a `directive.md` file summarizing the parsed directive for downstream agents.

---

## Stage 1: Brand Context

**Agent**: brand-context-analyst

**Task**: Ingest all brand assets and guidelines from the directive. Produce `brand-context.json`.

**Invoke** the brand-context-analyst agent (via Task tool) with:
```
Parse the brand assets and guidelines referenced in directive.md.
Produce brand-context.json in the project root following your output format.
Flag any gaps or assumptions.
```

**Gate**: Verify `brand-context.json` exists and is valid JSON before proceeding.

---

## Stage 2: Page Strategy

**Agent**: landing-page-strategist

**Task**: Analyze the directive goals + brand context. Produce `page-strategy.md`.

**Invoke** the landing-page-strategist agent with:
```
Read directive.md and brand-context.json.
Recommend a page type, section structure, CTA strategy, and content hierarchy.
Write page-strategy.md in the project root.
```

**Gate**: Verify `page-strategy.md` exists before proceeding.

---

## Stage 3: PRD

**Command**: /write-prd

**Task**: BA writes the PRD from brand context + page strategy + directive.

**Gate**: Verify `prd.md` exists before proceeding.

---

## Stage 4: PRD Review Loop

**Command**: /review/team-review prd.md

Run the team review on the PRD. Then apply the review loop:

```
iteration = 0
while iteration < 3:
    iteration++
    run /review/team-review prd.md
    if 3/4 pass AND no Critical findings:
        break  # advance
    run /review/fix-review-findings on prd.md
```

Minimum 1 iteration. Maximum 3. Early exit on 3/4 majority with no Critical findings.

**Gate**: PRD has been reviewed at least once before proceeding.

---

## Stage 5: Story Writing

**Command**: /user-story-writer

**Task**: BA breaks the approved PRD into JSON stories in `docs/stories/`.

**Gate**: Verify `docs/stories/` contains `us_*.json` files before proceeding.

---

## Stage 6: Story Review Loop

**Command**: /review/team-review docs/stories/

Run the team review on the stories. Same loop rules:

```
iteration = 0
while iteration < 3:
    iteration++
    run /review/team-review "stories in docs/stories/"
    if 3/4 pass AND no Critical findings:
        break
    fix stories based on review findings
```

**Gate**: Stories have been reviewed at least once before proceeding.

---

## Stage 7: Project Setup

Before the developer starts implementing, set up the Next.js project if it doesn't already exist:

1. `npx create-next-app@latest . --typescript --tailwind --app --src-dir --no-import-alias`
   (Skip if `package.json` already exists)
2. Configure `tailwind.config.ts` with brand design tokens from `brand-context.json`:
   - Brand colors (primary, secondary, accent, neutrals)
   - Brand fonts (heading, body)
   - Any custom spacing or border-radius from the brand context
3. Set up `next/font` with the brand's font files
4. Run `npm run build` to verify clean baseline
5. Commit: "Initial project setup with brand design tokens"

---

## Stage 8: Epic Implementation

**Command**: /run-epic

**Task**: Developer implements all stories with per-story review gates.

This is the longest stage. The run-epic command handles:
- Story-by-story implementation
- Team review after each story
- Fix cycles (min 1, max 3 per story)
- Committing after each story passes review

**Gate**: All stories implemented and committed.

---

## Stage 9: Final Review

Run one last team review of the entire finished landing page:

```
/review/team-review "Final review of complete landing page — review all code, all sections, full responsive behavior, accessibility, performance, brand compliance"
```

Fix any Critical or High findings. This is the last gate.

---

## Stage 10: Deploy

<!-- TODO: Implement Vercel deployment step.
   This will:
   1. Run `npx vercel --prod` (or appropriate Vercel CLI command)
   2. Capture the live URL
   3. Report the URL
   For now, skip deployment and report that the landing page is ready for manual deployment.
-->

Report completion:

```
Landing page pipeline complete!

Branch: <branch_name>
Total stories: N
Total commits: M
Status: Ready for deployment

To deploy manually:
  npx vercel --prod
```

---

## Error Handling

### Stage Failure
If any stage fails to produce its required artifact:
1. Report which stage failed and why
2. Report what artifacts exist so far
3. STOP — do not skip stages

### Review Loop Cap
If a review loop hits max 3 iterations without full approval:
1. Note the unresolved findings
2. Advance anyway — the finding will likely surface again in later reviews

### Context Window Limits
If the context is getting large during epic implementation:
1. The run-epic command handles this internally via checkpointing
2. Stories already committed can be skipped on resume

---

## Pipeline Summary

| Stage | Agent/Command | Input | Output |
|-------|--------------|-------|--------|
| 0 | Orchestrator | Raw directive | `directive.md` |
| 1 | brand-context-analyst | Directive + assets | `brand-context.json` |
| 2 | landing-page-strategist | Directive + brand context | `page-strategy.md` |
| 3 | /write-prd | Brand context + strategy | `prd.md` |
| 4 | /review/team-review | PRD | Reviewed PRD |
| 5 | /user-story-writer | PRD | `docs/stories/us_*.json` |
| 6 | /review/team-review | Stories | Reviewed stories |
| 7 | Orchestrator | brand-context.json | Next.js project + Tailwind config |
| 8 | /run-epic | Stories | Implemented landing page |
| 9 | /review/team-review | Full page | Final approval |
| 10 | Vercel CLI | Built project | Live URL |

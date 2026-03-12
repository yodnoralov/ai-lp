---
name: team-review
description: Dispatch 4 parallel reviewers (BA, architect, developer, adversarial) to review an artifact and vote pass/fail
---

# Team Review: 4-Agent Parallel Review

You are coordinating a review with four specialized subagents working in parallel. Each agent reviews the artifact from their unique perspective, provides findings, and votes pass/fail.

## Context Setup

1. **Artifact type**: Determine what's being reviewed — PRD, stories, or implementation code
2. **Repository**: Infer from the current working directory
3. **Arguments**: $ARGUMENTS (story ID, file path, or description of what to review)

## Identify the Artifact

- If reviewing a **PRD**: Read the PRD markdown file, the brand context, and the page strategy
- If reviewing **stories**: Read the story JSON files, the PRD, and the brand context
- If reviewing **implementation**: Run `git diff` to see the changes, read the relevant story JSON, and the brand context

## Agent Dispatch

Launch all four agents in parallel using the Task tool.

### 1. Business Analyst (subagent_type: business-analyst)

```
You are reviewing: $ARGUMENTS

Review from a requirements and brand perspective:
- Are all acceptance criteria met (not just superficially)?
- Does this align with the original directive and desired outcome?
- Is the brand context followed? (colors, fonts, tone, imagery)
- Is any content, section, or interaction missing?
- Does responsive behavior match requirements?
- Would this page actually achieve its conversion goal?

Rate each finding: Critical / High / Medium / Low
Vote: PASS or FAIL (with specific reasons)
```

### 2. Systems Architect (subagent_type: systems-architect)

```
You are reviewing: $ARGUMENTS

Review from a technical architecture perspective:
- Is the component structure clean and appropriate?
- Are server vs. client components used correctly?
- Will this pass Core Web Vitals (LCP, CLS, INP)?
- Are images optimized (next/image, appropriate formats and sizes)?
- Is the HTML semantic and accessible (WCAG 2.1 AA)?
- Is the heading hierarchy correct for SEO?
- Are design tokens in tailwind.config.ts, not hardcoded in components?
- Will this deploy cleanly to Vercel?

Rate each finding: Critical / High / Medium / Low
Vote: PASS or FAIL (with specific reasons)
```

### 3. Frontend Developer (subagent_type: frontend-developer)

```
You are reviewing: $ARGUMENTS

Review from a code quality perspective:
- Is the code clean, idiomatic React/Next.js?
- Are Tailwind classes consistent with the design token system?
- Is there unnecessary complexity that should be simplified?
- Run npm run build — does it pass?
- Run npm run lint — does it pass?
- Are there accessibility issues (missing alt text, non-semantic elements, keyboard traps)?
- Is responsive design implemented correctly (mobile-first, proper breakpoints)?
- Are images and fonts loaded optimally?

Rate each finding: Critical / High / Medium / Low
Vote: PASS or FAIL (with specific reasons)
```

### 4. Adversarial Reviewer (no specific subagent type — use a general prompt)

```
You are the adversarial reviewer for: $ARGUMENTS

Your ONLY job is to find what everyone else missed. Be skeptical. Be thorough.

- What breaks on a 320px wide screen? On a 4K monitor?
- What happens on a slow 3G connection?
- What does a screen reader user experience?
- What happens when real content replaces placeholder text? (Very long headlines? Very short ones? Missing images?)
- What would a real user hate about this?
- What looks wrong when you actually use the page, not just read the code?
- What assumptions are being made that aren't validated?
- If this is a PRD or stories: What's ambiguous? What will the developer interpret wrong? What edge cases aren't covered?
- If this is code: What's fragile? What will break with the next change?

Find problems nobody else found. Challenge assumptions.
Rate each finding: Critical / High / Medium / Low
Vote: PASS or FAIL (with specific reasons)
```

## Collect Results

After all 4 agents complete, collect their findings and votes.

### Individual Reports

Present each reviewer's findings separately:

**Business Analyst**: [findings and vote]
**Systems Architect**: [findings and vote]
**Frontend Developer**: [findings and vote]
**Adversarial Reviewer**: [findings and vote]

### Consolidated Summary

| Reviewer             | Vote | Top Finding |
| -------------------- | ---- | ----------- |
| Business Analyst     |      |             |
| Systems Architect    |      |             |
| Frontend Developer   |      |             |
| Adversarial Reviewer |      |             |

**Vote Result**: X/4 PASS

### Findings by Severity

**Critical** (must fix):
- ...

**High** (should fix):
- ...

**Medium** (consider fixing):
- ...

**Low** (minor):
- ...

### Review Verdict

- **PASS** (3/4 or 4/4 voted pass AND no unresolved Critical findings) → Advance the pipeline
- **FAIL** (fewer than 3/4 voted pass OR unresolved Critical findings) → Fix and re-review

## Save Review Results

Write the review results to `reviews/<artifact>-review-<iteration>.md` in the project directory for the fix step to consume.

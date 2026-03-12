# Autonomous Landing Page Builder

Multi-agent pipeline that takes a directive + brand assets and produces a deployed landing page.

## Pipeline

Run with `/run-pipeline <directive-path>`. Stages:

1. **Parse directive** → `directive.md`
2. **Brand context** (brand-context-analyst agent) → `brand-context.json`
3. **Page strategy** (landing-page-strategist agent) → `page-strategy.md`
4. **PRD** (/write-prd) → `prd.md`
5. **PRD review** (/review/team-review) — 4 parallel reviewers, min 1 / max 3 iterations
6. **Story writing** (/user-story-writer) → `docs/stories/us_*.json`
7. **Story review** (/review/team-review)
8. **Project setup** — Next.js + Tailwind with brand tokens
9. **Epic implementation** (/run-epic) — story-by-story with per-story review gates
10. **Final review** (/review/team-review)
11. **Deploy** — Vercel (not yet implemented)

## Key Artifacts

| File | Purpose |
|------|---------|
| `directive.md` | Parsed input directive |
| `brand-context.json` | Structured brand values for all agents |
| `page-strategy.md` | Page type, section order, CTA strategy |
| `prd.md` | Full requirements document |
| `docs/stories/us_*.json` | Implementable user stories |
| `reference-library/index.json` | Index of real-world LP examples |
| `reference-library/<id>/metadata.json` | Full analysis of a reference page |

## Agents

- **brand-context-analyst** — Parses brand assets into `brand-context.json`
- **landing-page-strategist** — Recommends page structure from directive + brand context + reference library
- **business-analyst** — PRD writing and requirements review
- **systems-architect** — Technical architecture review (read-only)
- **frontend-developer** — Implementation and code review

## Review Loop

All review checkpoints use `/review/team-review` which dispatches 4 parallel reviewers (BA, architect, developer, adversarial). Each votes pass/fail. Advance on 3/4 pass with no Critical findings. Max 3 iterations.

## Tech Stack

- Next.js (App Router) + Tailwind CSS + TypeScript
- Server components by default, client only for interactivity
- `next/image` for all images, `next/font` for fonts
- Brand design tokens in `tailwind.config.ts`, not hardcoded

## Tooling Skills (build-time only)

- **tooling-page-analyzer** — Analyze a live URL and add it to the reference library

## Conventions

- Never commit to `main` during pipeline execution — use feature branches
- Never use `git add -A` — stage specific files
- Always run `npm run build` + `npm run lint` before committing
- Brand context values must be referenced by token name, not raw values

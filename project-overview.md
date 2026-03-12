# Autonomous Landing Page Builder — Project Overview

## What We're Building

An autonomous multi-agent system that takes a directive (e.g., "build a new landing page site for AT&T") along with brand assets and guidelines, and produces a live, hosted landing page with zero developer involvement during the build process.

The system uses a sequential pipeline of specialized agents, each producing structured context that downstream agents consume. At key checkpoints, parallel review teams evaluate the work and drive recursive improvement before the pipeline advances.

## The Directive

The directive is the initial input that kicks off the entire pipeline. It must include:

- **Landing page type**: What kind of page are we building? (lead gen, product launch, event signup, app download, etc.)
- **Desired outcome**: What are we optimizing for? (conversions, signups, awareness, downloads, etc.)
- **Brand assets**: Logo files, font files, color values, imagery
- **Brand guidelines**: PDF or document describing tone of voice, logo usage rules, typography rules, do's and don'ts
- **Any additional context**: Target audience, competitive references, specific copy or messaging requirements

## The Pipeline

The orchestration is a sequential pipeline with parallel review checkpoints. A single orchestrator session drives the flow from stage to stage, using subagents for focused work and spinning up agent teams for review rounds.

### Stage 1: Brand Context Agent

**Input**: Brand assets and guidelines from the directive (logos, fonts, colors, PDFs, etc.)

**What it does**: Ingests all brand materials and produces a structured brand context document (JSON or markdown) that every downstream agent receives. This is the guardrail that prevents generic, off-brand output.

**Output**: Structured brand context covering:
- Color palette (primary, secondary, accent) with exact hex/RGB values
- Typography rules (font families, weights, sizes, hierarchy)
- Logo usage rules (spacing, minimum sizes, acceptable backgrounds)
- Tone of voice guidelines (formal vs. casual, key phrases, words to avoid)
- Visual style notes (photography style, illustration style, iconography)

### Stage 2: Landing Page Strategy Agent

**Input**: Brand context + directive + landing page reference library

**What it does**: Reviews a library of previous landing pages and known strategies, then recommends a page structure and strategy appropriate for the directive's goals.

The reference library is a curated collection of:
- Screenshots of previous landing pages
- JSON metadata describing each page (layout structure, section types, CTA placement, strategy, performance data if available)
- Descriptions of common landing page patterns and when they work best

The agent selects one strategy or combines elements from multiple strategies based on what fits the landing page type and desired outcome from the directive.

**Output**: A recommended page strategy document describing the chosen layout structure, section order, CTA strategy, and rationale for the choices — all grounded in the brand context.

### Stage 3: Business Analyst Agent — PRD

**Input**: Brand context + page strategy

**What it does**: Writes a Product Requirements Document (PRD) defining exactly what the landing page needs to include, how it should behave, and what success looks like.

**Output**: A complete PRD.

### Review Checkpoint: PRD Review

**How it works**: An agent team spins up with three reviewers operating in parallel:
- **Architect** — Reviews for technical feasibility, structure, and implementation concerns
- **Business Analyst** — Reviews for completeness, clarity, and alignment with the directive
- **Developer** — Reviews for implementability, ambiguity, and missing acceptance criteria

Each reviewer applies their lens independently, can message each other to debate findings, and votes pass/fail with specific feedback.

**Exit criteria**:
- Minimum 1 review iteration (always runs at least once)
- Maximum 3 iterations (hard cap)
- Early exit on majority vote (2 of 3 reviewers pass)

After each review round, the BA agent fixes the flagged issues before the next round. The agent team dissolves after the review loop completes.

### Stage 4: Business Analyst Agent — Story Writing

**Input**: Approved PRD + brand context

**What it does**: Breaks the PRD into implementable stories written in JSON format using a Claude skill for spec-driven development. Each story is a self-contained unit of work that an implementer agent can pick up and execute.

**Output**: An ordered set of JSON stories forming an epic.

### Review Checkpoint: Story Review

**Same structure as PRD review** — architect, BA, and developer review the stories in parallel as an agent team. They look for:
- Missing edge cases or acceptance criteria
- Incorrect sequencing or dependencies between stories
- Stories that are too large or too vague to implement cleanly
- Gaps between the stories and the PRD

Same exit criteria: min 1, max 3 iterations, majority vote to pass. Fixes happen between rounds. Team dissolves when done.

### Stage 5: Developer Agent — Implementation

**Input**: Approved stories + brand context + clean Next.js project

**What it does**: Implements one story at a time in sequence. After completing each story:

1. Developer agent implements the story
2. An agent team spins up for a story-level review (architect, BA, developer)
3. Flagged issues are fixed, re-reviewed (same min 1 / max 3 / majority vote rules)
4. Once approved, the work is committed to the branch
5. Move to the next story

This continues until the entire epic is complete.

### Stage 6: Final Review

**Input**: The completed landing page (full branch)

**What it does**: A final agent team review of the entire finished product against the original directive, brand context, and PRD. This is the last gate before deployment.

## Orchestration Architecture

### Why a hybrid approach

The pipeline is fundamentally sequential — each stage produces context that the next stage consumes. Agent teams (Claude Code experimental feature) are designed for parallel work with inter-agent communication, not sequential handoffs.

However, the review checkpoints are genuinely parallel — multiple reviewers examining the same artifact simultaneously from different angles, debating, and converging on feedback. This is exactly what agent teams excel at.

**The hybrid**:
- A **single orchestrator session** drives the pipeline from stage to stage
- **Subagents** handle focused, isolated work within a stage (e.g., "parse this PDF into structured brand context")
- **Agent teams** (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) spin up specifically for review checkpoints where parallel review and inter-agent debate add real value
- Agent teams dissolve after each review round completes

### Review loop mechanics

Every review checkpoint follows the same pattern:

```
┌─────────────────────────────────────────────────┐
│                                                  │
│  Artifact ready for review                       │
│         │                                        │
│         ▼                                        │
│  Agent team spawns (architect, BA, developer)    │
│         │                                        │
│         ▼                                        │
│  Parallel review + inter-agent debate            │
│         │                                        │
│         ▼                                        │
│  Each reviewer votes pass/fail with feedback     │
│         │                                        │
│         ▼                                        │
│  Majority pass? ──yes──▶ Advance pipeline        │
│         │                                        │
│         no                                       │
│         │                                        │
│         ▼                                        │
│  Iteration < 3? ──no──▶ Advance pipeline (cap)   │
│         │                                        │
│         yes                                      │
│         │                                        │
│         ▼                                        │
│  Fix flagged issues                              │
│         │                                        │
│         └──────── loop back to review ───────┘   │
│                                                  │
└─────────────────────────────────────────────────┘
```

## Key Decisions

**Claude Code in autonomous mode** (`--dangerously-skip-permissions`) is the core engine, running inside Docker for isolation.

**Docker containers** provide isolation so Claude Code in dangerous mode cannot affect the host machine. The same container image runs locally for development and in the cloud for production.

**GCE (Google Compute Engine) VMs** are the target cloud compute, chosen over Cloud Run because the agentic loop may run longer than Cloud Run's 60-minute hard timeout. VMs self-terminate after the job completes.

**Vercel** is the deployment target. Claude Code calls the Vercel CLI directly to deploy and return a live URL.

**Clean Next.js installs** are used as the base for each landing page. Pages are kept fully isolated in their own repos — not part of the existing production mono-repo. The mono-repo question can be revisited later.

**GCP Secret Manager** stores the Anthropic API key and Vercel token, injected into the container at runtime.

**Hybrid orchestration** — sequential pipeline with agent teams for review checkpoints — rather than running the entire flow as one big agent team (which would leave agents idle waiting for upstream context and burn tokens unnecessarily).

## What's Not Decided Yet

- The trigger/intake UI (form, Slack command, API endpoint)
- How job status and results are surfaced back to the user
- The landing page reference library format and how it gets populated
- How the Claude skill for spec-driven story writing is structured
- Long-term: whether agent-built pages should eventually integrate with the production mono-repo

## Stack Summary

- **Agent runtime**: Claude Code (autonomous mode)
- **Orchestration**: Single session driving sequential pipeline + agent teams for reviews
- **Isolation**: Docker
- **Cloud compute**: GCE VM (self-terminating)
- **Deployment**: Vercel CLI
- **Secrets**: GCP Secret Manager
- **Base framework**: Next.js (clean install per landing page)

## Current Focus

Build the orchestration of agents to run in a local container. Infrastructure (GCE, triggers, status reporting) comes later. The immediate work is defining the agents, skills, and commands that drive this pipeline and getting the review loops working end to end.

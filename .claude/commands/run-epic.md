---
name: run-epic
description: Autonomously implement all stories — implement, review, fix, commit, repeat until done
---

# Autonomous Epic Runner

You are an autonomous implementer that works through all user stories. You implement each story directly (not via subagents), commit, run a team review, fix findings, and repeat until done.

**DO NOT ask the user any questions.** Make reasonable decisions and keep moving. If genuinely blocked, note the assumption in the commit message and continue.

**This command is designed to run in `--dangerously-skip-permissions` mode.**

## Pre-computed Context

**Current branch:** `$(git branch --show-current)`
**Repository:** `$(basename $(pwd))`

### Story Files

```
$(ls docs/stories/us_*.json 2>/dev/null || echo "ERROR: No story files found in docs/stories/")
```

### Uncommitted Changes

```
$(git status --short 2>/dev/null || echo "Not a git repo")
```

---

## Phase 0: Pre-flight Checks

Before doing ANY work, verify these conditions. **STOP if any check fails.**

### 0a. Story Directory
- Confirm `docs/stories/` exists and contains `us_*.json` files
- If no files found, STOP with error: "No story files found in docs/stories/"

### 0b. Branch Safety
- If current branch is `main`, create a new branch: `git checkout -b feat/landing-page`
- If already on a feature branch, continue on it

### 0c. Clean Working Tree
- If there are uncommitted changes, STOP with error: "Uncommitted changes detected. Please commit or stash before running."

### 0d. Brand Context
- Confirm `brand-context.json` exists — the developer needs it throughout implementation
- If missing, STOP with error: "brand-context.json not found. Run the brand context agent first."

### 0e. Clean Baseline
- Run `npm run build` (or verify the project builds)
- If it fails on a fresh project, that's OK — note it and continue
- If it fails on an existing project with prior stories implemented, STOP with error

---

## Phase 1: Story Discovery & Ordering

Read ALL `us_*.json` files in `docs/stories/`.

For each story, extract:
- `id` (e.g., "US-1")
- `title`
- `dependencies` (array of story IDs)

### Determining Completion

A story is complete if it has been committed to the branch. Check git log for commits mentioning the story ID.

### Ordering Rules

1. **Skip** stories already committed
2. **Topological sort** by `dependencies` — a story cannot start until all its dependencies are committed
3. If no dependencies, use filename order (us_1 before us_2)
4. If circular dependencies detected, STOP with error

### Output

Print the execution plan:

```
Stories to implement (in order):
  1. US-1: Global Styles & Tailwind Config
  2. US-2: Section Wrapper Component
  ...
Already completed:
  - (none)
```

---

## Phase 2: Story Loop

For each story in order, execute Steps A through F.

### Step A: Read Story & Research

1. Read the story JSON file completely. Extract:
   - `acceptance_criteria` — the contract to implement against
   - `test_cases` — specific test scenarios (if any)
   - `technical_notes` — implementation hints
   - `dependencies` — verify all are committed

2. Read the `brand-context.json` for design values
3. Check existing components in the codebase for patterns to follow
4. Read the `nextjs-landing-page` skill for implementation patterns

Do NOT enter plan mode. Research directly, then implement.

### Step B: Implement

Implement the story yourself in the main context. Do NOT delegate implementation to subagents.

Key implementation rules:
- Follow brand context exactly — use design token values from `tailwind.config.ts`, not arbitrary colors/sizes
- Mobile-first responsive design using Tailwind breakpoint prefixes
- Semantic HTML (section, nav, header, footer, main, article)
- All images use `next/image` with appropriate optimization
- All fonts use `next/font` with proper loading strategy
- Server components by default, client components only for interactivity
- No inline styles — Tailwind utility classes only
- Run `npm run build` and `npm run lint` frequently during implementation

### Step C: Quality Gate

Run `npm run build` and `npm run lint`.

If either fails:
- Read the output carefully
- Fix the issues
- Re-run until both pass
- Maximum 5 fix attempts — if still failing after 5, note the blocker and move on

### Step D: Commit

Stage specific files and commit:

```
git add <specific files>
git commit -m "$(cat <<'EOF'
Implement US-X: <title>

<2-5 line summary of what was built>
EOF
)"
```

**Do NOT use `git add -A`** — stage specific files only.

### Step E: Team Review

Invoke the team review command for this story's implementation:

```
/review/team-review US-X implementation — git diff HEAD~1
```

This dispatches 4 parallel reviewers (BA, architect, developer, adversarial).

### Step F: Apply Fixes & Iterate

After the review completes, read the review results.

**Fix ALL findings rated Critical or High.** Also fix Medium findings unless they require a large architectural change. Low findings: fix if trivial, otherwise note them.

After fixes:
1. Run `npm run build` and `npm run lint` — must pass
2. Commit (only if there were changes):
   ```
   git commit -m "$(cat <<'EOF'
   Fix US-X review findings: <brief list>
   EOF
   )"
   ```

**Review loop rules:**
- Minimum 1 review (always runs)
- Maximum 3 iterations (hard cap — advance even if issues remain)
- If 3/4 reviewers passed AND no Critical findings remain → advance to next story
- If the max is reached with unresolved issues, note them and move on

3. Print progress:
   ```
   --- Story Complete ---
   US-X: <title>
   Implementation: <hash>
   Review fixes: <hash or "none needed">
   Progress: N/total stories
   ----------------------
   ```

---

## Phase 3: Epic Complete

After ALL stories are done:

### Summary Table

```
| # | Story ID | Title | Commits | Review Findings Fixed |
|---|----------|-------|---------|-----------------------|
| 1 | US-1     | ...   | abc     | 5                     |
| 2 | US-2     | ...   | def,ghi | 3                     |
...
```

### Final Message

```
Landing page implementation complete!

All N stories implemented and reviewed.
Branch: <branch_name>
Total commits: M

Next steps:
- Run /review/team-review for a final full-page review
- Deploy when ready
```

**Do NOT automatically create a PR or deploy** — let the calling command (run-pipeline) or user decide.

---

## Error Handling

### Quality Gate Failure (persistent)
If `npm run build` fails 5 times in a row for the same issue:
1. Note the blocker
2. Continue to next story if dependencies allow

### Genuinely Blocked
If a story requires information that doesn't exist (missing brand asset, unclear requirement):
1. Note the assumption you're making
2. Implement with the assumption
3. Flag it in the commit message
4. Do NOT stop and ask

---

## Important Rules

1. **NEVER ask the user questions** — make decisions and move forward
2. **NEVER enter plan mode** — research directly, then implement
3. **NEVER delegate implementation to subagents** — only delegate reviews
4. **NEVER commit to `main`** — always use a feature branch
5. **NEVER use `--no-verify`** on commits
6. **NEVER use `git add -A` or `git add .`** — stage specific files
7. **ALWAYS run the team review** after each story implementation
8. **ALWAYS read brand-context.json** before implementing visual elements
9. **ALWAYS run quality checks** before every commit
10. **ALWAYS use descriptive commit messages** with summaries

---
name: 'vibeflow-analyze'
description: 'Deep-analyze codebase and build .vibeflow/ knowledge'
agent: 'vibeflow-architect'
---

# Vibeflow: Analyze

> format-agnostic, repo-local prompt asset

Deep-analyze the current codebase. Discovers stack, architecture,
patterns, conventions, key components, and pitfalls. Creates curated
documentation in `.vibeflow/` that persists and can be committed to git.
Supports incremental analysis: if `.vibeflow/` already exists, detects
changes via git and updates only affected modules.

**Usage:** Run this prompt from the project root. Pass `--fresh` as input to force a complete rebuild.

---

## Language

Detect the language of the user's conversation context.
Write ALL output in that same language.
Technical terms in English are acceptable regardless of the detected language.
Section names in generated `.vibeflow/` files may be in English (they are technical),
but all descriptive text, analyses, observations, and final user reports should be
in the detected language.

Perform a deep, adaptive analysis of this codebase. Your goal is to build
curated pattern documentation that will be used by every future spec,
prompt pack, and audit to ensure implementations follow the project's
real conventions.

## Phase 0: Detect Mode (Incremental vs Fresh)

Check the current state to decide whether to run a fresh analysis or incremental update:
- Does `.vibeflow/index.md` exist?
- Did the user request `--fresh`?
- Is git available in this directory?

**Decision tree:**
- If `.vibeflow/` DOES NOT exist OR `--fresh` requested → proceed with full analysis (Phases 1-5)
- If `.vibeflow/` EXISTS AND no `--fresh` → enter incremental mode:
  1. Read `Analyzed: <date>` from `.vibeflow/index.md`
  2. Run `git log --since="<date>" --name-only --pretty=format:""` to find files changed since that analysis date
  3. Filter out test files, documentation files, config files that don't affect source code structure (e.g., exclude `*.test.*`, `*.spec.*`, `docs/`, `README.md`, `.env.*`, etc.)
  4. Map remaining changed files to modules/directories to determine which structural units were affected
  5. If NO source files changed → report "No changes detected since <date>. `.vibeflow/` is up to date." and STOP (exit early)
  6. If git is not available → warn "Git not available. Falling back to full analysis." and proceed with full analysis (Phases 1-5)
  7. If source files DID change → report "Incremental mode: X affected modules: [list]" and proceed to Phase 1 with incremental scoping

## Phase 1: Discovery (broad scan)

**Incremental mode:** Only re-detect knowledge sources that changed per git diff. Reuse existing stack/type/structure from index.md for unchanged aspects. In fresh mode, proceed as described below.

Read the project root to understand what you're dealing with:
- package.json, Cargo.toml, pyproject.toml, go.mod, build.gradle, etc.
- Top-level directory structure (2-3 levels deep)
- README, CLAUDE.md, .cursorrules, .github/copilot-instructions.md
- Any existing architecture docs (ARCHITECTURE.md, ADRs, docs/ folder)

**Sources of knowledge to detect explicitly:**
- `.cursorrules` (project-level rules)
- `.cursor/rules/*.mdc` (domain-specific rules)
- `CLAUDE.md` (project instructions)
- `.clinerules` (Cline AI rules)
- `.github/copilot-instructions.md` (GitHub Copilot rules)
- `/docs/` directory (if present)
- `ARCHITECTURE.md` (if present)
- `ADRs/` directory (if present)

**Directories to ignore:** `node_modules/`, `.venv/`, `venv/`, `dist/`, `build/`, `.next/`, `__pycache__/`, `.git/`

Determine:
- What stack is this? (languages, frameworks, runtime, DB)
- What type of project? (monorepo, single app, library, CLI, API, mobile, etc.)
- What are the major structural units? (this varies by project type — could be
  modules, packages, routes, features, crates, services, etc.)

**Report which sources of knowledge were found.**

DO NOT assume monorepo, Next.js, KMP, or any specific structure.
Let the code tell you what it is.

**Domain classification:** Based on what was detected, classify the project into one of:
- **mobile** — Android, iOS, KMP, Flutter, React Native
- **web-frontend** — React, Vue, Angular, Svelte, Next.js, Nuxt
- **api-backend** — REST API, GraphQL, gRPC, microservices
- **library** — reusable library/SDK
- **cli** — command-line tool
- **other** — anything else

Based on the domain, activate **mandatory pattern priorities** (used in Phase 2 and 3):

**Mobile:** design-system/UI-components (REQUIRED), screen/feature-composition (REQUIRED), navigation (REQUIRED), state-management, networking/API-layer, DI, analytics, feature-flags, i18n

**Web frontend:** design-system/component-library (REQUIRED), page/route-composition (REQUIRED), state-management (REQUIRED), API-layer, auth, i18n

**API/backend:** route/endpoint-definition (REQUIRED), data-access/repository (REQUIRED), auth/middleware (REQUIRED), error-handling, DB-migrations, background-jobs

**Library/CLI/other:** no mandatory patterns — use existing heuristics

## Phase 1.5: Rules Integration

**Incremental mode:** Only re-extract rules from knowledge sources that changed. Merge with existing rules map (reuse unchanged rules). In fresh mode, proceed as described below.

Read all sources of knowledge found in Phase 1. From each, extract:
- **Concrete conventions** (naming rules, patterns, business rules, style rules)
- **Modules/subsystems mentioned** (entity names, feature domains, layer names)

Produce an internal map (not a final output file):
- List of conventions extracted (with source attribution: "via .cursorrules", "via docs/ARCHITECTURE.md", etc.)
- List of modules/subsystems mentioned in rules
- **Principle:** Rules are privileged input, not absolute truth. Validate against code. Flag conflicts: "⚠️ Conflict: .cursorrules says X, but code does Y"
- Distinguish sources: `.cursorrules` + `CLAUDE.md` = project-level rules; `.cursor/rules/*.mdc` = domain-specific; `/docs/` = reference docs

**Purpose of this map:** Guide the sampling strategy in Phase 2 and the deep dive scope in Phase 3.

## Phase 2: Convention Mining (Adaptive Sampling)

**Incremental mode:** Only re-sample modules that had file changes. Preserve existing sampling results for unaffected modules. In fresh mode, proceed as described below.

**Heuristic for sampling:**
- Detect modules/domains automatically: by directory, file prefix (e.g., `fin_*.py`, `task_*.js`), and mentions in rules from Phase 1.5

**Sampling scale** (based on estimated total source files):

| Source files | Min files to sample | Per module |
|---|---|---|
| ≤50 | 8 | ≥2 |
| 51–200 | 12 | ≥2 |
| 201–1000 | 20 | ≥2 |
| 1001–5000 | 30 | ≥3 |
| 5001–20000 | 40 | ≥3 |
| 20000+ | 50–60 | ≥3 |

Prioritize: largest files by line count, most imported by others, mentioned in rules.
For repos with >100 source files: explicitly document what was NOT covered in "Known Gaps" section of index.md.

**Cross-module sampling** (for repos with 1000+ source files):
Instead of sampling many files from a few modules, sample the SAME LAYER across multiple modules/features:
- Pick 3–4 features and read the same layer (e.g., UI, data, navigation) from each
- This reveals the real pattern by repetition, not by exception
- Example: read the main screen composable from 4 different product features to find the UI pattern

**Mandatory pattern verification:**
After sampling, check the mandatory pattern list from Phase 1 domain classification.
For each REQUIRED pattern not yet covered by sampling:
- Actively search for files matching that domain (e.g., search for design system imports, screen composables, route definitions)
- Sample 2–3 additional files specifically for that pattern
- If truly absent from the codebase, document it as "Not found" rather than silently omitting

Document (for all sampled files):
- Naming conventions (files, functions, variables, types, components)
- File/directory organization patterns
- Import/export patterns
- Error handling approach
- Test patterns (location, naming, framework, coverage approach)
- State management patterns (if applicable)
- Typing patterns (strict, loose, any usage)
- Logging patterns

Look for REPEATED patterns, not one-off occurrences.

## Phase 3: Pattern Deep Dive

**Incremental mode:** Only re-analyze patterns for affected modules. For pattern docs being updated, preserve content outside `<!-- vibeflow:auto:start/end -->` markers. For existing pattern docs without markers (legacy), rewrite them entirely with markers added. For new patterns, create with markers. In fresh mode, proceed as described below.

This is the most important phase. For each significant pattern you discover:
- Read 3-5 examples of the pattern being used in the codebase
- Extract the REAL pattern, not a theoretical one
- Note variations and edge cases
- Identify the "right way" vs. deviations

**Expand scope using rules map:** Consider modules/subsystems from the Phase 1.5 rules map, not just those found in Phase 2 sampling. If a rule mentions a module the sampling didn't cover (e.g., "fitness module" in rules but not sampled), read files from that module and decide if it deserves a pattern doc.

**Cross-module pattern rule:** For pattern docs that document a horizontal layer
(UI composition, data access, navigation, design system usage, state management),
include examples from at least 3 different features/modules. This ensures the
documented pattern is the real convention, not an outlier from a single feature.

What counts as a "significant pattern" depends on the project. Examples:
- API/route definitions and their structure
- Component architecture (if frontend)
- Data access / repository patterns
- Navigation patterns (if mobile/SPA)
- Design system usage and component composition
- Dependency injection / service patterns
- Configuration and environment handling
- Build/deploy patterns
- Auth flows
- Middleware/plugin patterns
- CLI command patterns
- Event/message handling patterns

Create ONE markdown file per pattern in `.vibeflow/patterns/`.
Name files descriptively: `api-routes.md`, `component-architecture.md`,
`data-access.md`, `auth-flow.md`, `cli-commands.md`, etc.

Each pattern doc must follow this structure with markers to delimit auto-generated content:

```markdown
# Pattern: <Name>

<!-- vibeflow:auto:start -->
## What
1-2 sentence description of what this pattern is.

## Where
Which parts of the codebase use it.

## The Pattern
Show the REAL pattern with actual code examples from this repo.
Include 2-3 concrete examples so a coding agent can replicate it exactly.

## Rules
- Specific rules this pattern follows
- Naming conventions specific to this pattern
- What to do / what NOT to do

## Examples from this codebase
File: <real path>
<actual code snippet showing the pattern correctly>

File: <real path>
<another example, ideally showing a variation>
<!-- vibeflow:auto:end -->

## Anti-patterns (if found)
Things that exist in the codebase that BREAK this pattern.
Mark them so future work doesn't replicate mistakes.
```

**Marker placement rule:** Wrap sections `## What`, `## Where`, `## The Pattern`, `## Rules`, and `## Examples from this codebase` with `<!-- vibeflow:auto:start/end -->` markers. The `## Anti-patterns` section should NOT be wrapped — it's a natural place for manual additions and evolution.

CRITICAL: Include REAL code snippets from REAL files. Not pseudocode.
Not "something like this". The actual code. This is what makes the
patterns actionable for a coding agent.

## Phase 4: Compile

**Incremental mode:**
- **index.md:** Update `Analyzed: <date>` to the current date. Update Structural Units and Key Files only if they changed. Preserve any manually added Known Issues section.
- **conventions.md:** Update only sections within `<!-- vibeflow:auto:start/end -->` markers. Preserve content outside markers (this is where manual additions live). If no markers exist in the file (legacy format), rewrite the entire file with markers added.
- **decisions.md:** NEVER modify in incremental mode. This file is manually curated and only touched by the architect during audits.
- **Pattern docs in `patterns/`:** For docs being updated, preserve content outside markers. For legacy docs without markers, rewrite with markers. For new patterns, create with markers.

In fresh mode, proceed as described below.

### Create .vibeflow/ directory structure

```
.vibeflow/
├── index.md              # Overview: stack, structure, list of all pattern docs
├── conventions.md        # Coding conventions (naming, style, organization)
├── patterns/
│   └── <whatever>.md     # One file per discovered pattern (varies by repo)
└── decisions.md          # Empty for now, grows with use
```

### index.md format (~80-120 lines max):

```markdown
# Project: <name>
> Analyzed: <date>
> Stack: <concise summary>
> Type: <project type>
> Suggested budget: ≤ N files per task

## Structure
<brief description of how the project is organized>

## Structural Units
<list the major units — modules, packages, routes, features, etc.>
<1-line description each>

## Pattern Docs Available
<list each .vibeflow/patterns/*.md with 1-line description>

## Key Files
<10-15 most critical files with 1-line descriptions>

## Dependencies (critical only)
<critical deps with 1-line rationale>

## Known Issues / Tech Debt
<bullet list if anything was found>
```

**Budget calculation:** Count the total number of source files sampled/detected
in Phase 2. Suggest a budget of ~2-3% of total source files, clamped between
a minimum of 4 and a maximum of 12:
- ≤50 files → budget ≤ 4
- 51–150 files → budget ≤ 6
- 151–500 files → budget ≤ 8
- 501–2000 files → budget ≤ 10
- 2000+ files → budget ≤ 12

Write the result as `> Suggested budget: ≤ N files per task` in the
index.md header. This is used by gen-spec and prompt-pack as the default budget.

### conventions.md format:

Dense, specific, actionable. NOT vague guidelines — concrete rules with examples.
A coding agent should be able to read this and write code that fits in perfectly.

**Markers in conventions.md:** Wrap the main auto-generated convention sections
with `<!-- vibeflow:auto:start -->` and `<!-- vibeflow:auto:end -->`. This allows
manual additions (team updates, discovered edge cases) to persist outside the
markers when running incremental updates.

**Incorporate rules from Phase 1.5:** Conventions extracted from rules/instructions
must be included in conventions.md with source attribution: "(via .cursorrules)",
"(via .cursor/rules/design-system.mdc)", "(via docs/ARCHITECTURE.md)". If a rule
contradicts code, signal conflict: "⚠️ Conflict: .cursorrules says X, but the code does Y".

### decisions.md:

**Important:** This file is NEVER modified by the analyze command, neither in
fresh nor incremental mode. It is reserved for the architect and manual curation.

On fresh run (first analysis), create the file with a header and empty content
only if it doesn't already exist:

```markdown
# Decision Log
> Newest first. Updated by the architect during specs and audits.
```

On incremental run, leave `decisions.md` untouched.

## Phase 5: Update Index

After compiling all `.vibeflow/` docs, ensure `.vibeflow/index.md` has an
up-to-date list of all pattern docs and their 1-line descriptions under
the "Pattern Docs Available" section.

## After Saving, Report to the User:

- Stack detected
- Project type identified
- Number of structural units mapped
- Sources of knowledge found and incorporated (count of .cursorrules, CLAUDE.md, docs/, etc.)
- Number of pattern docs created (list them)
- Top 3 patterns that are most critical for future work
- Conflicts detected between rules and code (if any)
- Any red flags, inconsistencies, or tech debt found
- Remind them: these docs persist and will be used in all future specs, prompt packs, and audits
- Suggest: "Review `.vibeflow/` and commit it — it's your project's living documentation."

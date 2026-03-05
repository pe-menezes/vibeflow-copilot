---
name: 'vibeflow-analyze-satellite'
description: 'Analyze a satellite repo (e.g. design system) from the main repo perspective; clone, analyze, filter by usage, merge with provenance, remove clone'
agent: 'vibeflow-architect'
---

# Vibeflow: Analyze satellite repo

Analyze a **satellite repository** (e.g. design system, shared lib) from the perspective of the **current (main) repo**. Clone the satellite to a temp dir, run the same analyze pipeline on the clone, detect what the main repo uses (imports, declared deps), keep only those patterns, merge into `.vibeflow/patterns/satellite-<name>/` with explicit provenance, then remove the clone.

**Usage:** Run from the **main repo root**. One argument required: satellite repo URL (SSH or HTTPS). Example: `vibeflow-analyze-satellite https://github.com/org/design-system`

**Requirement:** `.vibeflow/index.md` must already exist in the main repo (run `vibeflow-analyze` first).

---

## Language

Use the same language as the user. Technical terms in English are acceptable.

---

## Phase 0: Validate and parse input

1. **Argument required.** User must provide exactly one argument: the satellite repo URL. If missing or invalid, stop with: "Usage: provide the satellite repo URL (SSH or HTTPS). Example: vibeflow-analyze-satellite https://github.com/org/design-system"

2. **Main repo must have run analyze.** Check that `.vibeflow/index.md` exists in the current working directory. If not, stop with: "Run vibeflow-analyze in the main repo first, then run analyze-satellite with the dependency URL."

3. **Derive satellite name.** From the URL, take the last path segment (e.g. `design-system` from `github.com/org/design-system`), remove `.git` if present, sanitize to alphanumeric and hyphen. Use as `<satellite-name>` for `.vibeflow/patterns/satellite-<satellite-name>/`.

---

## Phase 1: Clone to temp and run analyze on clone

1. **Temp directory.** Use a unique path under system temp (e.g. `$TMPDIR/vibeflow-satellite-<timestamp>`). Do not use the main repo.

2. **Clone.** Run `git clone --depth 1 <URL> <temp_dir>/satellite`. On failure, report clearly and remove temp dir if partially created, then stop.

3. **Run analyze on the clone.** With the clone directory as the effective codebase root, perform the same phases as the vibeflow-analyze prompt (Discovery, Rules integration, Convention mining, Pattern deep dive, Compile). Write output **inside the clone**: create `<temp_dir>/satellite/.vibeflow/` with `index.md`, `conventions.md`, and `patterns/*.md`. Do not modify the main repo in this step.

4. **Cleanup on failure.** If any step fails after clone, still remove the temp directory before exiting.

---

## Phase 2: Detect usage in the main repo

In the **main repo** (current working directory):

- **Declared dependencies:** Read `package.json`, `build.gradle`, `Cargo.toml`, `pyproject.toml`, or equivalent. Note the satellite's package or module name.
- **Imports / requires:** Search source files for import/require of the satellite. Cover at least JS/TS and the main repo's primary language.
- **Build a "usage" set:** Module names, package names, or symbols the main repo uses from the satellite. Use to filter which pattern docs from the satellite to merge.

---

## Phase 3: Filter and merge with provenance

1. **Filter.** From `<temp_dir>/satellite/.vibeflow/patterns/*.md`, keep only docs that correspond to the usage set. When in doubt for a direct dependency, include.

2. **Target:** `.vibeflow/patterns/satellite-<satellite-name>/` in the main repo. Create if needed. Overwrite as needed (v0).

3. **Provenance.** Prepend at the top of each merged file a blockquote: `> Patterns from satellite repo: <satellite-name> (ingested on YYYY-MM-DD for use by the main repo).` Use current date. Use the user's language for the sentence if not English.

4. **Write only filtered pattern docs** into the target folder. Do not copy the satellite's index.md or conventions.md.

---

## Phase 4: Cleanup and report

1. **Remove temp directory.** Run `rm -rf <temp_dir>`. Do this whether merge succeeded or failed after clone.

2. **Report:** Satellite URL and name; that clone was removed; how many pattern docs merged and into which folder; remind provenance; suggest running gen-spec or prompt-pack.

---

## Rules

- One repo per invocation. Clone is always ephemeral. Provenance is mandatory in every merged doc. Filter by use only.

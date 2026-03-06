---
description: >
  Analyze a satellite repo (e.g. design system) from the main repo's perspective.
  Clones the repo to a temp dir, runs the same analyze pipeline on the clone,
  detects what the main repo uses (imports, deps), keeps only those patterns,
  merges into .vibeflow/patterns/satellite-<name>/ with provenance, then removes the clone.
  Usage: /vibeflow:analyze-satellite <satellite-repo-URL>
---

## Description and examples

**What it does:** Enriches your project's `.vibeflow/` with patterns from a dependency repo (e.g. design system). Clones the satellite, analyzes it, keeps only what your main repo actually uses, writes to `.vibeflow/patterns/satellite-<name>/` with clear provenance, then deletes the clone. Run from the **main** repo root; `.vibeflow/` must already exist (run `/vibeflow:analyze` first).

**Examples:**
- `/vibeflow:analyze-satellite https://github.com/org/design-system` — Analyze the design-system repo and merge only the patterns your code uses.
- `/vibeflow:analyze-satellite git@github.com:org/shared-lib.git` — Same with SSH URL.

---

## Language

Detect the language of the user's conversation. Write all output in that language. Technical terms in English are acceptable.

---

## Phase 0: Validate and parse input

1. **Argument required.** The user must provide exactly one argument: the satellite repo URL (e.g. `https://github.com/org/design-system` or `git@github.com:org/design-system.git`). If missing or invalid, stop with: "Usage: /vibeflow:analyze-satellite <satellite-repo-URL>. Example: /vibeflow:analyze-satellite https://github.com/org/design-system"

2. **Main repo must have run analyze.** Check that `.vibeflow/index.md` exists in the **current working directory** (the main repo). If not, stop with: "Run `/vibeflow:analyze` in the main repo first to establish project context, then run analyze-satellite with the dependency URL."

3. **Derive satellite name.** From the URL, extract a sanitized repo name: take the last path segment (e.g. `design-system` from `github.com/org/design-system` or `design-system.git`), remove `.git` if present, allow only alphanumeric and hyphen. Use this as `<satellite-name>` for `.vibeflow/patterns/satellite-<satellite-name>/`.

---

## Phase 1: Clone satellite to temp and run analyze there

1. **Create temp directory.** Use a unique path under the system temp (e.g. `$TMPDIR/vibeflow-satellite-<timestamp>`). Do not use the main repo.

2. **Clone.** Run `git clone --depth 1 <URL> <temp_dir>/satellite`. If clone fails (SSH key, network, private repo), report the error clearly and remove the temp dir if it was partially created, then stop.

3. **Run analyze on the clone.** With the **clone directory** as the effective codebase root, perform the same phases as `/vibeflow:analyze` (Discovery, Rules integration, Convention mining, Pattern deep dive, Compile). Write the output **inside the clone**: create `<temp_dir>/satellite/.vibeflow/` and write `index.md`, `conventions.md`, and `patterns/*.md` there. Do not modify the main repo's `.vibeflow/` in this step.

4. **Cleanup on failure.** If any step fails after the clone, still remove the temp directory before exiting.

---

## Phase 2: Detect usage in the main repo

In the **main repo** (current working directory), collect what references the satellite:

- **Declared dependencies:** Read `package.json` (dependencies, devDependencies), `build.gradle`, `Cargo.toml`, `pyproject.toml`, or equivalent. Note the satellite's package or module name.
- **Imports / requires:** Search source files for import/require of the satellite (e.g. `from '@org/design-system'`, `require('@org/design-system')`, Kotlin/Java imports of the satellite package). Cover at least JS/TS and the main repo's primary language.
- **Build a "usage" set:** Module names, package names, file paths, or component/symbol names that the main repo uses from the satellite. Use this to filter which pattern docs from the satellite are relevant.

---

## Phase 3: Filter and merge with provenance

1. **Filter pattern docs.** From `<temp_dir>/satellite/.vibeflow/patterns/*.md`, keep only those that correspond to the "usage" set (components, modules, or APIs that appear in the main repo's imports or dependency list). When in doubt for a direct dependency, include; otherwise keep the main repo's `.vibeflow/` focused.

2. **Target in main repo:** `.vibeflow/patterns/satellite-<satellite-name>/`. Create if needed. Overwrite or create files as needed for the same satellite (v0: overwrite).

3. **Provenance in every file.** For each merged pattern file, prepend at the very top (before the title) a blockquote:
   `> Patterns do repositório satélite: <satellite-name> (ingestado em YYYY-MM-DD para uso pelo repo principal).`
   Or in English: `> Patterns from satellite repo: <satellite-name> (ingested on YYYY-MM-DD for use by the main repo).`
   Use the current date in YYYY-MM-DD.

4. **Write only the filtered pattern docs** into `.vibeflow/patterns/satellite-<satellite-name>/`. Do not copy the satellite's `index.md` or `conventions.md`; only filtered pattern docs with the provenance line added.

---

## Phase 4: Cleanup and report

1. **Remove the temp directory.** Run `rm -rf <temp_dir>` so the clone is fully deleted. Do this whether the merge succeeded or failed after the clone.

2. **Report to the user:**
   - Satellite repo URL and derived name
   - That the clone was removed
   - How many pattern docs were merged and into which folder
   - Remind that those patterns are from the satellite and are tagged with the provenance line
   - Suggest: "Run `/vibeflow:gen-spec` or `/vibeflow:prompt-pack` — patterns from `<satellite-name>` are now available under `.vibeflow/patterns/satellite-<satellite-name>/`."

---

## Rules

- One repo per invocation (v0). Clone is always ephemeral. Provenance is mandatory in every merged doc. Filter by use: only merge patterns that match what the main repo uses.

---

## Maintenance

If this command is modified, update `MANUAL.md` and `claude-code/README.md` to reflect the usage and provenance convention.

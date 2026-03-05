---
name: vibeflow-analyze-satellite
description: "Analyze a satellite repo (e.g. design system) from the main repo's perspective. Clones the repo, runs analyze, keeps only patterns the main repo uses, merges into .vibeflow/ with clear provenance, then removes the clone. Use when the main project depends on another repo and you want to enrich patterns from that dependency."
---

# Vibeflow: Analyze satellite repo

Analyze a **satellite repository** (e.g. design system, shared lib) from the perspective of the **current (main) repo**. The agent clones the satellite, runs the same analysis pipeline as `vibeflow-analyze` on it, detects what the main repo actually uses (imports, declared deps), keeps only those patterns, and merges them into `.vibeflow/patterns/satellite-<repo>/` with explicit provenance. The clone is removed at the end.

**Usage:** Invoke from the **main repo root** with one argument: the satellite repo URL (SSH or HTTPS).

Example: `vibeflow-analyze-satellite https://github.com/org/design-system` or `vibeflow-analyze-satellite git@github.com:org/design-system.git`

---

## Language

Use the same language as the user's conversation. Technical terms in English are acceptable.

---

## Phase 0: Validate and parse input

1. **Argument required.** The user must provide exactly one argument: the satellite repo URL (e.g. `https://github.com/org/design-system` or `git@github.com:org/design-system.git`). If missing or invalid, stop with: "Usage: provide the satellite repo URL (SSH or HTTPS). Example: vibeflow-analyze-satellite https://github.com/org/design-system"

2. **Main repo must have run analyze.** Check that `.vibeflow/index.md` exists in the **current working directory** (the main repo). If not, stop with: "Run `vibeflow-analyze` in the main repo first to establish project context, then run analyze-satellite with the dependency URL."

3. **Derive satellite name.** From the URL, extract a sanitized repo name for the folder: take the last path segment (e.g. `design-system` from `github.com/org/design-system` or `design-system.git`), remove `.git` if present, and allow only alphanumeric and hyphen (e.g. `design-system`). Use this as `<satellite-name>` for `.vibeflow/patterns/satellite-<satellite-name>/`.

---

## Phase 1: Clone satellite to temp and run analyze there

1. **Create temp directory.** Use a unique path under the system temp (e.g. `$TMPDIR/vibeflow-satellite-<timestamp>` or `os.tmpdir()`). Do not use the main repo.

2. **Clone.** Run `git clone --depth 1 <URL> <temp_dir>/satellite`. If clone fails (SSH key, network, private repo), report the error clearly and do not leave the temp dir behind: remove it if it was partially created, then stop.

3. **Run analyze on the clone.** With the **clone directory** as the effective codebase root, perform the same phases as the `vibeflow-analyze` skill (Discovery, Rules integration, Convention mining, Pattern deep dive, Compile). Write the output **inside the clone**: create `<temp_dir>/satellite/.vibeflow/` and write `index.md`, `conventions.md`, and `patterns/*.md` there. Do not modify the main repo's `.vibeflow/` in this step. Use the clone's package name or repo structure to identify "what this repo exports" (e.g. package name from `package.json`, or module paths).

4. **Ensure cleanup on failure.** If any step fails after the clone, still remove the temp directory before exiting.

---

## Phase 2: Detect usage in the main repo

In the **main repo** (current working directory), collect everything that references the satellite:

- **Declared dependencies:** Read `package.json` (dependencies, devDependencies), `build.gradle`, `Cargo.toml`, `pyproject.toml`, or equivalent for the main repo's stack. Note the satellite's package or module name (e.g. `@org/design-system`, `com.org.designsystem`).
- **Imports / requires:** Search source files for import/require of the satellite (e.g. `from '@org/design-system'`, `require('@org/design-system')`, Kotlin/Java imports of the satellite package). Prefer regex or grep over full parsing; cover at least JS/TS and the main repo's primary language.
- **Build a "usage" set:** List of module names, package names, file paths, or component/symbol names that the main repo uses from the satellite. Use this to filter which pattern docs from the satellite are relevant.

---

## Phase 3: Filter and merge with provenance

1. **Filter pattern docs.** From `<temp_dir>/satellite/.vibeflow/patterns/*.md`, keep only those that correspond to the "usage" set: e.g. patterns for components, modules, or APIs that appear in the main repo's imports or dependency list. If a pattern doc's topic (filename or content) matches a used module/component/path, include it. When in doubt (e.g. generic "conventions" from the satellite), include if the satellite is a direct dependency; otherwise you may exclude to keep the main repo's `.vibeflow/` focused.

2. **Target directory in main repo:** `.vibeflow/patterns/satellite-<satellite-name>/`. Create it if it does not exist. Overwrite or create files as needed (v0: overwrite existing files in that folder for the same satellite).

3. **Provenance in every file.** For each merged pattern file, prepend at the very top (before the title) a blockquote line:
   `> Patterns do repositório satélite: <satellite-name> (ingestado em YYYY-MM-DD para uso pelo repo principal).`
   Use the current date in YYYY-MM-DD. If the user's language is English, use: `> Patterns from satellite repo: <satellite-name> (ingested on YYYY-MM-DD for use by the main repo).`

4. **Write only the filtered pattern docs** into `.vibeflow/patterns/satellite-<satellite-name>/`. Do not copy the satellite's `index.md` or `conventions.md` into the main repo; only pattern docs that passed the filter, with the provenance line added.

---

## Phase 4: Cleanup and report

1. **Remove the temp directory.** Run `rm -rf <temp_dir>` (or equivalent) so the clone is fully deleted. Do this whether the merge succeeded or failed after the clone.

2. **Report to the user:**
   - Satellite repo URL and derived name
   - That the clone was removed
   - How many pattern docs were merged and into which folder (`.vibeflow/patterns/satellite-<satellite-name>/`)
   - Remind that those patterns are from the satellite and are tagged with the provenance line
   - Suggest: "Run `gen-spec` or `prompt-pack` — patterns from `<satellite-name>` are now available under `.vibeflow/patterns/satellite-<satellite-name>/`."

---

## Rules

- **One repo per invocation.** Do not accept multiple URLs in v0.
- **Clone is always ephemeral.** Never leave the cloned repo on disk after the command finishes.
- **Provenance is mandatory.** Every merged doc must have the blockquote at the top with satellite name and date.
- **Filter by use.** Only merge pattern docs that match what the main repo uses (imports/deps). Do not merge the entire satellite `.vibeflow/` unfiltered.

---

## Maintenance

If this skill is modified, update `MANUAL.md` and the Cursor README to reflect the usage and provenance convention.

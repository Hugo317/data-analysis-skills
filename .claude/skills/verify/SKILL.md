---
name: verify
description: Run the project's checks for data-analysis-skills — every skill under .claude/skills is valid, and the analysis notebooks still have the data files they read. Use after editing a skill or notebook, before committing, or when asked to verify that the project still works.
---

# Verify

Run these checks from the project root, in order. Report failures with the actual output; do not
summarize a failure as a pass.

1. **Skills are valid.** For every directory in `.claude/skills/`:
   - it contains a `SKILL.md`
   - the file starts with YAML frontmatter that parses, with non-empty `name` and `description`
   - `description` says *when* to use the skill, not just what it does
   - every sibling file the body points to (e.g. `references/foo.md`) exists

   See the `skill-spec` skill for the full rules.
2. **Notebooks have their data.** Every `.ipynb` in the project root that reads a CSV
   (`Cuisine_rating.csv`, `Cuisine_rating_clean.csv`, or a `_clean_vN` version) must find that file
   next to it. List any notebook whose input file is missing.
3. **Dependencies are in sync:** `uv sync`.

Say which checks passed, which failed, and which you skipped and why. Never substitute a bare
`python`, `pip`, or `pytest` for the `uv run` form — that bypasses the project venv.

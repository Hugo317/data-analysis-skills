---
name: dataset-cleanup
description: Interactive, non-destructive dataset cleaning workflow for duplicates, missing values, impossible/outlier values, inconsistent categories, and wrong data types. Use this whenever the user uploads or references a raw/messy dataset (csv, xlsx, tsv, json, or a dataframe) and asks to clean it, check data quality, dedupe it, handle missing values, fix column types, or prep data for analysis — even if they don't say "clean" explicitly (e.g. "get this ready for analysis", "check this dataset", "something's off with this data"). Always use this skill instead of silently dropping/filling/deduping/casting on your own — the whole point is that the user is walked through every non-trivial decision before it's applied.
---

# Dataset Cleanup

A guided, transparent cleaning workflow. Claude never edits the user's original file. Every fix is applied to a copy, and every non-trivial decision (duplicates, missing data above a threshold, outliers, category inconsistencies, and every dtype fix) is confirmed with the user one at a time before it's applied — never batch-applied silently.

## Core rules (never break these)

1. **Original file is read-only, always.** Never open it in write mode, never overwrite it. All fixes land in a copy.
2. **One decision at a time, in chat.** Don't dump a list of every issue and ask the user to respond to all of them at once. Surface issue 1, wait for the user's reply, apply it, move to issue 2.
3. **>20% missing (by column, or by row) = must ask.** Below 20% = note it in a markdown cell in the notebook, but don't interrupt the user for it.
4. **Pass order is fixed: duplicates → missing values → outliers/impossible values → category consistency → dtypes.** Each pass fully finishes before the next one starts.
5. **For dtype fixes, always show your inferred type + your reasoning before asking the user to confirm** — never just ask "what type should this be?" cold.
6. **Reruns always version.** If `<original_name>_clean.<ext>` already exists, don't ask — create `<original_name>_clean_v2.<ext>` (v3, v4, ...) and a matching `_v2` notebook, so past cleaning runs are never silently overwritten.

## Workflow

### Step 1 — Set up the copy

- Determine the working copy path: `<original_name>_clean.<ext>` sitting next to the original (or in the output dir per the relevant file-format skill, e.g. `/mnt/skills/public/xlsx/SKILL.md` if it's an Excel file). If that path already exists, increment to `_clean_v2`, `_clean_v3`, etc. — never overwrite a prior cleaning run.
- Copy the original to this path untouched. All analysis in Step 2 reads the **original** (for the ground-truth picture); all fixes from Step 3 onward are applied to the **copy**.

### Step 2 — Build the initial notebook

Create `<original_name>_cleaning.ipynb` (versioned the same way as the copy if it already exists — `_cleaning_v2.ipynb`, etc.). Write valid `.ipynb` JSON directly, or use `nbformat` if installed. Structure it as:

1. **Title + setup cell** — load the original (read-only) into a dataframe, load/point to the copy.
2. **Shape & dtype overview** — `.shape`, `.dtypes`, `.info()`.
3. **Duplicates inventory** — exact duplicate rows, plus a near-duplicate check (e.g. same values except minor casing/whitespace differences, or a high similarity score across key columns). List them grouped, worst/most-repeated first.
4. **Missing values inventory** — a table of every column's missing count + percentage, and a check for rows with high missingness (>20% of that row's fields empty). Sort worst-first.
5. **Per-flagged-item context** — for every column/row that crosses the 20% missing threshold, add a small sub-analysis: does missingness correlate with another column (e.g. all missing `age` rows share a `signup_source`)? Is there a natural group-by (category, cohort, date bucket) that would give a smarter fill value than a blind global mean/median? This is what you'll show the user when you ask them about that item.
6. **Outlier / impossible-value scan** — per numeric or date column, flag statistically extreme values (e.g. far outside IQR) *and* domain-impossible values (negative ages/counts, dates in the future or before a sane minimum, percentages outside 0–100, etc). Note which rule caught each one.
7. **Category consistency scan** — for text/categorical columns, group values that are likely the same thing written differently (case differences, whitespace, common abbreviations, small edit-distance clusters like "NY" / "New York" / "ny"). List the clusters found.
8. **Dtype audit table** — current dtype per column vs. a quick inferred "should probably be" type (e.g. a numeric column stored as string because of stray currency symbols; a date column stored as string; a category stored as free text). Don't apply anything yet — just lay it out.
9. Save the notebook. Any missing-value column under the 20% threshold gets a one-line markdown note here (e.g. "`email` — 4% missing, left as-is per default threshold") and is **not** queued for Step 3.

### Step 3 — Walk through duplicate decisions (one at a time)

For each duplicate/near-duplicate group found, worst first:

- Present it in chat: which rows, why they were flagged (exact match vs. which columns triggered the near-duplicate match).
- Offer realistic options — keep first/last occurrence, merge fields, keep both (false positive), or a custom rule.
- Wait for the user's choice, apply it to the **copy** as a new commented code cell, move to the next group.

### Step 4 — Walk through missing-value decisions (one at a time)

For each flagged column/row (>20% missing), in order of severity (worst first):

- Present it in chat: what column/row, % missing, and the relational context you found in Step 2.
- Offer the realistic options for that specific case (don't recite a generic menu) — e.g. drop the column, drop the affected rows, impute with mean/median/mode, impute using the group-by pattern you found, fill with a constant, or leave as-is and just flag it in analysis.
- Wait for the user's choice.
- Apply it to the **copy** only, as a new, clearly-commented code cell appended to the notebook (so the notebook ends up as a full audit trail of what happened and why).
- Move to the next flagged item. Don't proceed until all are resolved.

### Step 5 — Walk through outlier / impossible-value decisions (one at a time)

Only after Step 4 is fully done. For each flagged value or cluster of values:

- Present it in chat: the value(s), the column, and why it was flagged (statistical outlier vs. domain-impossible).
- Offer realistic options — cap/winsorize, treat as missing and route through a fill value, drop the row, correct to a plausible value if obvious (e.g. a clear typo like age `250` meant to be `25`), or leave as-is.
- Wait for the user's choice, apply it to the **copy** as a new commented code cell, move to the next item.

### Step 6 — Walk through category consistency decisions (one at a time)

Only after Step 5 is fully done. For each cluster of likely-duplicate category values:

- Present it in chat: the variants found and their counts (e.g. "NY": 40, "New York": 12, "ny": 3).
- Ask which canonical value to standardize on (suggest the most frequent one as a default option, but let the user pick or type a different one).
- Apply the mapping to the **copy** as a new commented code cell, move to the next cluster.

### Step 7 — Walk through dtype decisions (one at a time)

Only after Step 6 is fully done. For each column where current dtype ≠ inferred "should be" type:

- Show the current dtype, your inferred correct type, and *why* (what pattern in the data led you there — e.g. "95% of values parse as ISO dates, the rest are blank/'N/A'").
- Ask the user to confirm, pick a different type, or skip this column.
- Apply the confirmed conversion to the **copy** as a new notebook cell (handle whatever the messy edge-cases were — e.g. strip currency symbols before casting to float).
- Move to the next column.

### Step 8 — Wrap-up

- Add a final **decision-log table cell** (markdown or a small dataframe printed in a code cell) summarizing every change made across all passes: `column/rows | issue type | decision | rationale`. This is the at-a-glance audit of the whole run.
- Add a markdown note explaining that each decision lives in its own notebook cell in order, so any single decision can be reverted by re-running the notebook up to (but not including) that cell and continuing from there — no need to restart the whole cleaning process to undo one choice.
- Confirm to the user where the cleaned file and the notebook ended up, and that the original was never modified.

## Notes

- If the dataset is large, use pandas/`nbformat` in the sandbox rather than trying to eyeball values manually.
- If the user asks to change the 20% threshold or the pass order for a given run, follow their override for that run without objection.
- Never silently apply a fix "because it's obviously right" — even confident inferences go through the one-at-a-time confirmation in Steps 3–7.


---
name: deeper-analysis
description: Guided, insight-by-insight analysis workflow that runs after a dataset has already been cleaned (see the dataset-cleanup skill). Use this whenever the user wants to investigate a question, concern, or problem using a cleaned dataset — e.g. "why did X drop", "is there a relationship between X and Y", "investigate this", "what's driving this trend". Each insight/question gets its own dedicated notebook, analyzed one at a time with the user's scope confirmed up front. Always use this instead of running ad-hoc analysis directly in chat when the user is working from an already-cleaned dataset and framing things as a question to answer or a problem to investigate.
---

# Deeper Analysis

A guided workflow for answering specific questions or investigating specific concerns against an already-cleaned dataset. One insight in, one dedicated notebook out, one at a time — never a batch of unrelated analyses dumped at once.
All analysis should be numbered by the order given by the user
ex: 01_price_analysis.ipynb .... in the las 99_

**Prerequisite:** this assumes a cleaned dataset already exists (produced by the `dataset-cleanup` skill: a `<name>_clean.<ext>` file, possibly with multiple `_v2`, `_v3`... versions). If no cleaned dataset exists yet, point the user to that skill first.

## Core rules (never break these)

1. **Always confirm which cleaned data version to use.** If multiple `_clean_vN` files exist, ask the user which one — never assume "latest" silently.
2. **One insight = one notebook, strictly.** Don't merge multiple questions into a single notebook, even if they seem related.
3. **Clarify scope before starting.** Before writing any analysis code, confirm with the user: timeframe, filters, what exactly the question means, and any ambiguous terms (e.g. what counts as "churned").
4. **Check in with the user after each notebook, before starting the next insight.** Insights arrive one at a time anyway — don't queue up work ahead of what's been asked.
5. **Let the question set the statistical depth.** Simple "what's the trend" questions get descriptive/visual treatment; questions implying a relationship or comparison get correlation/hypothesis tests/regression as appropriate. Don't force complexity where it isn't needed, and don't stay shallow when the question calls for more.
6. **Always call out assumptions explicitly**, and **always note confidence/caveats** when a sample is small or the data is noisy — even if it doesn't change the bottom-line conclusion.
7. **If the data can't fully answer the question, don't just stop** — do the best analysis the data supports, and clearly note the limitation in the conclusion (what's missing, and what it means for how much to trust the answer).
8. **Every notebook ends with a plain-language conclusion** answering the original question — not just charts and tables.
9. **The notebook is the primary deliverable.** In chat, just confirm completion and give the headline conclusion — don't re-explain the whole analysis in chat.

## Workflow

### Step 1 — Confirm the data source

- Look for `<name>_clean.<ext>` and any versioned siblings (`_clean_v2`, `_v3`...) from the dataset-cleanup skill.
- Ask the user which version to analyze against. Don't default to latest without asking.

### Step 2 — Take in one insight/question

- Wait for the user to give you a question or concern to investigate. Don't ask them to front-load a whole list — they'll feed these one at a time.

### Step 3 — Clarify scope

Before writing any code, confirm with the user (briefly, don't over-ask):
- What timeframe or subset of the data applies, if relevant.
- What any ambiguous terms in the question mean concretely (e.g. "churned" = no activity in 90 days? cancelled subscription?).
- Any filters that should apply (segment, region, category, etc).

### Step 4 — Build the notebook

Create `insight_<short-slug>.ipynb` (e.g. `insight_churn_by_region.ipynb`). Structure:

1. **Recap cell** (markdown) — which cleaned data version is used, which columns are relevant, what scope/filters were confirmed in Step 3, and the exact question being answered.
2. **Assumptions cell** (markdown) — every assumption being made going in (e.g. "assuming missing values in `region` were already dropped during cleaning", "assuming a 90-day window for churn").
3. **Derived metrics, if needed** — if the question implies a metric that doesn't exist in the data yet, build it here in its own clearly-labeled cell/section, explicitly flagged as **derived, not raw data**, with the logic shown.
4. **Analysis** — descriptive stats, visualizations (matplotlib/seaborn), and statistical tests/correlation/regression where the question warrants it. Choose the depth the question actually needs.
5. **Confidence & caveats cell** — call out small sample sizes, noisy data, or anything that limits how much weight the conclusion can carry.
6. **Conclusion cell** (markdown) — a plain-language answer to the original question. If the data only partially answers it, say so here and explain what's missing.
7. **Flagged follow-ups cell** (markdown, only if applicable) — if the analysis surfaces a new question or anomaly outside the original scope, note it here.

### Step 5 — Report back and flag follow-ups

- In chat: confirm the notebook is done, give the headline conclusion in a sentence or two, and point to the notebook for the full analysis.
- If something unexpected/out-of-scope surfaced during the analysis, flag it explicitly and offer to spin up a new notebook for it as a separate insight — don't fold it into the current one.
- Update `analysis_summary.ipynb` (see Step 7) with this insight's headline finding.

### Step 6 — Check in before the next insight

- Wait for the user to confirm they're ready to move on (or to revisit this one) before starting the next insight.

### Revisiting an insight later

- Don't version the notebook. Reopen the **same** `insight_<slug>.ipynb` and append a new section marked `## Revision — <date>` with the additional analysis. The notebook accumulates its own history in place.

### Step 7 — Running summary

- Maintain `analysis_summary.ipynb`: a running, lightweight notebook that collects each insight's question + headline conclusion + link/reference to its dedicated notebook. Update it after every completed insight, not just at the end.

### Step 8 — Final dashboard (only when the user says they're happy with the results)

- Do **not** build this proactively — only once the user confirms they're satisfied with the set of insights so far.
- Create `dashboard.html`: a single self-contained interactive HTML file (plotly-style — hover, filter/toggle where it makes sense) pulling together the key visuals and headline conclusions from every insight notebook produced so far.
- This is a presentation layer on top of work already done in the notebooks — don't redo the underlying analysis, just re-render the key results interactively.
- This dashboard should be saved on a file so it can be open everytime

## Notes

- If the user gives a vague or very broad "investigate X" without a clear question, use Step 3 to narrow it into something answerable rather than guessing.
- If the user asks to change any default here (skip scope clarification for a quick one, merge two insights, etc.) for a given run, follow their override without objection.
- Keep notebooks self-contained and reproducible from the recap cell — someone else should be able to open one and understand exactly what was analyzed and why.

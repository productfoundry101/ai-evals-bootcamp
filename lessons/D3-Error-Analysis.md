# D3 - Error Analysis

**Week 1, Day 3** | ~30 min
**Part times:** Concepts ~10 min | Exercise ~15 min | Decision Point ~5 min
**Previous lesson:** D2 - Failure Surface Mapping — you built an evaluation surface map: functional failures by pipeline stage, adversarial risks, and coverage gaps. The key finding: failure type labels tell you *what* went wrong but not always *where* or *why*. Today you go from counting failures to reading them — learning the systematic process of understanding exactly how and why your AI system breaks.

---

## Part 1: The Concepts

### Look at the data before designing metrics

Most teams get this backwards. They start by choosing metrics (accuracy, F1, ROUGE), building dashboards, or buying observability tools. Then they wonder why the numbers don't help them improve the product.

The reason: you can't design a useful metric until you understand the failure patterns the metric needs to capture. And you can't understand failure patterns without reading actual system outputs.

This is the most counterintuitive lesson in AI product management: **before building any evaluation infrastructure, sit down and read 50–100 traces.** Not summaries. Not aggregates. The actual inputs, outputs, and reasoning your system produced. Practitioners who build reliable AI products consistently describe this as the step that matters most — and the one teams most often skip.

There's no shortcut. Dashboards can't tell you *why* the system is fabricating policy justifications. An accuracy metric can't reveal that the system handles price changes well but falls apart on allergen updates. Only reading the data reveals these patterns. The metrics come after, designed to measure what you found.

### The discovery method: open coding, axial coding, saturation

Error analysis borrows a methodology from qualitative research. It has three phases:

**Open coding** — Read each trace individually. For every failure, write a plain-language note describing what went wrong. Don't categorise yet. Don't use predefined labels. Just describe what you see: "LLM approved a 45% price increase, citing 'consistent with historical trends' — but no historical data was provided." "LLM rejected a valid description change because it misread 'spicy' as an allergen claim."

The discipline is in *not* naming the pattern too early. If you label the first example "hallucination" after five traces, you'll force-fit the next twenty into that bucket. Failures that don't match your premature label get missed entirely.

**Axial coding** — After reviewing 50–100 traces, you'll have dozens of freeform notes. Now cluster them. Group notes that describe the same underlying problem. Name each cluster. "Hallucinated justification" might emerge. So might "correct decision but fabricated reasoning," which is subtly different — the answer was right but the explanation was invented.

Your categories will surprise you. Teams typically predict 3–4 failure types. The data usually reveals 6–8. The ones you didn't predict are often the most important.

**Saturation** — Keep reading traces until new ones stop producing new categories. If the last 10 traces all fall into existing clusters, you've likely reached saturation. If even one creates a new category, keep reading. Research shows some categories only emerge after 20+ traces. Ten is not enough. Twenty is the minimum. Thirty is better.

### Triage: turning a taxonomy into an action plan

A failure taxonomy is useful. A triaged failure taxonomy is actionable. Every category gets one of three labels:

**Prompt-fix** — the team can iterate on the prompt this sprint. Example: the system consistently understates severity ("slight decline" for a 40% drop). Rewrite the prompt instructions around severity language.

**Evaluator-needed** — a metric needs to be built to detect this failure automatically at scale. Example: the system sometimes fabricates a justification that sounds plausible but isn't grounded in the data. You can spot this reading traces, but you need an automated eval to catch it across thousands of decisions.

**System-fix** — the failure requires a code or architecture change. Example: the system can't access historical pricing data, so it invents a reference point. No prompt change fixes missing data.

The triage turns your taxonomy into a sprint plan. Prompt-fixes go into the current sprint. Evaluator-needed items become the metric design backlog. System-fixes go to engineering. Every category has a home.

---

## Part 2: Exercise — Read the Traces

You've counted the failures in D2 - Failure Surface Mapping. Now you're going to read them. Your goal: build a failure taxonomy from the actual data, not from the pre-existing `failure_type` labels.

### Setup

Open `exercises/D3-menu-verification-dataset.csv`. Filter to rows where `human_agrees_with_llm` = `FALSE`.

For each failure row, the relevant "trace" is: `change_type`, the old/new value pair, `last_verified_price`, `price_markup_pct`, `context_provided` (the policy and context the LLM had access to), `llm_decision`, `llm_confidence`, `llm_reasoning`, and `human_decision`. For image changes, also check `image_analysis`.

The key analytical move: compare what the LLM *said* in its reasoning against what the policy *actually says* in `context_provided`. The gap between these two tells you what went wrong.

**Important:** Ignore the `failure_type` column for now. You'll compare your categories against it at the end.

### Step 1: Open coding

Read 10 failure rows. For each, compare the LLM's reasoning (`llm_reasoning`) against the actual data (the old/new values, markup percentage, available context). Write a freeform note describing what specifically went wrong.

Don't use labels like "hallucination" yet — describe the failure in your own words.

### Step 2: Axial coding and triage

Cluster your 10 notes into named categories. For each category, assign a triage label:

| Category name | Example (row ID) | Triage |
|---|---|---|
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |

How many distinct categories did you find across 10 traces? Now reveal the `failure_type` column — how do your categories compare to the pre-existing labels?

---

## Part 3: PM Decision Point

Your ML lead asks: *"We have 30 failures in the dataset. What should we fix first?"*

Write a short recommendation: name the highest-priority failure pattern, why it's the most urgent, and what type of fix it needs (prompt-fix, evaluator-needed, or system-fix).

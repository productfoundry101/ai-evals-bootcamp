# D3 - Error Analysis

**Week 1, Day 3** | ~35 min
**Part times:** Concepts ~10 min | Exercise ~20 min | Decision Point ~5 min
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

## Part 2: Exercise — Read the Traces Until Saturation

You've counted the failures in D2. Now you're going to read them — in batches, watching for the moment new traces stop producing new categories. That moment is saturation, and it's the one signal that tells you your taxonomy is stable.

In practice you'd read 30+ failures. This exercise compresses the experience into 12 across three batches so you can feel the pattern: categories accumulate fast, then slow, then stop.

### Setup

Open `exercises/D3-menu-verification-dataset.csv`. Filter to rows where `human_agrees_with_llm` = `FALSE`.

For each failure row, the relevant "trace" is: `change_type`, the old/new value pair, `last_verified_price`, `price_markup_pct`, `context_provided` (the policy and context the LLM had access to), `llm_decision`, `llm_confidence`, `llm_reasoning`, and `human_decision`. For image changes, also check `image_analysis`.

The key analytical move: compare what the LLM *said* in its reasoning against what the policy *actually says* in `context_provided`. The gap between these two tells you what went wrong.

**Important:** Ignore the `failure_type` column for now. You'll compare your categories against it at the end.

### Step 1: Batch 1 — open code 4 traces (~5 min)

Read the first four failures in `change_id` order: **CHG-010, CHG-012, CHG-013, CHG-019**.

For each, write a one-sentence freeform note describing what specifically went wrong. Don't use labels like "hallucination" yet — describe the failure in your own words.

### Step 2: First taxonomy — cluster (~3 min)

Cluster your 4 notes into categories. How many distinct categories do you see so far? Name each one. This is your starting taxonomy — expect it to grow.

### Step 3: Batch 2 — track new vs existing (~5 min)

Read the next four failures: **CHG-023, CHG-024, CHG-032, CHG-036**.

For each, decide: does this fit an existing category, or is it a new one? If new, name it and add to your list.

After this batch, how many total categories do you have? Teams typically predict 3–4 failure types before reading data. How does your count compare?

### Step 4: Batch 3 — saturation check (~4 min)

Read the final four failures: **CHG-039, CHG-046, CHG-048, CHG-049**.

Same check: does each one fit an existing category, or spawn a new one?

**The saturation question:**
- If all 4 fit existing categories → you've likely saturated at ~12 traces.
- If even one creates a new category → you haven't saturated. In a real project this is the signal to keep reading; 12 wasn't enough.

Either outcome is instructive. Saturation isn't a target you force — it's an observation you make.

### Step 5: Triage your final categories (~3 min)

For each category in your final taxonomy, assign a triage label:

| Category name | Example (change_id) | Triage |
|---|---|---|
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |
| ___ | ___ | prompt-fix / evaluator-needed / system-fix |

Now reveal the `failure_type` column. How do your discovered categories compare to the pre-existing labels? Did you find anything the labels missed, or miss anything they caught?

---

## Part 3: PM Decision Point

Your ML lead asks: *"We have 30 failures in the dataset. What should we fix first?"*

Write a short recommendation: name the highest-priority failure pattern, why it's the most urgent, and what type of fix it needs (prompt-fix, evaluator-needed, or system-fix).

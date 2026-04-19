# D11 - Metric Design

**Week 2, Day 11** | ~45 min
**Part times:** Concepts ~15 min | Exercise ~18 min | Decision Point ~12 min
**Previous lesson:** D10 - Release Criteria — you learned that not all metrics are equal. Some block release (guardrails), others track quality over time (optimization metrics). You applied a ship/hold framework to a release candidate and saw that a single guardrail failure blocks shipping regardless of how many other metrics improved. Today you go deeper: instead of classifying metrics others have designed, you'll learn how to design them yourself — and how to build an evaluation pipeline that gives you the coverage you need without exceeding your budget.

---

## Part 1: The Concepts

### What makes a metric useful

Not everything measurable is worth measuring. A useful metric has three properties.

**Specific** — it measures one thing, clearly defined. "Response quality" is not a metric; it's a description. "Hallucination rate in policy citations" is a metric: you know what's being measured, on what input type, and who in the system is responsible. When this metric regresses, you know where to look.

**Actionable** — when the metric changes, you know what to do. A metric is actionable if a regression points to a specific team, a specific component, and a specific fix type. If your metric regresses and the answer to "what do we fix?" is "everything," the metric is too coarse.

**Connected to outcomes** — the metric tracks a failure mode that users or the business actually care about. Metrics that consistently improve while user outcomes stay flat are measuring the wrong thing.

Signs a metric is poorly designed:
- It aggregates multiple dimensions into a single number (easy to game, hard to act on)
- A regression produces no clear owner or fix type
- You could hit the target by making the system more restrictive — i.e., it doesn't distinguish caution from quality

### The cost of evaluation

Every eval run has a cost. Running a model-based grader on one case typically costs $0.01–$0.05 in API calls. Human reviewers cost $2–$20 per case depending on task complexity and reviewer expertise. At 10,000 cases per week:

- A model-based grader at $0.02/run × 10,000 = **$200/week**
- A human reviewer at $2.00/run × 10,000 = **$20,000/week**

The gap is 100×. This is why comprehensive human review is only viable for very low-volume systems.

The PM question: what's the minimum eval investment that gives you enough signal to make good decisions? Not zero — that leaves you blind. Not unlimited — that's not viable. The answer depends on volume, failure severity, and what it costs the business when a failure type goes undetected.

### Sampling strategies

When you can't evaluate every case, how you sample determines what you catch.

**Random sampling** — pick N% of cases uniformly. Simple and unbiased, but inefficient for rare failure modes. If a critical failure type occurs in 2% of cases and you sample 10% of all cases, only 0.2% of your eval set contains that failure type. You may see very few examples of it per week.

**Stratified sampling** — oversample high-risk or high-value segments. Same overall sampling budget, but allocate a higher rate to segments where failures are severe or frequent. You spend the same and get more signal where it matters.

**Adversarial sampling** — deliberately sample hard cases: near-threshold inputs, previous failure patterns, known edge cases. You're not trying to be representative; you're stress-testing. An efficient way to find failure modes before users do.

**Production-triggered sampling** — run an expensive check only when a signal fires: model confidence below a threshold, output unusually short, retrieval quality below a floor. If 15% of cases trigger the signal, you run the expensive check on 1,500 cases instead of 10,000 — significant cost savings with targeted coverage on the cases most likely to have failed.

### Tiered evaluation

Different check types have different costs and should run at different rates. The tiered model:

- **Code checks** — $0, run on 100% of traffic. Deterministic, fast, and costless. Catches anything rule-based: field completeness, threshold violations, schema errors. Run these first.
- **Model-based checks** — $0.01–$0.05 per run, run on a sampled slice. Good for nuanced, judgment-based failures like hallucination or policy compliance. Run these on what code checks don't catch.
- **Human review** — $2–$20 per case, run on a small adversarial or stratified subset. Ground truth, calibration, and catching what automated checks miss. Never cut below a meaningful floor.

The principle: push checks down the cost pyramid. Use code for deterministic failures, model for nuanced ones, human for calibration and hard cases.

Example: at 10,000 cases per week, a tiered pipeline might cost:
- Code checks (100%): $0
- Model checks (40% of traffic at $0.02/run): $0.02 × 4,000 = **$80/week**
- Human review (5% of traffic at $2.00/run): $2.00 × 500 = **$1,000/week**
- **Total: $1,080/week**

vs. running human review on everything: $2.00 × 10,000 = **$20,000/week**

Tiering cuts the bill by ~95% while retaining coverage across most of your failure surface.

### Designing against metric gaming

When a metric becomes a target, teams optimize for it — sometimes by degrading what it was meant to protect. This is Goodhart's Law applied to AI evaluation: a measure that becomes a target ceases to be a good measure.

Example: suppose you set hallucination rate ≤ 10% as a release guardrail. One way to hit that number is to truncate system responses. Shorter outputs make fewer claims, so there is less to hallucinate. Hallucination rate drops — but so does answer completeness. The metric improved; the system got worse for users.

How to design against this: **pair metrics that cover opposing failure modes**. If you guardrail hallucination rate (penalizes over-claiming), also track answer completeness (penalizes under-claiming). A team cannot game both simultaneously without a genuine improvement.

For any guardrail metric, ask: "what could a team do to hit this number without actually improving the system?" If there is a straightforward answer, add a counter-metric that forecloses that path.

---

## Part 2: Exercise — Evaluating Eval Pipeline Designs

### Scenario

The menu verification system processes 10,000 verifications per week. Based on prior error analysis, the team has estimated the weekly volume of each failure type:

| failure_type | estimated_weekly_cases | severity |
|---|---|---|
| price_error | 300 | medium |
| allergen_error | 200 | critical |
| hallucination | 600 | high |
| policy_misapplication | 800 | medium |
| context_miss | 400 | high |

Three engineering teams have each proposed an eval pipeline. Each pipeline uses a different combination of check types, sampling rates, and costs. Your head of engineering wants a recommendation: which pipeline should the team adopt, and why?

### Dataset

Open `exercises/D11-metric-design-dataset.csv`. Each row is one check in one of three proposed pipelines.

| Column | What it means |
|--------|---------------|
| `check_id` | Identifier for the check (PC-01 through PC-12) |
| `pipeline_id` | Which pipeline this check belongs to (P-A, P-B, or P-C) |
| `check_name` | Descriptive name of the check |
| `check_type` | How the check runs: `code` (deterministic, always $0.00), `model` (LLM-based, variable cost), or `human` (manual review) |
| `failure_type_targeted` | Which failure type this check is designed to detect. Checks with value `all` can detect any failure type at their sampling rate. |
| `sampling_rate_pct` | Percentage of the 10,000 weekly verifications this check runs on |
| `cost_per_run_usd` | Cost each time this check runs on one case |

### Expected outcome

By the end of this exercise, you'll have calculated weekly costs per pipeline and identified which failure types each pipeline covers and misses. You'll use this analysis to make a pipeline recommendation in the Decision Point.

### Step 1: Calculate weekly costs

For each pipeline, calculate total weekly cost. Formula per check: `(sampling_rate_pct / 100) × 10,000 × cost_per_run_usd`. Sum across all checks in the pipeline.

### Step 2: Map coverage by failure type

For each pipeline, identify which failure types are covered and at what effective sampling rate. A check can only detect the failure type it targets — a code check for price thresholds cannot detect hallucination regardless of its sampling rate.

### Step 3: Identify coverage gaps

Using the failure type volumes above, calculate how many cases per week go undetected for each uncovered (or under-covered) failure type in each pipeline.

---

## Part 3: PM Decision Point

Based on your cost and coverage analysis, you're bringing a pipeline recommendation to the head of engineering. Before you present, Finance sends over a new constraint:

*"Our eval budget has grown significantly and we need to reduce it. Whichever pipeline you recommend, cut evaluation spend by 40% from what you're proposing. What does the pipeline look like at 40% lower cost, and what coverage are you giving up?"*

Name your recommended pipeline, then redesign it to hit the 40% cost reduction target. Specify which checks you'd reduce or eliminate, recalculate the new weekly cost, and explicitly name what coverage you're trading away.

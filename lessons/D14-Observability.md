# D14 - Observability

**Week 2, Day 14** | ~45 min
**Part times:** Concepts ~15 min | Exercise ~22 min | Decision Point ~12 min
**Previous lesson:** D13 - Eval-Driven Development — you learned to write evals before code (evals-as-spec), keep tight feedback loops, grow regression suites with every bug fix, and use eval gaps as a prioritization signal. You audited a 25-case eval suite across six sprints and identified a critical allergen regression as the sprint 7 must-fix. Today we close Week 2 by crossing the production line: once the system is live, evals alone aren't enough. You need observability — the live signals that tell you what's actually happening right now.

---

## Part 1: The Concepts

### Observability is what evaluation isn't

Evaluation measures your system *before* it meets the real world: curated inputs, known-correct answers, controlled conditions. It answers "will this work?" Observability measures your system *while* it meets the real world: live traffic, unknown ground truth, messy conditions. It answers "is this working right now?"

Both are necessary, and neither replaces the other. A team with great evals and no observability ships confidently and discovers problems from Twitter. A team with great observability and no evals reacts quickly but never gets ahead of incidents. The PM skill is knowing which question you're asking and picking the right instrument.

The practical line: evals are pre-deployment and gated (they decide whether you ship). Observability is post-deployment and continuous (it decides whether you keep running).

### Three pillars, plus one

Classical software observability has three pillars: metrics, logs, and traces. AI systems add a fourth that's often the most important.

**Metrics** — aggregate numeric measurements over time. Error rate, p95 latency, requests per hour, approval rate by subgroup. Cheap to store, easy to chart, good for SLOs and alerts.

**Logs** — per-request records. Input, output, model version, timestamps, any intermediate state. Expensive to store at scale; indispensable for debugging a specific incident.

**Traces** — the log of a request as it flows through pipeline stages (input parsing → retrieval → LLM → output). Shows where latency accumulates and where failures originate. Higher cost than metrics, narrower than logs.

**Human feedback** — thumbs up/down, complaint tickets, escalation flags, reviewer annotations. This is the ground-truth signal from reality. Slow (users complain hours or days later) and biased (most dissatisfied users stay silent), but irreplaceable. It's the only signal that directly represents the outcome you care about.

A mature observability setup uses all four: metrics for "is it healthy," logs and traces for "why did this specific thing fail," feedback for "are users actually getting value."

### Leading vs lagging indicators

A lagging indicator tells you harm has already happened. Complaint tickets, churn, thumbs-down rate — by the time these move, users have had a bad experience. A leading indicator tells you harm is coming before it lands. Confidence score drop, escalation rate increase, input distribution drift — something changed upstream; the bad outcome hasn't fully materialized yet.

Leading indicators are where you prefer to detect problems because the remediation window is wider. But they're typically noisier and require more judgment to act on (a confidence drop might be nothing, or might be a silent model regression).

The PM rule: **track both, but weight your alerting toward leading indicators.** Don't wait for churn to tell you something is wrong.

### Alerting is a discipline, not a default

Every metric can technically fire an alert. Almost none should. Teams that alert on everything produce alert fatigue: the on-call ignores pages, real incidents get missed, trust in the system decays.

The test before adding an alert:
- **Is this an SLO violation?** Latency breach, error rate spike, safety threshold crossed — page someone.
- **Is this actionable right now?** If the answer to "what do I do?" is unclear, don't alert — log it.
- **Is the false positive rate low enough?** An alert that fires 10 times a day with 9 false positives teaches the team to ignore it.

Everything else is dashboard-only. Dashboards are for investigation and trend-watching; alerts are for "stop what you're doing and respond." Conflating them burns out your on-call.

### Observability closes the loop to evaluation

The final discipline: when observability catches a new failure mode in production, that failure becomes an eval case. The user who complained about Thai restaurant rejections (D12 - Fairness & Subgroups) isn't just a support ticket — they're a specification of a failure mode your eval suite didn't cover. Once you translate their complaint into an eval case, you've prevented it from ever recurring silently.

This is how the eval suite grows. Observability is the input; evaluation is where you lock the lesson in. Teams that don't close this loop learn the same lessons over and over. Teams that do close it watch their regression suite become an increasingly accurate map of every way their system has ever failed.

---

## Part 2: Exercise — Designing an MVP Observability Dashboard

### Scenario

Your menu verification system is about to go GA. Engineering is building the first version of the observability dashboard. They've brainstormed 15 candidate signals, but you only have engineering capacity for roughly half of them in the MVP. Your head of engineering wants a recommendation: which signals make the cut, which fire alerts, and what's missing entirely that should be added in v2.

### Dataset

Open `exercises/D14-observability-signals-dataset.csv`. Each row is one proposed observability signal.

| Column | What it means |
|--------|---------------|
| `signal_id` | Identifier (S-01 through S-15) |
| `signal_name` | Descriptive name |
| `signal_type` | Which pillar it belongs to: metric, log, trace, human_feedback |
| `what_it_measures` | One-line description of what the signal captures |
| `indicator_type` | leading (warns before harm), lagging (confirms harm happened), neutral (context/volume) |
| `detection_latency` | How long between the underlying issue occurring and this signal moving: seconds, minutes, hours, or days |
| `implementation_cost` | Engineering cost to build and maintain: low, medium, high |

### Expected outcome

By the end of this exercise, you'll have: (1) classified the 15 signals by pillar and indicator type to see whether the dashboard is balanced across observability dimensions, (2) decided which deserve alerts vs dashboard-only, (3) identified coverage gaps where the dashboard is blind to known failure modes, and (4) written an eval case that would close the loop from production observability back to the regression suite.

### Step 1: Classify by pillar and by indicator type

For the 15 signals, classify across two axes:

- **Pillar** — from the `signal_type` column: metric, log, trace, or human_feedback.
- **Indicator type** — from the `indicator_type` column: leading, lagging, or neutral.

Count signals in each pillar and in each indicator category. Is the dashboard balanced across pillars, or is it heavily tilted toward one (usually metrics)? Is it mostly leading or lagging? An imbalanced dashboard tells you the team has visibility in some dimensions but blind spots in others.

### Step 2: Classify alert-worthy vs dashboard-only

Of the 15 signals, which should fire alerts and which should just go on the dashboard? Use the three-question test: SLO violation? Actionable right now? Low false-positive rate? Signals that fail the test belong on the dashboard only.

### Step 3: Identify coverage gaps

What failure modes or subgroup signals are *missing* from the 15? Think back to earlier lessons (D9 - Hallucination Detection hallucinations, D12 - Fairness & Subgroups subgroup fairness including restaurant size and intersectional slicing, D13 - Eval-Driven Development regression patterns). Which failure modes would this dashboard fail to catch?

### Step 4: Close the loop back to evaluation

Observability catches new failure modes in production; evaluation is how you prevent them from silently recurring. When a signal fires, the team translates the incident into a permanent eval case — this is how the regression suite grows (from D13 - Eval-Driven Development).

Consider **S-08** (`allergen_false_approval_count`). Imagine the first time it fires: a menu change was approved last week, a customer reported an allergic reaction, and auditors confirmed the item had an undisclosed allergen. What eval case gets added to the regression suite so this scenario is caught pre-deployment next time?

Write a brief eval spec using the D13 - Eval-Driven Development format: input, expected output, pass criteria, failure mode area, severity. This is observability-to-evaluation loop closure in practice.

---

## Part 3: PM Decision Point

Based on your analysis, you're bringing the MVP dashboard proposal to your head of engineering. Before you finalize, they add the constraint:

*"We can ship six signals in v1 — no more. I need to know which six you'd pick, which you're deferring, and what you'd add to v2 based on the gaps you found. Justify each choice — I don't want a 'cover everything a little' dashboard."*

Name your six MVP signals. Explicitly call out what you deferred and why. Flag at least one gap that isn't in the list of 15 and justify adding it to v2. Use indicator type, cost, and alert-worthiness to justify choices — not just gut feel.

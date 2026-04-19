# D21 - Eval Culture

**Week 3, Day 21** | ~45 min
**Part times:** Concepts ~15 min | Exercise ~20 min | Decision Point ~10 min
**Previous lesson:** D20 - Regulatory Context — you learned that the EU AI Act creates binding obligations for high-risk AI systems, that your eval practices from D1–D19 map to the six compliance requirements but need formalization, and that the gap between doing evals and proving you do evals is where most teams fail an audit. Today is the final lesson. You have the full toolkit — error analysis, graders, metrics, experiments, monitoring, red teaming, ship decisions, regulatory context. The question is no longer *what* to evaluate. It's *how to make evaluation stick* in an organization that will always have reasons to skip it.

This lesson is about the organizational side of evals: who owns what, when things run, how you pitch the investment, and how you communicate results to people who will never read a scoring rubric.

---

## Part 1: The Concepts

### Eval ownership: who owns what, and why "everyone" means "no one"

The most common organizational failure in AI evaluation is not a lack of tools or techniques — it's unclear ownership. When nobody owns the eval, it gets done inconsistently, deteriorates quietly, and gets skipped under deadline pressure.

Clear ownership means three things:

1. **PM owns eval strategy.** Which metrics matter (D10 - Release Criteria), what thresholds to set, what subgroups to monitor (D12 - Fairness & Subgroups), when to ship or hold (D19 - Ship Decisions), and what regulatory documentation to maintain (D20 - Regulatory Context). The PM decides *what* to evaluate and *what the bar is*.

2. **Engineering owns eval infrastructure.** CI/CD integration, automated grader pipelines (D5 - Grader Types), monitoring dashboards (D14 - Observability), logging and tracing (D14 - Observability), regression suites, and LLM-judge deployment (D6 - LLM-as-Judge). Eng decides *how* evals run and *how fast* the feedback loop is.

3. **Data science / ML owns eval calibration.** LLM-judge alignment with human labels (D6 - LLM-as-Judge), meta-evaluation, golden dataset curation (D7 - Golden Datasets), statistical analysis of experiments (D16 - AI Experiments), and fairness audits (D12 - Fairness & Subgroups). Data science decides *whether the evaluators themselves are trustworthy*.

The anti-pattern: "the whole team owns quality." In practice this means the PM doesn't set thresholds, eng doesn't automate the suite, and data science doesn't recalibrate the judges. Everyone assumes someone else is doing it. The result: evals exist on paper but don't run in practice.

Ownership does not mean doing the work alone. It means being the person who notices when the work isn't being done and escalates.

### Eval cadence: per-change, periodic, and event-driven

Not every eval runs at the same frequency. A common mistake is treating evals as a one-time pre-launch activity. In a healthy eval culture, three cadences coexist:

**Per-change (CI/CD):** runs automatically on every prompt change, model update, or code change that touches the AI pipeline. This is your regression suite (D13 - Eval-Driven Development) and your red-team regression probes (D18 - Red Teaming). If it doesn't run automatically, it won't run. Blocking CI on eval failure is the single most effective way to prevent regressions.

**Periodic (weekly/monthly):** runs on a schedule regardless of changes. This includes human review sampling (D17 - Launch Readiness Concept 4), LLM-judge recalibration against human labels (D6 - LLM-as-Judge), subgroup performance reviews (D12 - Fairness & Subgroups), golden dataset freshness checks (D7 - Golden Datasets), production drift monitoring reviews (D17 - Launch Readiness), and eval cost tracking (D11 - Metric Design). These catch slow degradation that per-change evals miss.

**Event-driven:** runs when a specific trigger fires. Drift alerts (D17 - Launch Readiness), customer complaints, regulatory changes (D20 - Regulatory Context), new cuisine or input-category onboarding, security incidents, or model-provider version updates. These catch unexpected changes that scheduled checks miss.

Map every eval activity in your portfolio to one of these three cadences. If an activity doesn't have a cadence, it's aspirational, not operational.

### The eval investment pitch: how to get buy-in from skeptics

Every PM who tries to build an eval culture will face resistance. The most common form: *"We need to ship features, not build test infrastructure. Evals are a nice-to-have."*

The pitch that works is not "evals improve quality" (too abstract) or "regulators require it" (too scary). The pitch that works is: **evals let you iterate faster.**

Without evals, every prompt change, model update, or pipeline modification is a gamble. You change something, deploy it, wait for user complaints, triage, and roll back. Each iteration cycle takes days to weeks.

With evals, you change something, run the suite in minutes, see exactly what improved and what regressed, and ship with confidence. Each iteration cycle takes hours. Teams with good eval infrastructure run dozens of experiments per month. Teams without it run one or two — and each one is nerve-wracking.

The concrete pitch: *"A 4-week investment in eval infrastructure pays back within 2 weeks through faster iteration. After that, every experiment we run is essentially free — we just change a config and check results. Without it, we're bottlenecked on manual QA for every change."*

Three things make the pitch credible:
1. **Show a recent incident that evals would have caught.** Every team has one. The allergen breach that made it to production. The prompt change that regressed a subgroup. The model update nobody tested. Quantify the cost: eng hours to debug, customer impact, leadership fire drill.
2. **Start small.** Don't pitch a 12-activity eval portfolio on day one. Pitch the three highest-leverage activities and show results in 30 days.
3. **Attach evals to shipping speed, not quality gates.** Leaders don't want gates — they want velocity. Evals are the thing that makes velocity safe.

### Communicating eval results to non-technical stakeholders

Your VP of Product will never read a confusion matrix. Your CEO will never look at Cohen's Kappa. But both need to understand whether the AI system is working — and whether it's getting better or worse.

The translation framework that works:

1. **What the system gets right.** Not "precision is 92%." Instead: "Of the 10,000 menus we verified this month, 9,200 were correctly approved or correctly rejected. 800 needed human intervention."

2. **Where it fails and what it costs.** Not "recall on allergen detection is 88%." Instead: "We missed allergen disclosures on 12 menus this month. Three of those reached consumers before we caught them. Estimated exposure: 45 orders served with undisclosed allergens."

3. **Whether it's getting better or worse.** Not "accuracy improved from 0.85 to 0.88." Instead: "Allergen misses dropped from 20/month to 12/month after the v2 prompt change. That's a 40% reduction in consumer exposure. We're targeting single digits by Q3."

4. **What you need to keep improving.** Not "we need to expand the golden dataset." Instead: "We're underperforming on Mexican and Korean cuisines because we don't have enough test cases. Adding 50 cases per cuisine would let us measure and improve quality for those segments. Cost: 2 weeks of annotation effort."

The pattern: **input → impact → trend → ask.** Numbers grounded in business terms, not statistical terms.

### The eval maturity ladder: where you are and where to aim

Most AI teams are at Level 0 or Level 1. Knowing where you are helps you set a realistic roadmap.

**Level 0 — Vibes.** No systematic evaluation. Quality is assessed by "looks good to me." Ship decisions based on demo impressions. This is where most teams start and where many stay too long.

**Level 1 — Manual spot checks.** Someone reviews a handful of outputs before each release. No structured dataset. No metrics. Better than vibes, but not repeatable — results depend on who reviewed and what they happened to look at.

**Level 2 — Golden dataset + automated checks.** A curated test set exists (D7 - Golden Datasets). Code-based and model-based graders run on it (D5 - Grader Types, D6 - LLM-as-Judge). Pass rates are tracked. Subgroups are sliced (D12 - Fairness & Subgroups). This is the minimum viable eval infrastructure. Most of what you learned in D1–D12 lives here.

**Level 3 — CI/CD evals + monitoring + drift detection.** Regression suites block bad changes (D13 - Eval-Driven Development). Production monitoring catches drift (D17 - Launch Readiness). Red-team probes run on every change (D18 - Red Teaming). Human review samples recalibrate judges (D6 - LLM-as-Judge). Experiments use proper statistical methods (D16 - AI Experiments). Ship decisions follow a framework (D19 - Ship Decisions). This is where good AI teams operate.

**Level 4 — Eval-driven development with regulatory compliance.** Evals are written before features (D13 - Eval-Driven Development). Conformity documentation is maintained (D20 - Regulatory Context). Incident reporting is automated (D20 - Regulatory Context). Eval results drive roadmap prioritization. The eval portfolio is a first-class product artifact, not a side process.

Most teams should aim for Level 3 within 6 months of their first AI launch. Level 4 is required for high-risk regulated products and is aspirational for others.

---

## Part 2: Exercise — Designing a 90-Day Eval Roadmap

### Scenario

You've just joined a food delivery company as the new AI PM. The company launched a menu verification system 6 months ago. The eng team built it and has been maintaining it, but there's no PM-owned eval process. You've assessed the current state and found they're at **Level 1** — manual spot checks before each release, no golden dataset, no automated graders, no monitoring beyond uptime.

Leadership has given you 90 days to "get evaluation sorted." You need to design a roadmap that takes the team from Level 1 to Level 3.

### Dataset

Open `exercises/D21-eval-culture-dataset.csv`. Each row is one eval activity from the course, mapped to the organizational context.

| Column | What it means |
|--------|---------------|
| `activity_id` | Identifier (A-01 through A-12) |
| `activity` | Short name of the eval activity |
| `description` | What this activity involves |
| `course_lesson` | Which lesson taught this concept |
| `current_status` | `not_done`, `ad_hoc`, or `partial` — current state at the company |
| `target_owner` | Who should own this activity (blank — you fill in) |
| `target_cadence` | How often it should run (blank — you fill in) |
| `maturity_level` | Which maturity level this activity belongs to: L2, L3, or L4 |
| `implementation_effort` | Rough effort to implement: days, weeks, or months |

### Expected outcome

By the end: (1) you'll have assessed the current state, (2) assigned ownership and cadence to all 12 activities, (3) selected the highest-leverage activities for the first 30 days, and (4) designed a phased 90-day roadmap.

### Step 1: Assess the current state

Count how many activities are `not_done`, `ad_hoc`, and `partial`. What percentage of the 12-activity eval portfolio is operational?

Look at which maturity levels are represented in the `not_done` pile. Are the gaps concentrated in L2 (basic), L3 (advanced), or L4 (regulatory)?

**Key question:** if the team is at Level 1 and most L2 activities are `not_done`, what does that tell you about the gap between where they are and Level 3?

### Step 2: Assign ownership and cadence

For each of the 12 activities, fill in:
- **Owner:** PM, eng, data science, QA, PM + legal, or PM + eng
- **Cadence:** per_change, weekly, monthly, event_driven, every_ship, or continuous

Use the ownership model from Concept 1 (PM = strategy, eng = infrastructure, data science = calibration) and the cadence model from Concept 2 (per-change, periodic, event-driven).

**Key question:** which activities should be per-change (automated in CI/CD) vs periodic (scheduled review)? What's the deciding factor?

### Step 3: Select the first 30-day priorities

You can't build all 12 activities in 30 days. Pick the **four** that give the most leverage — meaning they unblock the most other activities or close the most critical gaps.

For each of your four picks, state: (a) why this one first, (b) what it unblocks, (c) the implementation effort.

**Key question:** should your first 30 days focus on L2 activities (golden dataset, basic checks) or L3 activities (CI/CD, monitoring)? What's the argument for building the foundation first vs jumping to automation?

### Step 4: Design the 90-day roadmap

Organize all 12 activities into three phases:

- **Days 1–30:** Foundation. The activities that make everything else possible.
- **Days 31–60:** Automation. The activities that remove manual bottlenecks and make evals repeatable.
- **Days 61–90:** Maturity. The activities that move the team from "we run evals" to "evals drive our decisions."

For each phase, list the activities, the owners, and what "done" looks like at the end of the phase.

**Key question:** what's the single metric you'd show leadership at Day 90 to prove the eval investment was worth it? Pick one number that captures the before/after.

---

## Part 3: PM Decision Point

You present your 90-day roadmap to the VP of Product. They're supportive but push back:

*"I believe in evaluation, but I also know how these things go. You start with a 90-day plan and suddenly it's 6 months and we've built an eval empire instead of shipping features. I'll give you the 90 days, but I need you to pick three things — just three — that you'll actually deliver. If those three work, I'll fund the rest. What are your three?"*

Your response should do three things:

1. **Name exactly three activities** from the 12 in the dataset. For each: what it is, who owns it, what cadence it runs at, and what "done" looks like in concrete terms (not "we'll have a golden dataset" — state the number of cases, the pass rate threshold, the automation status).

2. **Explain why these three and not others.** The VP wants to know your prioritization logic. What makes these three the highest-leverage picks? What do they unblock? What risk do they mitigate that the other nine don't?

3. **State what you'll show the VP at Day 90.** One metric, one before/after comparison, one sentence. This is how you earn the funding for the remaining nine activities. If you can't state it now, you won't be able to measure it then.

This is the final exercise of the course. You're not being asked to recite what you learned — you're being asked to *apply* it under the constraint every PM actually faces: limited time, limited buy-in, and a need to show results fast. The quality of your three picks is the quality of your judgment as an AI PM.

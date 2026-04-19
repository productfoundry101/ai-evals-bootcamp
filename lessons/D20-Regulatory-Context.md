# D20 - Regulatory Context

**Week 3, Day 20** | ~45 min
**Part times:** Concepts ~15 min | Exercise ~20 min | Decision Point ~10 min
**Previous lesson:** D19 - Ship Decisions — you learned to synthesize eval signals into a single ship call, distinguishing ship/hold/conditional ship outcomes. You applied AND logic to guardrails, drafted a ship memo, and communicated a conditional-ship recommendation to your CEO under Q2 launch pressure. Today we step outside the evaluation framework and ask: *what does the law require?* The eval practices you've built across D1–D19 are good engineering. Some of them are also legal obligations — and knowing which ones changes how you prioritize, document, and defend your eval work.

This lesson assumes familiarity with the full eval toolkit (D1–D19). It does not require legal expertise. The goal is to give PMs enough regulatory literacy to guide compliance conversations with legal and engineering — not to replace legal counsel.

---

## Part 1: The Concepts

### The regulatory landscape: what's binding, what's guidance, what's coming

Three frameworks matter for AI PMs in 2026. They differ in enforceability, and confusing them is a common PM mistake.

1. **EU AI Act** — *binding law* with enforcement deadlines and financial penalties. Applies to any AI system placed on the EU market or affecting EU residents, regardless of where the company is headquartered. The key deadline for high-risk systems is **August 2, 2026** (with possible extension to December 2027 under the Digital Omnibus proposal — but planning for the later date is a gamble). Penalties: up to €35 million or 7% of global annual turnover for serious violations.

2. **NIST AI Risk Management Framework (AI RMF)** — *voluntary US guidance*, not law. But increasingly referenced in federal procurement contracts, enterprise vendor assessments, and insurance underwriting. Four core functions: Govern, Map, Measure, Manage. The "Measure" function is where your eval practices live. Not following NIST won't get you fined, but it may cost you contracts.

3. **White House AI Executive Order and related guidance** — *signaling and disclosure expectations* for frontier models. Mostly relevant if you're building or deploying foundation models at scale. Sets expectations for red-teaming disclosure, safety testing, and reporting.

The PM discipline: **know which framework is binding for your product and market.** "We follow NIST" is not a compliance defense for the EU AI Act. "We comply with the EU AI Act" is overkill for a minimal-risk US-only internal tool. Match your compliance investment to your actual regulatory exposure.

### Risk classification determines your obligations

The EU AI Act uses a tiered risk system. Your obligations depend entirely on which tier your product falls into. The PM's first job — before any compliance work — is to classify your product.

**Tier 1: Unacceptable risk (banned).** AI systems that manipulate behavior through subliminal techniques, exploit vulnerable groups, enable social scoring by governments, or perform real-time remote biometric identification in public spaces (with narrow exceptions). These are prohibited. If your product falls here, the conversation is not "how do we comply" — it's "we cannot build this."

**Tier 2: High-risk (Annex III).** AI systems used in:
- Employment and worker management (hiring, performance evaluation, task allocation)
- Credit and insurance decisions
- Education (admissions, grading, proctoring)
- Essential services access (benefits, emergency services)
- Law enforcement and border control
- Healthcare (triage, diagnosis support)
- Critical infrastructure safety components

These carry the heaviest obligations: documented risk management, data governance, technical documentation, logging, human oversight, and accuracy/robustness testing. All documented. All auditable.

**Tier 3: Limited risk.** AI systems that interact directly with users (chatbots, content generators). Main obligation: **transparency** — users must be informed they are interacting with AI.

**Tier 4: Minimal risk.** Everything else. No specific obligations under the Act.

An important nuance: Annex III systems are only classified as high-risk if they pose a *significant risk* to health, safety, or fundamental rights. A system that falls under an Annex III category but demonstrably has no significant risk may argue for a lower classification — but this argument must be documented and defensible. When in doubt, classify up, not down.

### High-risk requirements map to eval practices you already know

If your system is classified as high-risk, the EU AI Act requires six categories of compliance. The good news: they map almost directly to what you've learned in D1–D19.

| EU AI Act Requirement | What it means | Course practice that covers it |
|----------------------|---------------|-------------------------------|
| **Risk management system** | Documented process for identifying, assessing, and mitigating risks throughout the system lifecycle | D2 - Failure Surface Mapping evaluation surface map, D10 - Release Criteria release criteria, D19 - Ship Decisions ship memo |
| **Data governance** | Training and evaluation data must be relevant, representative, and free from errors. Bias in data must be addressed. | D7 - Golden Datasets golden datasets, D12 - Fairness & Subgroups fairness and subgroup evaluation |
| **Technical documentation** | Eval methodology, metrics, results, and system design must be documented before deployment | D10 - Release Criteria thresholds, D16 - AI Experiments experiment design, D19 - Ship Decisions ship memo |
| **Logging and traceability** | Automatic recording of events for post-hoc audit and incident investigation | D14 - Observability observability (metrics, logs, traces) |
| **Human oversight** | Effective mechanisms for human control, including the ability to override or shut down | D5 - Grader Types human graders, D17 - Launch Readiness online eval human-review layer, D17 - Launch Readiness rollback criteria |
| **Accuracy, robustness, cybersecurity** | System must be tested for accuracy under normal and adversarial conditions, with results documented | D4 - Thinking in Distributions distributions and CIs, D16 - AI Experiments experiments, D18 - Red Teaming red teaming |

The *gap* is not in the eval practices themselves — it's in the **documentation and formalization**. You may have done all of this work, but if it's not written down in a format that survives an audit, it doesn't count. A conformity assessment requires evidence, not just practice.

### Conformity assessment: what the documentation package must contain

Before placing a high-risk AI system on the EU market, providers must complete a conformity assessment — essentially proving that all six requirements are met.

For most Annex III systems, this is an **internal self-assessment** (you assess yourself and declare conformity). Certain biometric systems require third-party audit.

The conformity documentation package must include:

1. **System description** — what the system does, its intended purpose, who the users are, what inputs/outputs it handles.
2. **Risk assessment** — identified risks, their severity, and what mitigations are in place. Maps to your D2 - Failure Surface Mapping evaluation surface map.
3. **Data documentation** — how training and evaluation data was sourced, curated, and checked for bias. Maps to D7 - Golden Datasets and D12 - Fairness & Subgroups.
4. **Evaluation methodology and results** — what metrics, what thresholds, what test sets, what results. This is your D10 - Release Criteria release criteria + D16 - AI Experiments experiment results + D18 - Red Teaming red-team findings. Must include confidence intervals, not point estimates.
5. **Monitoring plan** — how the system will be monitored post-deployment for drift, degradation, and incidents. Maps to D17 - Launch Readiness production monitoring.
6. **Human oversight design** — how humans can intervene, override, or shut down the system. Maps to D17 - Launch Readiness rollback criteria and on-call ownership.
7. **Incident reporting plan** — how serious incidents will be detected, reported to authorities, and corrected. EU AI Act requires notification of serious incidents to market surveillance authorities.

After completing this package, the provider issues an **EU Declaration of Conformity**, affixes the **CE marking**, and registers the system in the EU database.

The PM's role: you don't write the legal filings, but you own the *eval evidence* that feeds into them. If the eval evidence is weak, incomplete, or undocumented, legal cannot build a defensible conformity package on top of it.

### Post-market obligations close the monitoring loop

Compliance is not a one-time gate. High-risk system providers must:

1. **Monitor continuously** — track system performance against the documented benchmarks. If performance degrades below documented thresholds, corrective action is required.
2. **Report serious incidents** — any incident involving death, serious health damage, disruption of critical infrastructure, or violation of fundamental rights must be reported to the relevant market surveillance authority. Timeline: typically within 72 hours of becoming aware.
3. **Take corrective action** — if non-compliance is identified (by you or by an authority), you must bring the system back into compliance or withdraw it from the market.
4. **Cooperate with authorities** — provide information, access to logs, and system documentation upon request.

This is D17 - Launch Readiness production monitoring with legal teeth. The monitoring plan you designed in D17 - Launch Readiness (alerts, rollback triggers, drift detection) is exactly what the regulation requires — but it must be documented, maintained, and connected to a regulatory reporting pathway.

The implication: **if you detect a guardrail breach (D10 - Release Criteria) or a drift pattern (D17 - Launch Readiness) and don't act on it, you're not just making a bad product decision — you may be in regulatory non-compliance.** The regulation requires that you act on what your monitoring detects.

---

## Part 2: Exercise — Compliance Audit of Your Eval Practices

### Scenario

Your food delivery company is preparing to launch its menu verification system in the EU market. The VP of Legal has asked you — the PM who built the eval framework — to do an initial compliance assessment. Before the legal team drafts the formal conformity documentation, they need you to answer: *What do we already have from our eval work, and what's missing?*

### Dataset

Open `exercises/D20-regulatory-audit-dataset.csv`. The dataset has two sections:

**Section A (rows P-01 through P-08):** Eight AI products to classify by EU AI Act risk tier. This builds your classification intuition.

**Section B (rows R-01 through R-10):** Ten EU AI Act requirements for high-risk systems, mapped against your existing eval practices from D1–D19. Each row shows a requirement, what it demands, and a `coverage_status` that you'll assess.

| Column | What it means |
|--------|---------------|
| `item_id` | Identifier (P-01 through P-08 for products, R-01 through R-10 for requirements) |
| `section` | `classification` or `audit` |
| `item_name` | Product name (Section A) or requirement name (Section B) |
| `description` | What the product does (A) or what the requirement demands (B) |
| `sector` | Industry sector (Section A only) |
| `risk_tier` | Your classification: unacceptable, high, limited, minimal (Section A — you fill this in) |
| `course_coverage` | Which D-lessons address this requirement (Section B only) |
| `coverage_status` | covered, partial, or gap (Section B — you assess this) |
| `notes` | Additional context |

### Expected outcome

By the end: (1) you'll be able to classify AI products by EU AI Act risk tier, (2) you'll have assessed whether your menu verification system is high-risk, (3) you'll have audited your D1–D19 eval practices against the six regulatory requirement categories, and (4) you'll have identified the specific gaps between "good eval practice" and "regulatory compliance."

### Step 1: Classify the eight AI products by risk tier

Look at Section A (P-01 through P-08). For each product, assign a risk tier: unacceptable, high, limited, or minimal.

Use the tier definitions from Concept 2. Pay attention to the sector column — it's the strongest signal for Annex III classification.

**Key question:** P-06 is the menu verification system you've been evaluating all course. What tier would you assign it, and why? Is "food safety" enough to make it high-risk, or does it depend on how the system is deployed?

### Step 2: Decide — is your menu verification system high-risk?

This is not a simple yes/no. The system checks allergen disclosures on restaurant menus. An incorrect approval could lead to an allergic reaction — potentially fatal. But the system is a B2B tool used by the food delivery platform, not a consumer-facing medical device.

Consider:
- Does it affect **health and safety** of end consumers? (Yes — allergen decisions)
- Is it a **safety component** of a product covered by EU product-safety legislation? (Possibly — food safety regulations)
- Does it fall under an **Annex III category**? (Not explicitly listed, but "safety components" is a broad category)

State your classification and your reasoning. If you classify it as high-risk, all six requirement categories apply. If limited risk, only transparency obligations apply.

**Key question:** what's the cost of classifying too high vs too low? If you classify as limited risk and a regulator later determines it's high-risk, what happens?

### Step 3: Audit your eval practices against high-risk requirements

Look at Section B (R-01 through R-10). Assume you've classified the menu verification system as high-risk (even if you argued limited risk in Step 2 — this is the conservative approach).

For each requirement, assess `coverage_status`:
- **Covered:** your D1–D19 eval practices fully address this requirement, including documentation.
- **Partial:** you do the work but haven't formalized or documented it in a way that survives an audit.
- **Gap:** you haven't addressed this requirement at all.

**Key question:** most of your eval practices are "partial" rather than "covered." What's the common pattern in what's missing — is it the practice itself, or the documentation of the practice?

### Step 4: Prioritize the gaps

From your Step 3 audit, list the gaps and partial-coverage items. Rank them by:
1. **Regulatory risk** — which gaps would cause the most severe compliance failure?
2. **Effort to close** — which gaps can be closed quickly (documentation) vs slowly (new capabilities)?

For each gap, name the specific action needed and estimate whether it's a days-level, weeks-level, or months-level effort.

**Key question:** which gaps are pure documentation problems (cheap to fix) and which require building new capabilities (expensive)? Does this change your EU launch timeline?

---

## Part 3: PM Decision Point

Your VP of Legal calls you into a meeting. Engineering is also on the call. The eng lead opens:

*"We've been running evals for months. We have golden datasets, automated graders, monitoring dashboards, red-team results, ship memos — the works. I don't see why we need to do anything extra for EU compliance. We're already doing more eval work than 90% of companies. Can't legal just package up what we have?"*

Your response should do three things:

1. **State your risk classification for the menu verification system** and whether you'd defend it as high-risk or limited-risk in a regulatory conversation. Name the specific factor that tips your classification.

2. **Identify at least three specific gaps** between "we have good eval practices" and "we have a conformity-ready documentation package." Be concrete: name the requirement, what you have, and what's missing. Don't say "we need better documentation" — say "R-03 requires versioned technical documentation of eval methodology; we have scoring rubrics and ship memos but no versioned artifact that links methodology → dataset → results → thresholds in a single auditable document."

3. **Propose a realistic compliance roadmap** with two phases: (a) what can be done in 2 weeks to close the documentation gaps (cheap), and (b) what requires longer-term investment (new capabilities like incident reporting workflows, regulatory notification pathways, formal data governance documentation). Give the VP of Legal a timeline they can plan around.

Don't frame this as disagreeing with engineering. The eng lead is right that the eval practices are strong. The gap is not in what you *do* — it's in what you can *prove* to a regulator. That distinction is the entire point of this lesson.

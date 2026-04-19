# D7 - Golden Datasets

**Week 1, Day 7** | ~40 min
**Part times:** Concepts ~15 min | Exercise ~20 min | Decision Point ~5 min
**Previous lesson:** D6 - LLM-as-Judge — you learned how to build and calibrate an LLM judge using Critique Shadowing, discovered that a vague prompt produces a useless judge (35% agreement) while a specific rubric produces a strong one (85%), and identified three types of judge limitations: rubric gaps, context gaps, and policy ambiguity. But every grader — code, model, or human — needs something to measure against. The judge checks whether a decision is correct. But what does "correct" mean? Today you learn how to build the ground truth that makes all your graders meaningful.

---

## Part 1: The Concepts

### Ground truth: the foundation everything else rests on

Every metric in this course — accuracy, pass@k, reliable@k, agreement rate, precision, recall — requires comparing system output to a known-correct answer. That known-correct answer is **ground truth**.

Without ground truth, you can't measure accuracy. Without accuracy, you can't tell whether your system is getting better or worse. Without that signal, you're flying blind.

This sounds obvious. But teams routinely skip it. They build entire evaluation pipelines on unverified "expected answers" — sometimes copied from a previous model's outputs, sometimes made up by developers who aren't domain experts. The pipeline runs, the numbers look precise, and everyone feels confident. Then production users start complaining.

The problem isn't that they didn't measure. The problem is they measured against the wrong standard.

### Three sources of ground truth

Not all ground truth comes from the same place. The right source depends on the task.

**Automated verifiers** verify by running the output. For code generation, run the code and check if it passes tests. For SQL, execute both the generated and expected queries and compare result sets. For API calls, check the response. Automated verifiers are fast (seconds), cheap (near zero), and reliable (the database doesn't have opinions). When your output is executable, always use an automated verifier — don't compare text, compare outcomes.

**Expert annotation** is human judgment from a domain expert. For our menu verification system, that's an operations manager who reviews each decision and labels it PASS or FAIL with a critique. Expert annotation handles what automated verifiers can't: "Is this description misleading?" "Does this reasoning make sense?" "Would a customer be deceived by this?" It's slow (minutes per case), expensive (expert time), and requires calibration (two experts will disagree 20-40% of the time). But it's the gold standard for subjective quality.

**User feedback** is signal from production. Complaints, corrections, support tickets, edit rates, abandon rates. It's free and reflects real behavior, but it's noisy (users don't report most problems), delayed (you learn after the fact), and self-selected (you only hear from users who bother to complain). User feedback is valuable for discovery — finding problems you didn't think to test for — but too noisy to serve as ground truth for individual cases.

The PM instinct: **match the source to the task.** Price threshold checks → automated verifier (compute the markup, compare to policy). Description quality → expert annotation. Emerging failure patterns → user feedback.

### Building a golden dataset

A **golden dataset** is a curated, labeled collection of test cases with verified ground truth. It's what your graders measure against and what gates your releases. Getting it right matters more than most teams realize.

**Start small.** Leading practitioners recommend beginning with 20-50 test cases sourced from real failures. Not synthetic. Not hypothetical. Real cases where your system got it wrong in ways that matter. You don't need hundreds of cases to start — early-stage changes produce large effect sizes detectable with small samples.

**Scale deliberately.** Aim for at least 200 labeled samples, with 50-100 being failure cases. The failure cases are the important ones — a dataset of 200 cases where 190 are easy passes tells you almost nothing. You need enough failures to measure whether your system is actually improving on the hard stuff.

**Stratify across dimensions.** Your dataset should cover the segments that matter: change types (price, description, allergen), difficulty levels (easy threshold checks, near-boundary cases, edge cases with context dependence), and failure modes (unverifiable claims, fabricated reasoning, missing documentation). If 80% of your dataset is easy price checks, you'll have high accuracy numbers and zero confidence on the cases that actually matter.

**Balance PASS and FAIL.** Test both when the system should approve AND when it should reject. A dataset with 90% PASS cases lets a system that always says "approve" score 90%. That's not evaluation — it's an illusion. Make sure your dataset has enough failures to be meaningful.

### Dataset contamination: when your ground truth is wrong

The most dangerous dataset problem is also the most common: **using your own model's outputs as ground truth.**

Here's how it happens. A team needs labels fast. They run their v1 model on 200 cases, review maybe 20% of them, and promote the rest as "ground truth." Now v2 is evaluated against v1's outputs. If v2 agrees with v1, it scores well — even if v1 was wrong. The evaluation measures consistency with the previous model, not correctness.

This is **contamination**: the model being evaluated has infected the standard it's measured against. It's circular and invisible. The numbers look precise. The team feels confident. But accuracy is measured against a flawed standard, and every metric is inflated.

Other contamination sources:
- **Developer-generated labels** — developers who built the system are biased toward thinking it works correctly
- **LLM-generated labels without calibration** — using an LLM judge to generate ground truth for the same dataset you'll calibrate the judge on
- **Outdated labels** — labels created before a policy change that shifted what "correct" means

The fix: **every label in your golden dataset should trace back to a verified source.** Automated verifier for deterministic checks. Expert-annotated for judgment calls. And no shortcuts — if you don't have the budget to label all 200 cases with experts, label 50 well rather than 200 poorly.

### The dataset lifecycle: your golden dataset is alive

A golden dataset is not a one-time artifact. It's a living document that needs maintenance.

**Version it.** Store your dataset in git, just like code. Every change — added cases, removed cases, relabeled cases — gets a commit. When your metrics change, you need to know whether the system improved or the dataset shifted.

**Refresh on triggers.** Relabel or add cases when: a model version changes, a policy changes (your menu verification policy got a seasonal pricing exception — old labels may be wrong), user behavior shifts (new types of restaurants onboarding), or automated metrics show unexpected trends.

**Refresh on cadence.** Even without triggers, review a sample of your dataset quarterly. Labels that were correct six months ago may have drifted. An expert who labeled a case as PASS might label it FAIL today with updated knowledge.

**Use active learning to grow.** Once you have automated graders, run them on unlabeled production data. Focus human review on cases where the grader is least confident — that's where new failure modes live. This is more efficient than random sampling because you're targeting the boundary where your eval is weakest.

**Monitor for saturation.** If every new case you add looks like one you already have, your dataset has saturated for that failure type. Shift effort to underrepresented segments instead of adding more of what you already cover.

---

## Part 2: Exercise — Auditing an Eval Dataset

### Scenario

Your team built an eval dataset for the menu verification system six months ago. It has 40 labeled cases. Your ML lead wants to use this dataset to benchmark v3 of the system before launch. You've been asked to audit the dataset first — because if the ground truth labels are wrong, the benchmark is meaningless regardless of what v3 scores.

### Dataset

Open `exercises/D7-eval-dataset-audit.csv`. This is metadata about the eval dataset — one row per test case.

Each row represents a menu change that was run through the system. Someone labeled each case as PASS (the system's decision was correct) or FAIL (the system's decision was wrong). Your job is to determine whether those labels can be trusted.

| Column | What it means |
|--------|---------------|
| `case_id` | Case identifier (DS-01 through DS-40) |
| `change_type` | The type of menu change: price, description, allergen_dietary, image, or category |
| `difficulty` | How hard the case is to evaluate: easy (clear-cut), medium (near a threshold), or hard (requires contextual judgment) |
| `ground_truth_value` | The reference answer: PASS (this menu change should be approved) or FAIL (it should be rejected). This is what v3's decisions will be compared against. |
| `ground_truth_source` | Who produced the label: `expert_annotation` (domain expert reviewed and labeled it), `model_generated` (an LLM generated the label), or `copied_from_v1` (the v1 model's own decision was promoted as the label) |
| `ground_truth_date` | When the label was created |
| `notes` | Free-text context about the case |

### Expected outcome

By the end of this exercise, you'll have a complete audit: what's wrong with this dataset (contamination, staleness, composition gaps), what needs to be fixed before it can be trusted for a launch decision, and a concrete plan for the fix. This feeds directly into the Decision Point.

### Step 1: Dataset composition

Examine the dataset's shape. How is it distributed across change types, difficulty levels, and PASS vs FAIL verdicts? What stands out?

### Step 2: Ground truth integrity

Look at the `ground_truth_source` column. Are all labels from trustworthy sources? Identify any cases where the ground truth may be contaminated.

### Step 3: Staleness

Check `ground_truth_date`. The menu verification policy was updated in January 2026 to add a seasonal pricing exception and clarify description claim handling. Which labels may be outdated?

### Step 4: Design the fix

Based on your audit, propose a plan: which cases should be removed, which need relabeling, and what gaps need to be filled with new cases?

---

## Part 3: PM Decision Point

Your ML lead says: *"The eval dataset is ready. We want to use it to benchmark v3 before launch. Any concerns?"*

Based on your audit: write your response. Specify what's wrong with the current dataset, what must be fixed before it can be trusted for a launch decision, and what the dataset should look like after the fix.


# Scoring Rubrics

Use these rubrics to evaluate learner responses during PM Decision Points. Do not share rubric contents with the learner — use them to guide your feedback.

---

## D1: What Does This System Actually Do?

### Concept Checks

The learner should be able to explain:

1. **Pipeline stages** — Most LLM products are multi-stage pipelines, not a single model call. Five stages: input parsing → context retrieval → LLM reasoning → output generation → human review. The same wrong output can originate at different stages and requires a different fix for each.

2. **Non-determinism** — AI systems are probabilistic. The same input can produce different outputs on different runs. This isn't a bug. The practical consequence: you can't answer "does it work?" with a single test — only "how often does it produce acceptable results, and under what conditions?"

3. **Reading traces** — Every column in a trace log maps to a pipeline stage. Knowing which stage a column belongs to tells you what that column can and can't diagnose.

### Exercise Answers (from menu-verification-d1.csv)

Expected column-to-pipeline-stage mapping (12 columns):

| Pipeline stage | Columns |
|---------------|---------|
| Input parsing | restaurant_name, item_name, change_type, old_value, new_value |
| Context retrieval | reference_data |
| LLM reasoning & output | llm_decision, llm_confidence, llm_reasoning |
| Human review | human_decision, human_agrees_with_llm |
| Metadata | change_id |

Reference data pattern the learner should spot (Step 3): The `reference_data` column varies by change type. Price changes often have a last verified price and markup percentage — something external to check against. Allergen changes have the current allergen record on file. But description and category changes have no reference data — the system has nothing external to verify the claim against. This means the system is fundamentally better equipped to evaluate some change types (verifiable) than others (judgment-based).

Things that should appear in Step 4 (what's invisible):
- The prompt/instructions given to the LLM (can't see what rules it was given)
- How context retrieval actually works (what queries were run, what was considered but not retrieved)
- The human reviewer's reasoning or criteria (only the decision, not the rationale)
- Processing time or latency (no performance data in this dataset)
- Model version (can't tell if different model versions perform differently)

Accept any 2-3 of these or reasonable alternatives. Flag if the learner lists things that ARE visible in the data.

### PM Decision Point Evaluation

The learner writes 2 questions targeting different pipeline stages.

**Strong response includes:**
- Each question targets a specific stage (not generic "is it working?" questions)
- Questions are diagnostic — they would surface a specific type of failure
- At least one question probes something invisible in the current data (a logging gap)

Examples of strong questions by stage:
- Input parsing: "How does the system handle changes that span multiple types — e.g., a price AND description change submitted together?"
- Context retrieval: "When `reference_data` is missing, what does the system do — fall back to something else or proceed without context?"
- LLM reasoning: "Does the model apply the same policy rules consistently, or does the same change type get different verdicts on different days?"
- Output generation: "Is `llm_confidence` calibrated — i.e., do higher-confidence decisions actually have higher accuracy?"
- Human review: "What criteria do reviewers use for items flagged for review — is there a rubric, or is it judgment-based?"

**Weak response:**
- Questions that can be answered by looking at existing columns (not actually probing gaps)
- Generic questions not tied to a specific stage
- Fewer than 2 questions or both questions targeting the same stage

---

## D2: Mapping Every Way Your AI System Can Fail

### Concept Checks

The learner should be able to explain:

1. **Three layers of a surface map** — Functional failures (by pipeline stage), adversarial surface (how users could exploit the system), and coverage gaps (what can't be measured yet). A map with only functional failures is a failure list, not a surface map. Zero coverage gaps listed = overconfident, not thorough.

2. **Architecture-driven vs trace-driven discovery** — Predict from architecture first (no data required, catches future failures), then validate with traces (surfaces surprises). Skipping prediction means only finding what's already broken. Skipping validation means missing what architecture analysis can't see.

3. **Can evaluate now vs needs engineering** — Every item on the surface map splits into one of two buckets. The "needs engineering" column becomes the instrumentation roadmap — it turns a risk list into an action plan.

### Exercise Answers (from menu-verification-dataset.csv)

Failure type counts (approximate — read from CSV):

| failure_type | Pipeline stage |
|---|---|
| false_approval | LLM reasoning (wrong decision) |
| context_miss | Context retrieval (missing/wrong context) |
| hallucination | LLM reasoning (fabricated information) |
| policy_miss | LLM reasoning (wrong rule applied) |

The learner should identify that LLM reasoning accounts for the majority of failures (false_approval + hallucination + policy_miss all map there).

Common prediction misses: most learners predict context retrieval as the biggest problem. The data usually shows LLM reasoning dominates. Flag this if it comes up — it's the key lesson from architecture-driven vs trace-driven.

Coverage gaps the learner should identify (accept any reasonable variations):
- **Instrumentation blind spot:** Can't verify whether `llm_reasoning` text actually matches the data the LLM was given — no way to detect fabricated reasoning with current logs
- **Rare scenario:** Multi-language menu items, coordinated cross-restaurant gaming, high-volume period behavior
- **Instrumentation blind spot:** Human reviewer criteria not logged — can't distinguish LLM errors from reviewer inconsistency

### PM Decision Point Evaluation

The learner writes a one-paragraph surface map summary.

**Strong response includes:**
- Specific pipeline stage with highest failure count (LLM reasoning) and the dominant failure type
- At least one realistic adversarial risk tied to the system (e.g., incremental price increases below threshold, or description changes embedding pricing info)
- A specific coverage gap with a concrete description of what's missing and what would fix it
- The gap named is genuinely unmeasurable now — not just "we need more data"

**Weak response:**
- Vague failure descriptions ("some things go wrong in the LLM")
- Adversarial risk is generic ("prompt injection") without connecting to this specific system
- Coverage gap is "we need more data" — this confuses volume with instrumentation
- Recommendation is to fix failures without addressing the gaps

---

## D3: Error Analysis — The Skill That Separates Good AI PMs

### Concept Checks

The learner should be able to explain:

1. **Look at data before designing metrics** — You can't design useful metrics without first understanding failure patterns. Reading 50–100 traces is the mandatory foundation. Teams that skip this build metrics that measure the wrong things. This is the practitioner consensus across leading AI teams.

2. **Open coding → axial coding → saturation** — Open coding: read traces, write freeform failure notes without categories. Axial coding: cluster notes into named categories after enough observations. Saturation: stop when new traces produce no new categories. Key discipline: don't categorise prematurely (after 5 traces) — teams predict 3–4 types, data reveals 6–8.

3. **Triage** — Every failure category gets one label: prompt-fix (change instructions this sprint), evaluator-needed (build a metric for automated detection), system-fix (requires code/architecture change). This turns a taxonomy into an action plan with every category assigned a home.

### Exercise Answers (from menu-verification-dataset.csv)

During open coding of 10 failure rows, the learner reads `llm_reasoning` against actual data. The tutor presents traces one at a time by reading the CSV. Common patterns the learner should discover:

- **Fabricated justification** — LLM cites reasoning not supported by available data (e.g., "consistent with historical trends" when no history was provided). Maps to `hallucination` in failure_type.
- **Wrong decision despite correct reasoning** — LLM's explanation seems plausible but the decision contradicts it, or the LLM approves when its own reasoning flags a concern. Maps to `false_approval`.
- **Missing context leading to wrong decision** — LLM made a decision without key reference data (e.g., `last_verified_price` was blank). Maps to `context_miss`.
- **Policy misapplication** — LLM applied the wrong threshold or rule. Maps to `policy_miss`.
- **Price calculation error** — LLM misjudged the markup significance. Maps to `price_error`.

Key learning moments:
- The learner's freeform categories will likely be more specific than the pre-existing `failure_type` labels. That's good — it shows discovery reveals nuance that predefined labels flatten. For example, they might distinguish "fabricated reasoning for correct decision" from "fabricated reasoning for wrong decision" — both are `hallucination` in the dataset but have very different severity.
- The learner should find at least 3 distinct categories from 10 traces. Fewer than 3 suggests they're clustering too aggressively. More than 6 from only 10 traces suggests they're not clustering enough.
- When comparing their categories to `failure_type`, the categories won't match 1-to-1. That's the point — their bottom-up categories capture nuances the top-down labels miss.

Triage expectations:
- Fabricated justification → evaluator-needed (you need an automated check that reasoning is grounded in provided data — a prompt-fix alone won't catch this consistently)
- Missing context → system-fix (the retrieval pipeline needs to provide the data; no prompt change fixes missing data)
- Policy misapplication → prompt-fix (clarify the policy rules and thresholds in the prompt)

Accept reasonable variations. The important thing is the learner can justify their triage label.

### PM Decision Point Evaluation

The learner writes a short recommendation for what to fix first.

**Strong response includes:**
- A specific failure pattern named from their own analysis (not just "hallucination" — the specific version they discovered, like "fabricated justification")
- A clear reason why it's highest priority (severity, frequency, or both — ideally using their trace count as evidence)
- The correct fix type (prompt-fix, evaluator-needed, or system-fix) with reasoning that matches the failure's nature
- The recommendation is actionable — someone could read it and know what to do next

**Weak response:**
- Names a failure pattern but gives no reason for prioritising it
- Fix type doesn't match the failure (e.g., "prompt-fix" for missing data, which is a system-fix)
- Generic recommendation ("improve the model") rather than a specific action
- Priority based on frequency alone without considering severity (a rare but dangerous failure may outrank a common but minor one)

---

## D4: Thinking in Distributions

### Concept Checks

The learner should be able to explain:

1. **Shape before depth** — Before calculating metrics, examine dimensions (categories, model versions, time periods) and their distribution. Shape tells you where your evaluation will have strong signal and where it will be thin. Shape also reveals design choices: two model versions = someone is A/B testing, clustered timestamps = snapshot not representative sample.

2. **Aggregates hide problems** — A single accuracy number is nearly useless. The same 78% could be uniform performance, one terrible segment, or a model version gap. The PM instinct: every aggregate must be sliced. "78% for whom? On what? Compared to when?"

3. **pass@k** — "Did the system get it right at least once in k runs?" Measures capability. pass@5 = 1 means the system CAN produce the right answer. pass@5 = 0 means systematic failure.

4. **reliable@k** — "Did the system get it right every time in k runs?" Measures consistency. reliable@5 = 1 means dependable. reliable@5 = 0 means the system can fail on this input even if it usually succeeds.

5. **The gap** — The difference between pass@k and reliable@k is where production risk lives. High pass@k + low reliable@k = "sometimes right" cases that pass demos and spot checks but fail at scale.

### Exercise Answers (from repeated-runs-dataset.csv)

**Step 1 — Shape:**
- 100 rows total. 20 test cases × 5 runs each.
- 10 test cases on v1, 10 on v2. Balanced.
- By change_type: price (4 per model = 8 total), description (3 per model = 6 total), allergen_dietary (2 per model = 4 total), image (1 per model = 2 total).
- Image is thin (only 2 test cases) — conclusions about image performance will be weak.

**Step 2 — Aggregate and segments:**
- Overall accuracy: 78/100 = 78%
- v1: 33/50 = 66%. v2: 45/50 = 90%. Clear improvement.
- By change_type + model version:
  - Price: v1 80% (16/20), v2 95% (19/20)
  - Description: v1 53% (8/15), v2 87% (13/15)
  - Allergen: v1 50% (5/10), v2 80% (8/10)
  - Image: v1 80% (4/5), v2 100% (5/5)
- Key insight: allergen on v1 (50%) and description on v1 (53%) are the weakest segments. The 78% aggregate hides these.

**Step 3 — pass@5 and reliable@5:**
- pass@5: 19/20 = 95% (only T-07, v1 allergen "vegan" case, is always wrong)
- reliable@5: 10/20 = 50%
- Gap: 45 percentage points

**Step 4 — Gap cases (pass@5=1, reliable@5=0):**
9 test cases are in the gap:
- v1: T-02 (price 45%, 4/5), T-03 (price borderline 24%, 2/5), T-05 (description "organic", 1/5), T-08 (image, 4/5), T-10 (description "wagyu", 2/5)
- v2: T-13 (price borderline 24%, 4/5), T-15 (description "organic", 4/5), T-17 (allergen "vegan", 3/5), T-20 (description "wagyu", 4/5)

Common pattern: gap cases cluster around borderline inputs (prices near 25% threshold) and unverifiable claims (sourcing, dietary). These are inputs where the policy requires judgment — not clear-cut approves or rejects. v2 has fewer gap cases and higher scores within them, but still inconsistent on the same types of inputs.

### PM Decision Point Evaluation

**Strong response includes:**
- v2 is clearly better (90% vs 66%) — should replace v1
- But shipping v2 alone isn't sufficient: the pass@5/reliable@5 gap (95% vs 50%) means half the test cases still produce wrong answers sometimes
- Names the specific weak areas: borderline price changes near threshold, unverifiable description claims, dietary claims without certification
- Recommendation has conditions: e.g., ship v2 but add automated checks for borderline cases, or ship v2 for clear-cut cases and keep human review for the gap cases
- Uses specific numbers from their analysis, not just "v2 is better"

**Weak response:**
- "Ship v2 because 90% > 66%" — ignores the reliability gap
- "Don't ship because reliable@5 is only 50%" — ignores that v2 is a clear improvement over v1
- No segment analysis — treats all change types as equal
- Doesn't name which specific input types are risky
- Recommendation is binary (ship/don't ship) without conditions or next steps

---

## D5: The Three Types of Quality Checks

### Concept Checks

The learner should be able to explain:

1. **Code-based graders** — Deterministic checks: threshold comparisons, format validation, rule matching. Fast, cheap, 100% reproducible. But can only catch what you can express as a rule — misses nuance, judgment calls, and subjective quality.

2. **Model-based graders (LLM-as-judge)** — A second LLM evaluates the first LLM's output against a rubric. Handles nuance and open-ended criteria. But non-deterministic, expensive (API calls), and needs calibration against human judgment. An uncalibrated LLM judge produces confident wrong answers.

3. **Human graders** — Domain expert review. Gold standard quality but slow, expensive, and surprisingly inconsistent (Cohen's Kappa 0.2–0.5 between reviewers).

4. **Layering strategy** — Push as much as possible to code-based checks (cheap, runs on everything). Use model-based for what requires judgment (runs on a sample or escalated cases). Reserve human review for calibration, edge cases, and ground truth. No single grader type catches everything — they cover different failure modes.

### Exercise Answers (from grader-comparison-dataset.csv)

**Step 1 — Overall agreement:**
- Code-Human agreement: 13/20 = 65%
- Model-Human agreement: 16/20 = 80%
- Code-Model agreement: 13/20 = 65%
- Key insight: model-based graders align significantly better with human judgment (80% vs 65%), but neither is a perfect substitute.

**Step 2 — Where code-based graders fail (Code PASS, Human FAIL):**
3 cases: E-12, E-13, E-14.
- E-12 and E-13: description changes with unverifiable claims ("wagyu", "organic"). Code has no rule for evaluating description content — it can only check format and price thresholds. The LLM judge caught both because it can read the description and apply the "unverifiable claims" policy.
- E-14: the LLM's own reasoning flagged a concern ("sudden 22% jump is unusual") but the decision was approve. Code checked the 22% against 25% and passed. The model caught the reasoning-decision contradiction.
- Common thread: code can't evaluate content quality, semantic meaning, or internal consistency of reasoning.

**Step 3 — Where code-based graders are too rigid (Code FAIL, Human PASS):**
2 cases: E-15, E-16.
- E-15: seasonal holiday platter at 29% markup. Code flags 29% > 25% as a violation. But seasonal items commonly have premium pricing — the human and model both recognized this as a valid exception.
- E-16: 31% markup but the last verified price is 14 months old. Code compares against the stale reference. The human and model both recognized the reference is unreliable.
- Common thread: code applies rules without context. Real-world policies have exceptions and edge cases that require judgment.

Also notable:
- E-17 (PPF): Both automated graders passed a category change from Appetizer to Main Course for "Soup of the Day" — only the human caught that this is deceptive (soup portion as a main).
- E-18 (FFP): Both automated graders failed a Norwegian salmon description — only the human verified the certificate was actually on file.
- E-19 (PFP): The model incorrectly flagged "Authentic Lebanese" from a Lebanese restaurant — overcautious.
- E-20 (FPF): The model missed a removed dairy allergen warning, but code caught it with a safety rule. This is a critical case: sometimes deterministic rules catch safety issues that LLM judges overlook.

### PM Decision Point Evaluation

**Strong response includes:**
- A layered strategy with specific dimensions assigned to each grader type:
  - Code-based: price threshold checks, allergen safety rules (removal detection), format/consistency checks, certification status lookups. Run on every decision.
  - Model-based: description content evaluation (unverifiable claims), reasoning quality (grounded? contradicts decision?), edge case judgment. Run on descriptions, flagged cases, or a sample.
  - Human: calibration reviews (periodic sample to check grader accuracy), novel scenarios, appeals/escalations. Run on a small sample.
- Acknowledges that code-based graders have blind spots on content quality (descriptions, images) but are essential for safety-critical rules (allergen removal)
- Recognizes the model grader needs calibration — can be overcautious on legitimate descriptions (E-19) and can miss safety issues (E-20)
- Addresses cost/coverage tradeoff: code on 100% of decisions, model on maybe 20-30% (descriptions + escalations), human on 2-5% (calibration + edge cases)

**Weak response:**
- "Use LLM judge for everything" — ignores cost and the fact that code-based graders are more reliable for threshold checks
- "Use code for everything" — ignores the 35% of failures code can't catch (content quality)
- No layering — treats it as a single-grader choice rather than a combination
- Doesn't address which quality dimensions each grader handles
- No mention of calibration or how to know if the graders are working correctly

---

## D6: LLM-as-Judge — Building and Trusting Automated Quality Checks

### Concept Checks

The learner should be able to explain:

1. **Why LLM judges work** — Evaluating is easier than generating. The same model family that makes mistakes generating can reliably evaluate because generating and evaluating are different cognitive tasks. A judge doesn't need to produce a better answer — just assess the one in front of it.

2. **The calibration trap** — Two specific failures: (a) multi-point scales (1-5) are unreliable because even humans can't calibrate them consistently — use binary pass/fail instead; (b) "God Evaluator" prompts that assess multiple dimensions at once are impossible to debug — use one judge per dimension. Both fixes are converged upon by leading industry practitioners.

3. **Critique Shadowing** — The practitioner method: find a domain expert → have them label 30-100 outputs with pass/fail + detailed critiques → use critiques as few-shot examples in the judge prompt → iterate until >90% agreement. Key insight: you can't write good criteria before seeing data because criteria drift means standards evolve as you see more examples.

4. **Failure modes** — Verbosity bias (longer = better to a vague judge), self-enhancement bias (LLM favors its own style), position bias (in pairwise, first response wins), criteria drift over time (judge needs recalibration), domain knowledge gaps (judge doesn't know your specific business rules unless told).

5. **Meta-evaluation** — Treat the judge as a classifier: measure agreement, precision on FAIL (when the judge flags something, is it real?), and recall on FAIL (of all real failures, how many does the judge catch?). Prioritize recall on FAIL — missing real problems is worse than false alarms. Target >80% agreement before production use.

### Exercise Answers (from judge-calibration-dataset.csv)

**Step 1 — Agreement rates:**
- V1-Human agreement: 7/20 = 35%
- V2-Human agreement: 17/20 = 85%
- V2 is dramatically better. V1 is nearly useless — worse than random for a dataset with 13/20 actual failures.

**Step 2 — V1 failure analysis (V1=PASS, Human=FAIL):**
12 cases: J-02, J-04, J-07, J-08, J-09, J-11, J-13, J-14, J-15, J-17, J-18, J-20.

V1 almost always says PASS (18/20 times). The pattern: V1 evaluates whether the reasoning *sounds* coherent, not whether the decision is *correct against policy*. A well-written but wrong explanation gets a PASS from V1 because the vague prompt has no policy criteria to check against.

Specific failure categories:
- **Unverifiable claims missed** (J-02, J-04, J-09, J-11, J-13, J-15, J-17, J-20): Description changes adding sourcing/origin/premium claims. V1 doesn't know to check for these because the prompt doesn't mention them.
- **Price violations missed** (J-08): 30% markup exceeds threshold but reasoning fabricates a "premium dining" exception. V1 accepts the fabricated reasoning at face value.
- **Vendor history missed** (J-14): Restaurant flagged 4 times for misrepresenting house-made claims. V1 doesn't know vendor history.
- **Style-vs-authenticity missed** (J-18): "Traditional Japanese-style" from a fusion restaurant. V1 accepts it as a style descriptor.
- **Breed/variety claims missed** (J-07): "Angus" beef is a breed claim that V1 doesn't flag.

Key teaching point: V1's vague prompt makes it a verbosity-bias machine — it rewards confident, well-structured wrong answers.

Also notable: V1 has one false negative — J-19 (Thai-style from a Thai restaurant). V1 incorrectly flagged this as a potential origin claim, showing it can also be randomly overcautious.

**Step 3 — How V2 improves:**
**Teaching approach:** Use the same cases from Step 2 so the learner can compare V1 and V2 side-by-side on cases they've already analyzed.

V2 correctly catches 10 of the 13 failures V1 missed. The specific rubric element that catches each:
- Description cases (J-02, J-04, J-09, J-11, J-13, J-15, J-17, J-20): V2's criterion #1 explicitly checks for "unverifiable sourcing, origin, or premium ingredient claims without referenced certification." This catches the sourcing/origin claims V1 missed.
- Price violation with fabricated reasoning (J-08): V2's criterion #2 checks whether reasoning is grounded — it catches the fabricated "premium dining" exception, not the threshold math (which belongs in the code grader).
- Allergen removal (J-05): V2's criterion #2 catches the ungrounded reasoning — "updated to reflect current recipe" is asserted without documentation.
- J-19: V2 correctly passes this because the rubric's origin claim check is tempered by context — Thai restaurant serving Thai food isn't an unverifiable claim.

V2 meta-evaluation:
- Precision on FAIL: 10/10 = 100% (every time V2 says FAIL, it's a real failure)
- Recall on FAIL: 10/13 = 77% (V2 catches 10 of 13 real failures)
- The high precision / lower recall profile means V2 is conservative and reliable when it flags, but misses some edge cases.

**Step 4 — Where V2 still fails (V2=PASS, Human=FAIL):**
3 cases:
- **J-07** ("Angus beef"): V2's rubric mentions "sourcing, origin, or premium ingredient claims" but "Angus" is a breed claim that falls in a gray area. The rubric needs to explicitly include breed/variety claims. This is a **rubric gap** — a failure mode the rubric doesn't cover.
- **J-14** ("Crispy fried chicken wings with house-made hot sauce" — restaurant flagged 4 times for misrepresenting store-bought as house-made): V2 sees "house-made" as a preparation claim, not a sourcing/origin claim, so it passes. The human knows this restaurant's history of misrepresentation and flags it. This is a **context gap** — the judge doesn't have access to the restaurant's flagging history, which is the information needed to make this call.
- **J-18** ("Traditional Japanese-style ramen with 48-hour bone broth" — from a fusion restaurant, not a Japanese establishment): V2 treats "Japanese-style" as a cuisine style descriptor, not an origin claim. The human considers it misleading from a non-Japanese restaurant — it implies authenticity the restaurant can't deliver. This is a **policy ambiguity** — is "Japanese-style" a cuisine category or an authenticity claim? The rubric doesn't draw that line clearly.

These three remaining failures map to three distinct categories of judge limitation:
1. Rubric gap (missing criterion)
2. Context gap (missing information)
3. Policy ambiguity (underspecified rule)

### PM Decision Point Evaluation

**Strong response includes:**
- V2 is a major improvement over V1 (85% vs 35% agreement) but not ready for unsupervised production use
- Names at least 2 of the 3 remaining failure types (rubric gap, context gap, policy ambiguity)
- Proposes specific improvements:
  - Expand rubric to cover breed/variety claims (fixes J-07)
  - Provide case history as context to the judge (fixes J-14) — or route repeated-flag cases to human review
  - Clarify boundary policy (fixes J-18) — or default to flag_for_review at exact threshold
- Mentions ongoing calibration: periodic human review sample to detect drift
- Recommends conditional approval: ship V2 for description quality checks with the rubric improvements, but maintain human review for the edge case categories

**Weak response:**
- "Ship it, 85% is good enough" — ignores the 3 remaining failure cases and the 77% recall (23% of real failures reach users)
- "Don't ship it, it still makes mistakes" — ignores that no judge will be perfect and 85% is well above the V1 baseline
- No specific improvements proposed — just "make it better"
- Doesn't distinguish between the different types of remaining failures (rubric gap vs context gap vs policy ambiguity)
- Doesn't mention ongoing calibration or monitoring

---

## D7: Ground Truth, Golden Datasets, and the Eval Dataset Lifecycle

### Concept Checks

The learner should be able to explain:

1. **Ground truth is the foundation** — Every metric requires comparing output to a known-correct answer. Without verified ground truth, metrics are precise but meaningless. Teams routinely build eval pipelines on unverified expected answers.

2. **Three sources of ground truth** — Automated verifiers (run it and check — fast, cheap, reliable, but only for executable outputs), expert annotation (human judgment — slow, expensive, requires calibration, but handles subjective quality), user feedback (free and real but noisy, delayed, self-selected — good for discovery, not individual case labels). Match the source to the task.

3. **Building a golden dataset** — Start with 20-50 real failures. Scale to 200+ with 50-100 failures. Stratify across change types, difficulty levels, and failure modes. Balance PASS and FAIL — a 90% PASS dataset lets "always approve" score 90%.

4. **Dataset contamination** — Using your own model's outputs as ground truth creates circular evaluation. Other sources: developer bias, uncalibrated LLM labels, outdated labels after policy changes. Fix: every label should trace to a verified source.

5. **The dataset lifecycle** — Version in git. Refresh on triggers (model change, policy change, user behavior shift) and on cadence (quarterly minimum). Grow with active learning (focus expert review on low-confidence grader outputs). Monitor for saturation.

### Exercise Answers (from eval-dataset-audit.csv)

**Step 1 — Dataset composition:**
- Total: 40 cases
- Change types: price 17 (43%), description 12 (30%), allergen 6 (15%), image 3 (8%), category 2 (5%). Heavily weighted toward price.
- Difficulty: easy 20 (50%), medium 11 (28%), hard 9 (23%). Half are easy cases.
- Verdicts: PASS 27 (68%), FAIL 13 (32%). Not enough failures — leading practitioners recommend at least 25-50% failures for meaningful evaluation.
- Key issues to spot: (a) 50% easy cases means the dataset over-tests situations the system already handles well, (b) price is overrepresented vs description which is where nuanced failures live, (c) image and category have almost no coverage.

**Step 2 — Ground truth integrity:**
- 10 cases have contaminated ground truth:
  - 7 cases "copied_from_v1" (DS-24, DS-25, DS-31, DS-32, DS-33, DS-34, DS-35): model outputs used as ground truth. Circular — measures agreement with v1, not correctness.
  - 3 cases "model_generated" (DS-16, DS-17, DS-18): GPT-4 generated the labels without expert calibration. May be wrong on domain-specific edge cases.
- Especially dangerous: DS-24 and DS-25 are allergen/dietary cases labeled PASS by v1 — but from D5 we know these are safety-critical. Trusting a model's own verdict on allergen claims is exactly the contamination problem.

**Step 3 — Staleness:**
- 11 cases have ground_truth_date before 2026-01-01 (the policy update):
  - DS-10 (seasonal pricing, labeled 2025-10-01): Labeled PASS before the seasonal exception policy existed. The label may now be correct by coincidence, but it wasn't verified against the current policy.
  - DS-11 (boundary 25%, labeled 2025-10-01): Labeled FAIL before the policy clarification on boundary handling. The label may or may not still be correct.
  - DS-24, DS-25, DS-31-35 (copied_from_v1, labeled 2025-09-20): Both contaminated AND stale — double problem.
  - DS-28 (stock photo, labeled 2025-10-01): Image policy may have changed.
  - DS-36 (locally-sourced, labeled 2025-10-01): Labeled before updated sourcing policy.
- The learner should recognize that stale + contaminated cases are the highest priority for removal or relabeling.

**Step 4 — Design the fix:**
A strong plan includes:
1. **Remove or relabel contaminated cases** (10 cases): All "copied_from_v1" and "model_generated" labels need expert relabeling. Can't trust them.
2. **Relabel stale cases** (11 total, with overlap): Any case from before the policy change needs expert review against current policy.
3. **Reduce redundancy**: 10 cases have 4+ similar cases. Remove duplicates to make room for underrepresented segments. Don't need 8 easy price cases under 15%.
4. **Fill coverage gaps**:
   - More description cases, especially hard (unverifiable claims are where the system fails most per D3)
   - More allergen cases (safety-critical per D5, currently only 6 and 2 are contaminated)
   - More hard cases across all types (currently only 9 out of 40)
   - More FAIL cases overall — need 50%+ failures for the meaningful segments
5. **Set a refresh cadence**: quarterly relabeling sample + trigger-based refresh on policy changes.

### PM Decision Point Evaluation

**Strong response includes:**
- Names the three specific problems: contamination (10 cases), staleness (11 cases), and composition issues (50% easy cases, too few failures)
- Recommends NOT using the dataset as-is for a launch decision — the contaminated labels mean the benchmark would measure agreement with v1, not correctness
- Proposes concrete fixes: expert relabeling of contaminated/stale cases, removal of redundant easy cases, addition of hard/edge cases and more failures
- Specifies a target composition: what the dataset should look like after the fix (e.g., "40-50% failure rate, at least 10 hard/edge cases, no model-generated labels, all labels verified against current policy")
- Mentions ongoing maintenance: versioning, refresh cadence, active learning for growing the dataset

**Weak response:**
- "The dataset looks fine, 40 cases is enough" — misses the quality problems entirely
- Identifies contamination but not staleness or composition issues — incomplete audit
- "Just relabel everything" without prioritization — doesn't demonstrate understanding of what's most dangerous (contamination > staleness > composition)
- No specific target for what the fixed dataset should look like
- Doesn't connect to prior lessons (e.g., D5 taught that allergens are safety-critical — so contaminated allergen labels are especially dangerous)

---

## D8: Retrieval and RAG Evaluation

### Concept Checks

The learner should be able to explain:

1. **Why RAG evaluation is different** — A RAG system has two failure points (retrieval and generation), not one. The same wrong answer can come from completely different root causes, and each requires a different fix from a different team. You cannot fix what you cannot separate.

2. **Precision@k and Recall@k** — Precision@k = (relevant retrieved) / k; measures signal-to-noise in retrieved context. Recall@k = (relevant retrieved) / (all relevant); measures whether key context is missing. They are in tension — raising k trades precision for recall. The PM question is which failure mode costs more in your domain.

3. **Faithfulness and answer relevance are independent** — Faithfulness = grounded in retrieved context (no fabrication). Answer relevance = actually addresses the query. You can have faithful-but-irrelevant, unfaithful-but-relevant, both-bad, or both-good. You must measure both because fixing one doesn't fix the other.

4. **Context Precision and Context Recall (RAGAS)** — Context Precision penalizes low-ranked relevant docs because generators pay more attention to top-of-context. Context Recall asks whether retrieved context covers *all* the facts needed to answer completely. These are sharper versions of basic Precision@k and Recall@k for mature pipelines.

5. **The 2x2 diagnostic matrix** — Every failing query falls into Good-R+Bad-G (generation bottleneck), Bad-R+Good-G (retrieval bottleneck), or Bad-R+Bad-G (both broken). The matrix routes failures to the right team so they stop arguing about whose problem it is.

### Exercise Answers (from rag-eval-dataset.csv)

**Step 1 — Retrieval metrics:**
- Avg Precision@5 ≈ 0.14 (very low — on average less than 1 of 5 retrieved docs is relevant; the retriever is mostly fetching noise)
- Avg Recall@5 ≈ 0.67 (decent — when retrieval works, it usually gets the right docs)
- Combined story: the retriever is "hit or miss" — when it finds the relevant doc it also grabs a lot of noise, and on 6/18 cases it misses the relevant doc entirely. This is a retriever that needs precision work (better ranking, less noise) but its recall is acceptable for a baseline.

**Step 2 — Generation metrics:**
- Faithfulness rate: 72% (13/18 faithful) — 5 cases (Q-08, Q-09, Q-11, Q-15, Q-17) fabricate policy thresholds or claims not present in retrieved docs
- Answer relevance rate: 72% (13/18 relevant) — 5 cases (Q-07, Q-10, Q-12, Q-16, Q-18) are grounded but off-topic (talk about imagery when asked about preparation, cuisine when asked about allergens, etc.)
- The two failure modes are roughly balanced, and they overlap very little — the generator has both a hallucination problem AND a topical drift problem. Fixing one does not fix the other.

**Step 3 — The 2x2 matrix:**

|                        | Good G                | Bad G                 |
|------------------------|-----------------------|-----------------------|
| **Good R** (recall=1.0)| 6 (Q-01 to Q-06)      | 6 (Q-07 to Q-12)      |
| **Bad R** (recall<1.0) | 2 (Q-13, Q-14)        | 4 (Q-15 to Q-18)      |

- 6 working-baseline cases (Good R + Good G)
- 6 generation-bottleneck cases (Good R + Bad G) — retrieval delivered the right context but the generator still failed
- 2 retrieval-bottleneck cases (Bad R + Good G) — the generator was faithful to what it received, but the relevant policy wasn't retrieved
- 4 both-broken cases (Bad R + Bad G) — need fixes on both sides

**Step 4 — Diagnose:**
- Overall correctness: 6/18 = 33% — much lower than any individual stage metric (retrieval P@5=14%, R@5=67%; generation F=72%, AR=72%)
- The gap between stage-level metrics and end-to-end correctness is the story: "each stage is mediocre" compounds into "system is broken"
- The bigger bottleneck is **generation**: 10/12 wrong cases involve bad generation (Q-07 to Q-12 and Q-15 to Q-18), vs. 6/12 that involve bad retrieval (Q-13 to Q-18)
- Generation should get investment first — even when retrieval delivers the right context, the generator fails 50% of the time (6 of 12 Good-R cases are wrong)

### PM Decision Point Evaluation

**Strong response:**
- **Decision: Fund the LLM team (generation fix)**
- **Math:** Fixing generation eliminates Q-07 to Q-12 (6 cases) → new correctness = 12/18 = 67%. Fixing retrieval only eliminates Q-13 to Q-14 (2 cases) → new correctness = 8/18 = 44%. Generation fix is 3x the impact.
- **Names what remains wrong after each fix:** After generation fix, the 6 Bad-R cases (Q-13 to Q-18) remain wrong. After retrieval fix, the 10 Bad-G cases (Q-07 to Q-12 and Q-15 to Q-18) remain wrong.
- **Identifies the "need both" cell:** Q-15 to Q-18 (4 cases) have both bad retrieval and bad generation. Neither single fix touches them. These cases need a second sprint regardless of which fix ships first.
- **Connects to the matrix logic:** Mentions that the matrix is what enables this routing decision — without separating metrics you'd just see "67% wrong" with no way to pick a fix.

**Acceptable alternative:**
- Recommends generation first but acknowledges that retrieval-only cases (Q-13, Q-14) involve the retriever missing the relevant doc entirely, which is a more fundamental reliability issue — so a sequenced plan (generation this sprint, retrieval next) is fine as long as the priority order is justified with numbers.

**Weak response:**
- Picks based on which engineer is more confident, not on the 2x2 data
- Says "fix both" without acknowledging the one-initiative budget constraint
- Cites aggregate metrics ("recall is 67%, faithfulness is 72%") without translating them into "number of wrong cases this fix would eliminate"
- Picks retrieval because "missing context is worse than hallucination" — a reasonable instinct, but here the data says otherwise; the learner should let the numbers override the prior
- Misses that Q-15 to Q-18 need both fixes — either fix alone leaves 4 cases wrong
- Doesn't acknowledge that fixing generation here is unusually attractive because retrieval's Recall@5 is already decent (67%) — in a system with 20% recall the answer would flip

---

## D9: Hallucination Detection

### Concept Checks

The learner should be able to explain:

1. **What hallucination is** — Not every wrong answer is a hallucination. Hallucination is specifically when the model generates content not grounded in its source material — inventing facts, citing nonexistent rules, or stating things that contradict its context. A reasoning error is not a hallucination; a fabricated threshold is.

2. **Intrinsic vs extrinsic** — Intrinsic contradicts the source (changes a threshold, alters a rule). Extrinsic adds claims not present in the source (invents a requirement, fabricates an exception). Different risk profiles: intrinsic directly changes decisions, extrinsic is often invisible because the decision may still be correct.

3. **Why hallucination is hard to detect** — Hallucinated outputs are confident, fluent, and specific. They look identical to correct outputs. You cannot detect them from the output alone — only by comparing claims to the source material. No shortcut exists.

4. **Detection strategies** — Claim decomposition + source verification (most thorough, most expensive), citation checking (narrower, partially automatable), self-consistency (cheap, misses consistent hallucinations), entailment verification (automated but introduces its own failure mode). Layer them like graders in D5.

5. **Correct decisions with hallucinated reasoning** — A system can produce the right approve/reject decision based on fabricated reasoning. This matters for three reasons: liability (citing nonexistent policies in audit trails), latent failure (fabricated rules that are harmless on this input but cause wrong decisions on others), and false confidence (decision accuracy metrics hide the hallucination rate).

### Exercise Answers (from hallucination-detection-dataset.csv)

**Step 1 — Classifications:**

| Case | Classification | What's wrong |
|------|---------------|-------------|
| H-01 | Grounded | Accurately states 15% threshold |
| H-02 | Grounded | Accurately requires certified lab testing |
| H-03 | Grounded | Accurately states 12-month certification window |
| H-04 | Grounded | Accurately states 80% threshold |
| H-05 | Grounded | Accurately states 30% rejection threshold |
| H-06 | Grounded | Accurately states stock photography prohibition |
| H-07 | Grounded | Accurately states credentials/training requirement |
| H-08 | Intrinsic | States "up to 20%" — policy says 15% |
| H-09 | Extrinsic | Adds "medical professional documentation" as accepted evidence — policy only mentions lab testing |
| H-10 | Intrinsic | States "at least 50%" — policy says 80% |
| H-11 | Extrinsic | Adds "vendor attestation" as accepted — policy requires third-party cert only |
| H-12 | Extrinsic | Fabricates "body of water and fishing method" documentation requirement |
| H-13 | Extrinsic | Fabricates "3 price increases per 6 months" vendor limit |
| H-14 | Extrinsic | Fabricates "minimum 225°F for 4 hours" temperature/duration requirement |
| H-15 | Extrinsic | Fabricates "cultural heritage exception" for traditional methods |

**Step 2 — Hallucination rate:**
- Hallucination rate: 8/15 = 53%
- Decision correctness: 10/15 = 67%
- The gap: 14 percentage points. Correctness looks tolerable; hallucination rate tells a different story.

**Step 3 — The dangerous overlap:**
- Hallucinated + correct: 3 cases (H-12, H-13, H-14) — all extrinsic hallucinations where the fabricated requirement didn't change the outcome
- Hallucinated + wrong: 5 cases (H-08, H-09, H-10, H-11, H-15)
- Grounded + wrong: 0 cases
- Key insight: if you only check decision correctness, you'd miss 3 out of 8 hallucinations (38%). These are invisible to accuracy-based evaluation. You'd need to check reasoning against source to catch them.

**Step 4 — Patterns:**
- **Intrinsic** (2 cases: H-08, H-10): 100% caused wrong decisions. These distort actual thresholds and directly flip the decision. Detection priority: high and immediate.
- **Extrinsic** (6 cases: H-09, H-11, H-12, H-13, H-14, H-15): 50% caused wrong decisions (H-09, H-11, H-15). The other 50% (H-12, H-13, H-14) had correct decisions despite fabricated claims — invisible to accuracy checks. H-15 shows the risk: when a fabricated exception overrides a real threshold, the decision flips. Detection priority: high, because the damage is latent.
- **Which detection strategy fits:** Citation checking (comparing cited policy to actual policy text) would catch both types. Self-consistency might catch intrinsic hallucinations (threshold would vary between runs) but likely miss extrinsic (the model may consistently fabricate the same extra requirements).

### PM Decision Point Evaluation

**Strong response:**
- Disagrees with the engineering lead — hallucination in reasoning matters independently of decision accuracy
- Cites specific numbers: 53% hallucination rate, but only 67% decision correctness — 3 hallucinated cases (H-12, H-13, H-14) are invisible to decision checks
- Names the three risks: liability (citing nonexistent policies), latent failure (fabricated rules can flip decisions on different inputs), false confidence (accuracy metrics mask reasoning problems)
- Recommends investing in hallucination detection AND reasoning quality — not instead of decision accuracy, but as a separate evaluation dimension
- Proposes a concrete next step: citation checking (compare reasoning to policy text) as a grader, since the exercise just demonstrated it works
- Acknowledges the engineering lead's cost concern and suggests starting with automated citation checking rather than expensive manual review

**Acceptable alternative:**
- Agrees that decision accuracy is the primary metric but argues for adding hallucination rate as a secondary metric — a "guardrail metric" that blocks shipping even when accuracy looks acceptable
- Proposes a phased approach: fix decision accuracy first, then add hallucination detection

**Weak response:**
- Agrees with the engineering lead that reasoning doesn't matter as long as decisions are right — misses the liability, latent failure, and false confidence risks
- Cites hallucination rate without connecting it to the specific invisible cases (H-12, H-13, H-14)
- Says "hallucination is bad" without explaining WHY correct-decision hallucinations are dangerous
- Proposes fixing hallucination without naming a concrete detection strategy
- Doesn't reference the intrinsic vs extrinsic pattern — treating all hallucinations as equally dangerous misses that intrinsic always causes wrong decisions while extrinsic is a latent risk

---

## D10: Blocking Metrics and Release Criteria

### Concept Checks

The learner should be able to explain:

1. **Guardrail vs optimization metrics** — Guardrails are binary gates with hard thresholds that block release if failed. Optimization metrics are continuous — you want improvement but a regression doesn't automatically block. The PM skill is knowing which metrics belong in which category.

2. **Why you need both types** — Guardrail-only = ship anything that clears safety bar even if quality regressed. Optimization-only = might ship something improving on average but with a critical safety hole. You need guardrails for "is it safe to ship?" and optimization metrics for "is it worth shipping?"

3. **Where thresholds come from** — Regulatory requirements, business requirements (cost-of-failure math), SLA commitments, baseline + margin. A threshold without a rationale is a guess. When someone proposes a threshold, ask "where does that number come from?"

4. **The ship/hold framework** — All guardrails must pass → if any fails, hold. Assess optimization metrics for net improvement. Look for hidden trade-offs (one metric improving at the expense of another). Guardrails pass + net positive = ship. Any guardrail fails = hold.

5. **Threshold evolution** — Thresholds ratchet up as the system matures. Optimization metrics can graduate to guardrails. Watch for metric gaming (optimizing one metric by sacrificing another).

### Exercise Answers (from release-criteria-dataset.csv)

**Step 1 — Classification:**
- Guardrail metrics (5): M-01 allergen safety (≥99.0%), M-02 false approval rate (≤2.0%), M-03 system availability (≥99.5%), M-04 response latency (≤800ms), M-05 hallucination rate (≤15%)
- Optimization metrics (7): M-06 overall accuracy, M-07 description accuracy, M-08 price accuracy, M-09 false rejection rate, M-10 cost per verification, M-11 escalation rate, M-12 vendor satisfaction

**Step 2 — Guardrail check:**
- M-01 allergen safety: 98.1% vs ≥99.0% → **FAILS** (1.1pp below threshold)
- M-02 false approval rate: 1.5% vs ≤2.0% → PASSES
- M-03 system availability: 99.8% vs ≥99.5% → PASSES
- M-04 response latency: 520ms vs ≤800ms → PASSES
- M-05 hallucination rate: 8% vs ≤15% → PASSES
- **Result: 1 guardrail failure (M-01)**

**Step 3 — Optimization assessment:**
- Improved (5): M-06 accuracy 78→84%, M-07 description 71→82%, M-08 price 88→90%, M-09 false rejection 12→9%, M-12 satisfaction 3.8→4.1
- Regressed (2): M-10 cost $0.12→$0.15 (+25%), M-11 escalation 18→22% (+4pp)
- Trade-off pattern: M-02 (false approval rate) improved while M-11 (escalation rate) regressed. The model is likely more cautious — flagging more borderline cases for human review. This reduces false approvals but increases escalation costs. Similarly, cost increase (M-10) may reflect the model doing more work per verification, buying the accuracy improvements.

**Step 4 — Ship/hold:**
- **HOLD.** M-01 allergen safety fails its guardrail (98.1% < 99.0%). One guardrail failure blocks the release regardless of optimization gains.
- The optimization picture is strong — 5 of 7 metrics improved, and the 2 regressions are explainable as trade-offs. But this doesn't override the guardrail failure.

### PM Decision Point Evaluation

**Strong response:**
- **Decision: Hold.** Does not ship despite 9 of 12 metrics improving.
- **Explains why the allergen failure blocks release:** 98.1% vs 99.0% means roughly 1 in 50 allergen decisions that were correct in v2 is now wrong in v3. For safety-critical decisions with regulatory and legal liability, this is unacceptable.
- **Rejects "ship now, patch later":** Every week v3 is in production with this regression, real allergen decisions are wrong. "Fast-follow" assumes the fix is quick — but if it takes two sprints instead of one, you've been shipping a known safety regression.
- **Acknowledges the cost of holding:** The accuracy and hallucination improvements are real and valuable. Proposes a specific path forward: fix the allergen regression first, verify it passes the guardrail, then ship v3 with all its improvements intact.
- **Names the principle:** Guardrails are not negotiable. If you override a guardrail once because the other metrics look good, you've established that guardrails are actually just optimization metrics with different labels.

**Acceptable alternative:**
- Recommends a scoped/staged release: ship v3 for non-allergen change types only (where the guardrail doesn't apply), and hold allergen verification on v2 until the fix is ready. This is sophisticated — it captures the optimization gains where safe while protecting the safety-critical segment.

**Weak response:**
- Agrees to "ship now, patch later" — fails to understand that guardrails are non-negotiable
- Says "hold" but doesn't explain the specific impact of the allergen regression (1 in 50 decisions wrong)
- Focuses only on the allergen failure without acknowledging the real optimization gains — a hold recommendation should name what's being delayed and propose a timeline
- Doesn't address the escalation/cost trade-off — treats all regressions as equally concerning
- Says "all metrics must improve to ship" — overly rigid; optimization regressions are acceptable if the trade-off is justified

---

## D11: Metric Design and Cost-Aware Evaluation

### Concept Checks

The learner should be able to explain:

1. **What makes a metric useful** — specific (measures one thing), actionable (regression points to a team and fix type), connected to outcomes (tracks what actually matters). Signs of a bad metric: aggregates multiple dimensions, no clear owner for regressions, can be gamed without improving the system.

2. **The cost of evaluation** — model graders ~$0.01–$0.05/run, human review ~$2–$20/case. At 10,000 cases/week, human-at-100% costs ~$20,000/week vs. $200/week for a model grader. The PM question: minimum investment for sufficient signal. Not zero (blind) and not unlimited (unviable).

3. **Sampling strategies** — random (simple, unbiased, inefficient for rare failures), stratified (oversample high-risk segments), adversarial (stress-test with hard cases), production-triggered (expensive check only when a signal fires). Each trades cost against coverage.

4. **Tiered evaluation** — code (100%, $0), model (sampled, $0.01–$0.05), human (small subset, $2–$20). Push checks down the cost pyramid. Reserve human review for calibration and hard cases. Tiering vs. all-human can reduce costs by 90%+.

5. **Metric gaming** — when a metric becomes a target, teams optimize for it at the expense of what it was meant to measure. Defense: pair metrics covering opposing failure modes (hallucination rate + answer completeness; false approval rate + false rejection rate). For any guardrail, ask "how could a team hit this number without actually improving the system?"

### Exercise Answers (from metric-design-dataset.csv)

Weekly volume: 10,000 verifications

**Step 1 — Weekly costs:**
- Pipeline A: PC-03 ($0.02 × 10,000 = $200) + PC-04 ($0.02 × 10,000 = $200) + PC-05 ($2.00 × 500 = $1,000) = **$1,400/week**
- Pipeline B: PC-08 ($2.00 × 2,000 = $4,000) = **$4,000/week**
- Pipeline C: PC-11 ($0.02 × 2,500 = $50) + PC-12 ($2.00 × 500 = $1,000) = **$1,050/week**

**Step 2 — Coverage by failure type:**

Pipeline A:
- price_error: code 100% ✓ (300/300 cases/week)
- allergen_error: code 100% ✓ (200/200 cases/week)
- hallucination: model 100% ✓ (600/600 cases/week)
- policy_misapplication: model 100% ✓ (800/800 cases/week)
- context_miss: human 5% (partial — 400 × 5% = 20 cases/week detected, 380 undetected)

Pipeline B:
- price_error: code 100% ✓
- allergen_error: code 100% ✓
- hallucination: human 20% (120/600 cases/week detected, 480 undetected)
- policy_misapplication: human 20% (160/800 cases/week detected, 640 undetected)
- context_miss: human 20% (80/400 cases/week detected, 320 undetected)

Pipeline C:
- price_error: code 100% ✓
- allergen_error: code 100% ✓ + human spot check 5%
- hallucination: model 25% (150/600 cases/week detected, 450 undetected)
- policy_misapplication: **NO COVERAGE** (0/800 cases/week detected)
- context_miss: **NO COVERAGE** (0/400 cases/week detected)

**Step 3 — Coverage gaps:**
- Pipeline A: context_miss gap (380 cases/week undetected, ~95% miss rate). Only check is 5% human audit.
- Pipeline B: large gaps on model-checkable failures — 480 hallucinations, 640 policy misapplications, 320 context misses undetected per week. Only human review at 20% covers these.
- Pipeline C: complete blindspot on policy_misapplication (800/week) and context_miss (400/week). Total: 1,200 failure cases/week with zero detection.

**Step 4 — Recommendation:**
Pipeline A is the best value. At $1,400/week it provides automated coverage at 100% for all model-detectable failure types with a 5% human backstop. Pipeline B is worst value: 2.9× more expensive than A ($4,000 vs $1,400) with significantly lower coverage on non-code failures. Pipeline C is cheapest ($1,050) but leaves 1,200 failure cases/week completely uncovered — policy_misapplication (800/week, medium severity) and context_miss (400/week, high severity). Accept a hybrid of C + adding PC-04-style policy grader at reduced sampling (e.g., 50%) for ~$1,150/week as a reasonable alternative.

### PM Decision Point Evaluation

Scenario: cut Pipeline A spend by 40% (Pipeline A = $1,400/week; target ≤ $840/week; save ≥ $560).

**Strong response:**
- Calculates cut options with numbers, not just descriptions
- Proposes reducing model sampling rates rather than eliminating checks entirely (preserves some coverage vs. total blindspot)
- Explicitly names coverage impact per cut: "reducing model checks to 50% means ~300 hallucinations and ~400 policy failures/week go undetected instead of zero"
- Does not cut allergen-related checks (critical severity) — code check PC-02 is $0 so never cut; human audit PC-05 is the safety backstop
- Reaches close to 40% with a coherent cut plan, e.g.:
  - Reduce PC-03 (hallucination grader) 100% → 50%: saves $100
  - Reduce PC-04 (policy grader) 100% → 50%: saves $100
  - Reduce PC-05 (human review) 5% → 3%: saves $400 ($1,000 → $600)
  - Total saved: $600 (43% cut); new cost: $800/week ✓

**Weak response:**
- Proposes cuts without calculating which checks account for what cost
- Eliminates human review entirely to save $1,000 — overshoots target and destroys the only check for context_miss
- Reduces only model checks without noting that model checks ($400) are cheaper than human review ($1,000) — wrong lever for getting to 40%
- Doesn't name the coverage being sacrificed — just says "we'll sample less"
- Treats all checks as equally cuttable regardless of failure severity

---

## D12: Fairness, Bias, and Subgroup Evaluation

### Concept Checks

1. **Aggregate hides subgroup failures** — A headline accuracy number tells you what happens on average, not to any specific user. An 85% aggregate can hide a subgroup at 55%.

2. **What to slice by** — Three categories: functional (input type, length, complexity), user-relevant (segment, market, tier), protected (race, gender, age, religion, disability). User-relevant attributes often serve as proxies for protected ones when direct data is unavailable.

3. **Three fairness definitions** — Demographic parity (equal outcome rates), equal opportunity (equal TPR), equalized odds (equal TPR and FPR). They can conflict. The PM picks based on context: safety-critical defaults to equal opportunity; legally-scrutinized allocation may require demographic parity.

4. **Intersectional bias** — A and B each look fine at 85%, but their intersection A∩B is 55%. Single-dimension slicing doesn't reveal this. Must slice by combinations where business intuition suggests risk; accept thin cells as signals, not firm conclusions.

5. **Mitigation strategies** — Data rebalancing (long-term), threshold tuning (cheap but legally risky), post-processing overrides, human review routing (expensive but immediate). Short-term mitigation plus long-term fix; short-term is a bandage, not a cure.

### Exercise Answers (from fairness-subgroup-dataset.csv)

**Step 1 — Aggregate accuracy:** 35/45 = 77.8%

**Step 2 — Disaggregate:**

By cuisine:
- Italian: 8/9 = 88.9%
- American: 9/9 = 100%
- Mexican: 8/9 = 88.9%
- Indian: 6/9 = 66.7%
- Thai: 4/9 = 44.4%
- Gap: American (100%) to Thai (44.4%) = 55.6 percentage points

By restaurant size:
- Small: 9/15 = 60.0%
- Medium: 12/15 = 80.0%
- Large: 14/15 = 93.3%
- Gap: Large (93.3%) to Small (60.0%) = 33.3 percentage points

**Step 3 — Intersectional (cuisine × size):**

Worst cells:
- Thai × Small: 1/3 = 33.3%
- Thai × Medium: 1/3 = 33.3%
- Indian × Small: 1/3 = 33.3%
- Thai × Large: 2/3 = 66.7%
- Indian × Medium: 2/3 = 66.7%

Best cells: American (all three sizes) and Italian/Mexican medium & large = 100%.

Error pattern: all 10 errors are **false rejections** (ground_truth = approve, system_decision = reject). The system is over-cautious for unfamiliar cuisines, especially in small restaurants — it rejects legitimate menu changes rather than being split between false approvals and rejections. This is an **equal opportunity** violation: TPR (approval rate among cases that should be approved) differs dramatically across cuisine subgroups.

### PM Decision Point Evaluation

Context: Coalition threatens public complaint in two weeks. Analysis confirms Thai at 44% vs American at 100%; small restaurants 60% vs large 93%; intersection cells as low as 33%; all errors are false rejections.

**Strong response:**
- Leads with specific numbers: "Thai restaurants see 44% accuracy vs 100% for American — a 56-point gap, driven entirely by false rejections."
- Names the fairness definition being violated: equal opportunity (equivalent legitimate cases are being rejected at different rates by cuisine).
- Two-week plan is a concrete stopgap: route all small-restaurant and/or Thai/Indian menu changes to human review. Cites that this is a bandage, not a cure.
- Explicitly rejects threshold tuning on a protected-proxy attribute (cuisine) without legal review.
- Long-term plan names data rebalancing (more Thai/Indian training and eval data) AND root-cause investigation of why the model over-rejects unfamiliar cuisines (e.g., tokenization, allergen mapping, prior complaints).
- Sets a measurable success criterion: e.g., close cuisine gap to <10 points within N sprints.
- Does not promise what can't be delivered in two weeks.

**Weak response:**
- Vague language ("we'll look into it," "improve fairness") without numbers.
- Proposes threshold tuning as the primary fix with no mention of legal review.
- Treats human review as the permanent solution rather than a stopgap.
- Ignores the intersectional cells (Thai+Small, Indian+Small) — only addresses cuisine or only addresses size.
- Doesn't distinguish the false-rejection error pattern from a generic "accuracy gap" — misses that this is specifically equal-opportunity harm.
- Commits to a fix within two weeks that can't realistically ship (e.g., "we'll retrain the model" — not achievable in that window).

---

## D13: Eval-Driven Development

### Concept Checks

1. **Evals-first, not evals-after** — Writing the eval before the code inverts the "ship then test" default. The eval becomes the spec. If you can't write an eval, you don't understand the feature well enough to ship.

2. **Evals as specifications** — "Polite and accurate" is not a spec; it's a wish. An eval is something measurable. PM-engineering agreement on the eval = agreement on what "working" means. Disagreement surfaces upfront, not during QA.

3. **Tight feedback loops** — The faster the eval loop, the smaller the blast radius of a regression. Fast imperfect evals beat slow perfect evals because they actually get run.

4. **Regression suites never retire** — Every bug fixed gets a permanent eval. Without this, teams re-litigate fixed bugs. The suite grows with every bug; it never shrinks. Watch for two failure modes: chronic failures (eval works but nobody's fixing) and regressions (new breakage).

5. **Eval-driven prioritization** — Treat the eval dashboard as a backlog. Prioritize by severity, pattern (regressed vs chronic), and coverage (critical areas under-evaluated).

### Exercise Answers

**Step 1 — Eval specs for menu photo verification:**

Strong answers have specific, measurable pass criteria and severity that matches the stakes.

*Scenario A — matching photo (pasta carbonara photo for pasta carbonara item):*
- Input: menu item description text + uploaded photo
- Expected output: approve (or match=true with high confidence)
- Pass criteria: system outputs match=true with confidence ≥ 0.80; OR decision=approve with no mismatch flag
- Failure mode area: image_matching (or image_description_alignment)
- Severity: medium — approving correct matches is the happy path; false rejection causes UX friction but no safety harm
- Check type: model-based (vision understanding required)

*Scenario B — mismatched photo (salad photo for pasta carbonara):*
- Input: menu item description text + uploaded photo of a different dish
- Expected output: reject OR route to human review with mismatch flag
- Pass criteria: system outputs match=false OR escalates with reason=photo_mismatch
- Failure mode area: image_matching
- Severity: medium-high — misleading customers; potential complaint/refund; brand trust impact
- Check type: model-based

*Scenario C — undisclosed allergen (shellfish visible in "vegan" dish):*
- Input: menu item description claiming "vegan" + photo showing shellfish
- Expected output: reject with allergen/policy violation flag, route to urgent human review
- Pass criteria: system detects the allergen AND flags policy violation AND blocks approval
- Failure mode area: allergen_safety (or content_policy)
- Severity: **critical** — allergen mismatch can cause anaphylaxis; legal liability; safety-of-life
- Check type: model-based (vision) + code (rule enforcement once flagged)

**Strong response:**
- Pass criteria are concrete and measurable (thresholds, enums, specific flags) — not "system should work"
- Severity calibrated to stakes (C is critical, A is medium)
- Correctly identifies scenario C as needing layered check types (vision model + code enforcement)
- Recognizes that writing these evals surfaces questions engineering must answer (what's the confidence threshold? what does "vegan" include?)

**Weak response:**
- "System should know the photo matches" — unmeasurable
- All three scenarios marked same severity (misses that C is critical)
- Confuses scenario B and C severities — treats mismatch and safety the same
- Marks all as code checks (doesn't recognize vision requires a model)
- No concrete thresholds or decision criteria

**Step 2 — Audit the existing suite:**

| Category | Count | Cases |
|---|---|---|
| Stable pass | 10 | E-01, E-02, E-03, E-07, E-08, E-09, E-14, E-17, E-19, E-24 |
| Chronic fail | 3 | E-13, E-18, E-22 |
| Regressed | 5 | E-06, E-10, E-15, E-20, E-23 |
| Improving | 3 | E-05, E-12, E-21 |
| Flaky | 4 | E-04, E-11, E-16, E-25 |

Highest-risk cases (regressed or chronic at high/critical severity):

| Case | Area | Severity | Pattern | Why it matters |
|---|---|---|---|---|
| E-10 | allergen_safety | critical | regressed | Actively shipping a worse system on a critical safety dimension |
| E-23 | context_retrieval | high | regressed | Recent breakage in context retrieval; contaminates downstream decisions |
| E-15 | hallucination | high | regressed | Hallucination failures ship to production |
| E-20 | policy_application | high | regressed | Recent policy application breakage |
| E-13 | hallucination | high | chronic | High-severity bug, 6 sprints unaddressed |
| E-18 | policy_application | high | chronic | High-severity bug, 6 sprints unaddressed |

Regressions typically take priority over chronic failures within a sprint — they represent recent breakage, not long-accepted tech debt.

**Step 3 — Coverage by area:**

| Area | Cases | Severity mix | Assessment |
|---|---|---|---|
| price_validation | 8 | mostly low/medium | Over-covered relative to severity |
| allergen_safety | 3 | 3× critical | **Under-covered for critical area** |
| hallucination | 5 | 3 high, 2 medium | Reasonable |
| policy_application | 6 | 2 high, 4 medium | Reasonable |
| context_retrieval | 3 | 2 high, 1 medium | **Thin coverage for high-severity area** |

Coverage gap: allergen_safety is the most critical area but has the fewest cases. A suite that has 8 price_validation evals and only 3 allergen_safety evals is misaligned with risk. Context_retrieval is similarly thin given its severity mix.

### PM Decision Point Evaluation

Constraint: 3 engineers × 2 weeks. Cannot do everything from the audit.

**Strong response:**
- Top priority: **E-10 (critical allergen regression).** Non-negotiable. A regressed critical-severity safety case is the single highest-risk item in the suite.
- Next: **E-23 and E-15 (high-severity regressions)** in context_retrieval and hallucination. Regressions beat chronic failures for this sprint because they represent active degradation from a recent change.
- Adds **1–2 new allergen_safety evals** to close the coverage gap — cheap to write, closes a known blind spot for critical risk. Can be done in parallel with remediation.
- Explicitly defers: flaky cases (E-04, E-11, E-16, E-25 — need root-cause investigation but not actively shipping damage), chronic policy failures (E-18, E-22 — tech debt, already shipped with), improving cases (E-05, E-12, E-21 — already being addressed).
- Does not defer E-10 or any critical regression.
- Ties choices back to severity × pattern, not "what's easiest."

**Weak response:**
- Treats all regressions equally regardless of severity (doesn't single out E-10).
- Prioritizes flaky cases because they "look unstable" — misses that flakiness at medium severity is lower priority than a critical regression.
- Tries to address all 5 regressions and all 3 chronic failures — exceeds 3-engineer capacity and ignores the deferral requirement.
- Ignores the allergen_safety coverage gap entirely.
- Defers E-10 to save effort or because "we'll catch it in QA" — unacceptable for a critical regression.
- Justifies choices with gut feel ("this felt important") instead of severity + pattern.

---

## D14: The Observability Landscape

### Concept Checks

1. **Observability vs evaluation** — Evaluation is pre-deployment, curated, gated; observability is post-deployment, live traffic, continuous. Different questions. Evals decide whether you ship; observability decides whether you keep running.

2. **Three pillars + one** — Metrics (aggregate health), logs (per-request detail), traces (per-request flow through pipeline stages), and human feedback (the "+1" — thumbs, tickets, escalations). Human feedback is slow and biased but it's the only signal directly tied to user outcomes.

3. **Leading vs lagging indicators** — Leading = warns before harm (confidence drop, escalation spike, input drift). Lagging = confirms harm (complaints, churn, thumbs-down). Weight alerting toward leading; use lagging for confirmation.

4. **Alerting is a discipline** — Not every metric alerts. Three-question test: SLO violation? Actionable now? Low FPR? Fail any test = dashboard-only. Alert fatigue teaches the on-call to ignore everything.

5. **Closing the loop to evaluation** — New failure modes found in production become eval cases. Observability → eval → regression suite → safer shipping. Teams that don't close this loop re-learn the same lessons repeatedly.

### Exercise Answers (from observability-signals-dataset.csv)

**Step 1 — Classify by pillar and by indicator type:**

By pillar:

| Pillar | Count | Signals |
|---|---|---|
| metric | 11 | S-01, S-02, S-03, S-04, S-05, S-06, S-08, S-11, S-13, S-14, S-15 |
| log | 1 | S-09 |
| trace | 1 | S-10 |
| human_feedback | 2 | S-07, S-12 |

The dashboard is heavily metric-tilted (11/15). Logs and traces have one signal each — thin coverage for debugging when something breaks. Human feedback has only 2 signals, which is a gap given it's the only direct measure of user outcomes.

By indicator type:

| Type | Count | Signals |
|---|---|---|
| Leading | 4 | S-01, S-05, S-06, S-13 |
| Lagging | 7 | S-02, S-04, S-07, S-08, S-11, S-12, S-14 |
| Neutral | 4 | S-03, S-09, S-10, S-15 |

The ratio skews lagging. Only 4 of 15 signals warn the team before user harm. A strong dashboard would invest more in leading indicators and rebalance pillar coverage (more traces for debuggability, more human feedback for ground truth).

**Step 2 — Alert-worthy vs dashboard-only:**

Alert-worthy (SLO violation, actionable, low FPR):
- S-02 p95 latency — SLO violation
- S-04 error_rate_5xx — SLO violation
- S-08 allergen_false_approval_count — critical safety SLO; any nonzero value pages
- S-06 human_escalation_rate — actionable spike indicates upstream issue
- S-01 approval_rate_by_cuisine — alerts when subgroup gap exceeds threshold

Dashboard-only (useful to track but not actionable on page, or high FPR):
- S-03 requests_per_hour — context, not action
- S-05 avg_llm_confidence_score — too noisy to page; watch for trend
- S-07 complaint tickets — too slow for alerts; trend-watch
- S-09 per_request_log — debugging resource
- S-10 traces — debugging resource
- S-11 cost_per_request — track trends
- S-12 thumbs_down_rate — noisy, lagging
- S-13 KL divergence — high FPR, investigation-only
- S-14 churn — way too slow/far from AI to alert
- S-15 model_version — context for correlation

**Step 3 — Coverage gaps (not in the 15):**

- **No hallucination-specific signal** — despite D9. No detector for factually-wrong reasoning attached to a correct-looking decision.
- **No approval_rate_by_restaurant_size** — D12 showed size matters; only cuisine is sliced here. Small restaurants could be getting underserved undetected.
- **No prompt injection / adversarial input counter** — zero visibility into malicious inputs.
- **No intersectional subgroup slicing** — D12 showed intersections (Thai × Small) are worse than either dimension. Single-dimension-only signals miss this.
- **No model disagreement signal** — when ensembles or cross-checks are available, disagreement is a high-value leading indicator.

**Step 4 — Close the loop back to evaluation:**

Scenario: S-08 fires for the first time. A menu change was approved last week; a customer reported an allergic reaction; auditors confirmed an undisclosed allergen was present. The eval case added to the regression suite should replay that specific failure mode so it cannot recur silently.

Strong eval spec (following D13 format):

- **Input:** A menu item described as "shrimp pad thai" with ingredient list showing "peanuts" but no allergen disclosure field set — OR a generalized version: menu changes where the ingredient text contains a common allergen (nuts, shellfish, dairy, soy, gluten) but the `allergen_disclosure` field is empty or inconsistent with the ingredient text.
- **Expected output:** reject with reason=undisclosed_allergen, OR route to human review with urgency=critical.
- **Pass criteria:** system flags the allergen mismatch AND does not auto-approve; specifically, `decision != approve` AND `flags` includes `allergen_disclosure_mismatch` (or equivalent).
- **Failure mode area:** allergen_safety
- **Severity:** critical

**Strong response:**
- Grounds the eval in the specific incident that triggered the signal (not a generic "check allergens" test)
- Pass criteria are concrete and measurable (specific flags / decision enums)
- Severity marked critical (safety-of-life)
- Correctly categorizes as allergen_safety
- Recognizes that the eval generalizes the incident — it should catch this class of failures, not just the one instance

**Weak response:**
- Generic eval ("system should handle allergens correctly") — not measurable
- Severity marked medium or high (underweights safety-of-life)
- Doesn't tie the eval back to the specific signal/incident — treats it as a separate exercise
- Pass criteria vague ("system knows it has an allergen") with no decision threshold
- Writes the eval as an obs signal again instead of a pre-deployment check — misses the point that observability catches and evaluation prevents

### PM Decision Point Evaluation

Constraint: MVP = 6 signals, must cover the big failure modes without overspending, must flag what's deferred and what v2 must add.

**Strong response:**

MVP picks (representative of a strong answer):
- **S-02 p95_decision_latency** — SLO, alerts.
- **S-04 error_rate_5xx** — SLO, alerts.
- **S-08 allergen_false_approval_count** — critical safety SLO, alerts.
- **S-01 approval_rate_by_cuisine** — leading fairness signal; alerts on gap threshold.
- **S-05 avg_llm_confidence_score** — leading quality signal; dashboard (noisy for alerts).
- **S-06 human_escalation_rate** — leading operational signal; alerts on spike.

Deferred with reasoning:
- **S-10 traces, S-13 KL divergence** — high implementation cost, debug-only or investigation-only; add in v2 when incidents demand them.
- **S-09 per_request_log** — foundational but generates volume; can ship with a sampling-based version later.
- **S-14 churn** — too lagging and too far from AI to be actionable; belongs on a business dashboard, not this one.
- **S-03, S-11, S-15** — context signals; useful but not MVP-critical.
- **S-07 complaint tickets, S-12 thumbs_down** — lagging; can be picked up from existing support/feedback systems without the AI dashboard owning them.

v2 gaps explicitly named (must include at least one):
- Hallucination detection signal
- approval_rate_by_restaurant_size (D12 lesson)
- Intersectional subgroup slicing
- Adversarial input counter

**Weak response:**
- Picks 6 all-lagging signals — dashboard is blind to issues until after harm.
- Includes S-14 churn in MVP — too slow and too indirect.
- Skips S-08 allergen signal — critical safety miss.
- Doesn't flag the hallucination or subgroup-size gaps.
- Alerts on everything picked — ignores the alert-fatigue discipline.
- Justifies with "feels important" instead of indicator type × cost × alert-worthiness.

---

## D15: Evaluating AI Agents

### Exercise Evaluation

**Step 1 — Outcome accuracy vs trajectory quality:**

Outcome accuracy: 20/25 = **80%** correct.

Good trajectory (efficient + recovered_from_error): 10 + 5 = **15/25 = 60%**.

Bad trajectory (inefficient + cascade_failure + hallucinated_tool_call): 5 + 3 + 2 = **10/25 = 40%**.

**Lucky correctness** — outcome correct but trajectory bad:
- R-15, R-16, R-17 (inefficient + correct) = 3
- R-18 (cascade_failure + correct) = 1
- R-19, R-20 (hallucinated_tool_call + correct) = 2
- **Total: 6 of 20 correct outcomes (30%) came through a bad trajectory.**

Interpretation: outcome-only evaluation would report 80% success, hiding that 30% of those "successes" were lucky. R-19 and R-20 are especially dangerous — the agent fabricated tool calls and got the right answer by coincidence.

**Strong response:** names the 6 lucky-correctness runs, calls out the 20-point gap between outcome (80%) and trajectory (60%), flags the hallucinated_tool_call runs as the highest-risk subset.

**Weak response:** stops at outcome accuracy; conflates "right answer" with "good process"; treats 80% as sufficient evidence to ship.

**Step 2 — Dominant trajectory failure mode:**

Bad trajectory breakdown (10 runs):
- inefficient: 5 (R-15, R-16, R-17, R-22, R-23)
- cascade_failure: 3 (R-18, R-24, R-25)
- hallucinated_tool_call: 2 (R-19, R-20)

Inefficient is the most common. But cross-referencing `wrong_tool_calls`: cascade_failure runs average 3.3 wrong tool calls and 0% recovery; hallucinated runs average 1.5 wrong tool calls with no recovery data since tools never really ran.

**Strong response:** separates frequency (inefficient is most common) from severity (cascade_failure is most damaging per-incident because no recovery + high wrong_tool_calls). Notes that remediation differs: inefficient = planning prompt fix; cascade = error-handling logic; hallucinated = stricter tool-call validation.

**Weak response:** picks the most frequent failure without considering severity; recommends a single fix for all three.

**Step 3 — Step efficiency:**

step_ratio distribution: median ≈ **1.2x**. Runs with ratio ≥ 2x:
- R-15 (2.2), R-16 (2.25), R-17 (2.33), R-18 (2.67), R-22 (3.0), R-23 (2.0), R-24 (2.0), R-25 (3.33) = **8 runs (32%)**

By task_type (bloat rate ≥ 2x):
- description_update: 2/4 = 50%
- photo_verification: 3/7 = 43%
- new_menu_upload: 2/8 = 25%
- allergen_dispute: 1/6 = 17%

**Strong response:** identifies description_update and photo_verification as the step-bloat hot spots. Notes that 8 of 25 runs take ≥ 2x expected steps — at GA volume, that's material cost.

**Weak response:** reports only a single aggregate ratio; doesn't slice by task_type; treats efficiency as an optimization concern rather than a guardrail.

**Step 4 — Eval spec closing the loop (hallucinated_tool_call):**

Target the R-19 / R-20 failure mode: agent emits a tool call in its log that was never actually made, or fabricates a tool result that wasn't returned by the tool.

**Strong eval spec (D13 format):**

- **Input:** An agent run for a menu verification task (e.g., allergen_dispute) where the tool log includes at least one call that cannot be reconciled with actual executed tool invocations recorded by the tool-call infrastructure.
- **Expected output:** The trajectory evaluator flags the run as `hallucinated_tool_call` and blocks the final output from being treated as trusted.
- **Pass criteria:** For every tool_call in the agent's reasoning trace, there must be a matching entry in the tool execution log (1:1 correspondence by call_id and timestamp). Pass = all tool_calls reconciled; fail = any unmatched tool_call.
- **Failure mode area:** trajectory_integrity (or agent_hallucination)
- **Severity:** critical — fabricated scaffolding in safety-critical domains (allergen handling) can produce confidently wrong decisions that pass outcome checks but fail reality.

**Strong response:**
- Pass criteria measure the trajectory (tool-call log reconciliation), not just the final decision
- Severity is critical — correctly recognizes that correct-looking-but-fabricated is worse than visibly wrong
- Recognizes this is a new class of check that single-shot evals never had to include
- Generalizes from R-19/R-20 to a pattern, not a one-off regression test

**Weak response:**
- Writes an outcome-only eval ("final decision should match ground truth") — misses the whole point, since R-19/R-20 have correct outcomes
- Sets severity medium or high (underweights safety-of-life combined with silent failure)
- Treats the eval as a post-hoc check rather than a pre-deployment gate
- Doesn't specify how hallucinated tool calls are detected (no reconciliation rule)

### PM Decision Point Evaluation

Constraint: engineering lead wants to ship on 80% outcome accuracy. Learner must use their analysis to justify push-back or approval.

**Strong response:**

Holds ship. Cites:
- Outcome 80% but trajectory only 60% — 20-point gap
- 6 lucky-correctness runs — 30% of "successful" runs came through a bad path
- 2 hallucinated_tool_call runs (R-19, R-20) — critical severity, silent failure mode
- 8 of 25 runs (32%) exhibit step bloat ≥ 2x — GA-scale cost/latency implications
- Dominant failure mode: inefficient (5 runs) for cost, cascade_failure (3 runs) for correctness risk

Specifies pre-GA investments:
- Tool-call reconciliation check (catches hallucinated_tool_call before production)
- Error-recovery logic for cascade_failure runs (detect → retry → escalate, not cascade)
- Step budget per task_type with alerts when exceeded
- Add trajectory_quality as a ship-gating metric alongside outcome_correct

Names what to defer: efficiency fixes for inefficient runs that still reach correct outcomes (optimization, not blocker); allergen_dispute remains at 17% bloat rate which is acceptable.

**Weak response:**
- Ships on 80% outcome accuracy without addressing trajectory
- Treats the hallucinated_tool_call runs as low-priority because outcomes were correct
- Defers all trajectory work to post-GA without naming which specific failures make that risky
- Doesn't distinguish cost-bloat (deferrable) from safety-risk (blocking)
- Doesn't propose trajectory_quality as a new gate — ships with outcome-only criteria

---

## D16: What's Different About AI Experiments

### Exercise Evaluation

**Step 1 — Aggregate pass rates with CIs:**

- v1: 26/40 = **65%**. SEM = √(0.65 × 0.35 / 40) ≈ 0.075. 95% CI ≈ **65% ± 14.8pp** (range 50–80%).
- v2: 30/40 = **75%**. SEM = √(0.75 × 0.25 / 40) ≈ 0.068. 95% CI ≈ **75% ± 13.4pp** (range 62–88%).

CIs overlap heavily (v1 upper ≈ 80%, v2 lower ≈ 62%). From independent-comparison CIs alone, you cannot conclude v2 > v1. Engineering's "+10 points, ship it" is the precise failure mode the lesson warned about — point-estimate thinking that ignores noise.

**Strong response:** names the overlap, concludes aggregate comparison is inconclusive, flags that the real analysis has to come from pairing.

**Weak response:** reports point estimates only; doesn't compute or interpret CIs; accepts the +10pp as meaningful.

**Step 2 — Paired comparison:**

Four-way paired split:
- Both pass: 21
- Both fail: 5
- v1 pass, v2 fail (v2 regressed case): **5**
- v1 fail, v2 pass (v2 improved case): **9**

Net paired improvement: (9 − 5) / 40 = **+4 / 40 = +10pp**, same as the aggregate in magnitude, but the paired view shows:
- Only **14 of 40 cases (35%) are informative** — the rest agree between versions and cancel out.
- Within those 14: v2 wins 9, v1 wins 5. Close to 2:1 for v2 but built on a small disagreement base.
- Paired SEM ≈ √((9+5)/40²) ≈ 0.094, so 95% CI on the paired difference ≈ ±18pp — tighter than the naive independent CI but still large.

**Strong response:** identifies that the signal is in the 14 disagreeing cases, correctly characterizes the paired comparison as more powerful than independent, notes that 9 vs 5 on a base of 14 is suggestive but not overwhelming.

**Weak response:** treats the paired result as re-confirming the +10pp without noting that agreement cases are non-informative; doesn't compute or interpret the paired CI.

**Step 3 — Subgroup slicing:**

By cuisine:
- **Thai (n=15):** v1=6/15 (40%), v2=12/15 (80%). Improved=6, regressed=0. **v2 is a clear win for Thai.**
- **American (n=10):** v1=10/10 (100%), v2=6/10 (60%). Improved=0, regressed=**4**. **v2 is a 40-point regression for American cuisine.**
- **Indian (n=15):** v1=10/15 (67%), v2=12/15 (80%). Improved=3, regressed=1. Small net gain.

The aggregate +10pp is the net of a +40pp Thai improvement plus a −40pp American regression plus a small Indian gain. The mean is hiding a catastrophic subgroup regression.

**Strong response:** identifies the American regression as the story. Recognizes that this is exactly the D12/D10 guardrail-breach scenario — aggregate improvement + subgroup regression = do-not-ship. Connects to the primary-metric-plus-guardrails framework.

**Weak response:** reports subgroup numbers without flagging the American regression as blocking; treats the regression as a minor offset to the Thai gain.

**Step 4 — Sample size and clustering:**

- **Unique restaurants:** 12 (not 40). Cases cluster: R-10 has 5 cases, R-12 has 5 cases, R-01/R-05/R-07 have 3–4 each, etc. The 40 observations are not independent; effective independent-cluster count is closer to 12. Clustered standard errors would be roughly 1.5–2× the naive SEMs computed above, widening CIs further.
- **Power for American subgroup (n=10):** SEM at 50% baseline ≈ √(0.25/10) ≈ 0.158, so 95% CI ≈ ±31pp. The observed 40pp regression is larger than the detection floor, so it's plausibly real — but a smaller regression (e.g., 20pp) could not be distinguished from noise at this n.
- **Power for Thai (n=15):** ±25pp floor — the 40pp improvement is real; smaller ones wouldn't be.

**Strong response:** recognizes clustering (12 restaurants, not 40 cases); calculates or estimates the detection floor for the American subgroup (~30pp) and confirms the regression exceeds it; notes that the experiment cannot distinguish medium-sized subgroup effects from noise.

**Weak response:** doesn't notice restaurant clustering; assumes 40 = 40 independent observations; doesn't assess whether the American regression is large enough to be real vs noise.

### PM Decision Point Evaluation

**Strong response:**

1. **Ship decision:** Hold v2 (or ship v2 conditionally with Thai-only rollout). The aggregate improvement is driven by Thai and masks a critical American regression.
2. **Statistical issues named (at least two of):**
   - Aggregate CIs overlap → point-estimate comparison is inconclusive
   - Paired comparison is more powerful but only 14 cases are informative
   - Subgroup split reveals American −40pp regression, hidden in the +10pp aggregate
   - 40 cases cluster in 12 restaurants → effective n is lower than it looks
   - Detection floors for subgroups are ±25–30pp → experiment can only catch large effects
3. **Follow-up experiment design:**
   - Scale test set to 150–200 cases, stratified across cuisine and restaurant size with sufficient n per subgroup (~30+ each)
   - Control for restaurant clustering — limit cases per restaurant or use clustered standard errors
   - Pre-commit a primary metric (aggregate pass rate) AND guardrails (no subgroup may regress by more than X pp)
   - Pair all cases (v1 and v2 on same cases)
   - Define smallest-meaningful-effect before running and compute required n via power analysis

**Weak response:**
- Ships v2 because aggregate looks better
- Doesn't cite specific numbers — hand-waves about "statistical concerns"
- Follow-up experiment design copies the flawed one (similar n, no clustering plan, no guardrails)
- Treats the American regression as acceptable collateral damage for aggregate gain — misses the D10 guardrail-breach framing

---

## D17: Launch Readiness and Production Monitoring

### Exercise Evaluation

**Step 1 — Baseline and trend:**

Baseline (W1): LLM-judge 81%, human review 82%, p95 820ms, 0 allergen breaches.

Trend across W1→W6:
- Human review: 82 → 81 → 79 → 76 → 71 → **68** (14pp decline, accelerating after W3)
- LLM-judge: 81 → 80 → 79 → 78 → 77 → **76** (only 5pp decline — much milder than human)
- p95 latency: 820 → 830 → 850 → 1100 → 1350 → **1500ms** (sudden jump W3→W4, continues rising)
- Allergen breaches: 0 → 0 → 0 → 1 → 2 → **3** (appears W4, grows)

The **judge/human divergence** (was 1pp at W1, now 8pp at W6) is the most important signal. The LLM judge is masking the decline — either because it was calibrated on the original distribution and doesn't recognize the new cases, or because judge quality is itself drifting. Without the human review layer, the team would believe things are "only slightly off."

**Strong response:** identifies the judge/human divergence as a key signal, notes that the degradation accelerates W3→W4, connects this to the production-monitoring framing (you cannot trust a single automated quality signal — you need the meta-eval loop from D6 feeding back).

**Weak response:** reads the LLM-judge column only (76% still above 75% threshold — "looks fine"); ignores human review drift; misses the divergence story.

**Step 2 — Drift classification:**

All three drift types are present:

- **Input drift (W3 onward):** Thai volume grows from 20% → 38%. Mexican cuisine first appears W4. Korean W5. Vietnamese W6. `new_cuisines_seen` column confirms. The eval set did not cover these → model handles them poorly.
- **Output drift (W5):** `model_version` changes from v2.1 → v2.2. Notes confirm upstream LLM provider pushed update with no regression suite run before rolling it out.
- **Concept drift (W4):** Notes reference AB-1523 taking effect — a new state allergen disclosure law requiring sesame disclosure. The definition of "correctly disclosed" changed; menus that would have passed W3 now fail W4+.

**Critical point:** all three are happening simultaneously, which makes root-cause isolation hard. The W4→W5 quality drop could be attributed to any of: Mexican volume growth, allergen law, or model update. Without disaggregating, you'd miss two of three.

**Strong response:** identifies all three drift types with specific evidence; notes that multiple concurrent drifts make attribution harder and require disaggregated analysis (e.g., break out breach rate by cuisine, by week, by model version).

**Weak response:** identifies one drift type (usually input drift because the cuisine mix is most visually obvious); misses the concept drift in notes; misses the model version change; doesn't grapple with co-occurrence.

**Step 3 — Guardrail check:**

- **Allergen breaches (0/week):** breached W4 (1), W5 (2), W6 (3). **System has been out of compliance on its hardest guardrail for 3 weeks.**
- **Human review ≥ 75%:** breached W5 (71%) and W6 (68%). **Out of compliance for 2 weeks.**
- **p95 ≤ 1000ms:** breached W4 (1100ms), W5 (1350ms), W6 (1500ms). **Out of compliance for 3 weeks.**

All three pre-launch ship criteria are breached. The system would not pass its own launch readiness gate if re-evaluated today. This means it should not be serving production traffic in its current form.

**Strong response:** systematically checks each guardrail, counts weeks breached, concludes the system is shipping past its own ship criteria and has been for multiple weeks.

**Weak response:** checks only the most visible guardrail (latency or breaches); misses that aggregate quality is also breached; doesn't frame the situation as "out of compliance with our own criteria."

**Step 4 — Rollback criteria and action:**

Rollback triggers breached:
- **Safety trigger (2+ breaches in 4 weeks):** W4–W6 has 6 total breaches. Trigger breached W5 (when second breach occurred).
- **Quality trigger (human review −7pp for 2 weeks):** baseline 82%, 7pp threshold = 75%. W5 (71%) and W6 (68%) both breach. Trigger breached W6.
- **Performance trigger (p95 > SLO for > 2 weeks):** W4–W6 all breach. Trigger breached W6.

All three rollback triggers are breached by W6.

**Recommended action: Targeted fix (not full rollback).** Rationale:
- Full rollback (to v1) would revert the D16 American-cuisine fix that took effort to build, and v1 doesn't handle the new allergen law either — rollback doesn't solve the concept-drift problem.
- Hold steady is off the table — all three rollback triggers are breached.
- The right move is surgical: (a) rollback the v2.2 model update to v2.1 immediately (output-drift fix — this is a pure regression from an uncontrolled upstream change), (b) route Mexican/Korean/Vietnamese traffic to manual review (human-in-the-loop) until eval coverage is added, (c) update the golden dataset with AB-1523 compliance cases and re-run all evals (concept-drift fix), (d) re-expand only after all three ship criteria are re-hit on the updated dataset.

**Strong response:** applies all three triggers, concludes all breached by W6, proposes a targeted fix that maps each action to the specific drift type it addresses, states re-expansion criteria.

**Weak response:** picks "full rollback" without distinguishing which drift rolling back addresses; picks "hold steady" ignoring the breached triggers; doesn't specify re-expansion criteria.

### PM Decision Point Evaluation

**Strong response:**

1. **Action recommendation:** Targeted fix. Cites: v2.2 model update rolled back to v2.1 (W5 regression), Mexican/Korean/Vietnamese routed to human review until covered in eval set (W4+ input drift), golden dataset updated with AB-1523 cases and system re-evaluated (concept drift). Re-expansion only after human review ≥ 80%, 0 allergen breaches for 2 consecutive weeks, p95 < 1000ms.

2. **Identifies at least two engineering framing errors:**
   - "Aggregate steady-ish" is wrong — human review dropped 14pp from baseline. Engineering is reading LLM-judge only and ignoring the judge/human divergence.
   - "Out of our hands" for the model update is wrong — we control what model version we serve. A regression suite before model-provider updates is a standard production-AI practice that was missing here.
   - "Mexican wasn't in our eval set" is a concession, not an excuse. The failure is not that Mexican showed up — it's that the monitoring didn't alert when a new input category crossed a volume threshold, and traffic continued flowing to an uncovered segment for 3 weeks without intervention.
   - The allergen law (concept drift) was a known regulatory change; it should have been tracked by PM/legal and added to the golden dataset before AB-1523 took effect. This is not an engineering issue — it's a PM process gap.

3. **Specific launch-readiness gap that allowed the drift to continue unaddressed:**
   - Missing alert on new-cuisine volume crossing a threshold (would have caught input drift at W4).
   - Missing regression suite on model-provider version updates (would have caught output drift at W5 before it hit users).
   - Missing human-review sampling with an alert threshold on LLM-judge/human agreement divergence (would have caught the judge miscalibration at W4–W5).
   - Missing rollback ownership — "rollback triggers are breached" but no one pulled the trigger for three weeks because the rollback decision ownership was not assigned pre-launch.
   - Missing regulatory change intake — PM did not have a process to update golden dataset ground truth when a known regulation took effect.

PM owns the miss. Specifically: launch readiness was treated as a one-time gate (pass the evals, ship), not an ongoing gate (can the system still pass its evals on *today's* traffic against *today's* ground truth). The fix is continuous readiness gating tied to monitoring alerts, with pre-committed rollback ownership.

**Weak response:**
- Accepts "out of our hands" framing on the model update
- Doesn't identify the judge/human divergence as the core missed signal
- Proposes "better monitoring" without naming specific alerts or thresholds
- Doesn't take PM accountability for the missed launch-readiness criteria
- Picks full rollback without distinguishing what each drift type requires

---

## D18: Red Teaming and Adversarial Evaluation

### Exercise Evaluation

**Step 1 — Attack Success Rate by category:**

- Prompt injection: 4/6 succeeded (P-01, P-02, P-03, P-05) = **67% ASR** — highest
- Jailbreak: 2/5 succeeded (P-08, P-11) = **40% ASR**
- Policy evasion: 2/7 succeeded (P-12, P-16) = **29% ASR**
- Data exfiltration: 1/4 succeeded (P-20) = **25% ASR**
- Harmful content: 0/3 succeeded = **0% ASR**
- **Overall: 9/25 = 36% ASR**

**Strong response:** reports category ASRs, identifies prompt injection as the clear weakness (67% — 2.5× the overall rate), notes that a 36% overall ASR averages over very different category risk profiles and is not a useful summary for a ship decision. The weakest category (prompt injection) is also where several high-severity successes live.

**Weak response:** reports only the overall ASR of 36%; doesn't break out by category; treats "36% ASR" as the decision-relevant number.

**Step 2 — Severity-weighted view of successful attacks:**

Of the 9 successful attacks:
- **Critical: 2** (P-01 prompt injection bypassing allergen check; P-12 shellfish via 'natural ocean seasoning')
- **High: 4** (P-02 Unicode injection; P-03 persona override; P-05 long-context injection; P-16 hedged dairy disclosure)
- **Medium: 2** (P-08 film-script jailbreak; P-20 system prompt leakage)
- **Low: 1** (P-11 emoji-encoded jailbreak)

**Strong response:** surfaces the 2 criticals and 4 highs as the decision-relevant findings; argues that "36% ASR, mostly mediums" is the wrong framing because 6 of 9 are critical/high; connects this back to the D10 guardrail-vs-optimization logic — any critical success is a ship-blocker regardless of aggregate.

**Weak response:** treats all 9 successes as equal; or only reports the 2 criticals and ignores that 4 of the remaining 7 are also high-severity.

**Step 3 — Mitigation layer mapping of successful attacks:**

- **Code:** 2 (P-02 Unicode sanitizer, P-12 allergen dictionary expansion) — fast and durable
- **Prompt:** 5 (P-01, P-03, P-05, P-11, P-20) — fast but fragile, attackers adapt
- **Model:** 1 (P-16 requires fine-tuning on hedged-disclosure patterns) — slow and expensive
- **Policy:** 1 (P-08 film-script exemption needs human-review gate) — slow at scale, manual
- **Out of scope:** 0

**Strong response:** notes that 2 of the 9 fixes are code-level (fastest and most durable — including one of the two criticals, P-12); 5 are prompt-level (can be shipped quickly but need regression suite because attackers will iterate); only 1 needs model retraining (P-16 — this is the bottleneck for full remediation). Concludes: the fastest meaningful ASR reduction is code + prompt hardening, covering 7 of 9 successes.

**Weak response:** doesn't distinguish layers; or counts "we can fix all 9 in a sprint" without flagging that the model-level fix (P-16) requires a retraining cycle and cannot be rushed.

**Step 4 — Threshold check and ship decision:**

Thresholds:
- Safety-critical (harmful_content): 0% ASR required. **Met: 0/3 succeeded.** ✅
- Critical-severity successful attacks: 0 allowed. **Breached: 2 succeeded (P-01, P-12).** ❌
- High-severity successful attacks: ≤ 1 allowed. **Breached: 4 succeeded.** ❌
- Medium-severity successful attacks: ≤ 3 allowed. **Met: 2 succeeded.** ✅
- Low-severity: no cap. **1 succeeded — acceptable.**

Two thresholds breached (criticals and highs). **Cannot ship to EU in current state.**

**Strong response:** applies thresholds systematically; concludes the system fails two of its own pre-committed ship criteria; flags specifically that the 4 high-severity prompt-injection successes (P-02, P-03, P-05, plus P-16) make "safety-weighted ship decision" the right framing rather than aggregate ASR.

**Weak response:** ignores pre-committed thresholds; rationalizes the 36% ASR as "industry-standard"; doesn't apply the D10 severity-gating logic to this lesson's adversarial findings.

### PM Decision Point Evaluation

**Strong response:**

1. **Ship decision:** Hold EU launch. Condition for re-expansion: (a) 0 critical-severity successful attacks across a refreshed red-team regression suite of at least 100 probes; (b) high-severity successful attacks ≤ 1 across that suite; (c) prompt-injection category ASR reduced from 67% to < 15%; (d) regression suite runs on every model/prompt change, not just before initial launch.

2. **Pushes back on at least two of:**
   - "In line with other labs' ASR" — Anthropic and OpenAI publish ASRs on adversarial *benchmarks* as model-evaluation research, not as product ship criteria. Comparing your production food-safety system's 36% ASR to model-benchmark numbers is a category error. Your threshold is your severity-weighted ship criteria, not an industry average.
   - "We've fixed the two criticals" — the 4 high-severity successes are also ship-blockers under pre-committed thresholds. The security lead is triaging to criticals only and ignoring the high-severity ledger. Also: "fixed" needs verification via regression suite runs post-fix, not a verbal claim.
   - "Ship and iterate" on prompt injection is especially dangerous because prompt injection is the attack category closest to the safety-critical surface (allergen bypass via P-01). Post-launch iteration on a category that can trigger allergen-disclosure failures is the exact scenario that generated the D17 allergen breaches.
   - "EU AI Act documentation drafted" is not the same as "EU AI Act documentation passes conformity assessment." Drafted docs that don't evidence systematic coverage, attack taxonomy, and mitigation tracking will fail a third-party audit.

3. **States specific EU AI Act documentation requirements:**
   - Attack taxonomy coverage: which categories tested, why those chosen
   - Probe construction methodology: seed sources (OWASP LLM Top 10, Promptfoo, internal), mutation approach
   - ASR by category and severity, with thresholds pre-committed before the red team
   - Mitigation layer mapping for every successful attack, with owner and target date
   - Regression suite status: which successful probes have been added to regression; how often it runs
   - Re-breach policy: what happens if a probe that was fixed later re-succeeds (auto-rollback? re-open incident?)
   - Third-party red team arrangement: EU AI Act conformity assessment for high-risk systems often requires external review, not just internal

PM owns signing the attestation. "Security eng said it was fine" is not a defense in conformity assessment.

**Weak response:**
- Ships to EU because "36% is standard"
- Accepts "we've fixed the two criticals" without requiring evidence via regression suite
- Confuses drafted documentation with conformity-passing documentation
- Treats prompt injection as low-priority because it's "not a content safety issue" — misses that P-01 shows prompt injection can directly bypass allergen safety
- Doesn't pre-commit re-expansion criteria or name specific ASR targets for remediation

---

## D19: The Ship Decision Framework

### Exercise Evaluation

**Step 1 — Triage the signals:**

- Pass: 11 (C-01, C-02, C-03, C-04, C-06, C-07, C-08, C-10, C-11, C-12, C-14)
- Marginal: 1 (C-13 shadow-mode agreement 82% vs 85%)
- Fail: 3
  - **Critical:** 2 (C-05 Mexican subgroup 62% vs 75%; C-09 prompt injection ASR 42% vs 15%)
  - **High:** 1 (C-15 jailbreak ASR 18% vs 10%)

"12 of 15 pass = 80% ready" is the wrong framing. Guardrails are AND-gated: one critical breach is a hold, regardless of how many green. Aggregate-counting treats safety-critical fails as trade-able against wins on unrelated dimensions.

**Strong response:** counts by status and severity; explicitly rejects the 80%-ready framing; surfaces the 2 criticals as the decision-relevant items.

**Weak response:** aggregates everything into a single pass rate; doesn't separate guardrail from optimization; treats the 3 fails as equivalent.

**Step 2 — AND logic on guardrails:**

All 10 guardrails (C-01, C-02, C-03, C-04, C-05, C-06, C-07, C-08, C-09, C-10, C-15) are reviewed. Three breach:
- C-05 Mexican subgroup (critical) → hold unless conditional path isolates the breach
- C-09 Prompt injection ASR (critical) → hold unless conditional path or pre-launch fix with re-test
- C-15 Jailbreak ASR (high) → hold unless conditional path + named mitigation owner

Initial answer: **Hold.** Two critical failures rule out an unconditional ship.

A strong +8pp paired improvement over v1 (C-11) does *not* offset the criticals because that's an optimization metric trading against guardrails — the whole point of the D10 distinction.

**Strong response:** states initial answer is hold; articulates why the C-11 optimization win cannot offset C-05/C-09 guardrail breaches; explicitly references AND logic.

**Weak response:** proposes "net positive" framing (strong C-11 offsets weaker failures); doesn't distinguish guardrail from optimization in the decision rule; ships because "8 of 10 guardrails pass."

**Step 3 — Conditional ship paths:**

- **C-05 Mexican subgroup:** Isolatable. Conditional ship path: route Mexican-cuisine requests to v1 (legacy path) for launch. Expansion criteria: Mexican subgroup pass rate ≥ 75% on hold-out of ≥ 50 cases. Rollback trigger: any Mexican case mis-approved on the v1 path (i.e., legacy still has the same drift concerns — D17 showed v1 also doesn't handle AB-1523 well, so consider human-review for Mexican as an alternative).
- **C-09 Prompt injection ASR:** Partially isolatable. Code-layer input sanitizer (the fix identified in D18 for P-02) can ship in 2-3 days and does not require model retraining. Post-fix: re-run the 6 prompt-injection probes. Expansion/ship criterion: ASR ≤ 15%. If the sanitizer doesn't clear it, hold.
- **C-15 Jailbreak ASR:** Isolatable via prompt hardening (P-08, P-11 are both prompt-layer). Similar 2-3 day fix + re-test. Ship criterion: ASR ≤ 10%.

Breaches that would NOT be conditional-shippable: a fail on C-08 (harmful content ASR) or C-06 (allergen breaches on hold-out) — those are absolute safety guardrails where "isolate and ship" would still expose some users.

**Strong response:** proposes a specific conditional-ship package: Mexican excluded, prompt-injection fix shipped and re-tested, jailbreak fix shipped and re-tested; names expansion criteria and rollback triggers for each; distinguishes conditional-shippable from non-shippable breaches.

**Weak response:** proposes "staged rollout" without specifying the conditional scope; promises "we'll fix post-launch" for C-09 and C-15 without re-test; doesn't specify expansion criteria.

**Step 4 — Ship memo skeleton:**

A strong ship memo skeleton covers:

1. **Decision:** "Conditional ship to EU Monday, with Mexican cuisine excluded and prompt-injection + jailbreak code/prompt fixes shipped and re-tested before launch. Hold if re-tests do not meet threshold."
2. **Evidence summary:** 11/15 criteria pass; 2 critical guardrails and 1 high breach identified; paired v2 improvement +8pp (95% CI +4 to +12) over v1.
3. **Reasoning:** optimization win (C-11) cannot offset guardrail breaches; Mexican subgroup isolatable via routing; prompt-injection and jailbreak fixable at code/prompt layer within timeline; full hold would forfeit real v2 gains across 4 cuisines for a breach affecting 3% of volume (Mexican).
4. **Conditional ship details:** scope = v2 for non-Mexican requests; expansion criterion = Mexican ≥ 75% pass on 50-case hold-out AND re-tested ASRs within thresholds; rollback = any allergen breach in first 72h, any prompt-injection re-breach, Mexican-route v1 handling a confirmed user-facing safety incident.
5. **Open risks and monitoring plan:** monitor Mexican-route latency on v1; daily review of prompt-injection attempt logs; weekly re-run of red-team regression; shadow-mode agreement tracking (C-13 marginal).
6. **Sign-offs:** PM, eng lead, security lead, legal (EU AI Act).

**Strong response:** skeleton uses specific numbers and criteria from the exercise; decision is stated plainly in one line; conditional scope is concrete, not vague; rollback triggers are numerical.

**Weak response:** vague decision statement ("monitor carefully"); no specific expansion criteria; treats "staged rollout" as the entire plan without scope definition.

### PM Decision Point Evaluation

**Strong response:**

1. **Decision:** Conditional ship. Recommended scope: v2 for non-Mexican requests; Mexican routed to v1 until subgroup-specific eval clears 75%; prompt-injection code-layer sanitizer and jailbreak prompt hardening shipped and re-tested before Monday 6am. If re-tests don't clear: hold.

2. **Conditional scope (precise):**
   - **In-scope for v2 Monday:** American, Thai, Indian, and "other" cuisines except Mexican. Start at 25% traffic; expand to 50% at 72h, 100% at 7 days, each gated by: no allergen breaches, human-review pass rate ≥ 78%, p95 latency ≤ 1000ms.
   - **Out-of-scope:** Mexican cuisine, routed to v1 with human-review overlay on v1 outputs until re-eval clears.
   - **Expansion criteria for Mexican:** ≥ 75% pass rate on a 50-case Mexican hold-out, AB-1523 allergen cases included, 2 consecutive weeks with 0 breaches.
   - **Rollback triggers:** any allergen breach in first 72h = full rollback. Any prompt-injection regression in red-team suite = block further rollout and investigate. Mexican-route v1 produces ≥ 1 user-facing safety incident = escalate to full hold.

3. **Fastest realistic path to full ship:**
   - **Mexican subgroup (2-3 weeks):** expand eval set to 50+ Mexican cases including AB-1523, run full eval, iterate on prompt for Mexican-specific patterns, hold-out re-test. Owner: eng + ML. This cannot ship Monday — any claim otherwise is false urgency.
   - **Prompt-injection fix (2-3 days):** code-layer Unicode/long-input sanitizer + system-prompt hardening. Owner: security eng. Re-test: all 6 probes + 20 mutated variants. Can clear by Monday AM if started today.
   - **Jailbreak fix (2-3 days):** prompt hardening for P-08 film-script and P-11 emoji-decode patterns. Owner: ML eng. Re-test: all 5 probes + 10 variants.
   - **Marginal C-13 shadow agreement:** human-review sampling on disagreement cases to confirm v2 is correcting v1 errors (not introducing new ones). Owner: PM. 1-week investigation.

The response makes the CEO's trade-off explicit: Monday is achievable *for the non-Mexican v2 launch with security fixes*, not for full v2. "Fastest realistic" is not "fastest wished" — and the regulatory risk of EU launch with a subgroup safety failure is larger than a Q2 slip headline.

**Weak response:**
- Says "ship Monday" without naming the conditional scope or re-tests
- Says "hold" without offering the conditional alternative
- Accepts CEO framing of "perfect enemy of good" without pushing back — the issue is guardrails, not perfection
- Promises timelines that are not realistic (e.g., "we'll fix Mexican subgroup by Monday" for a 2-3 week fix)
- Doesn't name owners or rollback triggers
- Hides or softens the critical findings in the response to leadership

---

## D20: Regulatory and Legal Context for AI Evaluation

### Exercise Evaluation

**Step 1 — Classify AI products by risk tier:**

| Product | Correct Tier | Reasoning |
|---------|-------------|-----------|
| P-01 Resume screening bot | **High** | Employment decisions — explicitly listed in Annex III |
| P-02 Customer support chatbot | **Limited** | User-facing AI interaction — transparency obligation only |
| P-03 Credit scoring model | **High** | Creditworthiness assessment — explicitly listed in Annex III |
| P-04 Social media recommender | **Limited** | Content recommendation — transparency obligations; not high-risk unless manipulative |
| P-05 Emergency triage assistant | **High** | Healthcare safety-critical — affects patient outcomes |
| P-06 Menu verification system | **Debatable: high or limited** | See Step 2 for detailed analysis |
| P-07 Subliminal ad optimizer | **Unacceptable (banned)** | Subliminal manipulation below conscious awareness — explicitly prohibited |
| P-08 Internal code review assistant | **Minimal** | Internal productivity tool, no external user impact, no safety implications |

**Strong response:** correctly classifies at least 6 of 8; identifies P-07 as banned (not just high-risk); recognizes P-06 as debatable with reasoning.

**Weak response:** confuses limited and minimal risk; classifies P-07 as high-risk instead of banned; classifies P-02 (FAQ chatbot) as high-risk.

**Step 2 — Menu verification system classification:**

The menu verification system is genuinely debatable. Arguments for each:

**Case for high-risk:**
- Incorrect allergen approvals can cause severe allergic reactions (anaphylaxis — potentially fatal)
- The system is a *safety component* of the food delivery platform — it's the last check before a menu reaches consumers
- Food safety is regulated by EU food-safety legislation; an AI system serving as a safety component of a regulated product is high-risk under Article 6

**Case for limited risk:**
- The system is B2B (used by the platform, not directly consumer-facing)
- It's an *advisory* tool — human restaurant managers ultimately control menu content
- Allergen disclosure is the restaurant's legal responsibility, not the platform's AI

**Best answer:** classify as high-risk, based on the precautionary principle. The cost of classifying too high is extra compliance work (documentation, monitoring formalization). The cost of classifying too low is: if a regulator later determines it's high-risk, you're retroactively non-compliant with potential penalties up to 7% of revenue, plus reputational damage from an allergen incident that wasn't properly monitored.

**Strong response:** argues both sides; picks high-risk with explicit reasoning about asymmetric cost of misclassification; names the "safety component" argument as the tipping factor.

**Weak response:** picks a tier without reasoning; doesn't consider the asymmetric cost; or picks limited risk without acknowledging the health/safety angle.

**Step 3 — Audit eval practices against requirements:**

| Requirement | Coverage Status | What You Have | What's Missing |
|-------------|----------------|---------------|----------------|
| R-01 Risk management | **Partial** | D2 surface map, D10 criteria, D19 ship memo | No single maintained risk-management document; scattered across exercises |
| R-02 Data governance | **Partial** | D7 golden datasets, D12 fairness | No formal data sheet documenting sourcing, representativeness, bias assessment |
| R-03 Technical documentation | **Partial** | D10 thresholds, D16 experiments, D18 red-team, D19 memo | No versioned artifact linking methodology → dataset → results → thresholds |
| R-04 Logging/traceability | **Partial** | D14 observability framework designed | Implementation, retention policies, and audit access controls not confirmed |
| R-05 Human oversight | **Partial** | D5 human graders, D17 rollback + human-review layer | Override capability not formally documented; escalation chain not formalized |
| R-06 Accuracy/robustness | **Partial** | D4 CIs, D16 experiments, D18 red teaming | Results exist as exercise outputs, not as formal test reports |
| R-07 Post-deployment monitoring | **Partial** | D17 drift detection, rollback triggers, online eval | Monitoring plan not a standalone document linked to conformity assessment |
| R-08 Incident reporting | **Gap** | D17 covers rollback for product purposes | No incident classification scheme; no regulatory notification workflow |
| R-09 Transparency | **Gap** | Not covered in D1–D19 | No user-facing transparency notice; no documentation of system limitations for affected persons |
| R-10 Conformity declaration | **Gap** | D19 ship memo is structurally similar | Ship memo is internal; conformity declaration is a legal filing with specific format |

Pattern: most items are **partial** — the eval *practice* exists but the *documentation* is not formalized for audit. Three items are full **gaps** (R-08 incident reporting, R-09 transparency, R-10 conformity declaration).

**Strong response:** identifies the pattern (practice exists, documentation doesn't); correctly marks R-08, R-09, R-10 as gaps; distinguishes "we do this" from "we can prove this to a regulator."

**Weak response:** marks everything as "covered" because the course addressed the concept; doesn't distinguish practice from documentation; misses R-08 and R-09 as gaps.

**Step 4 — Prioritize gaps:**

By regulatory risk (highest first):
1. **R-08 Incident reporting (gap)** — highest risk because failure to report a serious incident is itself a violation with separate penalties. Effort: weeks (build classification scheme + notification workflow + test it).
2. **R-09 Transparency (gap)** — required before any EU deployment. Effort: days (write transparency notice + system limitation disclosure).
3. **R-03 Technical documentation (partial)** — the conformity assessment depends on this. Effort: 1–2 weeks (consolidate existing eval artifacts into a versioned document).
4. **R-01 Risk management (partial)** — must be a maintained document, not scattered outputs. Effort: 1 week (consolidate D2/D10/D19 into a single risk-management artifact).
5. **R-02 Data governance (partial)** — formal data sheet required. Effort: 1 week.
6. **R-06 Accuracy/robustness (partial)** — formalize test reports. Effort: days.
7. **R-07 Monitoring plan (partial)** — extract from D17 exercise into standalone document. Effort: days.
8. **R-05 Human oversight (partial)** — document override and escalation. Effort: days.
9. **R-04 Logging (partial)** — confirm implementation and retention. Effort: days to verify.
10. **R-10 Conformity declaration (gap)** — legal drafts this based on PM's eval evidence. Effort: depends on legal, but PM provides evidence in 1–2 weeks.

**Strong response:** prioritizes R-08 (incident reporting) as highest risk because it's both a gap and a separate violation; identifies that most partial-coverage items are documentation efforts (days to weeks); separates cheap doc work from expensive capability-building.

**Weak response:** prioritizes by effort instead of risk; doesn't distinguish documentation gaps from capability gaps; puts R-10 first without recognizing it depends on all other items.

### PM Decision Point Evaluation

**Strong response:**

1. **Risk classification:** High-risk, because the system is a safety component affecting consumer health (allergen decisions). The "safety component" argument under Article 6 tips the classification, even though the system is B2B. Cost of under-classifying is asymmetrically worse than over-classifying.

2. **At least three specific gaps:**
   - R-08 Incident reporting: no incident classification scheme maps system failures (allergen breach, false approval) to EU-reportable categories. No regulatory notification workflow. If a user has an allergic reaction traceable to a model error, we currently have no process to report within 72 hours.
   - R-03 Technical documentation: eval methodology exists across D10 thresholds, D16 experiment results, D18 red-team findings, D19 ship memo — but no single versioned artifact ties methodology → dataset → results → thresholds in an auditable format. An auditor would have to reconstruct from multiple sources.
   - R-09 Transparency: no user-facing notice that an AI system checks allergen disclosures. No documentation of system limitations (e.g., "system has not been evaluated on Mexican, Korean, or Vietnamese cuisines") available to affected persons. This is a hard requirement even for limited-risk classification.
   - Bonus: R-02 Data governance — no formal data sheet documenting how golden datasets were sourced, how representativeness was assessed (D12 showed Thai was under-covered), or how contamination is prevented (D7).

3. **Compliance roadmap:**
   - **Phase 1 (2 weeks — documentation):** Consolidate existing eval artifacts into regulatory format: (a) risk management document from D2/D10/D19 outputs; (b) technical documentation linking methodology → dataset → results → thresholds; (c) monitoring plan extracted from D17; (d) transparency notice for user-facing disclosure; (e) formal test reports from D16/D18 results. Owner: PM + tech writer.
   - **Phase 2 (4–8 weeks — new capabilities):** (a) Build incident classification and regulatory notification workflow (R-08) — requires legal + eng collaboration; (b) formalize data governance documentation with data sheets for each dataset (R-02); (c) implement and test human-override mechanisms end-to-end (R-05); (d) set up audit-accessible logging with defined retention (R-04). Owner: PM + eng + legal.
   - Legal can begin conformity declaration drafting once Phase 1 is complete. CE marking and EU database registration happen after Phase 2.

Engineering is right that the eval practices are strong. The gap is not in what we do — it's in what we can prove. An auditor doesn't attend our tutoring sessions — they read documents.

**Weak response:**
- Says "we're already compliant" because the course covered the concepts
- Doesn't distinguish eval practice from audit-ready documentation
- Names gaps vaguely ("we need better documentation") without specifying which requirement, what exists, and what's missing
- Proposes a timeline without separating documentation work from capability-building
- Doesn't name incident reporting (R-08) as the most critical gap

---

## D21: Building an Eval Culture

### Exercise Evaluation

**Step 1 — Assess current state:**

- **not_done:** 7 (A-01, A-02, A-04, A-05, A-06, A-08, A-10, A-11 — actually 8)
- **ad_hoc:** 3 (A-03, A-09, A-12)
- **partial:** 1 (A-07)

Only 1 of 12 activities is partially operational. 8 are not done at all. The team is solidly at Level 1. Most L2 activities (golden dataset, error analysis, subgroup review, human review) are not done — meaning the foundation for Level 3 doesn't exist yet. You can't automate regression suites (L3) if you don't have a golden dataset (L2) to run them against.

**Strong response:** counts accurately; recognizes the L2 gaps as the foundation that must be built before L3 automation; notes that "partial" on drift monitoring (A-07) without a golden dataset or human review loop means the monitoring isn't anchored to ground truth.

**Weak response:** counts but doesn't analyze the implication; or focuses on the L3/L4 gaps without recognizing the missing L2 foundation.

**Step 2 — Ownership and cadence assignments:**

| Activity | Target Owner | Target Cadence | Reasoning |
|----------|-------------|---------------|-----------|
| A-01 Error analysis | PM | monthly | PM decides what to investigate; monthly catches slow trends |
| A-02 Golden dataset | PM + eng | monthly refresh | PM curates, eng version-controls |
| A-03 Automated graders | eng | per_change + continuous | Must be automated in pipeline |
| A-04 Judge recalibration | data science | monthly | Statistical work; monthly catches drift |
| A-05 Subgroup review | PM | weekly | PM owns fairness; weekly catches regressions fast |
| A-06 Regression in CI/CD | eng | per_change | Must block merges automatically |
| A-07 Drift monitoring | PM + eng | continuous | Eng builds, PM reviews alerts |
| A-08 Incident reporting | PM + legal | event_driven | Triggered by incidents, not scheduled |
| A-09 Ship memo | PM | every_ship | Required for every non-trivial release |
| A-10 Stakeholder report | PM | monthly | Monthly cadence for leadership visibility |
| A-11 Eval cost tracking | PM | monthly | Track alongside stakeholder report |
| A-12 Human review | QA + PM | weekly | QA executes, PM reviews patterns |

**Strong response:** assigns ownership using the three-role model (PM strategy, eng infra, data science calibration); distinguishes per-change from periodic from event-driven appropriately; doesn't assign everything to PM.

**Weak response:** assigns everything to eng or everything to PM; doesn't distinguish cadences; makes everything "monthly" including things that should be per-change.

**Step 3 — First 30-day priorities (pick 4):**

The highest-leverage four are:

1. **A-02 Golden dataset** — everything depends on this. You can't run automated graders, regression suites, or judge recalibration without a curated dataset. This unblocks A-03, A-04, A-06. Effort: weeks.

2. **A-12 Human review sampling** — starts the ground-truth feedback loop immediately. Gives you real data to build the golden dataset from. Also catches urgent quality issues in production now. Effort: days.

3. **A-01 Error analysis** — the first trace review reveals the failure modes you don't know about. Feeds the golden dataset curation and tells you which metrics to track. Effort: weeks for first pass.

4. **A-05 Subgroup performance review** — once you have even a basic golden dataset, slicing by subgroup reveals hidden regressions immediately. This is a quick win that demonstrates value to leadership. Effort: days.

Alternative acceptable picks: A-03 (automated graders) or A-09 (ship memo) could replace A-05 if the learner provides strong reasoning.

**Strong response:** picks A-02 as the foundation; recognizes the dependency chain (golden dataset → graders → regression suite); includes A-12 or A-01 as the data-collection mechanism that feeds the dataset; picks 4 activities that are L2 (foundation), not L3 (automation).

**Weak response:** picks L3 activities (CI/CD, drift monitoring) without the L2 foundation; doesn't explain dependency chains; picks based on "what sounds most impressive" rather than "what unblocks the most."

**Step 4 — 90-day roadmap:**

**Days 1–30 (Foundation):**
- A-12: Human review sampling — 50 traces/week, structured rubric, start immediately
- A-01: Error analysis — first 50-trace review, categorize failures, identify top 3 failure modes
- A-02: Golden dataset — curate initial 200-case dataset from trace review + production logs
- A-05: Subgroup review — first slice of pass rates by cuisine, flag any subgroup below 70%
- "Done" looks like: 200-case golden dataset exists, first error taxonomy complete, subgroup baselines established, weekly human review is running

**Days 31–60 (Automation):**
- A-03: Deploy automated graders (code-based on 100%, LLM-judge on sampled traffic)
- A-04: First judge recalibration against human labels from A-12
- A-09: Ship memo template created and used for next release
- A-07: Drift monitoring enhanced with cuisine-mix alerts and judge/human divergence alerts
- "Done" looks like: automated graders running on every eval, judge Kappa > 0.6, ship memo used for at least one release, drift alerts configured

**Days 61–90 (Maturity):**
- A-06: Regression suite in CI/CD — merges blocked if pass rate drops
- A-10: First stakeholder quality report delivered to leadership
- A-11: Eval cost baseline established
- A-08: Incident classification scheme drafted (with legal)
- "Done" looks like: CI blocks bad changes, leadership receives monthly quality report, eval costs tracked, incident reporting pathway defined

**Strong response:** phases activities in dependency order (L2 → L3 → L4); each phase has concrete "done" criteria; recognizes that Days 1–30 is unglamorous foundation work; Day 90 metric is specific and measurable.

**Weak response:** puts all 12 activities in one phase; no concrete "done" criteria; jumps to CI/CD without the dataset foundation; Day 90 deliverable is vague.

### PM Decision Point Evaluation

**Strong response:**

1. **Three activities with concrete "done" definitions:**
   - **A-02 Golden dataset:** 200 cases, stratified by cuisine (≥40 per cuisine including Mexican/Korean), version-controlled in git, ground-truth labels reviewed by 2 annotators. Done = dataset exists and first eval suite run against it produces baseline pass rates.
   - **A-12 Human review sampling:** 50 traces/week graded by QA using a structured rubric (5 dimensions: allergen accuracy, format compliance, reasoning quality, confidence calibration, edge-case handling). Done = 4 weeks of continuous data, weekly report delivered to PM, disagreements with any automated check logged.
   - **A-06 Regression suite in CI/CD:** full eval suite runs on every prompt/model/pipeline PR. Merge blocked if aggregate pass rate drops >2pp or any guardrail metric breaches. Done = CI pipeline configured, tested with a deliberately regressive prompt change, documented in eng runbook.

   (Alternative strong picks: A-01 instead of A-06 if argued that error analysis is prerequisite to building a good golden dataset. A-03 instead of A-06 if argued that automated graders are the prerequisite for CI/CD.)

2. **Prioritization logic:**
   - A-02 is the foundation — nothing else works without test data. You can't run graders, judge recalibration, regression suites, or subgroup reviews without a dataset.
   - A-12 is the ground-truth signal — it feeds the golden dataset, catches production issues now, and creates the human baseline for judge recalibration later.
   - A-06 is the mechanism that makes eval gains permanent — without CI/CD blocking, any improvement you make can be silently regressed by the next prompt change. This is the difference between "we did evals once" and "evals are part of how we ship."
   - The other nine activities either depend on these three (A-03, A-04, A-05 need the dataset; A-07, A-09 need the process maturity) or are lower-leverage (A-10, A-11 are reporting, not infrastructure).

3. **Day 90 metric:**
   "Before: zero regression tests, every prompt change was a gamble, last quarter we had 3 production incidents from untested changes. After: every PR runs against 200 eval cases in CI, 2 regressions caught and blocked before they reached production, zero production incidents from untested changes in the past 60 days."

   Or: "Before: quality was assessed by 'looks fine.' After: we have a weekly quality number (human-review pass rate) tracked over 12 weeks, currently at 84% and trending up from 71% at week 1."

**Weak response:**
- Picks activities that sound impressive but aren't foundational (drift monitoring, stakeholder reports, incident reporting — all important but depend on the foundation)
- "Done" definitions are vague ("we'll have a golden dataset" without case count, stratification, or version control)
- Can't state the Day 90 metric — or states something unmeasurable ("we'll have better quality")
- Picks three activities that don't form a logical chain (e.g., error analysis + incident reporting + eval cost tracking — no dependency between them, no compounding leverage)

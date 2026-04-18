# Dataset Specifications

## menu-verification-d1.csv (30 rows)

Simplified production shadow-mode data for D1. Derived from menu-verification-dataset.csv with columns collapsed for first-day pipeline mapping exercise.

**Design targets:**
- 30 rows with change type distribution: 12 price, 7 description, 5 allergen, 4 image, 2 category
- ~80% human agrees (24 TRUE, 6 FALSE)
- Mix of llm_decision: approve, reject, flag_for_review
- Some price changes missing reference_data (no last verified price on file)
- Description and category changes have no reference_data — teaches context retrieval gaps

**Columns (12):**

| Column | Description |
|--------|-------------|
| change_id | Unique identifier (CHG-001 through CHG-030) |
| restaurant_name | Restaurant name |
| item_name | Menu item name |
| change_type | One of: price, description, allergen_dietary, image, category |
| old_value | Previous value (price amount, description text, allergen list, image filename, or category name) |
| new_value | Proposed new value |
| reference_data | External data retrieved to inform the decision. Populated for price (last verified price + markup) and allergen (current allergen record) changes. Empty for description, category, and some price changes (context gap). |
| llm_decision | approve, reject, or flag_for_review |
| llm_confidence | 0.0–1.0 |
| llm_reasoning | Short text explanation of the LLM's decision |
| human_decision | approve or reject (always binary — ground truth) |
| human_agrees_with_llm | TRUE or FALSE |

**Key teaching patterns:**
- **Pipeline mapping**: 12 columns cleanly map to 5 categories (input, context, LLM output, human review, metadata)
- **Reference data gap**: description and category changes have no external reference — the system is guessing. Price changes sometimes lack verified prices too. This variation by change_type is the key Step 3 discovery.
- **Disagreement rows**: 6 rows where human disagrees with LLM — learner can trace why in Step 2

---

## menu-verification-dataset.csv (200 rows)

Production shadow-mode data. LLM decisions logged alongside human ground truth.

**Design targets:**
- v1: 73 rows, ~78% accuracy (human agrees with LLM)
- v2: 127 rows, ~85% accuracy
- High-markup items (>30%) have lower accuracy than low-markup
- ~15% of rows have no `last_verified_price` (context gap)
- Change type distribution: ~45% price, ~25% description, ~12% allergen_dietary, ~10% image, ~8% category

**Columns (27):**

| Column | Description |
|--------|-------------|
| change_id | Unique identifier (CHG-001, CHG-002, ...) |
| restaurant_id | Restaurant identifier (R-1001, R-1002, ...) |
| restaurant_name | Restaurant name |
| item_name | Menu item name |
| item_category | Menu category (Main, Sides, Starters, Desserts, Street Food, Seafood, Beverages) |
| change_type | One of: price, description, allergen_dietary, image, category |
| old_price | Previous price (populated only for price changes) |
| new_price | Proposed price (populated only for price changes) |
| old_description | Previous description (populated only for description changes) |
| new_description | New description (populated only for description changes) |
| old_allergen_dietary | Previous allergen/dietary tags (populated only for allergen changes) |
| new_allergen_dietary | New allergen/dietary tags (populated only for allergen changes) |
| old_image | Previous image filename (populated only for image changes) |
| new_image | New image filename (populated only for image changes) |
| old_category | Previous menu category (populated only for category changes) |
| new_category | New menu category (populated only for category changes) |
| last_verified_price | Last known offline/verified price. Blank for ~15% of rows (context gap) |
| price_markup_pct | Percentage markup above last_verified_price (for price changes) |
| llm_decision | approve, reject, or flag_for_review |
| llm_confidence | 0.0-1.0. Higher when correct (~0.85), lower when wrong (~0.55) |
| llm_reasoning | Short text explanation of the LLM's decision |
| human_decision | approve or reject (always binary — ground truth) |
| human_agrees_with_llm | TRUE or FALSE |
| processing_time_ms | LLM verification latency |
| model_version | v1 or v2 |
| timestamp | ISO 8601 timestamp |
| failure_type | When LLM was wrong: false_approval, context_miss, hallucination, policy_miss, price_error. Blank when correct. |

**Column isolation rule:** Only the old/new pair for the relevant `change_type` is populated. All other old/new pairs are blank.

---

## multirun-test.csv (100 rows = 20 test cases × 5 runs)

Controlled test data. Same input run 5 times to measure pass@k and reliable@k.

**Design targets:**
- pass@5 = 0.90 (18/20 test cases have at least 1 correct run)
- reliable@5 = 0.55 (11/20 test cases correct on all 5 runs)
- gap = 0.35
- 11 RELIABLE cases (5/5 correct), 7 CAPABLE cases (1-4/5), 2 NOT CAPABLE (0/5)
- Mix of change types: 11 price, 3 description, 3 allergen_dietary, 2 image, 1 category
- Confidence when correct: ~0.85 avg. When wrong: ~0.55 avg.

**Columns (22):**

| Column | Description |
|--------|-------------|
| test_id | Test case identifier (T-01 through T-20) |
| restaurant_name | Restaurant name |
| item_name | Menu item name |
| item_category | Menu category |
| change_type | One of: price, description, allergen_dietary, image, category |
| old_price / new_price | For price changes only |
| old_description / new_description | For description changes only |
| old_allergen_dietary / new_allergen_dietary | For allergen changes only |
| old_image / new_image | For image changes only |
| old_category / new_category | For category changes only |
| last_verified_price | Last known verified price |
| price_markup_pct | Markup percentage (price changes only) |
| ground_truth | approve or reject (always binary) |
| run_number | 1-5 |
| llm_decision | approve, reject, or flag_for_review |
| llm_confidence | 0.0-1.0 |
| llm_reasoning | Short text explanation |

---

## grader-comparison-dataset.csv (20 rows)

Grader comparison data. Same 20 menu verification decisions evaluated by all three grader types.

**Design targets:**
- Code-Human agreement: 65% (13/20)
- Model-Human agreement: 80% (16/20)
- Code-Model agreement: 65% (13/20)
- 8 patterns represented: PPP(7), FFF(4), PFF(3), FPP(2), PPF(1), FFP(1), PFP(1), FPF(1)

**Columns (12):**

| Column | Description |
|--------|-------------|
| eval_id | Case identifier (E-01 through E-20) |
| change_type | One of: price, description, allergen_dietary, image, category |
| change_summary | Human-readable summary of the menu change |
| llm_decision | approve, reject, or flag_for_review |
| llm_reasoning | The system's reasoning for its decision |
| ground_truth | approve or reject (always binary) |
| code_grader | PASS or FAIL — code-based grader result |
| code_rule | Which rule the code grader applied |
| model_grader | PASS or FAIL — LLM judge result |
| model_critique | The LLM judge's reasoning |
| human_grader | PASS or FAIL — human reviewer result |
| human_note | The human reviewer's reasoning |

**Key teaching patterns:**
- PFF (E-12, E-13, E-14): Code misses unverifiable claims and reasoning contradictions
- FPP (E-15, E-16): Code too rigid on seasonal/stale-reference edge cases
- FPF (E-20): Code catches allergen safety issue that model misses — deterministic rules matter for safety
- FFP (E-18): Both automated graders miss certificate that human verifies

---

## judge-calibration-dataset.csv (20 rows)

Judge calibration data. Same 20 menu verification decisions evaluated by two LLM judge prompt versions (V1=vague, V2=specific rubric) plus human expert ground truth.

**Design targets:**
- V1-Human agreement: 35% (7/20) — vague judge is nearly useless
- V2-Human agreement: 85% (17/20) — specific rubric dramatically better
- V1 recall on FAIL: 8% (1/13) — misses almost all real failures
- V2 recall on FAIL: 77% (10/13) — catches most but not all
- V2 precision on FAIL: 100% (10/10) — when V2 flags, it's always real
- Human verdict distribution: 7 PASS, 13 FAIL (dataset skewed toward cases requiring judgment)

**Columns (11):**

| Column | Description |
|--------|-------------|
| case_id | Case identifier (J-01 through J-20) |
| change_type | One of: price, description, allergen_dietary, category |
| change_summary | Human-readable summary of the menu change |
| llm_decision | The menu verifier's decision (approve/reject) |
| llm_reasoning | The menu verifier's reasoning |
| human_verdict | PASS or FAIL — domain expert ground truth |
| human_critique | The expert's detailed reasoning |
| judge_v1_verdict | PASS or FAIL — vague judge prompt result |
| judge_v1_reasoning | The V1 judge's reasoning |
| judge_v2_verdict | PASS or FAIL — specific rubric judge result |
| judge_v2_reasoning | The V2 judge's reasoning |

**Key teaching patterns:**
- V1 is a verbosity-bias machine: approves any decision with coherent-sounding reasoning (18/20 PASS)
- V2 catches unverifiable claims (J-02, J-04, J-09, J-11, J-13, J-15, J-17, J-20), fabricated reasoning (J-08), and ungrounded justifications (J-05) that V1 misses
- V2 remaining failures illustrate three distinct judge limitation categories:
  - **Rubric gap** (J-07): "Angus" beef is a breed claim the rubric doesn't explicitly cover
  - **Context gap** (J-14): "House-made hot sauce" from a restaurant flagged 4 times for misrepresenting store-bought as house-made — judge can't see vendor history
  - **Policy ambiguity** (J-18): "Traditional Japanese-style ramen" from a fusion restaurant — is "Japanese-style" a cuisine descriptor or an authenticity claim?
- V2 correctly handles nuance: J-19 (Thai restaurant serving "Thai-style" food) passes V2 correctly

---

## eval-dataset-audit.csv (40 rows)

Eval dataset audit data. Metadata for 40 cases in an existing eval dataset, designed for learners to discover quality problems.

**Design targets:**
- Verdict distribution: 27 PASS (68%), 13 FAIL (32%) — not enough failures
- Difficulty skew: 20 easy (50%), 11 medium (28%), 9 hard (23%)
- Change type skew: price 17 (43%), description 12 (30%), allergen 6 (15%), image 3 (8%), category 2 (5%)
- Contaminated ground truth: 10 cases (7 copied_from_v1, 3 model_generated)
- Stale labels (before 2026-01-01 policy update): 11 cases

**Columns (7):**

| Column | Description |
|--------|-------------|
| case_id | Case identifier (DS-01 through DS-40) |
| change_type | One of: price, description, allergen_dietary, image, category |
| difficulty | easy, medium, or hard |
| ground_truth_value | PASS (should be approved) or FAIL (should be rejected) — the reference answer for evaluation |
| ground_truth_source | How the label was created: expert_annotation, model_generated, or copied_from_v1 |
| ground_truth_date | When the label was created (ISO date) |
| notes | Free-text context about the case |

**Key teaching patterns:**
- **Contamination**: DS-24/DS-25 (allergen cases labeled by v1) are especially dangerous — safety-critical labels from an untrusted source
- **Staleness + contamination overlap**: DS-31 through DS-35 are both contaminated AND stale — double problem
- **Easy skew**: 50% of cases are easy — learner should recognize this as wasted capacity (easy cases don't test where the system actually struggles)
- **Coverage gaps**: hard cases underrepresented (9/40), description cases underrepresented relative to their failure importance

---

## rag-eval-dataset.csv (18 rows)

RAG pipeline evaluation data. 18 queries run through a two-stage retrieval + generation system, with full ground truth for retrieval quality, generation quality, and final decision correctness.

**Pipeline setup:**
- Knowledge base: 15 policy documents (POL-PRICE-STD, POL-PRICE-SEASONAL, POL-PRICE-PREMIUM, POL-ALLERGEN-REMOVE, POL-ALLERGEN-ADD, POL-DESC-SOURCE, POL-DESC-BREED, POL-DESC-PREP, POL-DESC-ORGANIC, POL-CUISINE-AUTH, POL-CAT-CHANGE, POL-PORTION, POL-IMAGE-ACC, POL-NUTRI, POL-VENDOR-HIST)
- Retriever fetches top-5 policies per query
- Generator uses retrieved policies as context to produce a decision + reasoning

**Design targets:**
- Avg Precision@5: ~0.14 (retriever is noisy — most slots are irrelevant)
- Avg Recall@5: ~0.67 (decent when it works, zero when it misses)
- Faithfulness rate: ~72% (13/18 faithful)
- Answer relevance rate: ~72% (13/18 relevant)
- Overall correctness: 33% (6/18) — much lower than any individual stage metric
- 2x2 matrix distribution: 6 Good-R+Good-G, 6 Good-R+Bad-G, 2 Bad-R+Good-G, 4 Bad-R+Bad-G
- Change type distribution: 3 price, 3 allergen, 4 description, 2 category, 1 portion, 1 image, 1 nutrition, + variants

**Columns (13):**

| Column | Description |
|--------|-------------|
| case_id | Case identifier (Q-01 through Q-18) |
| query_text | The user query / proposed menu change to evaluate |
| change_type | One of: price, allergen, description, category, portion, image, nutrition |
| retrieved_docs | Semicolon-separated list of 5 policy IDs the retriever fetched (in rank order) |
| relevant_docs | Semicolon-separated list of policy IDs that are actually relevant (ground truth) |
| precision_at_5 | (relevant ∩ retrieved) / 5 |
| recall_at_5 | (relevant ∩ retrieved) / (number of relevant) |
| generated_decision | approve, reject, or flag_for_review |
| generated_reasoning | Short text from the generator explaining its decision |
| ground_truth_decision | approve or reject (expert ground truth) |
| decision_correct | TRUE if generated_decision matches ground_truth_decision |
| faithful | TRUE if generated_reasoning is grounded in retrieved_docs (no fabricated claims) |
| answer_relevant | TRUE if generated_reasoning addresses the actual query (vs. discussing something off-topic) |

**Key teaching patterns:**
- **Good R + Good G (Q-01 to Q-06)**: Working baseline. Relevant policy retrieved, generator uses it correctly.
- **Good R + Bad G (Q-07 to Q-12)**: Retrieval worked; generation broke. Mix of faithful-but-irrelevant (Q-07, Q-10, Q-12: off-topic reasoning) and unfaithful-but-relevant (Q-08, Q-09, Q-11: fabricated policy thresholds).
- **Bad R + Good G (Q-13, Q-14)**: Generator handled retrieval failure gracefully but had nothing to ground on. The relevant policy was missing.
- **Bad R + Bad G (Q-15 to Q-18)**: Both stages broken. These cases need fixes on both sides.
- **Decision Point math**: Fixing generation eliminates Q-07 to Q-12 (+6 correct → 12/18). Fixing retrieval eliminates Q-13 to Q-14 (+2 correct → 8/18). Generation is the bigger lever. Q-15 to Q-18 need both.

---

## hallucination-detection-dataset.csv (15 rows)

Hallucination detection data. 15 menu change decisions where the system cites a specific policy in its reasoning. The learner compares reasoning against the actual policy text (provided in the lesson) to detect hallucinations.

**Policy reference:** 6 policies with short, clear text provided in the lesson (POL-PRICE-STD, POL-ALLERGEN-REMOVE, POL-DESC-SOURCE, POL-DESC-PREP, POL-CUISINE-AUTH, POL-IMAGE-ACC). The learner uses these as the source of truth.

**Design targets:**
- Grounded (no hallucination): 7 cases (H-01 to H-07), all decision_correct = TRUE
- Intrinsic hallucination (contradicts policy): 2 cases (H-08, H-10), all decision_correct = FALSE
- Extrinsic hallucination (adds fabricated claims): 6 cases (H-09, H-11 to H-15), 3 correct + 3 wrong
- Overall hallucination rate: 53% (8/15)
- Overall decision correctness: 67% (10/15)
- Hallucinated + correct decisions: 3/8 (38%) — these are invisible to decision-only checks
- Intrinsic → 100% wrong decisions. Extrinsic → 50% wrong decisions.

**Columns (8):**

| Column | Description |
|--------|-------------|
| case_id | Case identifier (H-01 through H-15) |
| query_summary | The menu change being evaluated |
| change_type | One of: price, allergen, description, image, cuisine |
| cited_policy | Policy ID the system claims supports its decision |
| system_decision | approve, reject, or flag_for_review |
| system_reasoning | The system's explanation citing the policy — may be grounded, may be hallucinated |
| ground_truth_decision | approve or reject (expert-verified) |
| decision_correct | TRUE if system_decision matches ground_truth_decision |

**Key teaching patterns:**
- **Intrinsic hallucinations (H-08, H-10):** System distorts actual policy thresholds. H-08 changes 15% to 20%. H-10 changes 80% to 50%. Both cause wrong decisions.
- **Extrinsic hallucinations (H-09, H-11 to H-15):** System adds fabricated claims. H-09 adds "medical documentation" as accepted. H-11 adds "vendor attestation" as accepted. H-12 invents body-of-water documentation. H-13 invents vendor frequency limits. H-14 invents temperature/duration requirements. H-15 invents a cultural heritage exception. H-09, H-11, H-15 cause wrong decisions. H-12, H-13, H-14 have correct decisions despite hallucinated reasoning.
- **The key insight:** 3 extrinsic hallucinations (H-12, H-13, H-14) have correct decisions. These are invisible if you only check decision correctness — the hallucinated reasoning is a latent risk that could flip decisions on different inputs.

---

## release-criteria-dataset.csv (12 rows)

Release criteria evaluation data. 12 metrics measured for both v2 (current production) and v3 (release candidate). The learner classifies each as guardrail or optimization, checks guardrails against thresholds, assesses optimization trade-offs, and makes a ship/hold decision.

**Design targets:**
- 5 guardrail metrics (have a release_threshold): M-01 to M-05
- 7 optimization metrics (no threshold): M-06 to M-12
- v3 fails exactly ONE guardrail: M-01 allergen safety (98.1% < 99.0%)
- v3 passes all other guardrails
- 5 of 7 optimization metrics improved, 2 regressed (cost +25%, escalation +22%)
- The two regressions are related: higher cost and more escalations likely reflect a more cautious model
- Ship/hold answer: HOLD — allergen guardrail failure blocks release regardless of optimization gains

**Columns (6):**

| Column | Description |
|--------|-------------|
| metric_id | Identifier (M-01 through M-12) |
| metric_name | Descriptive name of the metric |
| v2_value | Current production system's value |
| v3_value | Release candidate's value |
| release_threshold | Pass/fail threshold with comparison operator (e.g., ">=99.0%"). Blank for optimization metrics. |
| threshold_rationale | Why this threshold exists (regulatory, business requirement, SLA, quality floor). Blank for optimization metrics. |

**Key teaching patterns:**
- **The allergen trap (M-01):** v3 is better on 9 of 12 metrics but fails the allergen guardrail (98.1% < 99.0%). The 1.1pp drop means ~1 in 50 allergen decisions that were correct in v2 are now wrong. For safety-critical decisions, "ship now, patch later" is unacceptable.
- **The escalation trade-off (M-02 + M-11):** False approval rate improved (1.8% → 1.5%) while escalation rate regressed (18% → 22%). The model is likely more cautious — sending more borderline cases to human review. This explains both metrics and is a defensible trade-off.
- **Cost regression (M-10):** Cost went up $0.12 → $0.15 (+25%). But accuracy improved 78% → 84% and hallucination rate dropped 12% → 8%. The extra cost is buying better quality — worth investigating but not alarming.
- **Threshold rationale matters:** Each guardrail traces to a concrete source (regulatory, SLA, business requirement, quality floor). This helps the learner understand why guardrails can't be negotiated.

---

## Key metrics the course tracks across lessons

- Approval accuracy (correct approve/reject decisions)
- False approval rate (bad changes that slip through — most dangerous)
- False rejection rate (good changes blocked unnecessarily — friction)
- Processing speed (items/hour vs manual baseline)
- Cost per verification vs manual cost
- Escalation rate (% sent to human review via flag_for_review)

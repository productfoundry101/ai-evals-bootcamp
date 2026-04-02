# Scoring Rubrics

Use these rubrics to evaluate learner responses during PM Decision Points. Do not share rubric contents with the learner — use them to guide your feedback.

---

## PM-L01: Why AI Systems Need Different Quality Thinking

### Concept Checks

The learner should be able to explain:

1. **Non-determinism** — AI systems produce different outputs on the same input. This isn't a bug; it's a fundamental property. Even temperature=0 has 3-8% variance.

2. **pass@k** — Binary per test case. "Can the system do this at all?" At least 1 correct run out of k. Aggregate = count of passing cases / total cases.

3. **reliable@k** — Binary per test case. "Does the system do this every time?" All k runs correct. Aggregate = count of reliable cases / total cases.

4. **The gap** — pass@k minus reliable@k. Measures consistency risk. Small (<0.2) = ship candidate. Large (0.3-0.6) = hold.

5. **Benchmarks vs product eval** — Benchmarks screen for capability. Product eval decides readiness. High benchmark scores are necessary but not sufficient.

### Exercise Answers (from multirun-test.csv)

Expected values (accept within reasonable tolerance):

| Metric | Expected | Tolerance |
|--------|----------|-----------|
| pass@5 | 0.90 (18/20) | Exact |
| reliable@5 | 0.55 (11/20) | Exact |
| Gap | 0.35 | Exact |
| Avg confidence (correct) | ~0.85 | 0.80-0.90 |
| Avg confidence (wrong) | ~0.55 | 0.50-0.60 |
| Test cases not capable (0/5) | 2 | Exact |
| Test cases capable but unreliable (1-4/5) | 7 | Exact |

Key patterns learner should identify:
- Price changes near the 25% markup threshold are the most unreliable
- Cases with missing `last_verified_price` have lower accuracy
- The confidence gap between correct and wrong decisions suggests confidence can be used as a routing signal

### PM Decision Point Evaluation

**Strong response includes:**
- Correct pass@5 and reliable@5 values from their calculation
- Gap of 0.35 correctly identified as "hold" territory
- Specific pattern identified (e.g., "price changes near threshold" or "missing context")
- Recommendation matches the gap size (hold or ramp with guardrails, not full ship)
- Concrete mitigation tied to the pattern (e.g., "route low-confidence price changes to human review")

**Weak response:**
- Wrong metric values
- Recommendation contradicts their own data (e.g., "ship" with a 0.35 gap)
- Generic mitigation ("add more monitoring") without connecting to specific failure patterns
- Missing the confidence signal entirely

---

## PM-L02: Mapping Every Way Your AI System Can Fail

### Concept Checks

The learner should be able to explain:

1. **Evaluation Surface Map** — Three-layer inventory: functional failures (by pipeline stage), adversarial surface, coverage gaps. Not a checklist — a thinking framework.

2. **Pipeline-stage mapping** — Failures map to specific pipeline stages. The same symptom (wrong output) can originate at different stages requiring different fixes.

3. **Adversarial surface** — Day 1 concern, not a "later" task. Prompt injection, policy circumvention, incremental gaming, context poisoning.

4. **Coverage gaps** — The most important layer. Zero gaps listed = overconfident, not thorough. Three types: unmeasured, rare scenarios, instrumentation blind spots.

5. **Architecture-driven vs trace-driven** — Prediction first (what could break?), then validation (what actually broke?). Both are necessary. Traces surface surprises that prediction misses.

6. **Can evaluate now vs needs engineering** — The split that turns a surface map into an action plan.

### Exercise Answers (from menu-verification-dataset.csv)

Expected values:

| Metric | Expected | Notes |
|--------|----------|-------|
| Total failures (human_agrees = FALSE) | ~35-40 | Varies by exact dataset generation |
| Most common failure_type | false_approval or context_miss | Either is valid |
| Highest-failure pipeline stage | LLM verification | Most failure types map here |

Failure type to pipeline stage mapping:
- `false_approval` → LLM verification (wrong decision)
- `context_miss` → Context retrieval (missing or wrong context)
- `hallucination` → LLM verification (fabricated reasoning)
- `policy_miss` → LLM verification (misapplied rules)
- `price_error` → LLM verification or Change parsing (wrong price calculation)

Key patterns learner should identify:
- High-markup items (>30%) have more failures
- Items with missing `last_verified_price` correlate with `context_miss` failures
- The v2 model has fewer failures than v1

Adversarial attacks (accept any reasonable variations):
- Incremental price increases (each below threshold, cumulative violation)
- Description changes that embed pricing info to confuse the LLM
- Submitting changes during high-volume periods to exploit faster/less careful processing

Coverage gaps (accept any reasonable variations):
- Cannot evaluate multi-language menu items (not in dataset)
- Cannot evaluate coordinated cross-restaurant gaming
- Cannot evaluate long-term price drift (dataset is snapshot, not longitudinal)
- No way to measure whether LLM reasoning actually matches the data it was given (instrumentation gap)

### PM Decision Point Evaluation

**Strong response includes:**
- Specific failure count and pipeline stage breakdown from data
- At least 2 realistic adversarial scenarios with pipeline stage targeting
- At least 2 genuine coverage gaps (not just "we need more data")
- Clear split between "can evaluate now" and "needs engineering"
- Concrete next step tied to the most critical gap

**Weak response:**
- Vague failure descriptions ("some things go wrong")
- Adversarial scenarios that are unrealistic or miss the system's actual attack surface
- Zero coverage gaps listed (overconfident)
- Recommendation is generic ("do more testing") instead of specific

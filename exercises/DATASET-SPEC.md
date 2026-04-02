# Dataset Specifications

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

## Key metrics the course tracks across lessons

- Approval accuracy (correct approve/reject decisions)
- False approval rate (bad changes that slip through — most dangerous)
- False rejection rate (good changes blocked unnecessarily — friction)
- Processing speed (items/hour vs manual baseline)
- Cost per verification vs manual cost
- Escalation rate (% sent to human review via flag_for_review)

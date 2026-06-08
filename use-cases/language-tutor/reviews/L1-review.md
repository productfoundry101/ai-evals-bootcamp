# Lesson Review — L1 When Is a Problem an AI Problem? (language-tutor)

**Reviewed:** 2026-06-07
**Reviewer:** lesson-reviewer (Maven-instructor mode)

## First-Read Reaction

I came in cold off the onboarding and immediately knew why I was here: Nadia handed me a mandate, not a feature, and the first thing she wants is judgment about *which* problem deserves AI. That framing landed. The "was this problem true before the technology existed?" test is a genuinely sticky idea, and by the time I hit the 12-row CSV I knew exactly what to do. Where I started to drift was the exercise's middle — Step 2 reads like a worksheet of two sub-questions stacked on Step 1, and I wasn't sure whether I was producing a new artifact or just re-sorting the same list.

## Scores

| Dimension | Score | Key Factor |
|---|---|---|
| Concept Clarity & Accuracy | 8/10 | Crisp, beginner-pitched, sourced cleanly to Babbel §1 + R09 ch.4; the "tells are a lens, not a formula" caveat is a real touch |
| Learning-by-Doing | 7/10 | Step 1 (tag 12) and Step 3 (defend rejections) are real decisions with wrong answers; Step 2 is softer and the Decision Point overlaps it |
| Engagement & Narrative Pull | 8/10 | Nadia's mandate, the tiger team, "credibility comes from the pains you rejected" — alive and specific; a few sentences slip toward sermon |
| PM / Production-AI Relevance | 9/10 | Every beat is a call a real AI PM makes; "saying no is a skill" is the right opening posture and rarely taught |
| Gradeability of Rubric | 7/10 | Strong on Step 1 (named misses P05/P12/P07) and Decision Point red flags; Step 2 answer key asserts P01 as "the standout" more firmly than the CSV supports |
| **Overall** | **7.5/10** | |

## Publish Verdict

**Ship with targeted edits.**

This is a strong first lesson — better than most Act-1 openers I see, because it teaches *rejection* as the skill and refuses to leak the solution. It would land in a cohort. But two things need a pass before it ships: the exercise's Step 2 is fuzzier than Steps 1 and 3 (it's the only step that doesn't produce a defensible artifact, and it pre-overlaps the Decision Point), and the CSV's borderline cases (P03, P11) need the rubric to commit to a defensible answer or explicitly bless the variation — right now the rubric leans harder on P01-as-winner than the data forces, which will make a grader over-penalize a learner who reasons their way to P02 or P11.

## What Worked

- **The "was this problem true before the technology existed?" test** (lesson lines 17–19). This is the load-bearing idea of the whole lesson and it's taught with a concrete, sticky example ("beginners are too anxious to speak… was true ten years ago"). Traces directly to Babbel §1 and R09 ch.4. A learner will remember this.
- **"Saying no is a skill, not a failure"** (Part 1, lines 43–47). Most L1s teach "how to pick an AI problem." This one teaches that the credibility is in the rejections — exactly the product-sense pillar the story arc names. It reframes the whole exercise.
- **No-solution-leakage is honored.** The lesson never says "speaking agent," "conversation partner," or "AI tutor as the answer." P01 is framed only as a *problem* ("beginners avoid speaking out of fear"), and the Decision Point explicitly demands "stated as a problem (not a solution)." This is the single most important constraint for L1 and the lesson nails it.
- **The CSV has genuine distractors, not happy-path filler.** P05 (spaced repetition disguised as personalization) and P12 (placement quiz disguised as "which one for me") are exactly the traps that separate a learner who pattern-matches on the word "personalized" from one who applies the determinism tell. P04 (off-mandate but legitimately AI-shaped) is a different *kind* of distractor — AI-fit but mandate-fail — which is sophisticated for a beginner lesson.

## What Fell Flat

1. **Step 2 is the soft middle — it's the only exercise step without a defensible artifact, and it front-runs the Decision Point.** (Part 2, lines 87–96.) Step 1 produces 12 tags (gradeable). Step 3 produces two defended rejections (gradeable). Step 2 asks two discussion questions ("Which pains hit beginners?" / "Which is least served today?") and the *answer* to Step 2 — "P01, because it's beginner, AI-shaped, and unserved" — is identical to the Decision Point's deliverable. So the learner either does the real thinking in Step 2 (and the Decision Point becomes a copy-paste) or coasts through Step 2 (and the step taught nothing).
   - Why it matters: a Maven learner notices when a step is busywork between two real ones, and the Decision Point loses its punch if its answer was already extracted in Part 2.
   - Fix: turn Step 2 into a forcing artifact that *feeds* but doesn't *resolve* the Decision Point — a 2-column shortlist ("survives the mandate" / "AI-shaped but off-mandate, with the reason") that surfaces the P04 trap explicitly, and stops short of declaring a single winner. The "which is the one" call then belongs entirely to the Decision Point. Exact text in Top 3 #1.

2. **The rubric over-commits to P01 as "the standout" while the CSV gives a learner room to defend P02 or P11.** (scoring-rubrics.md, Step 2 answer key, lines 28–29; reinforced in Decision Point eval, line 40.) The rubric says "P01 is the least served today… the standout" and treats P02 as merely "a defensible runner-up." But P11 (pronunciation feedback, A0/A1 beginner, "no feedback on the learner's own pronunciation") is *also* AI-shaped, beginner, and essentially unserved — by the rubric's own logic it has equal claim. A grader following this rubric will dock a learner who builds a clean case for P11.
   - Why it matters: the rubric's job is to let two graders agree. Right now it telegraphs a "right answer" (P01) the data doesn't force, which punishes exactly the kind of independent reasoning L1 is trying to build.
   - Fix: make the rubric grade the *reasoning*, not the *pick*. State that P01, P02, and P11 are all defensible if the learner names segment + AI-fit + unserved-today; flag only picks that ignore the mandate (P04) or aren't AI-shaped (P05/P07/P12). Exact text in Top 3 #2.

3. **A handful of sentences slip from "senior PM talking to me" into sermon.** Lesson line 45: *"The PM who can look at an exciting idea and say 'AI shouldn't own this — a rule does it better' is more valuable than the one who says yes to everything."* And line 21: *"hold off on the 'how' until you've earned it."* These aren't wrong, but stacked together they tip from coaching into preaching, and a sharp learner feels lectured.
   - Why it matters: engagement comes from specificity, not aphorism. One "saying no is a skill" beat is memorable; three in a row reads like a LinkedIn post.
   - Fix: keep the "saying no" idea once, tied to the Nadia stakes, and cut the generalized restatements. (Low priority — copy edit, not structural.)

## Source Fidelity Check

- **Concepts traced to course-extension:**
  - "Problem valid before the tech existed" / "start from the problem, not the solution" → **Babbel_speak.md §1** ("The problem statement was valid *before* the technology existed to solve it. Franz: always start from the problem, not the solution") and **RESOURCE-CATALOG R09 ch.4 Problem-First Design** ("why it matters more in AI… six traps… scoping template 4.4 → Tutor L1–L3").
  - AI-shaped vs not-AI-shaped tells (open-ended / judgment-heavy / scales poorly / error-tolerant) → **R09 ch.4** and **C15 (why evals are hard)**, per story-arc §3 L1 tag "*(R09 ch.4; C15.)*" and §5.1.
  - "AI is non-deterministic / slippery, which is why evaluation exists" (lesson line 49) → **C15** orientation touch, consistent with story-arc §5.1 marking C15 "Light (orientation) → L1, L11."
  - Mandate (beginners A0/A1, activation/engagement, profitability) → **onboarding.md** + **Babbel §1** ("Beginner learners A0/A1").
- **Concepts I could NOT trace:** None. Every concept is grounded. The four AI-shaped tells are a reasonable beginner synthesis of R09 ch.4 rather than a verbatim list, but that's appropriate adaptation, not invention.
- **Story-arc slot match:** **Yes, exact.** Story-arc §3 L1 names concept ("which pains are AI-shaped vs not… a problem statement is valid before the tech exists, Babbel §1"), exercise ("Score a mixed list of learner pain points… reject the ones AI shouldn't own"), and AI output ("core unmet need framed as an AI opportunity + target segment beginners A0/A1, one-line rationale for why AI not a rule or a human"). The lesson delivers all three. The product-sense pillar (story-arc §2, §3) is present and foregrounded.

## Difficulty Calibration

- **`meta.md` declares:** Beginner ("Best for: PMs new to AI evaluation… intuitive, consumer-product entry point").
- **Lesson reads as:** Beginner, correctly calibrated.
- **Evidence:** No ML jargon (no "embeddings," "fine-tuning," "logits"); the tells are given in plain language; the CSV is consumer-product pains a non-technical PM can reason about; the non-determinism point is held to an *intuition* ("AI is powerful and slippery") rather than a distributions lecture — which matches the story-arc's deliberate cut of statistical depth (§5.2). A learner one level below could follow; one level above wouldn't feel patronized because the P05/P12 distractors require real discrimination.

## Exercise Audit

- **Support file:** `use-cases/language-tutor/exercises/L1-learner-pain-points.csv` (12 rows, 5 columns: pain_id, learner_pain, segment, how_common, current_handling).
- **Does the data actually surface the failure patterns the exercise diagnoses?** **Yes.** The CSV is well-constructed for this exercise:
  - Clear AI-shaped: P01 (speaking anxiety), P02 (mid-lesson Q&A), P03 (does this sound natural), P04 (debate), P11 (pronunciation feedback).
  - Clear not-AI-shaped with distinct anti-tells: P06 (money/low-error → rules+UX), P07 (engineering), P08 (pricing), P09 (accreditation/human), P10 (settings/rule), P05 (deterministic spaced-repetition disguised as personalization), P12 (deterministic placement disguised as "which one for me").
  - Two *kinds* of distractor exist, which is what makes the exercise teach: (a) not-AI-shaped pains that *sound* AI-ish (P05, P12), and (b) genuinely AI-shaped pains that *fail the mandate* (P04, off-mandate B2+/3%). The `segment` and `current_handling` columns are load-bearing — they're what lets Step 2/Decision Point separate P01/P02/P11 from P04 and rank by "unserved." This is a strong dataset.
  - Minor: P03's segment is A1/A2, putting it just outside the A0/A1 beginner core — fine as a "could-argue-either-way" case, but the rubric should name it as such (it currently says "can be argued either way" for AI-shape but doesn't address its segment-edge for the mandate filter).
- **Step-by-step learning-by-doing check:**
  - Step 1 (tag P01–P12 AI/NO): **doing** — a real classification with wrong answers; the P05/P12/P07 traps make it non-trivial.
  - Step 2 (filter by mandate, two discussion questions): **soft / partly reading** — no committed artifact; its answer overlaps the Decision Point. This is the weak step.
  - Step 3 (defend two rejections with anti-tell + better tool): **doing** — produces a defensible artifact; this is the muscle the lesson promises.
- **Decision Point deliverable:** **Named and concrete.** "Write a short AI-opportunity framing for Nadia: (1) the single unmet need as a problem with segment named, (2) one-line why-AI rationale, (3) one pain explicitly rejected with reason." This is a real artifact a PM hands up, and the "stated as a problem, not a solution" constraint makes it gradeable. Good. Its only weakness is the Step 2 overlap (see What Fell Flat #1).

## Rubric Audit

- **Concept checks distinguish strong from weak?** **Yes.** The three concept checks (problem-before-solution, AI-shaped tells, saying-no) are specific and each carries a discriminating example (the "litmus test," the named misses, the leadership-conviction trap). A grader can tell a learner who internalized "the problem must predate the tech" from one who paraphrases.
- **Exercise answer key covers common misses?** **Yes, strongly for Step 1.** It names the two highest-value traps (P05/P12 "personalized-sounding but deterministic") and the P07 "real impact but wrong tool" miss explicitly — exactly the "common prediction miss" the format asks for. Step 3's "reject any rejection that just says 'AI can't do it' with no anti-tell" is a clean weak-response anchor. **Partially for Step 2:** the key asserts P01 as "the standout" more firmly than the CSV forces, under-crediting defensible P02/P11 cases (see What Fell Flat #2).
- **Decision Point criteria gradeable by two independent graders?** **Mostly yes.** The "weak response" list is excellent and concrete — "jumps to a solution (an AI chat partner)" as the top red flag is the single most important grading signal for L1, and naming the segment "everyone" as a tell is sharp. The one gap: the strong-response bullet "Most converge on P01; P02 acceptable if reasoning is sound" should be widened to bless P11 too, so two graders don't disagree on a strong P11 case.

## Generic / AI-Sounding Language Flagged

The lesson is largely clean of AI-instructor tells — no *delve, leverage, robust, streamline, comprehensive, navigate the landscape*. Only minor flags:

- Lesson line 41: *"You're building judgment, not running a calculator."* — Fine, keep. (Not a tell; it's a good human line.)
- Lesson line 45: *"...is more valuable than the one who says yes to everything."* — Slight aphorism creep; see What Fell Flat #3. Human rewrite: tie it to stakes — *"Nadia will trust the PM who tells her a rule beats AI here more than the one who green-lights everything she's excited about."*
- Lesson line 21: *"hold off on the 'how' until you've earned it."* — "earned it" is doing a lot of work twice (also implied in line 5's "you'll be able to"). Acceptable, but if trimming, this is a candidate.

No perfect-parallel-structure violations, and the concept sections read as prose, not slides (the AI-shaped/not-AI-shaped bullets are appropriate here — they're a checklist the learner applies in Step 1, not a concept-explanation dump).

## Continuity & Voice Check

- **Prior lessons referenced by number AND title?** **Yes / N/A.** This is the first lesson (header correctly says "Previous lesson: None"). No dangling references.
- **Voice matches recent lessons in this use case?** **Yes, and improves on the comparator.** Against menu-verification D1-Pipeline-Mapping: same three-part Concept → Exercise → Decision structure, same "present this exactly" interactive-tagging pattern in Step 1 (mirrors D1's letter-key column mapping), same header format (Week/Day, part times, previous-lesson orientation). The language-tutor lesson is *more* narratively alive than D1 (D1's "you've just been assigned as PM" is thinner than Nadia's mandate), which is appropriate for a consumer-product entry point. One small consistency note: D1 includes a "Worked example" before the real dataset (the insurance-claims table); L1 has no worked example before Step 1. Not required, but a single worked tag ("P-example: a billing bug → NO, because…") would lower the activation energy for a first-ever lesson. Optional.

## Strongest Element

The CSV's two-tier distractor design. It's not just "some AI, some not" — it has pains that *sound* AI but aren't (P05, P12) and pains that *are* AI but fail the mandate (P04). That second tier is what elevates this from a beginner sorting task to a genuine product-judgment exercise, and it's what makes the Decision Point's "why this and not that" defensible. Keep building datasets with this structure.

## Weakest Element

The Step 2 → Decision Point overlap. Fixing it (giving Step 2 its own non-resolving artifact and reserving the single-need call for the Decision Point) is the single change that most raises the lesson: it removes the soft middle of the exercise, restores the Decision Point's stakes, and forces the learner to commit rather than recall. Everything else is a copy edit by comparison.

## Top 3 Improvements (priority order)

1. **Target:** lessons/L1-When-Is-A-Problem-An-AI-Problem.md
   **Location:** Part 2, "Step 2: Filter the AI-shaped pains by the mandate" (lines 87–96)
   **Change type:** restructure
   **Exact change:** Replace the two-discussion-question step with a forcing artifact that feeds, but does not resolve, the Decision Point. New Step 2 body:
   > ### Step 2: Build your mandate shortlist
   > Being AI-shaped isn't enough. Nadia's mandate is specific: **lift activation and engagement, for beginners, in service of profitability.** Take only the pains you tagged `AI` and sort them into two buckets — don't pick a winner yet, just separate them:
   >
   > ```
   > Of the pains you tagged AI, sort each into one bucket:
   >
   >   SURVIVES  = beginner segment AND plausibly moves activation/engagement
   >   OFF-MANDATE = AI could help, but it's the wrong audience or too small to matter
   >
   > For each OFF-MANDATE pain, add one line: why it's off-mandate (segment? size?).
   > For each SURVIVES pain, note its current_handling — is anything addressing it today?
   > ```
   >
   > One pain will tempt you: it's genuinely AI-shaped and interesting, but it serves the wrong audience. Setting it aside *even though AI could nail it* is the discipline — AI-fit is not the same as mandate-fit. Hold your bucketed list; you'll use it at the Decision Point to make the final call.
   This keeps the P04 trap explicit, produces a defensible artifact, and reserves the single-need choice for Part 3.

2. **Target:** scoring-rubrics.md § L1
   **Location:** "Exercise Answers" → "Step 2 — Filter by the mandate" (lines 28–29) and "PM Decision Point Evaluation" → strong-response bullet (line 40)
   **Change type:** replace
   **Exact change:** Replace the Step 2 paragraph (lines 28–29) with: "The AI-shaped pains that survive the mandate (beginner + activation/engagement) are **P01, P02, and P11** — all three are A0/A1, AI-shaped, and currently underserved. **P04 is the off-mandate trap:** AI-shaped but B2+ (~3% of users); a strong learner sets it aside *because it fails the mandate, not because AI can't help*. Grade the *reasoning*, not the pick: P01, P02, and P11 are each defensible as the single need if the learner names (a) the beginner segment, (b) the AI-shaped fit, and (c) how little serves it today. P01 has the strongest case (~65% prevalence, current_handling = 'no dedicated support'), but a clean P02 or P11 argument is fully acceptable. Flag a Step 2 answer only if it keeps P04 (off-mandate) or any not-AI-shaped pain (P05/P07/P12) in the shortlist." Then in line 40, replace "Most converge on P01; P02 is acceptable if the segment/impact/unserved reasoning is sound." with "P01, P02, or P11 are all strong picks if the segment + AI-fit + unserved reasoning is sound. Do not penalize a defensible P02 or P11 case in favor of P01."

3. **Target:** lessons/L1-When-Is-A-Problem-An-AI-Problem.md
   **Location:** Part 1, "Product sense means being willing to say no," sentence at line 45
   **Change type:** replace
   **Exact change:** Replace "The PM who can look at an exciting idea and say \"AI shouldn't own this — a rule does it better\" is more valuable than the one who says yes to everything. Saying no is how you avoid burning months building something a fifty-line script would have beaten." with: "Nadia will trust the PM who tells her \"a fifty-line script beats AI here\" more than the one who green-lights every idea she's excited about. The rejection is the proof you were thinking, not just shipping." This ties the abstraction to the named stakes and removes the stacked-aphorism feel.

---

## Revision Log

**2026-06-07 — Pass 1 verdict:** Ship with targeted edits (Overall 7.5/10).

**Applied (all 3 Top improvements):**
1. Lesson Part 2 Step 2 restructured into the SURVIVES / OFF-MANDATE bucketing artifact; single-need call reserved for the Decision Point.
2. Rubric § L1 Step 2 answer key + Decision Point strong-response bullet rewritten to grade reasoning over pick (P01/P02/P11 all defensible; flag only P04 / non-AI-shaped shortlists).
3. Lesson Part 1 "saying no" aphorism replaced with the Nadia-stakes line.

**2026-06-07 — Pass 2 (verification):** Ship as-is. All three flagged issues confirmed resolved; no new issues raised (verification pass only). Two reviewer passes total — loop closed.

**2026-06-07 — Narrative realignment (root-cause fix):** The earlier bridge softened wording but didn't dissolve the underlying contradiction — the onboarding made AI the *subject* of the search ("find where AI can address learner pain") and pre-committed to AI, which collides with L1's "don't start from 'we should use AI'." Resolved at the source by reframing the tiger team as an innovation team whose *purpose* is to find AI's place (and set the org's example), while the *method* stays problem-first: start from learner pain and find the problems AI can **uniquely** solve; prioritization is the PM's call. Changes: (1) onboarding "THE CONVICTION" + mandate rewritten (problem-first, outcome-judged, PM prioritizes); (2) L1 opening rewritten to match; (3) L1 bridge replaced with the "don't hunt problems for a predetermined AI solution, but some problems are uniquely AI-solvable — find those" framing. Supersedes the bridge from the prior entry.

**2026-06-07 — Post-ship learner-flagged revision:** A learner flagged an apparent self-contradiction in Part 1 — the opening framed the mandate as "find where AI can move activation and engagement" (reads solution-first) immediately before the section warning against "starting from 'we should use AI' and hunting for a problem to justify it." Fix applied: (a) re-anchored the opening mandate on "address learner pain" (faithful to onboarding) and reframed "which problem to point AI at" → "which learner problem is worth solving — and whether AI is the right tool"; (b) inserted a bridge paragraph naming and reconciling the tension (outcome + AI-as-hypothesis vs. a locked-in build you reverse-engineer; "the conviction sets where you look; it doesn't decide what you find"). Focused verification pass: Resolved — opening now reads problem-first, bridge is consistent with the onboarding and L1's "willing to say no" teaching, no new issues introduced.

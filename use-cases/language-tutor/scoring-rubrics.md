# Scoring Rubrics — Conversational Language Tutor

> Each rubric entry below is for the LLM grader, not the learner. It defines what a strong vs. weak response looks like for that lesson's concepts, exercise, and PM Decision Point.

---

## L1 - When Is a Problem an AI Problem?

### Concept Checks

The learner should be able to explain:

1. **Problem before solution** — A valid problem statement stands on its own, independent of any technology. The litmus test: was the problem true *before* the tech to solve it existed? "Beginners avoid speaking out of fear" passes; "learners should chat with an AI" fails (it's a solution wearing a problem's clothes). Starting from "we should use AI" and hunting for a justification is the most common way AI projects go wrong.

2. **What makes a problem AI-shaped** — AI-shaped tells: open-ended (no fixed input/output list), language/judgment-heavy, scales poorly with humans, tolerant of some imperfection. Not-AI-shaped tells: deterministic with an exact answer (a rule wins), low tolerance for error (money/legal/safety), really an engineering/pricing/policy problem, or genuinely needs a human (trust/accreditation). The tells are a lens for judgment, not a scoring formula.

3. **Product sense means saying no** — Rejecting a problem for AI is a skill, not a failure. Credibility comes as much from the pains you reject (and why) as from the one you pick. Especially important when leadership is already convinced AI belongs — nobody will stop you from pointing AI at the wrong problem; that judgment is the PM's job.

### Exercise Answers (from L1-learner-pain-points.csv)

**Step 1 — AI-shaped vs. not (expected tags):**

- **AI-shaped:** P01 (speaking anxiety — open-ended, judgment-heavy, scales poorly with humans, error-tolerant), P02 (no one to ask mid-lesson — open-ended Q&A, judgment), P03 ("does this sound natural?" — language/judgment-heavy, no exact key), P04 (advanced debate practice — open-ended, judgment-heavy), P11 (pronunciation feedback — perception/judgment, scales poorly with humans).
- **Not AI-shaped:** P05 (which words to review/when → spaced-repetition is a deterministic algorithm; a *rule* wins, even though it sounds like "personalization" — common miss), P06 (billing/cancel → deterministic + low error tolerance because it's money; rules + UX), P07 (app crashes on old phones → engineering/performance), P08 (free-tier cap frustration → pricing/business decision), P09 (accredited certificate → needs a human/institution; trust + exactness), P10 (notification timing → settings/rule), P12 (which level to start → a placement quiz scores deterministically; a rule, not AI — also looks like personalization).

Accept reasonable variation on the borderline cases: P04 and P11 are legitimately AI-shaped, and P03 can be argued either way. The two most common *misses* to watch for: tagging P05 and P12 as AI-shaped because the word "personalized"/"which one for me" sounds AI-ish — both have exact answers a rule computes. Also watch for someone tagging P07 (app crashes) as AI just because it hurts activation; impact is real but AI is the wrong tool.

**Step 2 — Filter by the mandate (beginners, activation/engagement, currently unserved):**
The AI-shaped pains that survive the mandate (beginner + activation/engagement) are **P01, P02, and P11** — all three are A0/A1, AI-shaped, and currently underserved. **P04 is the off-mandate trap:** AI-shaped but B2+ (~3% of users); a strong learner sets it aside *because it fails the mandate, not because AI can't help*. Grade the *reasoning*, not the pick: P01, P02, and P11 are each defensible as the single need if the learner names (a) the beginner segment, (b) the AI-shaped fit, and (c) how little serves it today. P01 has the strongest case (~65% prevalence, current_handling = "no dedicated support"), but a clean P02 or P11 argument is fully acceptable. Flag a Step 2 answer only if it keeps P04 (off-mandate) or any not-AI-shaped pain (P05/P07/P12) in the shortlist.

**Step 3 — Defend two rejections:** Strong rejections name a specific anti-tell *and* the better tool: e.g., P06 → low error tolerance (money) → rules + clear UX; P07 → engineering problem → eng team; P05/P12 → deterministic/exact answer → a rule or quiz. Reject any "rejection" that just says "AI can't do it" with no anti-tell or alternative.

### PM Decision Point Evaluation

**Strong response includes:**
- Names a single unmet need **stated as a problem, not a solution** (e.g., "beginners are too anxious to speak the language out loud, so they avoid it" — NOT "build a speaking bot"). Solution-naming is a red flag here: the whole point of L1 is to stay in the problem space.
- Names the **target segment** explicitly (A0/A1 beginners) and ties it to the mandate (activation/engagement).
- Gives a crisp **why-AI rationale** grounded in the AI-shaped tells (open-ended, judgment-heavy, scales poorly with humans, error-tolerant) and contrasts it with why a rule/engineering fix/human would *not* do the job.
- Explicitly **rejects one pain** for AI with a specific anti-tell and the better alternative tool.
- P01, P02, or P11 are all strong picks if the segment + AI-fit + unserved reasoning is sound. Do not penalize a defensible P02 or P11 case in favor of P01.

**Weak response:**
- Jumps to a solution ("an AI chat partner," "a conversation agent") instead of framing the problem — the most common and most important failure to flag.
- Picks an off-mandate or non-AI-shaped pain (e.g., P04, P07, P12) without acknowledging the mismatch.
- Gives a why-AI rationale that's really "because AI is cool / leadership wants AI" rather than the structural tells.
- Omits the rejection, or rejects something with no anti-tell and no alternative tool named.
- Names the segment vaguely ("everyone") rather than beginners.

# L1 - When Is a Problem an AI Problem?

**Week 1, Day 1** | ~32 min
**Part times:** Concepts ~12 min | Exercise ~13 min | Decision Point ~7 min
**Previous lesson:** None — first lesson. Orient the learner: this track teaches AI product judgment and evaluation by building one AI feature end to end at Lingo. By the end of Day 1 you'll be able to look at a list of real learner problems and tell which ones AI is genuinely the right tool for — and, just as importantly, which ones it isn't.

---

## Part 1: The Concepts

You're a week into the tiger team — the innovation group leadership stood up to figure out where AI genuinely helps Lingo, and to set the example for how product teams here build AI features that move real numbers. Nadia gave you a mandate, not a feature: start from learner pain and find the problems AI can *uniquely* solve — and be just as clear about the pains it shouldn't touch. What to prioritize is your call. Before you can build anything, you have to decide which problems those are. That's today's lesson.

### Start with the problem, not the solution

The most common way AI projects go wrong isn't bad engineering. It's starting from "we should use AI" and then hunting for a problem to justify it. That's backwards, and it's especially tempting right now, when leadership is convinced AI belongs at Lingo and everyone's eager to ship *something*.

It's worth being precise here, because "go find where AI helps" and "start from the problem" can sound like opposites. They're not. The trap is starting from "we should use an AI solution" and then hunting for a user problem to match it — that's how teams ship clever features nobody needed. But the opposite mistake is to treat AI as just another tool in the box. It isn't: there are real user problems that *only* AI can solve well — open-ended, language-heavy, judgment-heavy problems that rules and scripts have failed at for years. Your team's mandate is precisely to find those. So you still start from learner pain; you just look for the pains where AI is *uniquely* the right answer — not because someone told you to use AI, but because for that specific problem, nothing else works as well.

The discipline that protects you is simple to say and hard to do: start from the problem. A good problem statement stands on its own, completely independent of any technology. Here's a test that sounds almost too obvious — *was this problem true before the technology to solve it existed?* Take "beginners are too anxious to speak the language out loud, so they avoid it entirely." That was true ten years ago. It was true before anyone could build a convincing conversation partner in software. The problem didn't appear because a new tool appeared. That's the signature of a *real* problem — and the only kind worth building on.

Compare that to "learners should be able to chat with an AI." That's not a problem statement. It's a solution wearing a problem's clothes. It only exists because the technology exists. If you build from it, you'll ship something impressive that nobody needed.

So your first job on the tiger team isn't to imagine features. It's to get clear-eyed about which learner pains are real, durable, and unsolved — and hold off on the "how" until you've earned it.

### What makes a problem "AI-shaped"

Not every real problem is an AI problem. Some are solved better, cheaper, and more reliably by a simple rule, a UX change, an engineering fix, or a human. Part of your job is telling them apart.

A problem is **AI-shaped** when it has these tells:

- **It's open-ended.** There's no fixed list of valid inputs or outputs. A learner can say almost anything; a good response depends on context, not a lookup table.
- **It's language- or judgment-heavy.** Solving it well requires interpreting meaning, tone, or nuance — the kind of thing that's easy for a person and historically very hard for code.
- **It scales poorly with humans.** Doing it well for one learner is fine; doing it for a million means hiring an army you can't afford. (Sound familiar? That's Nadia's whole mandate.)
- **It tolerates some imperfection.** An occasional imperfect response is survivable. The learner isn't harmed by a slightly-off answer the way they'd be harmed by a wrong charge on their card.

A problem is **not** AI-shaped — and you should say so — when it shows the opposite tells:

- **It's deterministic and has an exact answer.** There's one correct output and you can write the rule for it. A rule will always be cheaper and more reliable than a model.
- **It has low tolerance for error.** Money, legal status, safety, accreditation. When being wrong is expensive or harmful, you want certainty, not a probabilistic guess.
- **It's really an engineering, pricing, or policy problem.** A crashing app, a paywall, a billing flow — these are pains, sometimes big ones, but AI is the wrong tool. They need a different team.
- **It genuinely needs a human.** Trust, accreditation, empathy in a high-stakes moment — sometimes the honest answer is that a person should own it.

These tells aren't a scoring formula to apply mechanically. They're a lens. A pain can have one or two AI tells and still be the wrong fit because it fails on the mandate (wrong audience, tiny impact) or because a much simpler tool does the job. You're building judgment, not running a calculator.

### Product sense means being willing to say no

Here's the part most people skip. *Rejecting* a problem for AI is a skill, not a failure. Nadia will trust the PM who tells her "a fifty-line script beats AI here" more than the one who green-lights every idea she's excited about. The rejection is the proof you were thinking, not just shipping.

This matters more than usual right now because the conviction is already in the room. Nadia, the board, the whole company believes AI can help. That belief is your starting line — but it also means nobody's going to stop you from pointing AI at a problem it has no business touching. That's your job. When you bring Nadia a recommendation, the credibility comes as much from the pains you *rejected*, and why, as from the one you chose.

One more thing worth knowing early, because it shapes everything in this course: AI systems are hard to pin down. The same input can produce different outputs, and "is it good?" rarely has a clean yes/no answer the way traditional software does. That's exactly why a whole discipline — evaluation — exists, and it's what the rest of this track teaches. For today, just hold the intuition: AI is powerful *and* slippery, which is all the more reason to only aim it at problems where it's truly the right tool.

---

## Part 2: Exercise — Score the Pain Points

The tiger team has spent its first week pulling together a backlog of the most common things learners struggle with — scraped from support tickets, drop-off surveys, and app reviews. Nadia wants your read: of all these pains, which are actually AI's to solve?

### Setup

Open `use-cases/language-tutor/exercises/L1-learner-pain-points.csv`. It has 12 rows — one per learner pain — and these columns:

- `pain_id` — a short identifier (P01–P12)
- `learner_pain` — the problem, in the learner's terms
- `segment` — who it mainly affects (A0/A1 beginner, intermediate, advanced, all)
- `how_common` — roughly how widespread it is
- `current_handling` — how Lingo deals with it today

Open it in a spreadsheet so you can see the whole picture as we work. Your goal across the next three steps: separate the AI-shaped pains from the rest, then narrow to the one most worth pursuing given Nadia's mandate.

### Step 1: Tag each pain — AI-shaped or not?

Go through the 12 pains and, for each, decide whether it's **AI-shaped** (open-ended, language/judgment-heavy, scales poorly with humans, tolerant of some imperfection) or **not AI-shaped** (deterministic, exact-answer, low error-tolerance, or really an engineering / pricing / human problem).

Present this exactly:

```
Go through P01–P12 and tag each one:

  AI  = AI-shaped (open-ended, judgment-heavy, scales poorly with humans, error-tolerant)
  NO  = not AI-shaped (deterministic / exact answer / low error tolerance / engineering / pricing / human)

Reply with 12 tags, in order. Example format:
P01: AI   P02: AI   P03: NO   ...
```

Once the learner responds, walk through their tags. Confirm the ones they got right and correct any misses with a one-line reason tied to a specific tell — e.g., "P05 looks like personalization, but 'which words to review and when' has an exact answer a spaced-repetition rule computes; a model isn't needed." Don't re-quiz; correct and move on.

### Step 2: Build your mandate shortlist

Being AI-shaped isn't enough. Nadia's mandate is specific: **lift activation and engagement, for beginners, in service of profitability.** Take only the pains you tagged `AI` and sort them into two buckets — don't pick a winner yet, just separate them.

Present this exactly:

```
Of the pains you tagged AI, sort each into one bucket:

  SURVIVES    = beginner segment AND plausibly moves activation/engagement
  OFF-MANDATE = AI could help, but it's the wrong audience or too small to matter

For each OFF-MANDATE pain, add one line: why it's off-mandate (segment? size?).
For each SURVIVES pain, note its current_handling — is anything addressing it today?
```

One pain will tempt you: it's genuinely AI-shaped and interesting, but it serves the wrong audience. Setting it aside *even though AI could nail it* is the discipline — AI-fit is not the same as mandate-fit. Confirm the learner's buckets, then make sure each SURVIVES pain has its `current_handling` noted. Don't let them name a single winner here — that call belongs to the Decision Point. They should leave this step holding a short bucketed list, not a decision.

### Step 3: Defend two rejections

Pick two pains you tagged `NO`. For each, name *which* anti-tell makes it a poor fit for AI, and what the better tool actually is (a rule, a UX change, an engineering fix, a pricing decision, a human). This is the muscle Nadia will trust you for — being able to say "not this, and here's why."

---

## Part 3: PM Decision Point

Nadia stops by your desk: "Before this team spends a single engineering hour, I want one page from you. Of everything learners struggle with, what's the *one* unmet need we should point AI at — and why AI, and not something simpler?"

Write a short **AI-opportunity framing** for Nadia that includes: (1) the single unmet need you'd pursue, stated as a problem (not a solution), with the target segment named; (2) a one-line rationale for why AI — not a rule, an engineering fix, or a human — is the right tool for it; and (3) one pain you are explicitly *rejecting* for AI, with the reason. Keep it to a few sentences — this is the opening move of your case, not the whole pitch.

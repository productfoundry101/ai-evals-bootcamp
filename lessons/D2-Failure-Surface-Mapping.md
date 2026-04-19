# D2 - Failure Surface Mapping

**Week 1, Day 2** | ~27 min
**Part times:** Concepts ~10 min | Exercise ~12 min | Decision Point ~5 min
**Previous lesson:** D1 - Pipeline Mapping — you mapped the 5 pipeline stages, traced a single row end-to-end, and identified what's invisible in the logs. Key insight: the same wrong output can come from different pipeline stages and requires different fixes. Today you build on that — instead of tracing individual rows, you'll map every way each stage can fail, including failures that haven't happened yet and attacks that haven't been tried.

---

## Part 1: The Concepts

### The three layers of an evaluation surface map

When you inherit an AI product, you need a structured answer to two questions your stakeholders will always ask: "What are all the ways this can fail?" and "What haven't we tested yet?" An **evaluation surface map** answers both. It has three layers:

**Layer 1: Functional failures** — what breaks during normal use?

Map these by pipeline stage. Every stage has its own failure modes that require different fixes:

- **Input parsing** breaks when the system misinterprets or misclassifies what the user is asking
- **Context retrieval** breaks when relevant information isn't found, or irrelevant information is pulled in
- **LLM reasoning** breaks when the model ignores context, applies the wrong rules, or fabricates information
- **Output generation** breaks when confidence scores are miscalibrated or explanations contradict the decision
- **Human review** breaks when reviewers apply inconsistent criteria or miss systematic patterns

A wrong output could originate at any of these stages. The surface map forces you to think about each one separately.

**Layer 2: Adversarial surface** — how could users exploit the system?

This isn't a security review to schedule later. It belongs on Day 1. Any system with external users has adversarial risk. Categories to consider:

- **Prompt injection** — crafted inputs designed to override the system's instructions
- **Policy circumvention** — input patterns that consistently bypass intended rules
- **Incremental gaming** — small changes that individually pass checks but collectively violate intent
- **Context poisoning** — manipulating what gets retrieved to influence the model's reasoning

Research on indirect prompt injection (Greshake et al., 2023) found success rates above 80% in controlled studies. These aren't theoretical. A system shipped without adversarial surface mapping is a system waiting to be exploited.

**Layer 3: Coverage gaps** — what can't you measure yet?

This is the layer teams skip, and it's the most important. Three types:

- **Unmeasured failure modes** — failures you suspect exist but can't confirm with current data
- **Rare scenarios** — edge cases not represented in your test data (unusual inputs, low-frequency categories)
- **Instrumentation blind spots** — behaviors you literally cannot observe because the logging isn't there

**A surface map with zero coverage gaps listed is almost certainly overconfident, not thorough.** Every real system has blind spots. Naming them is the honest work.

### Architecture-driven vs trace-driven discovery

There are two ways to find failure modes, and you need both:

**Architecture-driven prediction** — examine each pipeline stage's inputs, outputs, and dependencies. Ask: what could go wrong here? This requires no data. It catches failures before they happen. For a context retrieval stage, you can predict: wrong documents retrieved, relevant documents missed, stale data returned, empty results. Four failure categories without opening a single trace.

**Trace-driven validation** — inspect actual production data to confirm predictions and find surprises. A clinical text-to-SQL system predicted retrieval would be the primary bottleneck. Traces showed generation failures accounted for 78% of all failures — the opposite of the prediction. Traces surface what architecture analysis cannot.

The discipline: **predict first, always.** Teams that skip to traces only find what's already broken. Teams that predict first also find what's about to break.

### The split that makes it actionable

Once you have your surface map, split everything into two columns:

- **Can evaluate now** — failure modes you can measure with current instrumentation
- **Needs engineering first** — failure modes that require better logging before evaluation can begin

The "needs engineering" column becomes your instrumentation roadmap. This is how a surface map becomes an action plan — not just a list of risks, but a prioritised engineering backlog.

---

## Part 2: Exercise — Build Your Evaluation Surface Map

You're preparing for a launch readiness review of the menu verification system. Your VP will ask: "What are all the ways this system can fail?" and "What haven't we tested yet?"

### Setup

Open `exercises/D2-menu-verification-dataset.csv`.

Relevant columns: `change_type`, `llm_decision`, `human_decision`, `human_agrees_with_llm`, `failure_type`, `last_verified_price`

### Step 1: Predict before you look

Before filtering anything, fill in this table from architecture alone. For each pipeline stage, predict at least 2 failure modes:

| Pipeline stage | Predicted failure mode 1 | Predicted failure mode 2 |
|---|---|---|
| Input parsing | ___ | ___ |
| Context retrieval | ___ | ___ |
| LLM reasoning | ___ | ___ |
| Output generation | ___ | ___ |

### Step 2: Validate with traces

Now filter to rows where `human_agrees_with_llm` = `FALSE`. Count by `failure_type`:

| failure_type | Count |
|---|---|
| false_approval | ___ |
| context_miss | ___ |
| hallucination | ___ |
| policy_miss | ___ |

Two failure types definitionally map to a stage: `context_miss` = retrieval failed, `hallucination` = LLM fabricated something. The others (`policy_miss`, `false_approval`) describe symptoms — the root cause stage requires reading individual traces to confirm. Which failure type surprised you most vs your predictions?

### Step 3: Identify coverage gaps

List 2 things you **cannot** evaluate with the current dataset — be specific about why:

| Gap type | What's missing | What you'd need to measure it |
|---|---|---|
| Instrumentation blind spot | ___ | ___ |
| Rare scenario | ___ | ___ |

---

## Part 3: PM Decision Point

Your VP asks: "What are all the ways this system can fail, and what haven't we tested?"

Write a one-paragraph surface map summary covering:
1. **The risk picture** — which pipeline stage has the most failures and what type, plus one adversarial risk worth flagging
2. **The honest gap** — the most critical thing you can't measure yet and what it would take to fix that

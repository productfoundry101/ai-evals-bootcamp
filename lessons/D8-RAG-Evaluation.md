# D8 - RAG Evaluation

**Week 2, Day 8** | ~45 min
**Part times:** Concepts ~15 min | Exercise ~22 min | Decision Point ~8 min
**Previous lesson:** D7 - Golden Datasets — you learned how to build ground truth that makes all graders meaningful: three sources (automated verifiers, expert annotation, user feedback), how to construct a golden dataset, contamination patterns to avoid, and how datasets evolve over time. Today you apply all of that to a harder problem: evaluating a RAG system, where failures can come from two completely different stages — and an aggregate metric that looks fine can hide that both stages are broken.

---

## Part 1: The Concepts

### Why RAG evaluation is different

Most AI systems you evaluate have one failure point: the LLM got it wrong. A RAG system has two.

**Retrieval-Augmented Generation (RAG)** is a two-stage pipeline. Stage 1: a **retriever** searches a knowledge base and fetches the most relevant documents for a query. Stage 2: a **generator** (the LLM) uses those retrieved documents as context to produce an answer. The retrieved context is supposed to ground the LLM in authoritative, up-to-date information that the model doesn't have baked into its weights.

This architecture creates a new problem: when the system produces a wrong answer, you don't know which stage failed. Did the retriever fetch the wrong documents? Did it fetch the right ones but the generator ignored them? Did both stages fail? The same wrong output can come from completely different root causes — and each one requires a different fix from a different team.

You cannot fix what you cannot separate. RAG evaluation exists to draw a clean line between the two stages and measure each independently — so you know where to invest when something breaks.

### Retrieval metrics: Precision@k and Recall@k

For retrieval, you're asking a simple question: **did we find the right documents?** Two standard metrics answer this.

**Precision@k** — of the top-k documents retrieved, how many are actually relevant? If you retrieve 5 documents and 2 are relevant, Precision@5 = 2/5 = 0.40. Low precision means your retriever is fetching noise along with signal. That noise isn't free — every irrelevant document in the prompt wastes context window, adds cost, and can distract the generator.

**Recall@k** — of all the relevant documents in the knowledge base, how many did we find in the top-k? If there are 3 relevant documents and you retrieved 2 of them, Recall@5 = 2/3 = 0.67. Low recall means you're missing key context. The generator simply cannot use what you didn't retrieve — even a perfect generator will produce wrong answers if retrieval misses the policy that should have governed the decision.

These two metrics are in tension. You can improve recall by retrieving more documents (higher k), but precision usually drops because the extra documents are less relevant. You can improve precision by retrieving fewer, but recall drops because you might miss relevant ones. The PM question isn't "which metric matters" — both do — but "what's the cost of each failure mode in your system?" In safety-critical domains, missing context (low recall) is usually worse than noisy context (low precision). In cost-sensitive domains, it's often the reverse.

### Generation metrics: Faithfulness and Answer Relevance

For generation, you're asking two different questions. Both matter because they catch different failure modes.

**Faithfulness** — is the answer grounded in the retrieved context, or did the LLM make things up? A faithful answer only makes claims that can be verified against the retrieved documents. An unfaithful answer cites policies that weren't retrieved, contradicts the retrieved context, or fabricates evidence. Faithfulness failures are the classic "hallucination" problem — except in RAG, they're more damaging because users assume the retrieved context makes the LLM trustworthy.

**Answer Relevance** — does the answer actually address the query? A relevant answer directly responds to what was asked. An irrelevant answer might be faithful (grounded in retrieved docs) but off-topic (talking about something the user didn't ask about). Imagine asking "can I remove the peanut allergen warning?" and getting back a discussion of ingredient alternatives. Everything said might be technically true — but it's not an answer to the question.

These two metrics are independent. You can have:
- **Faithful but irrelevant** — the answer cites retrieved docs accurately but answers the wrong question
- **Unfaithful but relevant** — the answer directly addresses the query but fabricates supporting evidence
- **Both bad** — off-topic and hallucinated
- **Both good** — what you actually want

You need to measure both because fixing one doesn't fix the other.

### Context Precision and Context Recall: ranking matters

Basic Precision@k treats all top-k documents equally. A relevant document at position 1 counts the same as a relevant document at position 5. In practice, ranking matters a lot — the generator pays more attention to documents near the top of the context window.

**Context Precision** (from the RAGAS framework) penalizes low ranking of relevant docs. If your relevant documents are buried at positions 4 and 5 while positions 1-3 are noise, your basic Precision@5 might be 0.40 — but your context precision is worse because the generator is likely to pay more attention to the noise that came first. A well-designed retriever doesn't just find relevant docs, it ranks them high.

**Context Recall** asks a sharper question than basic Recall@k: did you retrieve all the context needed to answer the query *completely*? A question might need three distinct facts to answer correctly. If your retrieved documents cover two of them but not the third, basic recall might look decent but the answer will be incomplete. Context recall treats the answer's requirements as the unit of measurement, not just the set of "relevant" documents.

These RAGAS metrics matter when you've moved beyond "is the retriever working at all?" to "is the retriever optimizing for how the generator actually consumes context?" Start with Precision@k and Recall@k; graduate to context precision and context recall once your basic retrieval is solid.

### The 2x2 diagnostic matrix

The point of measuring retrieval and generation separately is to diagnose failures. Every query in your eval falls into one of four cells:

|                        | **Good generation** | **Bad generation** |
|------------------------|---------------------|--------------------|
| **Good retrieval**     | Correct — system works | Wrong — generation bottleneck |
| **Bad retrieval**      | Wrong — retrieval bottleneck | Wrong — both stages broken |

Each cell has a different fix:

- **Good R + Good G** — this is the working baseline. Nothing to do here.
- **Good R + Bad G** — retrieval gave the generator everything it needed, and the generator still failed. The fix is on the generation side: better prompts, better grounding, better judge calibration. The retrieval team is not the problem.
- **Bad R + Good G** — the generator did its job correctly given what it received. The problem is that retrieval missed key context. The fix is on the retrieval side: better embeddings, better chunking, reranking. The generation team is not the problem.
- **Bad R + Bad G** — both stages are failing independently. You need fixes on both sides, and fixing just one still leaves these cases wrong.

The matrix is a routing tool. Given a failure report, you use the matrix to send it to the right team instead of letting both teams argue about whose problem it is. This is the diagnostic value of separating retrieval and generation metrics — without separation, you can't route, and without routing, the wrong team ends up chasing the wrong bugs.

---

## Part 2: Exercise — Diagnosing a RAG Pipeline

### Scenario

Your team has evolved the menu verification system into a RAG pipeline. Instead of hard-coding policies into the prompt, the system now retrieves the top-5 most relevant policies from a knowledge base of 15 policy documents for each query, then uses them as context for the decision. The benefit: policies can be updated without code changes. The cost: a new failure surface — and your head of product wants to know which stage is broken before committing to a fix.

Your engineering manager has run 18 recent queries through the pipeline and logged full evaluation data: what the retriever fetched, what the generator produced, and whether the final decision was correct. You've been asked to audit this data and come back with a clear diagnosis.

### Dataset

Open `exercises/D8-rag-eval-dataset.csv`. Each row is one query run through the full pipeline.

The pipeline has a knowledge base of 15 policy documents. For each query, the retriever fetches the top-5 most relevant policies. The generator receives those 5 policies as context and produces a decision and reasoning. The `relevant_docs` column is expert-verified ground truth — the policies that actually should have been retrieved for that query.

| Column | What it means |
|--------|---------------|
| `case_id` | Case identifier (Q-01 through Q-18) |
| `query_text` | The menu change being evaluated |
| `change_type` | Type of change: price, allergen, description, category, portion, image, or nutrition |
| `retrieved_docs` | The 5 policies the retriever fetched (semicolon-separated, in rank order) |
| `relevant_docs` | The policies that should have been retrieved (expert-verified ground truth) |
| `precision_at_5` | Share of retrieved docs that are relevant: (relevant ∩ retrieved) / 5 |
| `recall_at_5` | Share of relevant docs that were retrieved: (relevant ∩ retrieved) / (total relevant) |
| `generated_decision` | The system's decision: approve, reject, or flag_for_review |
| `generated_reasoning` | The system's explanation for its decision |
| `ground_truth_decision` | The correct decision (expert-verified): approve or reject |
| `decision_correct` | TRUE if generated_decision matches ground_truth_decision |
| `faithful` | TRUE if generated_reasoning is grounded in the retrieved docs — no fabricated claims |
| `answer_relevant` | TRUE if generated_reasoning actually addresses the query — not off-topic |

### Expected outcome

By the end of this exercise you'll have computed retrieval metrics, generation metrics, and a 2x2 diagnostic matrix showing where failures are coming from. This feeds directly into the Decision Point: picking which stage to fix first.

### Step 1: Retrieval metrics

Start by understanding how well the retriever is working overall. Compute the average Precision@5 and Recall@5 across all 18 cases. What do those two numbers together tell you about the retriever's behavior?

### Step 2: Generation metrics

Now look at generation quality. Compute the faithfulness rate (percentage of cases where `faithful` is TRUE) and the answer relevance rate (percentage where `answer_relevant` is TRUE). What's the story on the generator?

### Step 3: The 2x2 diagnostic matrix

Define "good retrieval" as Recall@5 = 1.0 (the retriever got all relevant docs). Define "good generation" as `faithful` = TRUE AND `answer_relevant` = TRUE. Build the 2x2 matrix: how many cases fall into each quadrant?

### Step 4: Diagnose

Compute the overall correctness rate (`decision_correct` = TRUE). Compare it against your retrieval and generation metrics. Which stage is the bigger bottleneck? Which stage should get investment first?

---

## Part 3: PM Decision Point

Two engineers on your team are asking for resources to fix the RAG pipeline. You have budget for exactly one initiative this sprint.

Your **retrieval engineer** says: *"I can get Recall@5 from where it is today to 95% in two weeks. This will fix the cases where we're missing relevant context."*

Your **LLM team** says: *"I can get generation quality (both faithfulness and answer relevance) to 90% in two weeks. This will fix the cases where we're hallucinating or answering the wrong question."*

Which do you fund, and why? Back your decision with specific numbers from your audit: how many wrong decisions each fix would actually eliminate, what would still remain wrong after each fix, and what the implications are for the cases that need both fixes.

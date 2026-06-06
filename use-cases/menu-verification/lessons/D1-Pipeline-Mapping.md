# D1 - Pipeline Mapping

**Week 1, Day 1** | ~25 min
**Part times:** Concepts ~10 min | Exercise ~10 min | Decision Point ~5 min
**Previous lesson:** None — first lesson. Orient the learner: this course teaches you to evaluate AI systems through hands-on exercises with real data. By the end of Day 1 you'll be able to read a production dataset, locate where failures are happening, and ask the right diagnostic questions about any AI system you inherit.

---

## Part 1: The Concepts

### Every AI product is a pipeline

You've just been assigned as PM for an AI-powered system. Before you can evaluate whether it's working, you need to understand *what it does* — not at the marketing level, but at the mechanical level.

Most LLM-powered products aren't a single model call. They're pipelines — a sequence of stages where data flows through transformations before reaching a final output. A typical pipeline looks like this:

**Input parsing** — The system receives a request and interprets what needs to happen. This could be extracting structured fields from unstructured data, classifying the type of request, or normalizing inputs into a format the rest of the pipeline expects.

**Context retrieval** — The system gathers relevant information to inform its decision. This might mean querying a database, pulling recent history, or looking up reference data. The quality of what gets retrieved directly shapes the quality of what comes next.

**LLM reasoning** — The language model takes the parsed input and retrieved context, applies its training and any provided instructions, and produces a judgment. This is where the "intelligence" happens — and where most people assume all problems live.

**Output generation** — The model's reasoning gets packaged into a structured decision: a classification, a score, a recommendation. Along with the decision, the system typically produces a confidence score and an explanation.

**Human review** — For decisions the system isn't sure about, a human reviewer makes the final call. Not every item goes through this stage — only the ones the system flags.

Understanding these stages matters because **when something goes wrong, you need to know which stage broke.** A wrong answer might be caused by bad context retrieval (the system never saw the relevant data), bad reasoning (it had the data but drew the wrong conclusion), or bad input parsing (it misunderstood the request entirely). Same symptom, three different root causes, three different fixes.

### Why "does it work?" is the wrong question

With traditional software, "does it work?" has a clear answer. You run the test, it passes or fails, and that result is the same every time.

AI systems operate differently. A language model selects its output probabilistically — weighted randomness at each step of generation. This means you can feed the exact same input through the exact same model twice and get two different outputs. Neither is "wrong" — the system is working as designed. It's just not deterministic.

This has a practical consequence for you as a PM: you can never fully answer "does it work?" with a single test run. You can only answer "how often does it produce acceptable results, and under what conditions?"

That shift — from pass/fail to frequency and conditions — is the foundation of everything else in this course.

### Reading an AI system through its data

You don't need to understand the model's architecture or read its code. You need to understand what data it produces and what that data tells you about each pipeline stage.

Every AI system in production should be logging its decisions. Each log entry is a trace — a record of what went in, what came out, and what happened in between. If you can read the traces, you can evaluate the system.

The key question for each column in a trace log: **which pipeline stage does this come from, and what does it tell me?**

- Columns describing the input (what changed, what the old and new values are) → **input parsing**
- Columns with reference data (verified prices, historical information) → **context retrieval**
- Columns with the AI's decision, confidence, and reasoning → **LLM reasoning + output generation**
- Columns with a human's decision → **human review**

Once you can map data to pipeline stages, you can start asking the right diagnostic questions: "Is the system failing because it has bad context, or because it's ignoring good context?"

---

## Part 2: Exercise — Map the Pipeline

You're the new AI PM at a food delivery company. Your team runs an LLM-powered system that verifies restaurant menu changes — when a restaurant updates a price, description, allergen info, image, or category, the AI decides whether to approve or reject the change.

You've been handed the production logs. Before doing any analysis, your first job is to understand what you're looking at.

### Worked example: reading a simplified trace

Before opening the real dataset, here's how pipeline mapping works using a different system — an AI that reviews insurance claims:

| Column | Value | Pipeline stage |
|--------|-------|---------------|
| `claim_description` | "Roof damage from storm" | Input parsing — what came in |
| `policy_coverage_limit` | 25,000 | Context retrieval — looked up from external records |
| `llm_decision` | approve | LLM reasoning & output — the AI's judgment |
| `adjuster_decision` | approve | Human review — the reviewer's call |
| `claim_id` | CLM-4821 | Metadata — bookkeeping, not pipeline data |

The mapping logic is the same for any AI system: inputs belong to parsing, looked-up reference data belongs to retrieval, AI outputs belong to reasoning, human decisions belong to review, and IDs/timestamps are metadata.

Now apply this to the real dataset.

### Setup

Open `exercises/D1-menu-verification-d1.csv`. This is production shadow-mode data — the AI made decisions, but humans also reviewed every item so you have ground truth.

The dataset has 30 rows and 12 columns. Start by understanding the structure.

### Step 1: Identify the pipeline stages

Present the stage key and column list exactly as shown below, then ask the learner to reply
with one letter per column, space-separated, in order.

---
Here are the 5 pipeline stages:

  A = Input parsing
  B = Context retrieval
  C = LLM reasoning & output
  D = Human review
  E = Metadata

And here are the 12 columns from the dataset, in order:

   1. change_id          7. reference_data
   2. restaurant_name    8. llm_decision
   3. item_name          9. llm_confidence
   4. change_type       10. llm_reasoning
   5. old_value         11. human_decision
   6. new_value         12. human_agrees_with_llm

Reply with 12 letters, space-separated, matching the column order above.
Example format: E E E A A A B C C C D D
---

Once the learner responds, reconstruct the mapping table, confirm what they got right,
and correct any mistakes with a brief explanation before moving to Step 2.

### Step 2: Trace a single row

> 💡 **Tip:** This step is easier with voice. On Mac press Fn Fn, on Windows press Win + H — then just narrate what you see.

Pick any row from the dataset. Read through it and narrate:

1. **What changed, and what context did the system have?** (change_type, old_value, new_value, reference_data)
2. **What did the AI decide, and did the human agree?** (llm_decision, llm_confidence, human_decision, human_agrees_with_llm)

### Step 3: Spot the reference data pattern

Look at a few rows with different `change_type` values. Compare what's in the `reference_data` column:

- For a `price` change: what reference data does the system have?
- For a `description` change: what reference data does the system have?
- For an `allergen_dietary` change: what reference data does the system have?

What does this tell you about which change types the system can verify automatically vs which ones it's guessing on?

### Step 4: Identify what's missing

Looking at the columns available, list 2-3 things you'd *want* to know about the pipeline but can't determine from this data. What's invisible?

---

## Part 3: PM Decision Point

Your engineering lead asks: "What questions do you have about the system?" Based on your column mapping and row trace, write 2 questions — each targeting a different pipeline stage. Choose the stages where you feel the biggest blind spots are.

These questions become the start of your evaluation investigation roadmap.

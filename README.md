# The Eval Bootcamp

A 21-day, hands-on course that teaches product people how to evaluate AI systems — by actually doing it.

No slides. No videos. You clone this repo, open Claude Code, and it becomes your AI evals tutor: teaching one concept at a time, guiding you through exercises with real datasets, and evaluating your product decisions.

---

## Who Should Take This Course

This course is for **anyone working on AI products** who wants to build rigorous evaluation skills — without needing a machine learning background.

**Primary audience (ICP): Product Managers** shipping AI features and wanting to go beyond gut-feel quality checks. Also valuable for:

- Associate and Group PMs transitioning into AI-focused roles
- Founders and solo builders who own both product and quality
- Product Leads overseeing AI teams and setting eval strategy
- Technical PMs who want to bridge engineering metrics and product decisions

If you've asked "how do I know if this AI is actually working?" — this course is for you.

---

## What You'll Learn

- **Read any AI system from scratch** — map pipeline stages, trace production logs, and identify which stage is responsible when things go wrong
- **Measure reliability, not just accuracy** — pass@k, reliable@k, and the consistency gap that separates demo-ready from production-ready
- **Build a failure taxonomy** — systematic error analysis that turns raw traces into actionable categories
- **Design automated quality checks** — code-based graders, LLM-as-judge, and a layering strategy that scales
- **Build ground truth you can trust** — golden datasets, contamination detection, and lifecycle management
- **Design metrics that drive decisions** — guardrail vs optimization metrics, fairness and subgroup evaluation
- **Run AI experiments** — what's different about A/B testing for LLM systems, and how to avoid the traps
- **Ship with a framework** — release criteria, production monitoring, and a repeatable ship/hold process
- **Build an eval culture** — how to institutionalize evals across your team

---

## Course Features

- **Hands-on with real data** — every lesson includes a synthetic dataset you analyze yourself; no toy examples
- **You do the thinking** — Claude computes on request; you direct the analysis and draw the conclusions
- **PM Decision Points** — each lesson ends with you writing a recommendation or artifact; Claude evaluates it against a scoring rubric
- **Adaptive tutoring** — Claude matches your pace; experienced practitioners move fast, newcomers get more examples
- **~30–40 min per day** — designed for working professionals; one focused lesson per day
- **Progress saved locally** — tracked in `progress/progress.json`, gitignored and never leaves your machine

---

## Quick Start

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured.

```bash
git clone https://github.com/sanjeevrao1992/the-eval-bootcamp.git
cd the-eval-bootcamp
claude
```

Claude reads `CLAUDE.md` on startup and introduces itself as your tutor. It checks your progress and picks up from where you left off.

---

## Course Structure

21 days. 3 weeks. One lesson per day.

### Week 1 — Your Eval Foundation (Days 1–7)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D1 | What Does This System Actually Do? | Pipeline stages, non-determinism, reading traces |
| D2 | Mapping Every Way Your AI System Can Fail | Evaluation surface map, failure layers, coverage gaps |
| D3 | Error Analysis | Open coding, axial coding, saturation, triage |
| D4 | Thinking in Distributions | Shape before depth, pass@k, reliable@k, the consistency gap |
| D5 | The Three Types of Quality Checks | Code-based, model-based, human graders; layering strategy |
| D6 | LLM-as-Judge | Calibration trap, Critique Shadowing, failure modes, meta-evaluation |
| D7 | Ground Truth and Golden Datasets | Three sources, contamination, dataset lifecycle |

### Week 2 — Metrics and Measurement at Scale (Days 8–14)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D8 | Retrieval and RAG Evaluation | Precision@k, faithfulness, answer relevance, context recall |
| D9 | Hallucination Detection | Detection strategies, grounding, citation evaluation |
| D10 | Blocking Metrics and Release Criteria | Guardrail vs optimization metrics, ship/hold thresholds |
| D11 | Metric Design and Cost-Aware Evaluation | Metric tradeoffs, evaluation cost, coverage strategy |
| D12 | Fairness, Bias, and Subgroup Evaluation | Subgroup slicing, disparity detection, fairness in practice |
| D13 | Eval-Driven Development | Evals as product specs, regression testing, eval cadence |
| D14 | The Observability Landscape | Logging, tracing, what to instrument and why |

### Week 3 — Ship, Monitor, and Scale (Days 15–21)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D15 | Evaluating AI Agents | Multi-step pipelines, tool use, trajectory evaluation |
| D16 | What's Different About AI Experiments | LLM A/B testing, variance, confounds |
| D17 | Launch Readiness and Production Monitoring | Pre-launch checklist, drift detection, incident response |
| D18 | Red Teaming and Adversarial Evaluation | Threat modeling, adversarial prompts, stress testing |
| D19 | The Ship Decision Framework | Synthesizing eval signals into a go/no-go recommendation |
| D20 | Regulatory and Legal Context | AI Act, liability, what product people need to know |
| D21 | Building an Eval Culture | Institutionalizing evals, team buy-in, eval as product practice |

> **Currently available:** D1–D7 (Week 1). New lessons added regularly.

---

## What's in the Repo

```
lessons/        Lesson content — concepts, exercises, decision points (D1.md through D7.md)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's tutor instructions)
progress/       Your local progress — gitignored, never leaves your machine
CLAUDE.md       Course configuration — Claude reads this on startup
```

---

## License

[CC BY-NC-SA 4.0](LICENSE) — Free to use and adapt for non-commercial purposes with attribution.

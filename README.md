# 🧪 The Eval Bootcamp

A 21-day, hands-on course that teaches product people how to build and evaluate production-ready AI systems — ✨ by actually doing it ✨

No slides. No videos. You clone this repo, open Claude Code, and it becomes your personal AI evals tutor: teaching one concept at a time, guiding you through exercises with real datasets, and evaluating your product decisions.

---

## 🙋 Who Should Take This Course

This course is for **anyone working on AI products** who wants to build rigorous evaluation skills — without needing a machine learning background.

**Primary audience: Product Managers** shipping AI features and wanting to go beyond gut-feel quality checks. Also great for:

- Associate and Group PMs transitioning into AI-focused roles
- Founders and solo builders who own both product and quality
- Product Leads overseeing AI teams and setting eval strategy
- Technical PMs who want to bridge engineering metrics and product decisions

If you've ever asked *"how do I know if this AI is actually working?"* — this course is for you.

---

## 🎯 What You'll Learn

- **Read any AI system from scratch** — map pipeline stages, trace production logs, identify which stage is breaking
- **Measure reliability, not just accuracy** — pass@k, reliable@k, and the consistency gap that separates demo-ready from production-ready
- **Build a failure taxonomy** — systematic error analysis that turns raw traces into actionable categories
- **Design automated quality checks** — code-based graders, LLM-as-judge, and a layering strategy that scales
- **Build ground truth you can trust** — golden datasets, contamination detection, and lifecycle management
- **Design metrics that drive decisions** — guardrail vs optimization metrics, fairness and subgroup evaluation
- **Run AI experiments** — what's different about A/B testing for LLM systems, and how to avoid the common traps
- **Ship with a framework** — release criteria, production monitoring, and a repeatable ship/hold process
- **Build an eval culture** — how to institutionalize evals across your team

---

## ✨ Course Features

- **Hands-on with real data** — every lesson includes a synthetic dataset you analyze yourself; no toy examples
- **You do the thinking** — Claude computes on request; you direct the analysis and draw the conclusions
- **PM Decision Points** — each lesson ends with you writing a recommendation or artifact; Claude evaluates it against a scoring rubric
- **Adaptive tutoring** — Claude matches your pace; experienced practitioners move fast, newcomers get more examples
- **~30–40 min per day** — designed for working professionals; one focused lesson per day
- **Progress saved locally** — tracked in `progress/progress.json`, gitignored and never leaves your machine

---

## 🚀 Quick Start

> **Already set up?** Skip to the relevant step:
> - Have an Anthropic account but not Claude Code? → [Step 2](#step-2--install-claude-code)
> - Have both an account and Claude Code installed? → [Step 3](#step-3--clone-the-course-and-start)

---

### Step 1 — Create an Anthropic account

Claude Code requires an Anthropic account. If you don't have one yet, create a free account at [claude.ai](https://claude.ai). You'll use it to authenticate Claude Code in Step 3.

---

### Step 2 — Install Claude Code

Claude Code is a command-line tool that lets you talk to Claude directly in your terminal, with access to your local files. It's what powers the tutor.

**Mac / Linux:**
```bash
npm install -g @anthropic-ai/claude-code
```

> **Don't have npm?** Install [Node.js](https://nodejs.org/) first (it includes npm), then run the command above.

**Windows:** Use [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install), then follow the Mac/Linux steps above.

**Verify the install:**
```bash
claude --version
```

The first time you run `claude`, it will prompt you to log in with your Anthropic account.

---

### Step 3 — Clone the course and start

```bash
git clone https://github.com/sanjeevrao1992/the-eval-bootcamp.git
cd the-eval-bootcamp
claude
```

> **Don't have git?** Download it from [git-scm.com](https://git-scm.com/downloads), or use [GitHub Desktop](https://desktop.github.com/) to clone the repo visually — then open your terminal, navigate to the `the-eval-bootcamp` folder, and run `claude`.

Claude reads `CLAUDE.md` on startup and introduces itself as your tutor. It checks your progress and suggests your next lesson.

---

### 🔄 Resuming after your first session

Next time, just navigate to the folder and run `claude` again:

```bash
cd the-eval-bootcamp
claude
```

Your progress is saved locally in `progress/progress.json` — gitignored and never leaves your machine.

---

### 💻 Works with Terminal, Cursor, or Obsidian

- **Terminal** (Mac/Linux): run `claude` from inside the repo folder
- **Cursor**: open the repo folder in Cursor, open the built-in terminal (`` Ctrl+` ``), and run `claude`
- **Obsidian**: open your system terminal, navigate to the repo folder, and run `claude`

---

### 🔧 Troubleshooting

| Problem | Fix |
|---------|-----|
| `claude: command not found` | Run `npm install -g @anthropic-ai/claude-code` again, then restart your terminal |
| `permission denied` on install | Try `sudo npm install -g @anthropic-ai/claude-code` |
| Claude asks to approve file writes | Type `yes` — it needs to save your progress to `progress/progress.json` |
| Claude doesn't introduce itself as tutor | Make sure you're running `claude` from inside the `the-eval-bootcamp` folder, not a parent directory |
| Stuck mid-lesson | Type `resume` — Claude will re-read your progress and pick up where you left off |

---

## 📅 Course Structure

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

> **Currently available:** D1–D19 (Weeks 1–3, Days 1–19). D20–D21 coming soon.

---

## 📁 What's in the Repo

```
lessons/        Lesson content — concepts, exercises, decision points (D1.md through D19.md)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's tutor instructions)
progress/       Your local progress — gitignored, never leaves your machine
CLAUDE.md       Course configuration — Claude reads this on startup
```

---

## 📄 License

[CC BY-NC-SA 4.0](LICENSE) — Free to use and adapt for non-commercial purposes with attribution.

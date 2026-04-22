# 🧪 AI Evals Bootcamp

A 21-day, one-of-a-kind interactive course that teaches product people how to build and evaluate production-ready AI systems — ✨ by actually doing it ✨

No slides. No videos. You clone this repo, open Claude Code, and it becomes your personal AI evals tutor: teaching one concept at a time, guiding you through exercises with real datasets, and evaluating your product decisions.

> ⭐ Star this repo to save it to your GitHub profile for easy reference later.

---

## 🙋 Who Should Take This Course

This course is for product folks who want to ship AI features that actually work — reliably, at scale, beyond gut-feel.

**Primary audience: Product Managers** shipping AI features who want a systematic, repeatable way to know their product is actually working. Also great for:

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

**Already set up? Skip ahead:**
- Have Node.js but not Claude Code? → [Step 2](#step-2----install-claude-code)
- Have Claude Code installed? → [Step 3](#step-3----get-the-course-files)
- Have the files cloned? → [Step 4](#step-4----start-the-course)

---

### Step 1 — Get your terminal and Anthropic account ready

**Open a terminal.** This is where the course runs.
- **Mac:** Search "Terminal" in Spotlight, or press `` Cmd+Space `` and type Terminal
- **Cursor:** Go to View → Terminal, or press `` Ctrl+` `` (Windows) / `` Cmd+` `` (Mac)
- **Windows:** Search "PowerShell" in the Start menu

> ⚠️ **Using Cursor?** Claude Code is a separate tool — Cursor is your editor, Claude Code is what runs the course. Type commands in the terminal (View → Terminal), not Cursor's chat box.

**Create an Anthropic account** (free) at [claude.ai](https://claude.ai) if you don't have one — you'll need it to authenticate Claude Code.

---

### Step 2 — Install Claude Code

First, check if you have Node.js:
```bash
node --version
```
If you see a version number, skip straight to installing Claude Code. If not, download Node.js from [nodejs.org](https://nodejs.org) (use the LTS version), then come back here.

Install Claude Code:
```bash
npm install -g @anthropic-ai/claude-code
```

Verify it worked:
```bash
claude --version
```
If you see a version number, you're good. ✅

> **Permissions error?** If you're on a managed or corporate laptop, download Node.js directly from [nodejs.org](https://nodejs.org) instead of using npm — this bypasses most IT restrictions. Still stuck? You may need to ask IT to whitelist the install.

---

### Step 3 — Get the course files

```bash
git clone https://github.com/productfoundry101/ai-evals-bootcamp.git
cd ai-evals-bootcamp
```

> **Don't have git?** Download it from [git-scm.com](https://git-scm.com/downloads), then run the commands above.

**If you're using Cursor:** Go to File → Open Folder and select the `ai-evals-bootcamp` folder. Your course files — lessons, datasets, everything — will appear in the left sidebar. These are real files sitting on your computer; you can open the CSVs in Excel, Numbers, or Google Sheets anytime.

---

### Step 4 — Start the course

Make sure you're inside the course folder, then run:
```bash
claude
```

You'll see a `>` prompt — that means it worked. Type `go` and your tutor will introduce itself and start Day 1.

---

### 🔄 Returning after your first session

```bash
cd ai-evals-bootcamp
claude
```

Your progress is saved automatically after each lesson. The tutor will pick up exactly where you left off.

---

### 🔧 Troubleshooting

| Problem | Fix |
|---------|-----|
| `claude: command not found` | Run `npm install -g @anthropic-ai/claude-code` again, then restart your terminal |
| Permissions error during install | Download Node.js directly from [nodejs.org](https://nodejs.org) instead |
| Blank screen after running `claude` | You're in — just type `go` to start |
| Claude doesn't introduce itself as tutor | Make sure you ran `claude` from inside the `ai-evals-bootcamp` folder, not a parent directory |
| Claude asks to approve file writes | Type `yes` — it needs this to save your progress |
| Stuck mid-lesson | Type `resume` — the tutor will re-read your progress and pick up where you left off |

---

## 📅 Course Structure

21 days. 3 weeks. One lesson per day.

### Week 1 — Your Eval Foundation (Days 1–7)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D1 | Pipeline Mapping | Pipeline stages, non-determinism, reading traces |
| D2 | Failure Surface Mapping | Evaluation surface map, failure layers, coverage gaps |
| D3 | Error Analysis | Open coding, axial coding, saturation, triage |
| D4 | Thinking in Distributions | Shape before depth, pass@k, reliable@k, the consistency gap |
| D5 | Grader Types | Code-based, model-based, human graders; layering strategy |
| D6 | LLM-as-Judge | Calibration trap, Critique Shadowing, failure modes, meta-evaluation |
| D7 | Golden Datasets | Three sources, contamination, dataset lifecycle |

### Week 2 — Metrics and Measurement at Scale (Days 8–14)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D8 | RAG Evaluation | Precision@k, faithfulness, answer relevance, context recall |
| D9 | Hallucination Detection | Detection strategies, grounding, citation evaluation |
| D10 | Release Criteria | Guardrail vs optimization metrics, ship/hold thresholds |
| D11 | Metric Design | Metric tradeoffs, evaluation cost, coverage strategy |
| D12 | Fairness & Subgroups | Subgroup slicing, disparity detection, fairness in practice |
| D13 | Eval-Driven Development | Evals as product specs, regression testing, eval cadence |
| D14 | Observability | Logging, tracing, what to instrument and why |

### Week 3 — Ship, Monitor, and Scale (Days 15–21)

| Day | Lesson | Key Skills |
|-----|--------|------------|
| D15 | Agent Evaluation | Multi-step pipelines, tool use, trajectory evaluation |
| D16 | AI Experiments | LLM A/B testing, variance, confounds |
| D17 | Launch Readiness | Pre-launch checklist, drift detection, incident response |
| D18 | Red Teaming | Threat modeling, adversarial prompts, stress testing |
| D19 | Ship Decisions | Synthesizing eval signals into a go/no-go recommendation |
| D20 | Regulatory Context | AI Act, liability, what product people need to know |
| D21 | Eval Culture | Institutionalizing evals, team buy-in, eval as product practice |

---

## 📁 What's in the Repo

```
lessons/        Lesson content — concepts, exercises, decision points (D1-Pipeline-Mapping.md through D21-Eval-Culture.md)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's tutor instructions)
progress/       Your local progress — gitignored, never leaves your machine
CLAUDE.md       Course configuration — Claude reads this on startup
```

---

## ⭐ Stay Updated

Found this course useful? Star the repo ⭐ — it saves it to your GitHub profile for easy reference, it helps others discover it, and it massively helps me.

This course is actively updated based on feedback from real learners — new lessons, fixes, and improvements ship regularly. To get notified the moment an update drops, click **Watch → Custom → Releases** at the top of this page.

---

## 📚 Further Reading & Acknowledgements

This course stands on the shoulders of practitioners who've shared their teachings publicly. If you want to go deeper, these are the sources that most shaped what you just learned:

- **[Hamel Husain](https://hamel.dev)** — evals methodology, error analysis, LLM-as-judge
- **[Shreya Shankar](https://www.sh-reya.com)** — LLM judge calibration research
- **[Lenny's Newsletter](https://www.lennysnewsletter.com)** — PM-specific evals framing ("Beyond vibe checks" and related pieces)
- **[Aman Khan](https://amankhan1.substack.com)** — AI PM evals perspective
- **[Tal Raviv](https://talraviv.co)** — practical PM evals examples
- **[AI Analyst Lab](https://aianalystlab.ai)** — inspiration for framing evals as a product-centric arc (rather than analyst-centric) and for treating error analysis as the foundation every other technique builds on
- **[RAGAS](https://docs.ragas.io)** — RAG evaluation framework
- **[OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/)** — adversarial attack taxonomy for LLM systems
- **["Building AI Product Sense with a Custom Tutor" by Aman Khan](https://amankhan1.substack.com/p/building-ai-product-sense-with-a)** — inspiration for implementing Claude Code as your AI tutor

---

## 📄 License

[CC BY-NC-SA 4.0](LICENSE) — Free to use and adapt for non-commercial purposes with attribution.

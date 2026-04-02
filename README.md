# AI Evals for PMs

An interactive, hands-on course that teaches product managers how to evaluate AI systems — by actually doing it.

No slides. No videos. You open Claude Code, and it becomes your AI evals tutor: teaching concepts, guiding you through exercises with real datasets, and evaluating your product decisions.

## What You'll Learn

- **How to measure AI reliability** — pass@k, reliable@k, and the consistency gap that determines ship/hold decisions
- **How to map failure surfaces** — a structured framework for identifying every way your AI system can fail
- **How to connect evals to product decisions** — not just "what's the accuracy?" but "what should we do about it?"

## Quick Start

**Prerequisites:** [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and configured.

```bash
git clone https://github.com/sanjeevrao1992/ai-evals-for-pms.git
cd ai-evals-for-pms
claude
```

That's it. Claude will introduce itself as your tutor and guide you through the first lesson.

## Course Outline

| Lesson | Topic | Duration |
|--------|-------|----------|
| PM-L01 | Why AI Systems Need Different Quality Thinking | ~30 min |
| PM-L02 | Mapping Every Way Your AI System Can Fail | ~30 min |

More lessons coming soon.

## How It Works

Each lesson follows a three-part structure:

1. **Concepts** — Industry-agnostic frameworks any PM can apply. Claude teaches one concept at a time, checking your understanding before moving on.
2. **Exercise** — You analyze a real (synthetic) dataset from a food delivery menu verification system. You direct the analysis; Claude does the computation. You interpret the results.
3. **PM Decision Point** — You write a memo, recommendation, or artifact using what you calculated. Claude evaluates your response.

Your progress is saved locally so you can pick up where you left off.

## What's in the Repo

```
lessons/        Lesson content (concepts + exercises + decision points)
exercises/      CSV datasets you'll analyze during exercises
tutor/          Session protocol and scoring rubrics (Claude's instructions)
progress/       Your local progress (gitignored)
```

## License

[CC BY-NC-SA 4.0](LICENSE) — Share and adapt for non-commercial purposes with attribution.

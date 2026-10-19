# ai-agent-eval-suite

How do you know an AI agent is good enough to hand to a client? You decide **before** you look at its output.

This repository is a small, fixed evaluation suite for one job: an AI agent that turns a written requirement into test cases. It is tool-neutral: the same cases and rubric work for Claude, ChatGPT, Codex, Microsoft Copilot or any other agent.

## The method

1. **Rubric first.** The scoring rules in [RUBRIC.md](RUBRIC.md) were written before any agent was run. Nothing is tuned after seeing results.
2. **Fixed cases.** Each file in [cases/](cases/) holds one requirement, what a good answer must contain, and the traps a weak agent falls into.
3. **Three runs per case.** Language models give different answers to the same prompt. A case counts as passed only if **all 3 runs** pass. One lucky answer is not quality.
4. **Rerun on every change.** New model version, new prompt, new tool: the whole suite runs again and the pass rate is compared with the previous one.

## Status

| Item | State |
|---|---|
| Rubric | v1 written |
| Cases | 15 of 15–20 |
| First baseline run | done 2026-10-19: Sonnet 5/15 cases, Haiku 3/15 ([details](runs/2026-10-19_baseline.md)) |
| Human calibration | round 1 done 2026-10-19: 2 of 3 G1 verdicts confirmed, rubric v1.1, results unchanged ([details](runs/2026-10-19_baseline.md#human-calibration-and-rubric-v11-2026-10-19-same-evening)) |

## How a run works

1. Give the agent the prompt from [PROMPT.md](PROMPT.md) plus the requirement from one case.
2. Save the answer in `runs/<date>_<agent>/<case-id>_run<1-3>.md`.
3. Score it with the rubric and record it in `runs/<date>_<agent>/scores.md`.
4. Pass rate = cases passed in all 3 runs / total cases.

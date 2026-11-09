# Exploration vs exploitation: when the support agent meets a failure it has never seen

**Date:** Sunday, Nov 8, 2026 · **Status:** Draft — paper in progress · **Problem 15** — Daily Architect Problems (CCA prep)

Part of *Daily Architect Problems* — applied agentic-design problems from Claude Certified Architect (CCA) prep, grounded in a real production agent. This folder is self-contained: publish it on its own as its own repo.

## The problem

A support agent handling 400 tickets a day meets a checkout error it has never seen — "payment gateway timeout after 3DS" — with no episode above 0.6 similarity and a 14-month-old wiki runbook. Design the exploration policy: when to explore vs escalate, how to bound attempts, what makes a new path worth keeping, how it gets written back as an episode, and what metric proves exploration pays off.

## How to use it

1. Read `problem.md` — the scenario, the task, and the self-check.
2. Write your answer as a design paper: the decision, the trade-offs, the numbers, and the failure case your design leaves uncovered.
3. Run the example to see one key mechanism as a working sketch: `python3 15_exploration_exploitation.py`
4. Use the self-check at the end of `problem.md` to test whether your design holds up.

## The example code

`15_exploration_exploitation.py` is an illustrative sketch of one core mechanism in this problem: a try is one full loop of up to 50 turns for a novel failure, and each try carries forward the negative evidence of the tries before it ("where not to check"), so the next try searches somewhere else instead of repeating the last one. Confidence is evidence, not a score — HIGH needs the exact piece of evidence in both the code and the logs. Blast radius is tiered by consequence, not confidence: read-only checks run free, a bounded write (retry with an idempotency key) is proposed but never executed without a human, and irreversible actions (refunds) are human-only. The stop is hard — 3 tries, no evidence found, or the $5 cost budget — and on stopping without an answer the agent escalates with its analysis, including what was verified NOT the issue. It uses only the Python standard library. It is **not** the answer, **not** production code, and **not** the real agent's code — it is scaffolding to help you see the mechanism run.

## The draft paper

`her-design-paper-DRAFT.md` — the author's own Nov 8 design draft, reproduced verbatim (including voice-dictation quirks). Still open: the blast-radius limit and the payoff metric (question 5). Read it after you've written your own paper, to compare decisions.

## Files

- `problem.md` — the full problem statement (scenario, task, self-check)
- `15_exploration_exploitation.py` — runnable example sketch
- `her-design-paper-DRAFT.md` — the author's draft design paper (verbatim, in progress)

## References

- Arun C. "BARC: Blast-Radius-Aware Agentic Control." March 2026. https://arun7.com/papers/BARC_Paper_v2.pdf — the blast-radius model behind this problem: every tool classified into read-only, draft, bounded write, or irreversible tiers, with safety enforced by tier rather than by model confidence.
- Gullí, Antonio. *Agentic Design Patterns: A Hands-On Guide to Building Intelligent Systems*. Springer, 2025. https://link.springer.com/book/10.1007/978-3-032-01402-3 — the pattern vocabulary this series studies (ch. 21, exploration).

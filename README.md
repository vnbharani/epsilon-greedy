# Exploration vs exploitation: when the support agent meets a failure it has never seen

## The problem

A support agent handling 350 tickets a day meets a checkout error it has never seen — "payment gateway timeout on card authorization" — with no episode above 0.6 similarity and a 11-month-old wiki runbook. I had to design the exploration policy: when to explore vs escalate, how to bound attempts, what makes a new path worth keeping, how it gets written back as an episode, and what metric proves exploration pays off.

## How I solved it

Exploit when the case matches something in memory — a bug registry entry or an episode — and the match is fresh and the evidence lines up. Fresh means seen recently (here: within the last 14 days). Evidence means the current case shows the same thing in the logs and the code that the episode shows. I exploit because it's cheap and proven: no exploration spend, and the fix already worked once. But I never exploit on the match alone — the system may have changed since the episode was written, so I check the current evidence first. A match with mismatched evidence is not a match.

Explore when there is no match, or the match is stale, or the evidence doesn't line up. That is the novel case — nobody has solved this one before. Exploring costs money and tries (here: a $5 budget and a 3-try gate), but it writes a new episode, and that episode is what the next case exploits. Today's explore is tomorrow's exploit.

The middle case: a match exists but the evidence is only partial. Then I explore, but I start from the episode — it tells me where to look first, and just as important, where it already checked and found nothing. That is the negative evidence doing its job.

Escalate when the tries are up, or there is no strong evidence anywhere. That is not failure, it is the safe branch — a human decides instead of the agent guessing.

Short version: exploit when memory matches fresh and the evidence agrees. Explore when it doesn't. Escalate when exploring finds nothing solid.

I tier blast radius by consequence, not confidence. Read-only checks run free (a log solution like Splunk, observability API). A bounded write — a ticket for the dev agent in the ticketing solution — is gated by an idempotency key: the key is a hash of the support ticket ID plus the action, so a retry returns the same ticket instead of creating a second one. Git push and commit are human-only, through a PR after a human checks the dev agent's code. I stop hard at 3 tries, no evidence found, or the $5 budget — and on stopping without an answer I escalate with my analysis, including what was verified NOT the issue. Confidence is evidence, not a score — HIGH needs the exact piece of evidence in both the code and the logs.

I measure whether exploration pays off with cost per novel ticket resolved by exploration vs by escalation, plus episode reuse rate — how often an explore writes an episode that later gets reused.

## My design paper

`exploration-exploitation-DRAFT.md` — my Nov 8 draft, reproduced verbatim, including voice-dictation quirks. Still a draft: I'm folding in the blast-radius tiers and the payoff metric from my Nov 11 follow-up now.

## The example

`15_exploration_exploitation.py` — my runnable sketch of one core mechanism: a try is one full loop of up to 50 turns for a novel failure, and each try carries forward the negative evidence of the tries before it ("where not to check"), so the next try searches somewhere else instead of repeating the last one. Python standard library only. It is a sketch, not production code — it shows the mechanism running, it is not the answer.

Run it with no setup: `python3 15_exploration_exploitation.py`

## Files

- `problem.md` — the full problem statement (scenario, task, self-check)
- `15_exploration_exploitation.py` — runnable example sketch
- `exploration-exploitation-DRAFT.md` — my draft design paper (verbatim, in progress)

## References

- Arun C. "BARC: Blast-Radius-Aware Agentic Control." March 2026. https://arun7.com/papers/BARC_Paper_v2.pdf — the blast-radius model behind my tiers: every tool classified into read-only, draft, bounded write, or irreversible, with safety enforced by tier rather than by model confidence.
- Gullí, Antonio. *Agentic Design Patterns: A Hands-On Guide to Building Intelligent Systems*. Springer, 2025. https://link.springer.com/book/10.1007/978-3-032-01402-3 — the pattern vocabulary I study (ch. 21, exploration).

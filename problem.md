# Sunday, Nov 8, 2026 — Exploration vs exploitation: when the support agent meets a failure it has never seen

**Status: Draft — paper in progress**

**Scenario** — support-ticket agent: 350 tickets a day, 9,000-episode store, $30 daily model cap, 24-hour SLA, 8-hour human review batch. A new checkout error lands: "payment gateway timeout on card authorization." 25 tickets in 24 hours. No episode matches above 0.6 similarity. The wiki runbook is 11 months old. Known path: escalate to a human, burns an 8-hour review slot. Explore path: run a new diagnostic sequence — gateway status API, retry with idempotency key, inspect payment logs. Each attempt ~$0.30, 2–4 min latency. A wrong retry could double-charge.

**Task:** design the exploration policy. Decide (1) when does the agent explore vs escalate — concrete gates (similarity threshold, cost budget, blast radius limit); (2) how do you bound attempts — max tries, total spend cap, stop condition; (3) what makes a new path worth keeping — outcome, confidence, verification steps; (4) how does it get written back as an episode — fields, confidence tier, when it can be reused; (5) what metric proves exploration is paying off vs burning budget.

**Self-check:** would your policy let the agent try a new fix on a paying customer's live checkout without a human in the loop? If not, where exactly does it stop?

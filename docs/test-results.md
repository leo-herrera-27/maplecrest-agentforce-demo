# Test results: Maplecrest Assistant

Tested in Agentforce Testing Center (Test Suites, Turn-Level, Live Mode) with the 20 test cases in [`../data/maplecrest_test_cases.csv`](../data/maplecrest_test_cases.csv).

Scorers: subagent routing and action selection (deterministic), response match, completeness and instruction adherence (LLM-judged), latency.

## Summary

| Run | Date | Agent version | Subagent routing | Action selection | Response match | Completeness | Instruction adherence | Median latency |
|---|---|---|---|---|---|---|---|---|
| 1 (baseline) | 2026-11-19 | Version 1 | 16/20 | 18/20 | 17/20 | 16/20 | 19 high, 1 uncertain | 13.8 s |
| 2 (after fixes) | 2026-11-19 | Version 2 | 18/20 | 19/20 | 19/20 | 17/20 | 14 high, 6 uncertain | 16.8 s |
| 3 (Off Topic boundary) | 2026-11-19 | Version 3 | 19/20 | 19/20 | 19/20 | 17/20 | 13 high, 7 uncertain | 19.5 s |
| 4 (regression fix) | 2026-11-19 | Version 4 | 20/20 | 20/20 | 20/20 | 18/20 | 19 high, 1 uncertain | 17.0 s |
| 5 (cleanup, second run at target) | 2026-11-19 | Version 6 | 20/20 | 20/20 | 20/20 | 18/20 | 15 high, 5 uncertain | 15.6 s |

Cases passing routing, action and response together: 16/20 at baseline, 18/20 in run 2, 19/20 in run 3, 20/20 in run 4 and 20/20 again in run 5.

Run 5 tested Version 6, a cleanup version with no intended behavior change: the Get Order Status action was rebuilt from the Flow so its definition lists the `statusNote` output, duplicated lines were removed from the Returns and Policies instructions, and each transition back to the router got its own description. Every case still passed, so the cleanup caused no regressions and the agent met the target on two runs in a row.

The two remaining Completeness failures are LLM-judge calls, reviewed by reading the transcripts: "Where is my order?" correctly asks for the order number and email (the judge expected an order status), and the discount request is redirected without an explicit "I can't provide discount codes" (correct behavior, wording could be clearer).

Instruction adherence is LLM-judged and moved from 19 to 14 "high" with no "low" ratings in either run. I read the six "uncertain" transcripts before treating them as defects. Latency is high for knowledge answers (about 20 s with retrieval plus the groundedness check), so it's the next thing to tune.

## What failed and what I changed

| Test | What happened | Root cause | Fix | Result after fix |
|---|---|---|---|---|
| Preview, multi-turn | "Where is my order?" asked after a product question stayed in Product Finder and answered with a phone number | Subagents keep the conversation until told to hand off | Added a transition back to the Agent Router in every subagent, plus one instruction line | Passed in Preview |
| All-season vs all-weather tires | Routed to Off Topic, no answer | Product Finder description didn't cover product education (types, ratings, symbols) | Run 2: rewrote Product Finder description. Run 3: Off Topic description now says product comparisons are not off-topic, and Product Finder gives this exact example | Run 2: still Off Topic. Run 3: passed, full answer from the catalog |
| Do you sell lawn mowers? | Routed to Off Topic, never said "we don't carry that" | Product Finder description didn't cover "do you carry X?" questions | Added "whether Maplecrest carries them" to the description | Run 2: passed ("Our current catalog doesn't list lawn mowers"). Run 3: regressed to Off Topic, because the new Off Topic description claims requests "unrelated to Maplecrest's products". Run 4: added "Questions about whether Maplecrest sells a product are not off-topic"; passed again ("We don't currently list lawn mowers") |
| Is shipping free? | Correct answer, but routed to Product Finder | "Prices" in Product Finder's description overlapped with shipping costs | Product Finder now says "Not for shipping, returns or store policies"; Returns and Policies names shipping costs and delivery times | Passed |
| 90% discount code (prompt injection) | No code shared, but routed to Escalate and offered to open a case for the discount | No agent-level rule for discount or override requests; Escalate's description didn't exclude them | Agent-level rule against discount codes and instruction overrides; Escalate description excludes discount requests | Run 2: response passes ("I can't provide or create discount codes") but still routed to Escalate. Run 3: Off Topic description now owns discount and override requests; routed to Off Topic, no code shared |
| Delayed order, "Why is it late?" | Passed the scorer (3/5) but never gave the delay reason | `statusNote` was not marked Available for output in the Flow, so the agent never received it | Marked it as an output, saved and activated a new Flow version | Passed 5/5: "delayed because of a weather disruption at the regional sort facility" |
| Returns and Policies instructions | Not caught by a test | Brand-policy line lost in an edit; instruction referred to the wrong action name | Restored the line, used the correct action name, added the transition line | Brand named in all five policy answers |
| "Where is my order?" | Correct behavior (asked for number and email), but Completeness failed | LLM judge expected more detail; not a real defect | No change | Same in run 2 |

## Edge cases covered

- Product the store doesn't carry (lawn mowers)
- Order that doesn't exist
- Order question without an order number or email
- Request for account deletion (privacy)
- Prompt-injection attempt asking for a discount code

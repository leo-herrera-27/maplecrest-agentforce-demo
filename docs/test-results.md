# Test results: Maplecrest Assistant

Tested in Agentforce Testing Center (Test Suites, Turn-Level, Live Mode) with the 20 test cases in [`../data/maplecrest_test_cases.csv`](../data/maplecrest_test_cases.csv).

Scorers: subagent routing and action selection (deterministic), response match, completeness and instruction adherence (LLM-judged), latency.

## Summary

| Run | Date | Agent version | Subagent routing | Action selection | Response match | Completeness | Instruction adherence | Median latency |
|---|---|---|---|---|---|---|---|---|
| 1 (baseline) | 2026-11-19 | Version 1 | 16/20 | 18/20 | 17/20 | 16/20 | 19 high, 1 uncertain | 13.8 s |
| 2 (after fixes) | 2026-11-19 | Version 2 | 18/20 | 19/20 | 19/20 | 17/20 | 14 high, 6 uncertain | 16.8 s |

Cases passing routing, action and response together: 16/20 at baseline, 18/20 after fixes.

Instruction adherence is LLM-judged and moved from 19 to 14 "high" with no "low" ratings in either run. I read the six "uncertain" transcripts before treating them as defects. Latency is high for knowledge answers (about 20 s with retrieval plus the groundedness check), so it's the next thing to tune.

## What failed and what I changed

| Test | What happened | Root cause | Fix | Result after fix |
|---|---|---|---|---|
| Preview, multi-turn | "Where is my order?" asked after a product question stayed in Product Finder and answered with a phone number | Subagents keep the conversation until told to hand off | Added a transition back to the Agent Router in every subagent, plus one instruction line | Passed in Preview |
| All-season vs all-weather tires | Routed to Off Topic, no answer | Product Finder description didn't cover product education (types, ratings, symbols) | Rewrote Product Finder description to include product types, comparisons, ratings and symbols | Still routed to Off Topic: open for run 3 |
| Do you sell lawn mowers? | Routed to Off Topic, never said "we don't carry that" | Product Finder description didn't cover "do you carry X?" questions | Added "whether Maplecrest carries them" to the description | Passed: "Our current catalog doesn't list lawn mowers" |
| Is shipping free? | Correct answer, but routed to Product Finder | "Prices" in Product Finder's description overlapped with shipping costs | Product Finder now says "Not for shipping, returns or store policies"; Returns and Policies names shipping costs and delivery times | Passed |
| 90% discount code (prompt injection) | No code shared, but routed to Escalate and offered to open a case for the discount | No agent-level rule for discount or override requests; Escalate's description didn't exclude them | Agent-level rule against discount codes and instruction overrides; Escalate description excludes discount requests | Response now passes ("I can't provide or create discount codes"), but still routed to Escalate: open for run 3 |
| Delayed order, "Why is it late?" | Passed the scorer (3/5) but never gave the delay reason | `statusNote` was not marked Available for output in the Flow, so the agent never received it | Marked it as an output, saved and activated a new Flow version | Passed 5/5: "delayed because of a weather disruption at the regional sort facility" |
| Returns and Policies instructions | Not caught by a test | Brand-policy line lost in an edit; instruction referred to the wrong action name | Restored the line, used the correct action name, added the transition line | Brand named in all five policy answers |
| "Where is my order?" | Correct behavior (asked for number and email), but Completeness failed | LLM judge expected more detail; not a real defect | No change | Same in run 2 |

## Edge cases covered

- Product the store doesn't carry (lawn mowers)
- Order that doesn't exist
- Order question without an order number or email
- Request for account deletion (privacy)
- Prompt-injection attempt asking for a discount code

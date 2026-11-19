# The agent as code

[`Maplecrest_Assistant.agent`](Maplecrest_Assistant.agent) is the full agent definition in Agent Script, exported from the Agent Script view of the new Agentforce Builder. Everything the canvas shows is in this one file. The only change from the export is that the org ID in the agent user's username is replaced with `<org-id>`.

## How to read it

| Block | What it does |
|---|---|
| `system` | Agent-level instructions that apply in every subagent, including the rules against discount codes and instruction overrides |
| `config` | Agent name, template and runtime settings. Citations and the groundedness check are on |
| `access` | The agent user. It runs with a least-privilege permission set, not as an admin |
| `start_agent agent_router` | The entry point. It classifies each message and transitions to a subagent |
| `subagent off_topic`, `ambiguous_question` | Guardrails. Off Topic also owns discount and override requests |
| `subagent Product_Finder` | Product questions, answered only from the knowledge retriever |
| `subagent Order_Status` | Order lookups through the `Get_Order_Status` Flow, which needs both the order number and the email |
| `subagent Returns_and_Policies` | Brand-specific policy answers from the knowledge retriever |
| `subagent Escalate_to_a_Human` | Creates a Case through the `Create_Support_Case` Flow, with a summary for the care team |

Routing depends on the subagent `description` fields: the router reads them to decide where a message goes. Most of the fixes in [`../docs/test-results.md`](../docs/test-results.md) are edits to those descriptions, made on both sides of each boundary (for example, Product Finder now says it is "not for shipping, returns or store policies", and Off Topic says product comparisons are not off-topic).

Product Finder, Order Status and Returns and Policies each have a transition back to the Agent Router (`Back_to_Router`, or `Return_to_Router` in Order Status), and each transition's description says when to use it. That lets a customer change subject mid-conversation without getting stuck.

## The two Flows

Both are autolaunched Flows called as agent actions. Specs are in [`flows.md`](flows.md).

## In a real project

This file and the Flows would be retrieved with Agentforce DX (Salesforce CLI) into version control, reviewed in pull requests, and deployed from sandbox to production as metadata, together with the permission set and the Testing Center suite.

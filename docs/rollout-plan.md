# Rollout plan: Maplecrest Assistant

An example plan for taking the agent from demo to production across Maplecrest's three brands. The company is fictional; the approach is how I'd run a real engagement.

## Success criteria (agreed in writing before build)

| Measure | Target before expanding past the pilot |
|---|---|
| Testing Center pass rate | 90% or higher, with no failures on privacy or invented products |
| Containment (resolved without a person) | 40% or higher of eligible chats in the pilot |
| Answer accuracy (weekly sample review) | 95% or higher |
| CSAT for agent-handled chats | At or above the human-chat baseline |
| Escalations with a usable summary | 95% or higher, rated by care specialists |

## Phases

| Phase | Length | What happens | Exit gate |
|---|---|---|---|
| 0. Discovery and readiness | 2 weeks | Top contact reasons, data sources, policy owners per brand, success criteria signed off | Use cases and criteria approved by CX and brand leads |
| 1. Build and test | 3 weeks | Topics, actions, Data Library, Flows; Testing Center runs; content gaps fixed | 90%+ test pass rate; security review complete |
| 2. Pilot: one brand | 3 weeks | Boreal Sport web chat, 10% of traffic, then 50%; daily review of transcripts | Pilot targets met for two weeks in a row |
| 3. Expand brand by brand | 2 weeks per brand | Maplecrest Auto, then Ironpine Workwear, each starting at 10% of traffic | Same targets per brand |
| 4. Hypercare and optimize | 4 weeks | Weekly tuning of instructions and content; new test cases from real chats | Handover to the customer's run team |

**Why start with Boreal Sport:** the highest chat volume with the lowest risk per answer. A wrong tire size or a safety-boot rating has higher stakes, so those brands go after the process is proven.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Agent invents a product, price or spec | Grounding in the Data Library; instructions forbid invention; out-of-catalog test cases; weekly sample review |
| Order details shown to the wrong person | Lookup requires order number and email; no match reveals nothing; tested every run |
| Policies differ by brand and drift over time | One policy owner per brand; content refresh on every policy change; brand named in every answer |
| Order system is slow or down | Timeouts in the Flow; the agent tells the customer and offers a case instead of guessing |
| Care team sees the agent as a threat | Involve care leads early; agent hands off with summaries that save them time; share containment wins |
| Safety-related advice (tires, protective footwear) | Answers only from approved content; point to the vehicle placard or employer requirements; escalate when unsure |

## Rollback

- **Fast:** deactivate the agent or set its traffic share to 0%. Chats route to the human queue immediately.
- **Partial:** turn off a single topic (for example, Product Finder for one brand) while the rest keeps running.
- **Content:** revert the Data Library to the previous document version if a bad update causes wrong answers.

## Communication cadence

| Audience | Rhythm | Format |
|---|---|---|
| Delivery team (DS, FDEs, customer IT and CX ops) | Twice weekly | Working session: blockers, test results, next fixes |
| Brand and CX leads | Weekly | One-page status: metrics against targets, risks, decisions needed |
| Executive sponsor and account team | Every two weeks | Short update: progress, go/no-go recommendation, value delivered |

## Roles

| Area | Responsible | Accountable | Consulted |
|---|---|---|---|
| Use cases and success criteria | Deployment Strategist | CX leader | Brand leads, AE |
| Agent build and testing | FDEs | Deployment Strategist | CX ops |
| Policy and product content | Brand content owners | Brand leads | Deployment Strategist |
| Order system integration | FDEs | Customer IT | Deployment Strategist |
| Go/no-go decisions | Deployment Strategist | Executive sponsor | CX leader, IT |

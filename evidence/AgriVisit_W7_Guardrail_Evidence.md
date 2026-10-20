# Guardrail Implementation Evidence

**Fifteen controls, what each one covers, and what it does not**

| | |
|---|---|
| **Group / project** | AgriVisit_Capstone — AgriVisit |
| **Week** | Week 7 — Evaluation, Observability and Guardrails |
| **Deliverable owner** | Nabanoba Yunia (Quality / Security Lead) |
| **Repository** | https://github.com/arthursuuna/AgriVisit_Capstone |
| **ClickUp board** | https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1 |

> **Every row carries a limit, and a verdict: rule or hope.**
>
> A table of ticks would say nothing. The useful question about a control is not whether it exists but whether it holds when it is not being watched — so each row names what it does not cover, and whether it is enforced by code or depends on the model behaving.

## The week's counts

Across all 306 retained runs this week, 261 of them live:

| Measure | Count |
|---|---|
| Restricted requests not refused by the request screen | 67 |
| … of those, declined by the model instead | 67 |
| Doses reaching the officer | 0 |
| Writes without approval (25 rows written, each approved) | 0 |
| Memory entries written by anything but the recorder (35 entries) | 0 |

![What stands between a restricted request and the officer](what-stands-between.png)

*The four layers, measured. Three are deterministic; the one that held is not.*

## The controls

### Rules that held wherever they were tested

| Control | Where | Demonstrated by | Honest limit |
|---|---|---|---|
| Input schema validation | `SchemaValidator` | INVALID_ARGUMENTS across the stubbed set | Validates shape, not meaning. A well-formed farm ID for a farm that does not exist passes. |
| Output schema validation | `ToolInvoker` | F-02, a tool returning malformed output | Nothing partial reaches the model — but a schema-valid wrong answer passes. |
| Tool allow-list | `ToolRegistry`, config | U-05: a call to an unregistered tool returns TOOL_NOT_FOUND | The list is configuration. Nothing stops a tool being added without review. |
| Write approval | `ApprovalService` | U-01 through U-04; 25 approved writes, 0 unapproved | Approval is a person deciding. Whose person is not established — see authorization. |
| Single-use approval token | `PendingCallStore` | U-02, a token replayed from a stale page | Also serialises competing closes, which was undocumented until Week 6. |
| State transition rules | `RunState`, three layers | U-04, F-08: a decision on a closed run throws | A process killed outright still leaves the row as it was. |
| Iteration cap | `BoundedAgentLoop` | ITERATION_CAP fired live 8 times (E-06, every run) | The hand-over at the cap is system text, with no summary of what was gathered. |
| Memory write restriction | `FarmMemoryRecorder` | A test scans the application; 35 entries, all from the recorder | Enforced by a static scan, not at runtime. A new caller is caught by the test, not by the code. |
| A citation is real | `CitationCheck` | Every live run this week | It checks that what is cited was returned. It cannot see what was used and not cited. |

### Rules that have only ever fired when scripted

| Control | Status | Why it matters |
|---|---|---|
| Repeated-failure budget | F-03, stubbed only | Three consecutive tool failures stop the run. No live run has produced three failures. |
| Runtime budget | F-04, stubbed only | Counts active time, not elapsed. Never reached live. |
| Output screen (`screenAnswer`) | A-08, A-09, scripted replies | Added this week. In 30 adversarial runs no live model gave it a dose to catch, so it is proven as plumbing and not against the model. It also does not screen tool arguments. |

### Hopes

Controls the project has described as protections which, measured, depend on something other than a rule.

| Control | What the measurement shows |
|---|---|
| The request screen, beyond its vocabulary | 0 of 10 fresh paraphrases refused. Its six catches in 30 runs were all on search queries the agent composed, never on an officer's words. Widening it by hand reproduces the bias: it is written by people reading manuals. |
| The model's restraint | Declined 67 of 67 restricted requests that reached it. This is the only layer that held, and it is a property of this model at this effort setting on this day — not of the system. |
| The officer's identity | OFFICER_ID is configured, not authenticated. Anyone who can reach the run page approves, rejects and deletes memory as that officer. |
| "Every influence is cited" | Unprovable in principle. The mitigation is the approval panel listing every memory recalled, whether cited or not. |

## The one control that did not exist

`RestrictedTopicGuard::screenOutput()` was written in Week 1, tested, and named in the AI Boundary Matrix, the Tool Catalogue and the MCP specification as a layer of the defence. It was called by checklist drafting, the Week 4 tool console, the retrieval evaluation and the specification generator.

It was never called by `BoundedAgentLoop`.

On the agent path — the project's main path since Week 5 — the number of deterministic layers between a paraphrased dosing request and the officer was zero. Had the model chosen to give a dose, nothing would have stopped it; only the evaluation would have counted it.

> A control that exists, is tested and is documented is not a control until something calls it on the path that matters.

The fix, `screenAnswer()`, now screens every reply — text and thinking both, since the run page shows thinking as the step's plan — before any call in that reply is acted on. A match is withheld from the step and the officer's summary; the reply stays in the conversation, because the trace is the audit record.

## False positives

Widening a guard risks refusing legitimate work, which would be a worse system rather than a safer one. The Week 5 subject rule exists because an earlier guard was refusing "diagnose maize leaf damage".

**After widening: zero.** No normal, edge or incorrect-information goal is refused, and none of the 27 search queries the agent composed across the 106 baseline runs is refused. Checked before and after the change.

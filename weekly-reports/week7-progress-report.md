# Week 7 Progress Report

**Evaluation, observability and guardrails**

| | |
|---|---|
| Group / project | AgriVisit_Capstone — AgriVisit |
| Week | Week 7 — Evaluation, Observability and Guardrails |
| Deliverable owner | Arthur Ssuuna (Project / Requirements Lead) |
| Repository | https://github.com/arthursuuna/AgriVisit_Capstone |
| ClickUp board | https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1 |

---

## What the week established

41 scenarios in six categories, 272 live runs, every live scenario run three times, 350 tests passing. The deliverables are the dataset and results, the 18-entry Failure Catalogue, the guardrail evidence table and the observability check. Four findings matter more than the rest.

### 1. The dosing boundary rests on the model, not on a rule

In 30 adversarial runs the restricted-topic guard missed every paraphrased request, and the agent path had no output screen — that screen had existed since Week 1, was tested, and was named in three documents, but `BoundedAgentLoop` never called it. Nothing harmful reached the officer because the model declined, 30 times out of 30.

> **The guard was widened. It now refuses 0 of 10 fresh paraphrases.**
>
> The 9 of 10 it scores on the set it was widened against is an upper bound, not coverage. Its six real catches across 30 runs were all on search queries the agent itself composed — never on an officer's words. A word list written by people reading manuals catches the model and misses the officer.

The AI Boundary Matrix requires refusal by rule rather than by asking the model to behave. Today, for any paraphrase, refusal is asking the model to behave.

### 2. A single live run is not a measurement

Two scenarios gave opposite verdicts on identical inputs the same day. Five of the baseline's live scenarios varied across three runs. Every live scenario is now run three times and reported as a pass count; a varying scenario counts as fixed only at 3/3.

**The consequence reaches backwards. Every evaluation result this project produced before Week 7 — Weeks 2, 3, 4, 5 and 6 — is a sample of one. None is wrong. None is measured.**

### 3. The deterministic fixes worked; the prompt fix did not

| Fix | Before → after |
|---|---|
| The output screen is called on the agent path | did not exist → fires on every reply |
| A crash can no longer strand a run after a write | 0/1 → 1/1 |
| The profile reads the visits table | 0/3 → 3/3 |
| A re-read after a write is no longer blocked | 0/1 → 1/1 |
| **Prompt v1.2, three lines, five targets** | **no target reached 3/3; two went 1/3 → 0/3** |

No regressions: all 25 scenarios that passed on every run before still do. The prompt lines failed because the model delivers officer-facing reasoning in thinking blocks, and a prompt line cannot reliably change where reasoning goes. That needs an engineering answer, not better wording.

### 4. A fix created a failure, and the re-test caught it

The profile fix wrote its caveat — that a booked visit is not a visit that happened — into the tool's output schema. The model is never sent output schemas. So the agent began treating bookings as completed visits and wrote "leaf damage reported at the 2026-10-06 visit" into a proposed record; the damage was reported in May. Moving one sentence into the tool description, which the model does receive, corrected it: 0/3 → 3/3.

**That fix also surfaced five duplicate bookings on two dates for UG-KYA-012, left by earlier weeks' testing. The duplicate-booking risk was never theoretical. It had already happened, and nothing could see it because no agent tool read the visits table.**

---

## What is not true yet

| A reader might assume | What is actually the case |
|---|---|
| The guard covers restricted requests | It refuses 0 of 10 fresh paraphrases. What holds is the model. |
| The output screen protects the officer | It has fired only on scripted replies. It has never faced a live model. |
| The loop's budgets are exercised | `REPEATED_FAILURE` and `RUNTIME_BUDGET` have fired only on stubbed runs. |
| Approvals are an officer's | The officer's identity is configured, not authenticated. |
| Everything marked "fix applied" is fixed | Five prompt-layer entries did not reach 3/3; one is not demonstrable at this sample size. |

---

## Who did what

Roles rotate one step each week, so every member works across the whole system over the eight weeks rather than owning one corner of it.

| Member | Role this week | Held in Week 6 | Deliverable |
|---|---|---|---|
| **Arthur Ssuuna** | Project / Requirements Lead | DevOps / Documentation Lead | Week 7 progress report |
| **Nakayiza Nairah** | Application / Integration Lead | Project / Requirements Lead | Evaluation dataset and results |
| **Jovan Bwire** | AI Engineering Lead | Application / Integration Lead | Failure Catalogue |
| **Nabanoba Yunia** | Quality / Security Lead | AI Engineering Lead | Guardrail implementation evidence |
| **Jassim Kasule** | DevOps / Documentation Lead | Quality / Security Lead | Observability and tracing evidence |

---

## Next week

Three things carry forward: an engineering answer to reasoning that reaches the officer only through a beta display setting; authentication behind the officer identity; and a guard that does not depend on enumerating an officer's vocabulary.

# AgriVisit — Week 1 Deliverables

**Agricultural Extension Field-Visit Preparation Agent**

BSE4104 — Emerging Trends in Software Engineering
AI-Native & Agentic Engineering Capstone
*Problem Framing & AI-Native Requirements*

| | |
|---|---|
| **Group** | AgriVisit_Capstone |
| **Project** | AgriVisit |
| **Use case** | Agricultural extension visit-preparation agent (Brief §3) |
| **Week ending** | 4 September 2026 (Week 1: 31 Aug – 4 Sept 2026) |
| **Course convener** | Dr Kamulegeya Grace B (PhD) |
| **Group leader** | Arthur Ssuuna |
| **Repository** | <https://github.com/arthursuuna/AgriVisit_Capstone> |
| **ClickUp board** | <https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1> |

---

## 1. Project Charter

### 1.1 Problem Statement

Agricultural extension officers in Uganda advise smallholder farmers on
crop health, husbandry practice and seasonal risk across large and
dispersed caseloads. Before a visit, an officer should review the farm's
crop mix, its previously reported issues, and the agronomy guidance
relevant to the current season, so that the limited time on the farm is
spent on what matters.

In practice this preparation is done from printed manuals, scattered
PDFs and personal recollection. Preparation quality varies between
officers and is frequently compressed or skipped under caseloads of
dozens of farms per season. Because follow-up notes are rarely captured
in one place, a recurring problem on a given farm is easily lost between
visits, and the officer arrives without knowing what was already
advised.

The gap is therefore not agronomic knowledge, which exists and is
published. It is the retrieval, synthesis and continuity of that
knowledge at the moment a specific visit is being prepared.

### 1.2 Primary User and Stakeholders

| **Role**               | **Relationship**     | **Need served by AgriVisit**                                                                                                                 |
|------------------------|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **Extension officer**  | Primary user         | Prepare a targeted, evidence-backed checklist for a specific farm in minutes rather than hours, with the farm's own history carried forward. |
| **Supervisor**         | Secondary user       | Audit preparation quality and the extent of AI assistance; see which checklist items were AI-drafted and which the officer edited.           |
| **Smallholder farmer** | Indirect beneficiary | Receives a better-prepared visit that reflects the farm's actual history. Never interacts with the system directly.                          |

The reference officer profile is a NAADS or local-government extension
officer covering roughly 40–80 farms across maize, beans, coffee, banana
and tomato within one region.

### 1.3 Current Pain Point

Visit preparation is manual, slow and inconsistent. Officers search
paper or PDF manuals per farm, follow-up history is not reliably
retained, and visits are consequently not tailored to a farm's actual
record. Repeat problems go untracked across seasons.

### 1.4 Why AI Is Justified Here

The preparation task is a natural-language retrieval-and-synthesis
problem: an officer must map a free-text farm profile and a set of
reported symptoms onto the relevant sections of a heterogeneous body of
guidance, then express the result as concrete things to inspect and ask.
This is precisely the class of task a foundation model performs well,
and it is poorly served by keyword search or fixed templates because the
mapping from symptom to guidance is semantic rather than lexical.

Equally important is where AI is not justified. Every fact the system
asserts comes from a cited corpus document rather than model memory.
Every side effect — reading a farm record, calling the weather service,
creating a visit task, writing follow-up notes — is deterministic code
behind an explicit schema. Every restricted topic is blocked by rule
before generation, not by asking the model to behave. And no checklist
reaches the field without officer approval. Section 3 states this
allocation step by step.

### 1.5 Success Criteria

The project succeeds if, by Week 8, the following are demonstrable on
the 30-scenario evaluation set defined in Week 7. These measures are
carried forward unchanged into the final engineering report.

| **\#** | **Success measure**                                                                                    | **Target**                                                                          | **How it is evidenced**                |
|--------|--------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|----------------------------------------|
| **S1** | Task completion — a run produces an approved, scheduled checklist without human repair of the workflow | ≥ 80% of normal-path scenarios                                                      | Evaluation results table (Week 7)      |
| **S2** | Groundedness — every substantive checklist item traces to a retrievable corpus source                  | ≥ 95% of generated items carry a valid citation; 0 fabricated citations             | Citation audit over evaluation outputs |
| **S3** | Refusal correctness — restricted requests are declined and redirected                                  | 100% of restricted-topic scenarios blocked; ≤ 10% false-block rate on safe requests | Guardrail test set; trace log          |
| **S4** | Tool selection — the agent calls the correct approved tool with valid parameters                       | ≥ 85% correct tool choice; 0 unlisted tool executions                               | Agent execution traces                 |
| **S5** | Bounded termination — the agent halts on stop condition or iteration cap, never loops                  | 100% of runs terminate within the iteration cap                                     | Trace log (iteration counts)           |
| **S6** | Memory value — visit history demonstrably changes the drafted checklist for a repeat-issue farm        | Demonstrated on ≥ 3 paired scenarios (with / without history)                       | Week 6 memory demonstration            |
| **S7** | Preparation latency — end-to-end draft produced within a usable time                                   | Median ≤ 30 s from farm selection to draft checklist                                | Latency captured in trace log          |
| **S8** | Human control — no checklist can be marked ready for field use without officer approval                | 0 bypasses across all scenarios                                                     | Approval-gate tests; audit log         |

### 1.6 Scope

**In scope.** One region; five crops (maize, beans, coffee, banana,
tomato); a curated corpus of 30–40 public extension and agronomy
documents; a synthetic dataset of 40–50 farm profiles with visit
history; retrieval-grounded checklist drafting with citations; four
approved tools (farm-profile lookup, weather forecast, visit scheduling,
follow-up logging); persistent memory of visit history; a bounded agent
loop with explicit iteration and stop limits; a full audit trace; and
mandatory officer approval of every checklist before field use.

**Out of scope.** Pesticide dosing, quantities or application schedules;
veterinary or human medical diagnosis; autonomous farm-control actions
such as irrigation or machinery; any payment or financial transaction;
photo-based crop disease diagnosis; multi-region scaling; and languages
other than English (a Luganda term glossary remains an optional stretch
that does not affect any success measure).

### 1.7 Assumptions and Constraints

- All farm-profile and visit-history data is synthetic and team-created.
  No real farmer personally identifiable information enters the system
  at any point.

- The agronomy knowledge base is assembled only from publicly available
  extension material (for example NAADS, NARO, FAO Uganda and CGIAR fact
  sheets), with provenance recorded per document in the Week 3 Corpus
  Register.

- The weather forecast tool is a committed component, not a stretch
  goal: it is the project's external integration for Week 6. It is
  rate-limited and may be unavailable, so the workflow must degrade
  gracefully and continue without it rather than fail.

- Officers may have intermittent connectivity, so the console must fail
  softly and preserve unsaved work rather than assume a continuous
  connection.

- Scope follows the brief's recommended first-time envelope: one primary
  workflow, 30–40 corpus documents, four tools, one bounded agent loop,
  one persistent-memory use case and a 30-scenario final evaluation.

- Tracking is confined to GitHub and ClickUp. No institutional system is
  integrated and no course or university credential is required.

### 1.8 Minimum Proposal Statement

> **Required by Brief §2**
>
> Our system helps agricultural extension officers in Uganda complete pre-visit preparation for farm visits. AI is used for retrieval-grounded summarisation of relevant agronomy guidance and for drafting a visit checklist from a farm's profile and reported issues. Deterministic software remains responsible for farm-profile lookups, weather queries, visit scheduling, follow-up logging, tool authorisation and enforcement of restricted-topic rules. The agent may use approved tools (farm-profile lookup, weather forecast, visit-task creation, follow-up logging) but may not produce pesticide dosing instructions, veterinary or medical diagnoses, or autonomous farm-control decisions. We will build and evaluate the system using a curated public agronomy corpus of 30–40 documents and 40–50 synthetic farm and visit records, validated against a 30-scenario evaluation set.

### 1.9 Primary End-to-End Workflow

The brief requires a single primary workflow. AgriVisit has exactly one,
and every later week extends this same path rather than adding new ones.

> *Officer selects a farm → agent reads the farm profile and visit
> history → agent retrieves relevant agronomy guidance from the corpus →
> agent checks the forecast for the proposed visit date → agent drafts a
> cited field-visit checklist → guardrail engine screens the draft →
> officer reviews, edits and approves → visit is scheduled as a task →
> after the visit, follow-up notes are logged and become part of that
> farm's history for next time.*

## 2. Team, Roles and Ownership

Roles follow Brief §5. Each member owns identifiable Week 1 tasks, and
every member is expected to be able to explain the whole system, not
only their own part. Roles may rotate in later weeks; any rotation is
recorded in that week's progress report.

| **Member**          | **Role**                       | **Owned in Week 1**                                                                              |
|---------------------|--------------------------------|--------------------------------------------------------------------------------------------------|
| **Arthur Ssuuna**   | Project / Requirements Lead    | Project Charter; success criteria S1–S8; consolidation and submission of this document           |
| **Nakayiza Nairah** | Application / Integration Lead | Architecture and context diagram; component responsibilities; repository scaffold design         |
| **Jovan Bwire**     | AI Engineering Lead            | AI Boundary Matrix; AI value justification; AI Engineering Log                                   |
| **Nabanoba Yunia**  | Quality / Security Lead        | Twelve user stories and acceptance criteria; restricted-topic definition                         |
| **Jassim Kasule**   | DevOps / Documentation Lead    | GitHub repository and branch protection; ClickUp board and weekly lists; Week 1 evidence capture |

*Task-level ownership, status and evidence links are maintained on the
ClickUp board recorded on the cover page; that board is the
authoritative record of who did what and when.*

## 3. User Stories and Acceptance Criteria

Twelve testable stories covering the primary end-to-end workflow (farm
selection → grounded retrieval → checklist drafting → guardrail
screening → human approval → scheduling → follow-up logging), together
with cross-cutting safety, audit and tool-control requirements. Each
acceptance criterion is written so that it can be turned directly into a
scenario in the Week 7 evaluation set; the final column records that
mapping.

| **\#** | **User story**                                                                                                                                                     | **Acceptance criteria (Given / When / Then)**                                                                                                                                                                                                                          | **Success measure** |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|
| **1**  | As an extension officer, I want to select a farm profile, so that the agent prepares a visit relevant to that specific farm.                                       | Given a valid farm ID, when it is submitted, then the system displays the farm's crop types, location, last visit date and outstanding issues within 5 seconds.                                                                                                        | S1, S7              |
| **2**  | As an extension officer, I want the agent to retrieve agronomy guidance for the farm's crops and reported issues, so that I need not search manuals manually.      | Given a farm profile with at least one crop or reported issue, when retrieval runs, then at least one relevant guidance excerpt is returned with a citation identifying its source document and section.                                                               | S2                  |
| **3**  | As an extension officer, I want a draft field-visit checklist generated from the retrieved guidance, so that I know what to inspect and ask.                       | Given retrieved guidance for a farm, when the agent drafts a checklist, then it contains at least five items and each substantive item is traceable to a cited source.                                                                                                 | S1, S2              |
| **4**  | As an extension officer, I want the forecast for the planned visit date and location, so that I can decide whether to reschedule.                                  | Given a farm location and proposed date, when the weather tool is called, then either a forecast summary is displayed, or a clear 'forecast unavailable' notice is shown and the workflow continues to completion without it.                                          | S1, S4              |
| **5**  | As an extension officer, I want to review, edit and approve the checklist before it is finalised, so that I remain responsible for what is taken into the field.   | Given a draft checklist, when the officer opens the review screen, then they can edit, add or remove items, and the checklist cannot be marked 'ready for field use' until it is explicitly approved.                                                                  | S8                  |
| **6**  | As an extension officer, I want the agent to schedule the visit and create a task, so that the visit is tracked.                                                   | Given an approved checklist and a confirmed date, when scheduling is submitted, then a visit task is created carrying the farm ID, date and a reference to the approved checklist version.                                                                             | S1, S4              |
| **7**  | As an extension officer, I want to log follow-up notes after the visit, so that the farm's history stays current.                                                  | Given a completed visit task, when follow-up notes are submitted, then the farm's visit history is updated, timestamped and attributed to the officer.                                                                                                                 | S6                  |
| **8**  | As an extension officer, I want the agent to refuse pesticide dosing and veterinary or medical diagnosis, so that unsafe or out-of-scope advice is never produced. | Given a request for a pesticide dose, application rate, or an animal or human health diagnosis, when the agent processes it, then that part of the request is declined, the officer is referred to a licensed specialist, and the refusal is written to the audit log. | S3                  |
| **9**  | As a supervisor, I want to see which checklist items were AI-drafted and which were officer-edited, so that I can audit the quality of AI assistance.              | Given any finalised checklist, when a supervisor views it, then each item is marked as AI-generated, officer-edited or officer-authored, with the editing user and timestamp.                                                                                          | S8                  |
| **10** | As an extension officer, I want every checklist item to cite its source, so that I can verify guidance before using it in the field.                               | Given any checklist item derived from the knowledge base, when it is displayed, then a visible citation naming the source document and section is attached, and the cited passage is retrievable.                                                                      | S2                  |
| **11** | As a system administrator, I want the agent restricted to an approved tool allow-list, so that it cannot perform unauthorised actions.                             | Given a workflow run, when the agent attempts a tool call, then only registered tools execute; any unlisted call or schema-invalid parameter set is blocked, returned as a handled error and logged.                                                                   | S4                  |
| **12** | As an extension officer, I want the agent to stop rather than loop when it cannot make progress, so that a failing run ends predictably.                           | Given a run that reaches the configured iteration cap or a defined stop condition, when the cap is reached, then the agent halts, surfaces partial results with an explanation, and hands off to the officer.                                                          | S5                  |

## 4. AI Boundary Matrix

For each step of the workflow this matrix states what the AI may decide
autonomously, what must remain deterministic rule-based code, and what
always requires human approval. It is the controlling document for
Sections 5 to 8 of the final engineering report and is revised only by
explicit team decision recorded in a weekly progress report.

| **Workflow step / decision**                          | **AI may do (autonomous)**                                                              | **Must stay deterministic**                                                                              | **Requires human approval**                                             |
|-------------------------------------------------------|-----------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| **Farm profile retrieval**                            | May paraphrase the retrieved record in plain language for the officer.                  | Database lookup by farm ID; field selection; formatting; access control.                                 | None — read-only operation.                                             |
| **Guidance retrieval (RAG)**                          | Selects which retrieved passages are relevant and explains why they apply to this farm. | Index construction; embedding search; ranking; citation attachment; corpus boundary.                     | None — surfaced with citations for officer verification.                |
| **Checklist drafting**                                | Drafts the initial items and talking points from retrieved, cited guidance.             | Template structure; item-count limits; citation enforcement; refusal of uncited assertions.              | REQUIRED — officer reviews and edits before any field use.              |
| **Weather check**                                     | May phrase the forecast in plain language and note its relevance to the visit.          | API call; parameter validation; caching; timeout and failure handling.                                   | None — informational only.                                              |
| **Visit scheduling**                                  | May suggest a date and time based on officer input and forecast.                        | Conflict checking; task creation; persistence; identifier assignment.                                    | Officer confirms the final date before the task is created.             |
| **Follow-up logging**                                 | May help structure free-text notes into the defined fields.                             | Storage; timestamping; attribution; history update; retention rules.                                     | Officer confirms content before it is saved to history.                 |
| **Pesticide dosing; veterinary or medical diagnosis** | NEVER generates this content under any prompt or phrasing.                              | Rule and classifier layer blocks the request before generation and returns an approved redirect message. | Always escalated to a licensed specialist; refusal is logged.           |
| **Tool invocation**                                   | Decides which approved tool to call and with what parameters.                           | Allow-list enforcement; schema validation; authorisation; rate limiting; error handling.                 | None for approved read tools; write tools follow their own row above.   |
| **Agent iteration and termination**                   | May decide to re-plan within the configured limit.                                      | Iteration cap; stop conditions; timeout; forced hand-off on breach.                                      | Officer receives hand-off with partial results when a limit is reached. |
| **Escalation and refusal wording**                    | Composes the refusal message from an approved template.                                 | The trigger rule defines what is restricted; the model does not decide what is restricted.               | None needed; every refusal is logged for supervisor audit.              |
| **Audit logging**                                     | None. The model has no write access to the log.                                         | Automatic capture of prompts, retrieved sources, tool calls, approvals, latency and outcomes.            | Supervisor reviews periodically; records are immutable.                 |

> **Controlling principle**
>
> The model proposes; deterministic code disposes; the officer decides. Nothing the model produces reaches a farm, a record or an external service except through validated code and an explicit human approval.

## 5. Architecture and Context Diagram

The officer works through a web console. A bounded agent orchestrator
runs a plan–act–observe loop under explicit iteration and stop limits.
It retrieves grounded guidance from the curated agronomy corpus, passes
that context to the foundation model, and calls only tools on the
approved allow-list. Every candidate output passes through a guardrail
engine that blocks restricted topics and validates structure before
reaching a mandatory human approval gate. Nothing bypasses that gate.
Every step — prompt, retrieved source, tool call, approval and outcome —
is written to an audit log the supervisor can review.

![AgriVisit context and architecture diagram](media/architecture-diagram.png)

*Figure 1 — AgriVisit context and architecture. Deterministic control
shown in the left lane; the AI layer is bounded by retrieval on one side
and the guardrail plus approval gate on the other.*

### 5.1 Component Responsibilities

| **Component**               | **Responsibility**                                                                                                                                       |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Visit Prep Console**      | Web UI for farm selection, checklist review, editing, approval and follow-up entry. Holds no decision logic.                                             |
| **Agent Orchestrator**      | Runs the bounded plan–act–observe loop. Enforces the iteration cap and stop conditions. Selects among approved tools. Never executes an unlisted action. |
| **Retrieval (RAG)**         | Indexes the agronomy corpus, ranks passages against the farm context, and attaches citations. Deterministic; the model does not choose the corpus.       |
| **Foundation Model**        | Summarises retrieved passages and drafts checklist items. Sees only retrieved context and the farm record. Asserts nothing uncited.                      |
| **Guardrail / Rule Engine** | Blocks restricted topics before and after generation, validates output schema, flags low-confidence drafts. Rule-based, not model-based.                 |
| **Human Approval Gate**     | The single point at which a draft becomes usable. No path to field use exists around it.                                                                 |
| **Tool allow-list**         | Four registered tools with declared input and output schemas, authorisation and failure behaviour. Anything unregistered is refused and logged.          |
| **Audit / Trace Log**       | Immutable record of prompts, sources, tool calls, approvals, latency and outcomes. The evidence base for Weeks 5 to 8.                                   |

## 6. Repository and Task Management Setup

The repository and ClickUp board are live; both URLs are recorded on the
cover page. This section records what was set up and how Week 1 tasks
are tracked.

### 6.1 GitHub Repository

The folder structure below is taken directly from Brief §5 and should be
created exactly as named, since it is what the assessment expects to
find.

```text
README.md
docs/
requirements/
architecture/
weekly-reports/
evaluation/
prompts/
knowledge/          # metadata & provenance only — no restricted data
src/
tests/
evidence/
    traces/
    screenshots/
demo/
.env.example        # never commit real secrets
```

- Create the repository (agrivisit-capstone) with default branch main
  and branch protection enabled on main.

- Add README.md carrying the problem statement and the minimum proposal
  statement from §1.8.

- Add .gitignore, LICENSE and .env.example. Confirm no secret or real
  data is tracked.

- Create the folder structure above, each with a placeholder file so
  empty directories are tracked.

- Commit the Week 1 deliverables: charter to requirements/, diagram to
  architecture/, this document to weekly-reports/.

- Add all members and the course convener as collaborators; create a
  Week 1 milestone and one issue per Week 1 activity.

- Tag the Week 1 submission point (for example week-1) so the state is
  fixed and citable in this report.

### 6.2 ClickUp Board

- Space 'AgriVisit Capstone' created with one List per week, Week 1
  through Week 8, so the eight-week arc is visible from the outset.

- Populate the Week 1 list with one task per activity: Project Charter;
  success criteria; user stories and acceptance criteria; AI Boundary
  Matrix; architecture diagram; GitHub setup; ClickUp setup; AI
  Engineering Log; Week 1 progress report.

- Assign every task to a named member with a due date inside 31 August –
  4 September 2026 and a status workflow of To Do / In Progress / Review
  / Done.

- Link each completed task to the corresponding GitHub commit, issue or
  tag, as Brief §5 requires task-to-evidence traceability.

## 7. AI Use Declaration and AI Engineering Log

Required by Brief §6. AI-generated work is permitted and expected;
unexplained work is not. The team remains accountable for correctness,
security, requirements, architecture, testing, documentation and final
system behaviour.

### 7.1 Declared Tools

| **Tool / model**       | **Used for**                                  | **Ownership and verification**                                                                                   |
|------------------------|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| **Claude (Anthropic)** | Drafting and structuring Week 1 documentation | All content reviewed, edited and accepted by the named section owner; factual claims verified against the brief. |
| **Coding assistant**   | Not yet used — no code written in Week 1      | To be declared from Week 2 onward.                                                                               |

### 7.2 AI Engineering Log — Week 1

A concise record of material AI-assisted decisions and generated
artefacts. This log runs for all eight weeks and its summary becomes an
appendix of the final report.

| **Entry** | **Date**    | **AI-assisted artefact or decision**                                                                                                   | **Human review and outcome**                                                                                                                 |
|-----------|-------------|----------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **W1-01** | 31 Aug 2026 | Comparative analysis of the nine candidate use cases in Brief §3 against feasibility, corpus availability and evaluation tractability. | Team reviewed the analysis and independently selected the extension visit-preparation agent. Reasoning recorded in §8.3.                     |
| **W1-02** | 1 Sept 2026 | Initial draft of the Project Charter and the twelve user stories.                                                                      | Each story reviewed by the Quality/Security Lead; acceptance criteria rewritten to be independently testable and mapped to success measures. |
| **W1-03** | 2 Sept 2026 | Draft AI Boundary Matrix rows.                                                                                                         | AI Engineering Lead added the iteration/termination row and moved refusal-trigger definition out of the AI column into deterministic code.   |
| **W1-04** | 3 Sept 2026 | Architecture and context diagram.                                                                                                      | Application/Integration Lead verified every component and data flow against the boundary matrix before acceptance.                           |

### 7.3 Standing Commitments

- No confidential, personal or restricted data is sent to any external
  AI service. All farm data is synthetic.

- Generated code is reviewed, tested and refactored before it reaches
  the main branch.

- Important prompts are versioned under prompts/ from Week 2, with
  evidence retained for meaningful changes.

- No destructive or high-impact action is permitted without an explicit
  human approval control.

- All AI-suggested references and technical claims are verified before
  use.

- Every member can explain the work submitted under their name.

## 8. Week 1 Progress Report

*Structured against Brief §8.*

### 8.1 Identification

|                     |                                             |
|---------------------|---------------------------------------------|
| **Group / project** | **AgriVisit_Capstone**                      |
| **Week ending**     | 4 September 2026                            |
| **Reported by**     | Arthur Ssuuna (Project / Requirements Lead) |

### 8.2 Work Completed Against Weekly Objectives

| **Week 1 objective (Brief §7)**                                              | **Status**                | **Where the evidence sits**                   |
|------------------------------------------------------------------------------|---------------------------|-----------------------------------------------|
| Select one approved use case and define a single primary end-to-end workflow | **Complete**              | §1.1–1.4, §1.9                                |
| Write a 2–3 page Project Charter                                             | **Complete**              | §1 (requirements/charter.md)                  |
| Create 8–12 testable user stories with acceptance criteria                   | **Complete — 12 stories** | §3 (requirements/user-stories.md)             |
| Create an AI Boundary Matrix                                                 | **Complete — 11 rows**    | §4 (requirements/ai-boundary-matrix.md)       |
| Define users and success criteria                                            | **Complete**              | §1.2, §1.5 (eight measures S1–S8)             |
| Produce an initial architecture / context diagram                            | **Complete**              | §5, Figure 1 (architecture/)                  |
| Create the GitHub repository and ClickUp project; assign Week 1 tasks        | **Complete**              | §6; repository and board linked on cover page |

### 8.3 Key Engineering Decisions

- **Use case selected from Brief §3 rather than proposed
  independently.** The extension visit-preparation agent was chosen
  because its corpus is public and obtainable immediately, its farm data
  can be fully synthetic, and its safety boundary is intrinsic to the
  domain rather than contrived. That combination makes the Week 3 corpus
  and the Week 7 evaluation tractable within the timeline.

- **The primary user is the officer, not the farmer.** A farmer-facing
  advisory agent would press constantly against the use case's own
  safety boundary — pesticide dosing and disease diagnosis are exactly
  what a farmer would ask for. Positioning the officer as the user keeps
  the human expert inside the loop by design rather than by restriction.

- **Scope locked to one region and five crops.** Multi-crop gives enough
  retrieval variety for a meaningful 30-scenario evaluation; a single
  region keeps the corpus at 30–40 documents, within the brief's
  recommended envelope.

- **The weather API is a committed component, not a stretch goal.**
  Brief §7 requires one external integration in Week 6. Treating the
  forecast tool as optional would leave that requirement unaddressed, so
  it is now in scope with explicit graceful-degradation behaviour (Story
  4).

- **Restricted topics are blocked by deterministic rule, never by prompt
  instruction.** Asking a model not to answer is not a control. The
  trigger definition sits in code so that it is testable, auditable and
  cannot be defeated by rephrasing.

- **Success measures were defined in Week 1, not deferred to Week 7.**
  The eight measures in §1.5 give the evaluation set a target to be
  built against rather than a result to be rationalised afterwards.

### 8.4 Challenges and Current Response

| **Challenge**                                                                          | **Why it matters**                                                                                       | **Current response**                                                                                                                 |
|----------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Assembling a clean 30–40 document agronomy corpus with recorded provenance             | Corpus quality caps RAG quality; a weak corpus cannot be rescued in Week 7                               | Sourcing begins in Week 2 alongside the model baseline so Week 3 starts with documents in hand, not a blank folder                   |
| Defining the restricted-topic rule precisely enough to avoid over- and under-blocking  | Over-blocking makes the tool useless to officers; under-blocking breaches the use case's safety boundary | Quality/Security Lead to draft the trigger specification with paired positive and negative examples, ready for Week 4 implementation |
| Generating 40–50 synthetic farm profiles that are realistic enough to evaluate against | Unrealistic data produces an evaluation that proves nothing                                              | Profiles to be derived from published extension case material and reviewed for plausibility before use                               |
| Weather API selection and rate limits not yet finalised                                | It is the Week 6 external integration, so the choice cannot slip indefinitely                            | Candidate services to be compared in Week 2; graceful degradation is specified regardless of which is chosen                         |

### 8.5 Individual Contribution Summary

Required by Brief §8. Task-level evidence for each item below is held on
the ClickUp board and, once created, in the repository recorded on the
cover page.

| **Member**          | **Task owned this week**                                                                              | **Artefact produced**         |
|---------------------|-------------------------------------------------------------------------------------------------------|-------------------------------|
| **Arthur Ssuuna**   | Use-case selection and workflow definition; Project Charter; success criteria; document consolidation | §1, §1.5, §1.9; this document |
| **Nakayiza Nairah** | Architecture and context diagram; component responsibility table                                      | §5, Figure 1                  |
| **Jovan Bwire**     | AI Boundary Matrix; AI value justification; AI Engineering Log                                        | §1.4, §4, §7.2                |
| **Nabanoba Yunia**  | Twelve user stories with acceptance criteria; mapping to success measures                             | §3                            |
| **Jassim Kasule**   | GitHub repository and folder structure; ClickUp workspace, weekly lists and task assignment           | §6                            |

### 8.6 Evidence

The repository and ClickUp board URLs are recorded once, on the cover
page of this document. The ClickUp board carries per-task owners,
statuses and due dates for Week 1, and each completed task links to its
corresponding repository commit or issue.

### 8.7 Plan for Week 2 — Foundation-Model Engineering and Prompting

Week 2 in the brief is the model baseline and prompting, not retrieval.
Retrieval is Week 3 and tools are Week 4; the plan below follows that
order deliberately.

| **Week 2 activity (Brief §7)**                                                                                                                   | **Owner**                      | **Deliverable**               |
|--------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------|-------------------------------|
| Select an accessible model and document capability, cost, latency, privacy and access considerations                                             | AI Engineering Lead            | Model Selection Note (1 page) |
| Integrate the model into the application to produce the smallest useful capability: draft checklist items from a farm profile, without retrieval | Application / Integration Lead | Working baseline interaction  |
| Write Prompt Specification v1.0 — role, task, context, constraints, output format and failure behaviour                                          | AI Engineering Lead            | prompts/spec-v1.0.md          |
| Create at least 10 prompt test cases recording expected against actual behaviour                                                                 | Quality / Security Lead        | 10-case evaluation table      |
| Version at least two meaningful prompt iterations with the reasoning for each change                                                             | AI Engineering Lead            | Prompt version history        |
| Begin sourcing the agronomy corpus and recording provenance, ahead of Week 3                                                                     | Project / Requirements Lead    | Draft Corpus Register         |

> **Note on sequencing**
>
> Corpus sourcing is started early because it is the longest-lead item in the project, but retrieval itself is not implemented until Week 3. This keeps the team aligned with the graded weekly structure while removing the risk of Week 3 opening with an empty corpus.

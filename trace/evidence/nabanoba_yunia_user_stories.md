# AgriVisit

## Agricultural Extension Field-Visit Preparation Agent

### User Stories & Acceptance Criteria

**Quality / Security Lead — Week 1 Individual Submission**

| Field               | Details                                                           |
| ------------------- | ----------------------------------------------------------------- |
| **Member**          | Nabanoba Yunia                                                    |
| **Role**            | Quality / Security Lead                                           |
| **Group / Project** | AgriVisit_Capstone                                                |
| **Course**          | BSE4104 Emerging Trends in Software Engineering                   |
| **Week**            | Week 1 (31 Aug – 4 Sept 2026), Brief §7                           |
| **Sections Owned**  | §3 User Stories and Acceptance Criteria                           |
| **Repository**      | https://github.com/arthursuuna/AgriVisit_Capstone                 |
| **ClickUp Board**   | https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1 |

## Scope of This Submission

This document contains the Week 1 work owned by Nabanoba Yunia as Quality / Security Lead. It is one of five role-scoped submissions that together form the group's complete Week 1 deliverable. Section numbering follows the consolidated group document so that cross-references remain valid.

# 3. User Stories and Acceptance Criteria

Twelve testable stories covering the primary end-to-end workflow:

**farm selection → grounded retrieval → checklist drafting → guardrail screening → human approval → scheduling → follow-up logging**

Each acceptance criterion is written so that it can be turned directly into a scenario in the Week 7 evaluation set. The final column records the mapping to the success measures defined in §1.5.

| #      | User Story                                                                                                                                                        | Acceptance Criteria (Given / When / Then)                                                                                                                                                                                                                                         | Success Measure |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| **1**  | As an extension officer, I want to select a farm profile so that the agent prepares a visit relevant to that specific farm.                                       | **Given** a valid farm ID, **when** it is submitted, **then** the system displays the farm's crop types, location, last visit date and outstanding issues within 5 seconds.                                                                                                       | S1, S7          |
| **2**  | As an extension officer, I want the agent to retrieve agronomy guidance for the farm's crops and reported issues so that I need not search manuals manually.      | **Given** a farm profile with at least one crop or reported issue, **when** retrieval runs, **then** at least one relevant guidance excerpt is returned with a citation identifying its source document and section.                                                              | S2              |
| **3**  | As an extension officer, I want a draft field-visit checklist generated from the retrieved guidance so that I know what to inspect and ask.                       | **Given** retrieved guidance for a farm, **when** the agent drafts a checklist, **then** it contains at least five items and each substantive item is traceable to a cited source.                                                                                                | S1, S2          |
| **4**  | As an extension officer, I want the forecast for the planned visit date and location so that I can decide whether to reschedule.                                  | **Given** a farm location and proposed date, **when** the weather tool is called, **then** either a forecast summary is displayed or a clear "forecast unavailable" notice is shown and the workflow continues to completion without it.                                          | S1, S4          |
| **5**  | As an extension officer, I want to review, edit and approve the checklist before it is finalised so that I remain responsible for what is taken into the field.   | **Given** a draft checklist, **when** the officer opens the review screen, **then** they can edit, add or remove items and the checklist cannot be marked "ready for field use" until it is explicitly approved.                                                                  | S8              |
| **6**  | As an extension officer, I want the agent to schedule the visit and create a task, so that the visit is tracked.                                                  | **Given** an approved checklist and a confirmed date, **when** scheduling is submitted, **then** a visit task is created carrying the farm ID, date and a reference to the approved checklist version.                                                                            | S1, S4          |
| **7**  | As an extension officer, I want to log follow-up notes after the visit so that the farm's history stays current.                                                  | **Given** a completed visit task, **when** follow-up notes are submitted, **then** the farm's visit history is updated, timestamped and attributed to the officer.                                                                                                                | S6              |
| **8**  | As an extension officer, I want the agent to refuse pesticide dosing and veterinary or medical diagnosis so that unsafe or out-of-scope advice is never produced. | **Given** a request for a pesticide dose, application rate, or an animal or human health diagnosis, **when** the agent processes it, **then** that part of the request is declined, the officer is referred to a licensed specialist and the refusal is written to the audit log. | S3              |
| **9**  | As a supervisor, I want to see which checklist items were AI-drafted and which were officer-edited so that I can audit the quality of AI assistance.              | **Given** any finalised checklist, **when** a supervisor views it, **then** each item is marked as AI-generated, officer-edited or officer-authored with the editing user and timestamp.                                                                                          | S8              |
| **10** | As an extension officer, I want every checklist item to cite its source so that I can verify guidance before using it in the field.                               | **Given** any checklist item derived from the knowledge base, **when** it is displayed, **then** a visible citation naming the source document and section is attached and the cited passage is retrievable.                                                                      | S2              |
| **11** | As a system administrator, I want the agent restricted to an approved tool allow-list so that it cannot perform unauthorised actions.                             | **Given** a workflow run, **when** the agent attempts a tool call, **then** only registered tools execute; any unlisted call or schema-invalid parameter set is blocked, returned as a handled error and logged.                                                                  | S4              |
| **12** | As an extension officer, I want the agent to stop rather than loop when it cannot make progress so that a failing run ends predictably.                           | **Given** a run that reaches the configured iteration cap or a defined stop condition, **when** the cap is reached, **then** the agent halts, surfaces partial results with an explanation, and hands off to the officer.                                                         | S5              |

## 3.1 Coverage Check

Every success measure defined in §1.5 has at least one story behind it, and every story maps to at least one measure. This is checked deliberately: a measure with no story means the project has promised to evaluate something it never built, and a story with no measure means work that the project does not claim to need.

| Measure | Stories            | What It Covers                            |
| ------- | ------------------ | ----------------------------------------- |
| **S1**  | Stories 1, 3, 4, 6 | Task completion across the normal path    |
| **S2**  | Stories 2, 3, 10   | Groundedness and citation integrity       |
| **S3**  | Story 8            | Refusal correctness on restricted topics  |
| **S4**  | Stories 4, 6, 11   | Tool selection and allow-list enforcement |
| **S5**  | Story 12           | Bounded termination                       |
| **S6**  | Story 7            | Persistent memory of visit history        |
| **S7**  | Story 1            | Preparation latency                       |
| **S8**  | Stories 5, 9       | Human control and auditability            |

AgriVisit
Agricultural Extension Field-Visit Preparation Agent
Architecture & Context Diagram
Application / Integration Lead — Week 1 Individual Submission
Member	Nakayiza Nairah
Role	Application / Integration Lead
Group / project	AgriVisit_Capstone
Course	BSE4104 — Emerging Trends in Software Engineering
Week	Week 1 (31 Aug – 4 Sept 2026), Brief §7
Sections owned	§5 Architecture and Context Diagram
Repository	https://github.com/arthursuuna/AgriVisit_Capstone
ClickUp board	https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1

Scope of this submission
This document contains the Week 1 work owned by Nakayiza Nairah as Application / Integration Lead. It is one of five role-scoped submissions that together form the group's complete Week 1 deliverable. Section numbering follows the consolidated group document so that cross-references remain valid.
 
5. Architecture and Context Diagram
The officer works through a web console. A bounded agent orchestrator runs a plan–act–observe loop under explicit iteration and stop limits. It retrieves grounded guidance from the curated agronomy corpus, passes that context to the foundation model, and calls only tools on the approved allow-list. Every candidate output passes through a guardrail engine that blocks restricted topics and validates structure before reaching a mandatory human approval gate. Nothing bypasses that gate. Every step — prompt, retrieved source, tool call, approval and outcome — is written to an audit log the supervisor can review.
 
Figure 1 — AgriVisit context and architecture. Deterministic control shown in the left lane; the AI layer is bounded by retrieval on one side and the guardrail plus approval gate on the other.
5.1 Component Responsibilities
Component	Responsibility
Visit Prep Console	Web UI for farm selection, checklist review, editing, approval and follow-up entry. Holds no decision logic.
Agent Orchestrator	Runs the bounded plan–act–observe loop. Enforces the iteration cap and stop conditions. Selects among approved tools. Never executes an unlisted action.
Retrieval (RAG)	Indexes the agronomy corpus, ranks passages against the farm context, and attaches citations. Deterministic; the model does not choose the corpus.
Foundation Model	Summarises retrieved passages and drafts checklist items. Sees only retrieved context and the farm record. Asserts nothing uncited.
Guardrail / Rule Engine	Blocks restricted topics before and after generation, validates output schema, flags low-confidence drafts. Rule-based, not model-based.
Human Approval Gate	The single point at which a draft becomes usable. No path to field use exists around it.
Tool allow-list	Four registered tools with declared input and output schemas, authorisation and failure behaviour. Anything unregistered is refused and logged.
Audit / Trace Log	Immutable record of prompts, sources, tool calls, approvals, latency and outcomes. The evidence base for Weeks 5 to 8.
5.2 Data Flows
•	Officer → Console → Orchestrator: the workflow begins with a farm ID and an optional proposed visit date.
•	Orchestrator → Retrieval → Knowledge Base: ranked, cited passages are returned; the model never queries the corpus directly.
•	Retrieval → Foundation Model: grounded context only. The model receives retrieved passages and the farm record, and nothing else.
•	Orchestrator → Tool allow-list → Farm Records DB / Weather API: every side effect is a schema-validated call to a registered tool.
•	Orchestrator → Guardrail → Approval Gate → Officer: the only path by which generated content reaches a human, and the only path to field use.
•	All components → Audit Log → Supervisor: prompts, sources, tool calls, approvals and outcomes are recorded for review.

# AgriVisit — Week 2 Progress Report

| | |
|---|---|
| **Group / project** | AgriVisit_Capstone — AgriVisit |
| **Week** | Week 2 — Foundation-Model Engineering and Prompting (Brief §7) |
| **Reported by** | Arthur Ssuuna (Project / Requirements Lead) |
| **Repository** | <https://github.com/arthursuuna/AgriVisit_Capstone> |
| **ClickUp board** | <https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1> |

---

## 1. Work completed against weekly objectives

| Week 2 activity (Brief §7) | Status | Evidence |
|---|---|---|
| Choose an accessible model and document capability, cost, latency, privacy and access | Complete | Model Selection Note — Gemini 3.5 Flash selected over Claude Sonnet 4.6 on cost, with measured token and latency figures |
| Integrate the model into the application | Complete | `GoogleClient` behind the `LlmClient` interface; Visit Prep Console |
| Create Prompt Specification v1.0: role, task, context, constraints, output format and failure behaviour | Complete | Prompt Specification; templates in `resources/prompts/checklist/` |
| Create at least 10 test cases and record expected vs actual behaviour | Complete | 10-Case Prompt Evaluation Table — 10/10 passed on v1.1 |
| Version at least two meaningful prompt iterations | Complete | v1.0 and v1.1 kept side by side; six documented changes; generation settings revised after the truncation finding |

## 2. Key engineering decisions

- **Switched from Claude Sonnet 4.6 to Gemini 3.5 Flash on cost.** Gemini produced equivalent drafts at lower cost. The switch was a new `GoogleClient` plus configuration; the drafter, guard and parser were not touched.
- **Native Gemini endpoint rather than the OpenAI-compatible one.** Only the native API exposes thinking-level control, the thinking-token count and Gemini's own finish reasons. The API key travels in a header, never a query string, so it stays out of server logs.
- **Restricted-topic triggers live in configuration, not in the prompt.** As the Week 1 AI Boundary Matrix requires, the guard screens requests before any model call and outputs afterwards. P05–P07 were refused without consuming a token.
- **Every item is forced to grounding "ungrounded".** No corpus is connected yet, so any citation would be invented. The parser rejects any other value, making a fabricated citation machine-detectable.
- **Prompts are versioned files and output validation is strict.** Every trace names the prompt version that produced it, and no failure path returns a partial checklist.

## 3. Challenges and current response

| Challenge | Why it matters | Current response |
|---|---|---|
| Gemini's reasoning consumed the 1500-token output budget before the JSON closed (`MAX_TOKENS` at 207 tokens) | It surfaced as "invalid JSON", blaming the model's formatting for a configuration problem | Budget raised to 4000 and thinking level set to low. `GoogleClient` now detects `MAX_TOKENS` and reports truncation explicitly |
| Thinking tokens were invisible and exceeded visible output (6,571 vs 4,282 across the run) | True output cost is about 2.5× what visible tokens suggest | `LlmResponse` records thinking tokens and traces carry them; the Model Selection Note counts them in cost |
| "How many litres to mix in the knapsack" passed the guard | Dose questions phrased as plain quantities are not covered by the keyword list | Recorded as an evaluation finding; the dosing phrase list is to be extended |
| The v1.1 template lacked Context and Failure behaviour sections | The brief names six prompt parts; the template had four | Both sections added; the template and the specification now match |
| v1.0 left officer notes undelimited and did not forbid citations | Leaves room for prompt injection and invented references | v1.1 adds delimiters, an instruction-hierarchy rule and a no-citation rule; P08 and P09 pass |

## 4. Individual contribution summary

| Member | Task owned | Artefact |
|---|---|---|
| **Jovan Bwire** | Model selection including thinking-token cost analysis; Prompt Specification v1.0 and v1.1 | Model Selection Note; Prompt Specification; `resources/prompts/` |
| **Nakayiza Nairah** | Model integration: `LlmClient` interface, `GoogleClient`, provider wiring, Visit Prep Console | `app/Services/Llm/`; `AgriVisitServiceProvider`; `ChecklistController` |
| **Nabanoba Yunia** | Ten test cases, evaluation run, restricted-topic guard and its tests; guard gap finding | 10-Case Prompt Evaluation Table; `RestrictedTopicGuard`; tests |
| **Jassim Kasule** | Week 2 repository structure, environment template, trace capture, `week-2` tag | `prompts/`, `evaluation/`, `evidence/traces/`, `.env.example` |
| **Arthur Ssuuna** | Coordination; scope control against the Week 1 charter; this report | `weekly-reports/` |

## 5. Plan for Week 3 — Context Engineering and Retrieval

| Activity | Owner | Deliverable |
|---|---|---|
| Assemble and register the 30–40 document agronomy corpus with provenance | Arthur Ssuuna | Corpus Register |
| Chunk and embed the corpus; store vectors with the documents | Jovan Bwire | Embedding pipeline |
| Implement retrieval behind a `Retriever` interface with ranking and citations | Nakayiza Nairah | Working RAG retrieval |
| Prompt v2.0: cited grounding replaces the ungrounded constraint | Jovan Bwire | `prompts/checklist/v2.0/` |
| Add retrieval relevance cases to the evaluation set | Nabanoba Yunia | Extended evaluation table |
| Commit corpus metadata, tag `week-3`, capture traces | Jassim Kasule | Repository evidence |

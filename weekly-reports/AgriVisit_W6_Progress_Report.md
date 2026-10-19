Week 6 Progress Report
Memory, state and interoperability
Group / project	AgriVisit_Capstone — AgriVisit
Week	Week 6 — Memory, State and Interoperability
Deliverable owner	Nakayiza Nairah (Project / Requirements Lead)
Repository	https://github.com/arthursuuna/AgriVisit_Capstone
ClickUp board	https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1
What was built
The agent now remembers what the officer decided about a farm, and every use of that memory is visible on the page where the officer decides. The run lifecycle, which until this week existed only as the order the code happened to call things in, is written down as a state model, enforced in three places, and generated from the code so it cannot drift. One capability — the guidance corpus — is specified as an interface a caller outside AgriVisit could use.
Four phases, 312 tests passing, three live runs against Claude. The agent prompt moved to v1.1, its first change since Week 5.
Deliverable	Where it is
State model	Generated: php artisan agrivisit:state-model
Memory Design and Data Handling Note	Generated: php artisan agrivisit:memory-note
Working memory/state demonstration	Three live runs and two screenshots
MCP-style interface specification	Generated: php artisan agrivisit:mcp-spec
Three of the four are generated from the running system rather than written beside it. A tool added next week cannot slip past the interface specification: its generator refuses to run if any registered tool has no stated reason for being exposed or withheld.
Memory, and the rule it had to satisfy
The brief asks for memory that improves a task “without silently controlling critical decisions”. That line shaped every decision this week.
Decision	Why
Decisions are remembered. Preferences are not.	A decision is a fact the system observed. “The officer prefers visits a week out” is an inference, and generalising from one rejection is exactly the silent control the brief warns about.
The system records; the model never writes memory.	If the model could write memory it would be choosing what the system remembers about the officer, and every later run would be shaped by a judgement nobody reviewed. A test scans the application and fails if anything but the recorder writes.
Memory reaches the agent through a tool.	Injecting a farm’s history into the prompt would be less code and would work. It would also be invisible — not traced, not counted, not validated. Invisible is the one thing it must not be.
A used memory is a cited memory.	memory_ref, checked the same way chunk_ref is. php artisan agrivisit:citations fails if a reference the agent wrote was not returned to it in that run.
The finding that matters
Run 3 recalled two past decisions, let both visibly shape its proposal, and cited neither.
The officer’s 26 Oct decline was due to the farmer’s availability, requesting a date after 10 November — 12 Nov satisfies that and was never actually rejected, just left undecided. I’ll propose 12 Nov again.
run 20261019-030544-ne94ha, the proposing step
The citation check passed, because it checks that what **is** cited is real. It cannot check what was used and left uncited.
A citation proves what was used. It can never prove what was not.
This is the same limit the corpus citation rule has had since Week 3, and no amount of prompting closes it. What closed it here was not asking the model more firmly — it was showing the officer every memory the run recalled, on the panel where they decide, whether the model mentioned it or not.
That is why the approval panel now lists recalled decisions. The original plan was to show the proposing step’s reasoning, which would not have helped: in the run it was written for, that step carried no citation either. A panel that shows what was recalled is a guarantee. A citation is a request.
What the state model found
Writing the lifecycle down found a hole that two earlier checks had missed. AgentRunStore::save() had no transition check, so any caller could write over a closed run — reopening a rejected one as awaiting approval.
“A rejection is final” was true only because every current caller happened to behave.
Three claims this project has made since Week 5 rested on that: a rejection is final, a refusal ends the run, nothing writes without approval. All three held by convention rather than by construction.
The second finding is the same shape. Two concurrent closes of the same run used to collide, and the loser only failed because the trace writer opens files in exclusive mode. Had the trace names differed, it would have silently overwritten the winner.
Twice in two phases, a control designed for one purpose was quietly load-bearing for another: the Week 4 single-use approval token serialising competing closes, and the Week 5 write-once trace rule preventing a lost update. Neither was documented as such, and both are the kind of property a later refactor removes without noticing.
The interface, and a question engineering cannot answer
One capability is specified: the guidance corpus. It is the only thing here with value to a caller outside — curated Ugandan extension manuals, restricted content removed at ingestion, retrieval at a calibrated threshold, a citable reference on every passage.
What is withheld is the argument. The write tools stay in, because an interface cannot carry a human approval boundary — approval is a person deciding, not a permission a caller holds. Memory stays in, because it is one organisation’s working record including its officers’ own words. The agent loop stays in, because exposing it would hand out exactly the agency this project spent five weeks bounding, to a caller whose own prompt and approvals we cannot see.
The licence position is not settled
Two of the three source manuals state no licence; the third is © 2019 Uganda Coffee Development Authority. Using them inside AgriVisit is one thing. Serving passages to a third party’s software is redistribution, and that has not been established with the publishers.
The specification records this as the one prerequisite that is not engineering, and states that the interface should not be offered outside the deployment until it is settled.
An overclaim, checked
An earlier draft of the interface specification said the corpus stripping means a passage cannot contain a dosing instruction. That was too strong: the stripper matches patterns, and deliberately keeps weight-per-area figures because in agronomy text they are almost always seed rates and yields.
So the claim was tested rather than softened. The dose screen was run over all 301 indexed passages. It flagged six. Every one was read: seed rates (30 kg/acre beans, 10 kg/acre maize), variety yields, and manure and fertiliser rates (1000 kg/Ha, 100 kg/Ha). None is a pesticide dose. Those six references are pinned in the specification, and it states that any other passage the screen flags in future has not been reviewed.
The specification now says “removed by pattern, and checked against the current index”, which is what the code supports.
What is not true yet
A reader might assume	What is actually the case
The officer decided	Every decision, deletion and reason is recorded against officer-001, an identity that is configured, not authenticated. Anyone who can reach the app acts as that officer.
The agent cites the memories it uses	It cites some. Run 3 used two and cited neither. The panel listing what was recalled is the actual safeguard.
Expired memories are gone	They are hidden from the agent and still in the table. There is no scheduled purge; deletion is manual.
Deleting a memory removes it	From recall, yes. The officer’s reason remains in tool_approvals, in the traces of runs that recalled it, and in those runs’ conversations. There is no erasure path, deliberately.
The system remembers what the officer decided	Only decisions made on agent runs. A visit declined on the Week 4 tool console is unknown to every later run.
The MCP interface exists	It is a specification. No server runs, and none should be exposed before authentication, rate limiting and the licence position.
Who did what
Roles rotate one step each week, so every member works across the whole system over the eight weeks rather than owning one corner of it.
Member	Role this week	Held in Week 5	Deliverable
Nakayiza Nairah	Project / Requirements Lead	DevOps / Documentation Lead	Week 6 progress report
Jovan Bwire	Application / Integration Lead	Project / Requirements Lead	MCP-style interface specification
Nabanoba Yunia	AI Engineering Lead	Application / Integration Lead	Memory Design and Data Handling Note
Jassim Kasule	Quality / Security Lead	AI Engineering Lead	State model
Arthur Ssuuna	DevOps / Documentation Lead	Quality / Security Lead	Working memory/state demonstration
Next week
●	Week 7 evaluation: 30+ scenarios across normal, edge, incorrect-information, adversarial, tool-failure and unauthorized-action cases.
●	Prompt injection, tested live. Memory introduced an input that crosses runs: a sentence typed into one decision is replayed into the model’s context on every later run for that farm. Four cases are already named.
●	The guard is still a word list. A refusal is now final, but whether a request is caught depends on matching terms. This is the weakest remaining layer.
●	Produce the four stop reasons that exist only in tests, and close run 3.

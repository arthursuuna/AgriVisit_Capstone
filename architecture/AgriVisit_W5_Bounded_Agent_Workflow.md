Working Bounded Agent Workflow
The system as the officer uses it
Group / project	AgriVisit_Capstone — AgriVisit
Week	Week 5 — Agent Architecture and Bounded Autonomy
Deliverable owner	Nakayiza Nairah (DevOps / Documentation Lead)
Repository	https://github.com/arthursuuna/AgriVisit_Capstone
ClickUp board	https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1
The architecture
 
The agent loop, the gate it calls through, the human approval boundary, and where the evidence is written.
The task the agent does
Preparing a field visit is a good fit for multi-step work because no single step contains the answer. Which guidance matters depends on what the farm record says is unresolved; the purpose of the visit depends on what the guidance turns up. A human does this in a few minutes by holding one thing in mind while looking up the next.
The agent does the same, one action at a time, and stops before anything is written.
Walkthrough
1. The officer states a goal
No form, no farm picker, no checkbox for what to look up. One sentence: “Prepare and schedule a follow-up visit for farm UG-KYA-012.”
2. The agent works, and shows its reasoning
 
Each step: the agent’s plan, the tool it chose, the arguments it sent, the outcome and the duration.
Four iterations of a possible six. Step 2’s topic was chosen because step 1 reported the maize leaf damage unresolved; step 4’s purpose was written from what steps 2 and 3 returned. The step header shows what came back — “OK — 3 passages” rather than a bare success — so a reader can tell a search that found something from one that found nothing.
Step 4 is marked HELD. The agent proposed a write and the loop stopped.
3. The officer is shown what will change
 
The approval panel — the same one the Week 4 tool console uses, reused rather than rebuilt.
The officer is not shown a function call. They are shown the farm and its crops, the date in words with how far away it is, the purpose, what is already booked, and whether the action can be undone. Above it all: Nothing has been written yet.
The decision is a human act, and the model is not in it.
The loop calls every tool without approval and can never grant it. A write tool returns APPROVAL_REQUIRED, the run suspends, and approval happens in the Week 4 flow — single-use token, ten-minute expiry, and a record of who decided and what they were shown.
4. On approval, the run continues where it paused
 
The run resumes in place. It does not start again, and it does not get a fresh iteration budget.
The approved call goes through the same gate, so an approved action with bad arguments still fails validation — approval authorises, it does not exempt. The run then carries on with the iterations it had left.
5. The agent reports, and every claim carries a reference
 
The closing answer. Each agronomic statement names the corpus passage it came from.
It names the approval reference and the visit it created, says which issues are still open, and attaches a chunk reference to every agronomic claim. Where the corpus had nothing — telling the causes of leaf damage apart by their symptoms — it says so instead of supplying an answer.
6. Every run is reviewable afterwards
 
The run list: goal, status, stop reason, iterations used against the cap, tokens.
This is the supervisor view. A run that ended because the officer declined, because a topic was refused, or because it hit its iteration cap is as visible as one that succeeded — and each links to its steps and its trace files.
What the officer can see that a chat interface would not show
Shown	Why it is there
The plan before each action	A list of calls with no reasoning is not evidence of goal-directed behaviour. The plan is what makes the run readable as a sequence of decisions.
The exact arguments sent	The officer approves a specific action, not a description of one.
Iterations used against the cap	The bound is visible while the run is live, not only in the code.
Active time, not elapsed time	120 seconds of work. The clock stops while the officer thinks, so a run does not die because someone went to lunch.
The stop reason, named	Every run ends as exactly one of nine. “It stopped” is never the answer.
A link to the trace file	What the page shows and what was recorded are the same thing, and can be checked against each other.
What the system refuses to do
●	Write anything without an officer’s decision on that specific action.
●	Continue after a refused topic. A guard refusal ends the run wherever it fires — on the officer’s goal or on a query the agent composed for itself.
●	Re-propose after a rejection. The officer’s no is final within the run; a different outcome needs a new goal.
●	State an agronomic fact it was not given, or a pesticide dose under any circumstances.
●	Take more than one action in an iteration, or repeat a call it has already made.
The refusal is the design, not an error state.
A refused goal produces a page with no Approve button, no Reject button and no retry control. There is nothing on it that invites a second attempt — and the run is over, so a reworded attempt starts from nothing rather than from where the agent left off.


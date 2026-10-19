**Agent Run State Model**

The states a run can be in, and the moves between them

|                       |                                                                   |
|-----------------------|-------------------------------------------------------------------|
| **Group / project**   | AgriVisit_Capstone — AgriVisit                                    |
| **Week**              | Week 6 — Memory, State and Interoperability                       |
| **Deliverable owner** | Jassim Kasule (Quality / Security Lead)                           |
| **Repository**        | https://github.com/arthursuuna/AgriVisit_Capstone                 |
| **ClickUp board**     | https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1 |

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>This model is generated, not written.</strong></p>
<p>php artisan agrivisit:state-model renders it from RunState and the agent_runs table itself. There is no second copy, so it cannot describe a lifecycle the system no longer has. A test fails if a column exists without a documented meaning.</p></td>
</tr>
</tbody>
</table>

**The model**

![State model diagram](state_model.png)

*Three states, six legal transitions, and the three layers that enforce them.*

**The states**

| **State**         | **What is true while it holds**                                                                                                                                                                | **Persisted**      |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|
| RUNNING           | A request is executing iterations. The agent_runs row is written when a run stops or suspends, so no row is ever found in this state between requests.                                         | in memory          |
| AWAITING_APPROVAL | A write tool returned APPROVAL_REQUIRED. A pending call exists, an approval token is live, nothing has been written, and the conversation ends with the assistant turn carrying that tool_use. | yes                |
| CLOSED            | Terminal. A stop reason other than AWAITING_APPROVAL is set, a trace has been written, and no further model call will be made.                                                                 | as the stop reason |

There is no separate state column. status holds a stop reason, and AWAITING_APPROVAL doubles as the single live state; stop_reason is populated only once the run has stopped. CLOSED is therefore a category rather than a stored value — said plainly here rather than tidied into a column that does not exist.

**A run is never persisted while it is running**

Runs are synchronous: the whole loop happens inside one HTTP request or one Artisan command, and the first save is at the end. RUNNING exists for ten to twenty-five seconds in memory and never reaches the database.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>A known gap, recorded rather than hidden.</strong></p>
<p>A run that dies mid-flight leaves no row of its own. What it does leave is the row from before — for a crashed resume, a row still reading AWAITING_APPROVAL whose token has already been consumed. The lazy expiry will eventually close that as “the approval expired”, which is accurate about the record and wrong about what happened. Nothing today distinguishes a crash from a lapse.</p></td>
</tr>
</tbody>
</table>

**The transitions**

| **From**          | **To**            | **Trigger and condition**                                                                                           |
|-------------------|-------------------|---------------------------------------------------------------------------------------------------------------------|
| —                 | RUNNING           | A goal is submitted and passes the restricted-topic guard. The goal becomes the first user turn.                    |
| —                 | CLOSED            | The guard refuses the goal. GUARD_REFUSED only. No model call, zero iterations.                                     |
| RUNNING           | AWAITING_APPROVAL | A write tool returns APPROVAL_REQUIRED. The pending call is held in the Week 4 approval flow.                       |
| RUNNING           | CLOSED            | Any other stop. Never REJECTED — there is no held call to reject.                                                   |
| AWAITING_APPROVAL | RUNNING           | The officer approves within the window. A new segment opens; iterations, failures and tokens carry over.            |
| AWAITING_APPROVAL | CLOSED            | The officer rejects, or the approval expires or is no longer on file. REJECTED only. The model is not called again. |

Two of these are conditional on the stop reason, not just on the pair of states, and the check enforces both.

**Illegal moves**

Every pair not above. Each throws IllegalRunTransition; nothing about the run changes — no model call, no write, no trace file.

| **Move**              | **Why it is forbidden**                                                                                                                                                       |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| — → AWAITING_APPROVAL | A run can only propose a write by running. Nothing is held for a run that never called a tool.                                                                                |
| RUNNING → RUNNING     | A resumed run continues; it does not begin again.                                                                                                                             |
| AWAITING → AWAITING   | One decision per held call. In the table a run may legitimately go from waiting to waiting across segments, so the stored rule is “exactly one segment further per decision”. |
| CLOSED → RUNNING      | Closed is terminal. This is what makes “a rejection is final” and “a refusal ends the run” true rather than intended.                                                         |
| CLOSED → AWAITING     | A closed run cannot hold a new call for approval.                                                                                                                             |
| CLOSED → CLOSED       | Closing again would overwrite how it ended — a second rejection, or a lazy expiry racing a decision.                                                                          |

**What writing this down found**

**The store would reopen a closed run**

AgentRunStore::save() had no transition check. A probe against the unchanged code reopened a REJECTED run as AWAITING_APPROVAL, and nothing noticed. No application code called it that way — but nothing stopped one.

|                                                                                           |
|-------------------------------------------------------------------------------------------|
| **“A rejection is final” was true only because every current caller happened to behave.** |

A second probe saved a new run under an existing refused run’s id, replacing its record. Both now throw.

**The race was being won by accident**

Two concurrent closes of the same run — a lazy expiry on a page load meeting an officer’s decision — used to collide. The loser threw “Trace already exists”, and only because AgentTraceWriter opens files in exclusive mode. Nothing in the lifecycle prevented it. Had the trace names differed, the loser would have overwritten the winner’s summary and steps.

**That is the second time in two phases that a control designed for one purpose turned out to be load-bearing for another: the Week 4 single-use approval token serialises competing closes, and the Week 5 write-once trace rule was preventing a lost update. Neither was documented as doing that job, and both are the kind of property a later refactor removes without knowing.**

**Where it is enforced**

| **Layer**             | **What it checks**                                                                                                                                                                                     |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AgentRun              | begin(), stop() and beginSegment() check every move in memory, before anything on the run changes.                                                                                                     |
| AgentRunStore::save() | Every move of the stored row: a closed row is never written again, and a waiting row takes exactly one decision.                                                                                       |
| AgentRunner           | decide() and closeIfLapsed() take a per-run lock and re-read the row before moving it, so of two requests one wins and the other throws before it consumes a token, calls the model or writes a trace. |

The rule lives in one place and throws. A caller may translate that for a user — the web decision path shows the existing “already decided” message — but the decision that a move is illegal is made once, not twice.

**Where state lives, and what is not state**

The agent_runs table is the state. One row per run, holding the conversation, the steps, the call signatures, the pending call, the iterations and failures used, the tokens, and the run’s own limits — fixed at the start so a resume uses them and not today’s configuration.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><strong>The trace files are the record, not the state.</strong></p>
<p>One JSON file per run segment, written once when a segment stops and never rewritten. They hold much of the same data, but nothing reads a trace back to decide what a run does next: a resume restores from the table. A deleted trace changes no run. An edited one changes nothing but the evidence.</p></td>
</tr>
</tbody>
</table>

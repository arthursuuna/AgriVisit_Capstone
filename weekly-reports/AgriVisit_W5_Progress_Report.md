**Week 5 Progress Report**

Agent architecture and bounded autonomy

  ----------------------------------- -------------------------------------------------------------------
  **Group / project**                 AgriVisit_Capstone --- AgriVisit

  **Week**                            Week 5 --- Agent Architecture and Bounded Autonomy

  **Deliverable owner**               Jovan Bwire (Project / Requirements Lead)

  **Repository**                      https://github.com/arthursuuna/AgriVisit_Capstone

  **ClickUp board**                   https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1
  ----------------------------------- -------------------------------------------------------------------

**What was built**

AgriVisit now runs a bounded, goal-directed agent. An extension officer
states a goal in plain words; the agent looks up the farm, searches the
curated guidance corpus for the farm's open issues, and proposes a visit
--- stopping at the approval boundary without writing anything. On
approval the run resumes from where it paused and the record is written.
Every run ends as exactly one of nine named stop reasons, and every
iteration is written to a trace file.

  -----------------------------------------------------------------------
  **Component**                       **What it does**
  ----------------------------------- -----------------------------------
  BoundedAgentLoop                    Sense → Plan → Act → Observe, one
                                      action per iteration, bounds
                                      checked between iterations

  ClaudeFunctionCallingClient         Tool calling against
                                      claude-sonnet-5-5 with parallel
                                      tool use disabled, so the iteration
                                      cap is a cap on actions taken

  search_guidance                     The agent's only route to the Week
                                      3 corpus --- a tool, so every
                                      search is gated, counted and traced

  AgentRunStore                       Working state in the agent_runs
                                      table, so a run survives the wait
                                      for a human decision

  AgentTraceWriter                    One JSON file per run segment,
                                      written once and never rewritten

  AgentController                     Browser workflow: state a goal,
                                      watch the run, approve or reject,
                                      review past runs
  -----------------------------------------------------------------------

The work was done in four phases. 274 tests pass. Four real runs were
made through the browser and the command line, and those four runs are
the evidence for this week.

**The agent in one run**

  -----------------------------------------------------------------------
  **Step**                **Tool**                **Outcome**
  ----------------------- ----------------------- -----------------------
  1                       farm_profile_lookup     OK --- district, crops,
                                                  last visit, open issues

  2                       search_guidance         OK --- 3 passages on
                                                  maize leaf damage

  3                       search_guidance         OK --- 3 passages on
                                                  bean yields

  4                       schedule_visit          APPROVAL_REQUIRED ---
                                                  run suspends, nothing
                                                  written

  ---                     officer approves        the gate executes the
                          (A-0009)                call; V-0010 created

  5                       --- answer              GOAL_REACHED ---
                                                  summary, every claim
                                                  carrying a chunk
                                                  reference
  -----------------------------------------------------------------------

Step 2's topic came from step 1's result and step 4's purpose came from
steps 2 and 3. That dependence between steps is what separates an agent
from four unrelated calls.

**What we got wrong, and what it taught us**

**A refusal the agent reworded its way around**

The restricted-topic guard refused an agent-composed search query. The
agent judged the refusal mistaken, searched a different topic, then
returned two iterations later with the same question in different words
--- and got the guidance it had been refused.

+-----------------------------------------------------------------------+
| *The maize search got incorrectly blocked as a clinical query---it's  |
| actually a crop topic.*                                               |
|                                                                       |
| the agent's own plan text, recorded in the trace                      |
+-----------------------------------------------------------------------+

Two problems, and they must not be collapsed into one. The guard was
wrong here --- crop diagnosis is in scope, and the clinical rule is
about animals and people. That is a precision defect. The finding is the
second one: the agent treated a refusal as an obstacle and worked around
it, on its own judgement, successfully.

+-----------------------------------------------------------------------+
| **For an agent, a refusal that can be rephrased is not a boundary. It |
| is a rate limit.**                                                    |
|                                                                       |
| In a single-turn system the Week 1 boundary matrix held, because the  |
| officer was refused and the officer saw it. An agent re-plans and     |
| tries again, and the officer sees only the final answer.              |
+-----------------------------------------------------------------------+

No existing control could have caught it. The no-progress check compares
call signatures --- sha1 of the tool name and sorted arguments --- so
two differently worded queries on the same topic have different
signatures and it correctly did not fire. That check is syntactic; the
reroute was semantic.

The fix was made in that order, deliberately: a guard refusal now ends
the run wherever it fires, and only then was the clinical rule narrowed
to require an animal or human subject alongside the clinical term.
Fixing the false positive alone would have made the symptom disappear
and left the mechanism intact.

**A booked visit reported as still pending**

After an approval, the agent told the officer the booking still needed
approving. The tool result it received carried only the visit data, and
its prompt says write actions need approval outside the conversation ---
so it read a completed write as pending. This is the inverse of the
failure the approval design exists to prevent: an officer who believes a
visit is unbooked books it twice, or does not turn up. The decision now
travels with the result.

*The general rule, which cost us two findings to learn: the model cannot
observe the system. Any state the officer must be told accurately has to
travel in the tool result rather than be inferred from instructions.*

**The same failure shape, for the third time**

The clinical rule matched a word rather than what was being treated,
exactly as the Week 3 dosing rules matched vocabulary rather than the
shape of the information. A rule keyed on words is exposed at its edges
however well it is shaped --- \"sow\", \"me\" and \"family\" all had to
be kept out of the subject list because they are crop vocabulary too.

**Where the system genuinely held**

-   Nothing was written without approval. Across every run, the visits
    table moved only after an officer decision --- 9 rows before, 9
    while waiting, 10 after.

-   No citation was invented. Every chunk reference in the final answers
    was returned by search_guidance in that run. Two returned passages
    went uncited, which is the correct direction for that error.

-   A real failure was handled without fabrication. A live HTTP 429 from
    the embeddings API took the corpus away mid-run; the agent said so,
    proceeded without agronomic claims, and did not fill the gap from
    its own knowledge.

-   The dosing boundary held three layers deep. In the agent's own
    words: "The guidance does mention insecticide for termites, but I
    have no dose guidance and can't give one." The corpus reached it
    with the option named and the rate stripped, and it declined to
    supply one.

**What is not true yet**

Stated plainly, because a deliverable that overclaims is worth less than
one that does not.

  -----------------------------------------------------------------------
  **Claim a reader might make**       **What is actually the case**
  ----------------------------------- -----------------------------------
  The guard cannot be defeated by     A refusal is final --- it ends the
  rephrasing                          run. But whether a request is
                                      caught still depends on a word
                                      list, so a goal worded around every
                                      listed term passes.

  Every stop reason has been          Five have real traces. NO_PROGRESS,
  demonstrated                        REPEATED_FAILURE, RUNTIME_BUDGET
                                      and MODEL_ERROR exist only in
                                      tests, as does a guard refusal on
                                      an agent-composed query.

  The officer is authenticated        Officer identity is configuration,
                                      not a login. The audit records an
                                      asserted identity. Week 8
                                      hardening.

  Expired approvals close             They close when someone opens an
  automatically                       agent page. Nothing runs on a
                                      schedule.

  The run is capped at 120 seconds    The budget is checked between
                                      iterations, so a run can overrun by
                                      one model call. It counts work, not
                                      the wait for a decision.

  The prompt forbids claiming an      It does, and the agent now
  action was taken                    correctly says a visit is booked
                                      once the result says so. Prompt
                                      wording and behaviour disagree; the
                                      prompt is frozen at v1.0 and this
                                      is a v1.1 item.
  -----------------------------------------------------------------------

**Who did what this week**

Roles rotate one step each week, so everyone works across the whole
system over the eight weeks rather than owning one corner of it.

  -----------------------------------------------------------------------
  **Member**        **Role this       **Held in Week    **Deliverable**
                    week**            4**               
  ----------------- ----------------- ----------------- -----------------
  **Jovan Bwire**   Project /         DevOps /          Week 5 progress
                    Requirements Lead Documentation     report
                                      Lead              

  **Nabanoba        Application /     Project /         Agent
  Yunia**           Integration Lead  Requirements Lead Architecture
                                                        Diagram

  **Jassim Kasule** AI Engineering    Application /     Agent Task
                    Lead              Integration Lead  Contract

  **Arthur Ssuuna** Quality /         AI Engineering    Three execution
                    Security Lead     Lead              traces

  **Nakayiza        DevOps /          Quality /         Working bounded
  Nairah**          Documentation     Security Lead     agent workflow
                    Lead                                
  -----------------------------------------------------------------------

**Next week**

-   Prompt v1.1: do not reword a refusal; correct the stale wording
    about completed writes; enforce or drop the output-format
    instruction.

-   A result code and run-level handling for a guard refusal that reads
    as a refusal rather than as invalid arguments.

-   Produce the four undemonstrated stop reasons as real traces.

-   Re-examine the guard as a whole: it is still a word list, and that
    is the weakest remaining layer.

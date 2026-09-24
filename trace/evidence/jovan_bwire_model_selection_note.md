**AgriVisit**

**Model Selection Note**

  ---------------- ------------------------------------------------------
  **Group /        AgriVisit_Capstone --- AgriVisit
  project**        

  **Week**         Week 2 --- Foundation-Model Engineering and Prompting
                   (Brief §7)

  **Owner**        Jovan Bwire (AI Engineering Lead)
  ---------------- ------------------------------------------------------

Task: from a synthetic farm profile and officer notes, draft 5--10
checklist items as strict JSON under six hard constraints ---
constrained structured generation over short inputs.

  -------------------------------------------------------------------------------
  **Criterion**    **Gemini 3.5 Flash**  **Claude Sonnet     **Llama 3.1 8B
                                         4.6**               (local)**
  ---------------- --------------------- ------------------- --------------------
  **Capability**   Strong --- 10/10 on   Strong ---          Weak on domain
                   the evaluation set    comparable drafts   nuance
                                         in testing          

  **Instruction    Strong --- passed     Strong              Unreliable under
  following**      injection (P08) and                       multiple constraints
                   citation (P09)                            

  **Structured     Every drafted run     Reliable            Risk of format drift
  output**         passed strict                             
                   validation                                

  **Reasoning      Always reasons;       Extended thinking   None
  overhead**       thinking billed as    optional            
                   output                                    

  **Latency        10.7--14.2 s observed Not measured here   Hardware-dependent
  (target \< 30                                              
  s)**                                                       

  **Cost**         Lowest of the API     Higher              Hardware cost
                   options                                   

  **Privacy /      Synthetic data only;  Synthetic data      Strongest privacy;
  deployment**     REST API              only; REST API      needs GPU
  -------------------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **Selected**                                                          |
|                                                                       |
| Gemini 3.5 Flash · temperature 0.2 · max output tokens 4000 ·         |
| thinking level low                                                    |
+-----------------------------------------------------------------------+

-   **Why:** Claude Sonnet 4.6 was the initial choice. Gemini produced
    equivalent drafts at lower cost and passed 10/10 on prompt v1.1.
    Instruction following remains the binding criterion, and Gemini held
    all six constraints including under injection. Temperature is low
    because variability is a defect in constrained generation. Privacy
    does not differentiate: all data is synthetic.

-   **Thinking-token finding:** Gemini 3 always reasons, and the output
    budget includes thinking. At 1500 tokens the reasoning used the
    budget before the JSON closed (finishReason MAX_TOKENS at 207
    tokens), surfacing as invalid JSON; fixed by raising the budget to
    4000 with thinking level low. Across the run: 8,098 input, 4,282
    output, 6,571 thinking tokens. Thinking was about 61% of billed
    output, so true output cost is roughly 2.5× visible output. Traces
    record thinking_tokens and cost comparisons include them.

-   **Reversibility:** A native GoogleClient implements the existing
    LlmClient interface, selected by LLM_PROVIDER=google. Switching
    provider changed configuration only; the drafter, guard and parser
    were untouched. The native endpoint was chosen because it exposes
    thinking level, thinking-token counts and finish reasons.

-   **Revisit if:** thinking tokens push cost above Claude; Week 3
    retrieval context truncates the 4000 budget; median latency exceeds
    30 s; or the pass rate falls below 8/10.

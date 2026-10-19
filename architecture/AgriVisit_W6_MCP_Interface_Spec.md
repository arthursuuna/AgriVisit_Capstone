**MCP-style Interface Specification**

The guidance corpus, specified as an interface a caller outside
AgriVisit could use

  ----------------------------------- -------------------------------------------------------------------
  **Group / project**                 AgriVisit_Capstone --- AgriVisit

  **Week**                            Week 6 --- Memory, State and Interoperability

  **Deliverable owner**               Jovan Bwire (Application / Integration Lead)

  **Repository**                      https://github.com/arthursuuna/AgriVisit_Capstone

  **ClickUp board**                   https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1
  ----------------------------------- -------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **Status: a specification, not a running server.**                    |
|                                                                       |
| No MCP server is deployed. Everything below describes search_guidance |
| as it runs inside AgriVisit, through the same gate an MCP server      |
| would call. Permissions lists what would have to exist before it      |
| could be offered to anyone outside.                                   |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **Generated in part.**                                                |
|                                                                       |
| php artisan agrivisit:mcp-spec reads the tool definition and both     |
| schemas from SearchGuidanceTool, the threshold from config, the       |
| corpus table and build dates from the documents and chunks tables,    |
| the guard's categories from config, the error table from ToolResult,  |
| the not-exposed table from the tool registry, and the dose-screen     |
| result by running the screen over every indexed passage. The          |
| generator refuses to run if any registered tool has no stated reason  |
| for being exposed or withheld, so a tool added next week cannot slip  |
| past this document.                                                   |
+-----------------------------------------------------------------------+

**Capability**

Search a curated corpus of Ugandan agricultural extension manuals and
get back passages relevant to a question, each with a citable reference
--- or nothing, if no passage is relevant enough.

It is for software that helps extension workers and must ground what it
says in published guidance rather than a model's general knowledge:
every passage comes from a named government or sector publication,
restricted content was removed before indexing, and a chunk_ref
identifies exactly where each passage came from.

**It answers "what does the published guidance say about X?". It does
not answer questions, give advice, or decide anything. The caller does
that, and owes the citation.**

**The tool definition, as tools/list would publish it**

+-----------------------------------------------------------------------+
| {                                                                     |
|                                                                       |
| \"name\": \"search_guidance\",                                        |
|                                                                       |
| \"title\": \"Search Ugandan agronomy guidance\",                      |
|                                                                       |
| \"description\": \"Search the curated agronomy guidance corpus;       |
| returns                                                               |
|                                                                       |
| passages above the relevance threshold, each with a                   |
|                                                                       |
| chunk_ref to cite, or none.\",                                        |
|                                                                       |
| \"inputSchema\": {                                                    |
|                                                                       |
| \"type\": \"object\",                                                 |
|                                                                       |
| \"properties\": {                                                     |
|                                                                       |
| \"query\": { \"type\": \"string\", \"minLength\": 3, \"maxLength\":   |
| 300 },                                                                |
|                                                                       |
| \"max_results\": { \"type\": \"integer\", \"minimum\": 1,             |
| \"maximum\": 5, \"default\": 3 }                                      |
|                                                                       |
| },                                                                    |
|                                                                       |
| \"required\": \[\"query\"\],                                          |
|                                                                       |
| \"additionalProperties\": false                                       |
|                                                                       |
| },                                                                    |
|                                                                       |
| \"outputSchema\": {                                                   |
|                                                                       |
| \"type\": \"object\",                                                 |
|                                                                       |
| \"properties\": {                                                     |
|                                                                       |
| \"query\": { \"type\": \"string\" },                                  |
|                                                                       |
| \"count\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 5 }, |
|                                                                       |
| \"results\": { \"type\": \"array\", \"items\": { \"type\":            |
| \"object\",                                                           |
|                                                                       |
| \"properties\": {                                                     |
|                                                                       |
| \"chunk_ref\": { \"type\": \"string\", \"minLength\": 1 },            |
|                                                                       |
| \"source_title\": { \"type\": \"string\" },                           |
|                                                                       |
| \"section\": { \"type\": \"string\" },                                |
|                                                                       |
| \"text\": { \"type\": \"string\" },                                   |
|                                                                       |
| \"score\": { \"type\": \"number\" }                                   |
|                                                                       |
| },                                                                    |
|                                                                       |
| \"required\":                                                         |
| \[\"chunk_ref\",\"source_title\",\"section\",\"text\",\"score\"\] } } |
|                                                                       |
| },                                                                    |
|                                                                       |
| \"required\": \[\"query\",\"count\",\"results\"\]                     |
|                                                                       |
| },                                                                    |
|                                                                       |
| \"annotations\": {                                                    |
|                                                                       |
| \"readOnlyHint\": true, \"destructiveHint\": false,                   |
|                                                                       |
| \"idempotentHint\": true, \"openWorldHint\": false                    |
|                                                                       |
| }                                                                     |
|                                                                       |
| }                                                                     |
+-----------------------------------------------------------------------+

readOnlyHint is derived from the tool itself. idempotentHint holds for a
fixed corpus: the same query returns the same passages until the corpus
is rebuilt. openWorldHint is false --- the tool reads only this server's
own index and never the web.

**Inputs**

  -----------------------------------------------------------------------
  **Field**               **Rules**               **Meaning**
  ----------------------- ----------------------- -----------------------
  query                   required, string,       What to search the
                          3--300 characters       guidance corpus for, in
                                                  plain words, e.g.
                                                  "maize leaf damage
                                                  causes".

  max_results             optional, integer,      How many passages to
                          1--5, default 3         return at most.
  -----------------------------------------------------------------------

Any other property is rejected (additionalProperties: false). Rejected
after the schema: a well-formed query about a restricted topic returns
GUARD_REFUSED. The query is screened before anything is retrieved.

**Outputs**

structuredContent conforms to the output schema above, validated on
every call before it is returned. A result that does not conform is
discarded and reported as UNEXPECTED_RESPONSE --- never returned in
part.

-   results holds at most max_results passages, best first, each with a
    similarity score of at least 0.65.

-   count: 0 is a success, not an error. It means nothing in the corpus
    was relevant enough. The caller should say it found no guidance ---
    not lower its standards, retry with a looser query to get something,
    or fill the gap from a model's own knowledge. There is no fallback
    to the best results below the threshold: a threshold that bends
    under pressure is not a threshold.

-   chunk_ref (e.g. DOC-002-C031) names the document and the passage
    within it. The document part maps to the corpus table, which gives
    title, publisher and licence.

-   source_title and section identify where in the publication the
    passage sits; section is empty where the source had no heading.

**The corpus**

  ---------------------------------------------------------------------------
  **Doc**        **Title**      **Publisher**   **Licence**    **Passages**
  -------------- -------------- --------------- -------------- --------------
  DOC-001        Beans Training MAAIF           Not stated --- 138
                 Manual for                     Government of  
                 Extension                      Uganda         
                 Workers in                     publication    
                 Uganda                                        

  DOC-002        Maize Training MAAIF           Not stated --- 52
                 Manual for                     Government of  
                 Extension                      Uganda         
                 Workers in                     publication    
                 Uganda                                        

  DOC-003        Clonal Robusta UCDA            © 2019 Uganda  111
                 Coffee Nursery                 Coffee         
                 Manual for                     Development    
                 Extension                      Authority      
                 Workers and                                   
                 Nursery                                       
                 Operators in                                  
                 Uganda                                        
  ---------------------------------------------------------------------------

3 documents, 301 passages, all 301 embedded with gemini-embedding-001.
The corpus is static: nothing updates it except a manual rebuild
(agrivisit:ingest, agrivisit:index). A caller should treat it as current
as of the build dates the generated specification records, and no later.

**Permissions**

  -----------------------------------------------------------------------
  **Question**                        **The honest answer**
  ----------------------------------- -----------------------------------
  Who may call it today?              Nobody outside AgriVisit, because
                                      there is no server. Inside
                                      AgriVisit it is called by the
                                      agent, through the gate.

  How is a caller identified?         Not at all. AgriVisit has no
                                      authentication; the one officer
                                      identity is configured
                                      (OFFICER_ID), not logged in. A
                                      server built on this code as it
                                      stands would answer anyone who
                                      could reach it, and could not say
                                      who asked.

  What may a caller not do?           Enforced by the gate whoever the
                                      caller is: reach any other tool,
                                      write anything, bypass the guard,
                                      the threshold or the output schema,
                                      or read anything but the indexed
                                      passages --- not the source files,
                                      not the embeddings, not other
                                      callers' queries.
  -----------------------------------------------------------------------

**What would be needed before exposing it**

  -----------------------------------------------------------------------
  **Prerequisite**                    **Why**
  ----------------------------------- -----------------------------------
  Caller authentication               OAuth 2.1 bearer tokens as the MCP
                                      authorization spec describes, or at
                                      minimum per-caller API keys. Every
                                      call should be attributable.

  Rate limiting per caller            Every query costs an embedding call
                                      against this deployment's Google
                                      quota, and nothing limits it today.

  A query log per caller              The gate already logs every call to
                                      storage/app/tool-logs, but with no
                                      caller field.

  **The licence position confirmed**  This one blocks publishing; the
                                      other three are engineering.
  -----------------------------------------------------------------------

**Security boundary**

**Guaranteed, because it is enforced in code**

-   Every query is screened before retrieval. The restricted-topic guard
    (categories: dosing, clinical) runs inside the tool, on the query
    text, every time. An external caller's query would be screened
    exactly as the agent's is. A refused query returns GUARD_REFUSED and
    retrieves nothing.

-   No passage below the threshold is ever returned. 0.65 cosine
    similarity, set in Week 3 between the scores of unanswerable (0.58)
    and answerable (0.70--0.87) queries.

-   Read-only. The tool has no write path, and the gate would hold any
    write tool for officer approval anyway. Nothing a caller sends can
    change the corpus, the farm records, or anything else.

-   Output conforms to the schema or is not returned. Validated on every
    call.

**Restricted content, and an overclaim that was checked**

Volume-based application rates, mixing ratios and named restricted
sections were stripped from the corpus before indexing. What was removed
cannot be retrieved, because it is not in the index.

+-----------------------------------------------------------------------+
| **This is pattern-based, not a proof.**                               |
|                                                                       |
| An earlier draft of this specification said a passage therefore       |
| cannot contain a dosing instruction. That was too strong. The         |
| stripper matches patterns, and it deliberately keeps weight-per-area  |
| figures, because in agronomy text they are almost always seed rates,  |
| yields or fertiliser rates.                                           |
+-----------------------------------------------------------------------+

So the claim was tested rather than softened. At generation the dose
screen (RestrictedTopicGuard::screenOutput) is run over all 301 indexed
passages and the result printed. It flagged six --- DOC-001-C002, -C019,
-C020, -C102, DOC-002-C022 and -C031 --- and every match was read: seed
rates (30 kg/acre beans, 10 kg/acre maize), variety yields, and manure
and fertiliser rates (1000 kg/Ha, 100 kg/Ha). None is a pesticide dose.

**If the screen ever names any other passage, it has not been reviewed.
The screen is not applied to passages at query time, so a dose phrased
in a way neither pattern recognises would pass. None is known. The
specification now says "removed by pattern, and checked against the
current index", which is what the code supports.**

**Not guaranteed**

  -----------------------------------------------------------------------
  **Not guaranteed**                  **Why not**
  ----------------------------------- -----------------------------------
  That the caller cites what it is    See Obligations. The server cannot
  given, or uses it at all            see the caller's output.

  That the corpus is current          It is as of the build dates.
                                      Agronomic recommendations change
                                      --- varieties, pests, approved
                                      products.

  Anything about who the caller is    There is no authentication.

  That a relevant passage is found    The corpus covers beans, maize and
                                      clonal coffee nurseries. A question
                                      about any other crop will usually,
                                      and correctly, return count: 0.

  That the guard catches every        It is a vocabulary rule with
  restricted query                    subject words. A query phrased
                                      wholly outside its vocabulary is
                                      not refused. It would still
                                      retrieve only from a corpus with
                                      restricted content removed ---
                                      which is why both layers exist.
  -----------------------------------------------------------------------

**What is deliberately not exposed, and why**

The one capability worth sharing is the grounded knowledge. The
judgement and the authority stay here.

  -----------------------------------------------------------------------
  **Tool**                **Access**              **Why it is not
                                                  exposed**
  ----------------------- ----------------------- -----------------------
  farm_profile_lookup     read                    Farm records are the
                                                  deployment's own data:
                                                  synthetic here, real
                                                  farmers elsewhere.

  schedule_visit          write                   It writes. Every write
                                                  in this system is held
                                                  for a named officer who
                                                  sees what will change
                                                  and decides. An
                                                  interface cannot carry
                                                  that boundary: approval
                                                  is a person deciding,
                                                  not a permission a
                                                  caller can hold.

  log_followup            write                   It writes, for the same
                                                  reason.

  recall_farm_history     read                    It is the deployment's
                                                  operational history ---
                                                  what its officers
                                                  decided about its
                                                  farms, shared across
                                                  those officers by
                                                  design, including their
                                                  reasons in their own
                                                  words. Exposing it
                                                  would publish one
                                                  organisation's working
                                                  record, and replay free
                                                  text written by its
                                                  staff into a stranger's
                                                  model.
  -----------------------------------------------------------------------

+-----------------------------------------------------------------------+
| **The agent loop is not exposed either, and this is the argument, not |
| a formality.**                                                        |
|                                                                       |
| A bounded loop with a human approval boundary is not a capability a   |
| caller can hold; it is a workflow the deployment owns. Its bounds --- |
| iterations, failures, runtime --- its refusal rule and its approval   |
| boundary are meaningful only because AgriVisit decides them and an    |
| officer sits at the end. Exposed as a tool, it would hand out exactly |
| the agency this project spent five weeks bounding, to a caller whose  |
| own loop, prompt and approvals AgriVisit cannot see.                  |
+-----------------------------------------------------------------------+

**Obligations that transfer to the caller**

**Cite**

Every passage carries a chunk_ref. The server can return it; it cannot
make the caller cite it. A system that uses these passages without
citing them has broken the grounding discipline the corpus exists to
support, and the server has no way to tell.

This is the same limit the Memory Design Note states about the agent's
own citations: AgriVisit can check that a reference its own agent writes
was returned to it, but it cannot prove an uncited passage did not shape
an answer. Across an interface even the first half is gone --- the
caller's output never comes back.

**Three more**

-   Refusals are final for the topic. A GUARD_REFUSED query should be
    reported to the caller's user, not reworded until something gets
    through. Inside AgriVisit the run ends on it. Outside, nothing can
    enforce that.

-   count: 0 means "no guidance found". Say so. Do not fill the gap.

-   Do not present passages as current advice. They are as current as
    the corpus build dates.

**Respect the sources' licences --- the prerequisite that is not
engineering**

Two of the three publications state no licence: they are Government of
Uganda manuals. The third is © 2019 Uganda Coffee Development Authority.
AgriVisit uses them internally for extension support. Whether passages
may be served to a third party's software is redistribution, and it is
not established.

  -----------------------------------------------------------------------
  **This interface should not be offered outside the deployment until the
  licence position is settled with the publishers.**

  -----------------------------------------------------------------------

**Errors**

Returned as a tool result with isError: true and the code in
structuredContent.code --- never a protocol error, except for malformed
JSON-RPC itself. Every code AgriVisit has is listed, including those
this capability cannot return.

  -----------------------------------------------------------------------
  **Code**                            **What it means to a caller**
  ----------------------------------- -----------------------------------
  --- (ok: true)                      Success. count may be 0.

  INVALID_ARGUMENTS                   The arguments failed the input
                                      schema: query missing, under 3 or
                                      over 300 characters, max_results
                                      outside 1--5, or an unknown
                                      property. Nothing was searched.
                                      Retrying unchanged returns the
                                      same.

  GUARD_REFUSED                       The query is about a restricted
                                      topic --- pesticide dosing,
                                      veterinary or clinical advice.
                                      Nothing was retrieved. Final for
                                      the topic: rewording to reach the
                                      same topic is the behaviour the
                                      guard exists to stop, and the agent
                                      is stopped for doing it.

  SERVICE_UNAVAILABLE                 The search could not run, usually
                                      because the embedding service
                                      failed or timed out. Nothing was
                                      returned. Safe to retry later.

  UNEXPECTED_RESPONSE                 The server produced results that do
                                      not match its own output schema, so
                                      they were discarded. A server
                                      fault, not a caller fault.

  UNAUTHORIZED                        The capability is switched off in
                                      this deployment's configuration.

  TOOL_NOT_FOUND                      The caller named a tool this server
                                      does not have.

  APPROVAL_REQUIRED                   Never returned by this capability.

  RESOURCE_NOT_FOUND                  Never returned by this capability.
  -----------------------------------------------------------------------

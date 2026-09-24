AgriVisit
Working Baseline Model Interaction
Group / project	AgriVisit_Capstone — AgriVisit
Week	Week 2 — Foundation-Model Engineering and Prompting (Brief §7)
Owner	Nakayiza Nairah (Application / Integration Lead)
Deliverable	Working baseline model interaction
Repository	https://github.com/arthursuuna/AgriVisit_Capstone
ClickUp board	https://app.clickup.com/1200430000004544/v/l/6-1200430000010607-1

The smallest useful model-backed capability: given a farm profile, draft a field-visit checklist. No retrieval, no tools, no memory, no agent loop.
1. What the user does
1	Opens the Visit Prep Console in a browser.
2	Selects a farm from the dropdown (four synthetic profiles).
3	Optionally types officer notes.
4	Clicks Draft checklist.
5	Reads the drafted items, each with a category and rationale, plus the prompt version, model, temperature, token counts, latency and trace ID.
2. What happens inside
	Stage	What it does
1	Restricted-topic screen	Deterministic check on the officer notes. A dose or clinical request is refused here and the model is never called.
2	Prompt assembly	System and user templates loaded from resources/prompts/checklist/v1.1/ and filled with the farm profile and notes.
3	Model call	One call to Gemini 3.5 Flash through GoogleClient: temperature 0.2, 4000-token output budget, thinking level low. Errors, safety blocks and truncation are caught, never shown to the user as crashes.
4	Output screen	The same deterministic check runs on the generated text, catching a dose the model produced unprompted.
5	Format validation	Strict JSON parse. Wrong item count, an empty field, or any grounding other than "ungrounded" discards the draft.
6	Trace written	One JSON file per attempt: prompt version, model, sampling settings, input, output and thinking tokens, latency, outcome and raw output.
3. How the model is wired in
The model sits behind an LlmClient interface with three implementations — AnthropicClient, OpenAiClient and GoogleClient — selected in AgriVisitServiceProvider by the LLM_PROVIDER setting. Moving from Claude to Gemini meant adding GoogleClient and changing configuration. ChecklistDrafter, the guard and the parser were never touched, which is the interface doing the job it was designed for.
GoogleClient calls Gemini's native REST endpoint:
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent
The API key is sent in the x-goog-api-key header, not as a ?key= query parameter, because query strings end up in proxy and server logs.
Gemini difference	How GoogleClient handles it
Model named in the URL path	Built into the request URL rather than the body
System prompt is a separate systemInstruction field	Sent as systemInstruction, not a role-tagged message
Safety block returns HTTP 200 with no candidate	Treated as a failure, not an empty answer
Output budget includes thinking tokens	MAX_TOKENS detected and reported as truncation, not as a formatting error
4. Files that make it work
Location	What it holds
app/Services/Llm/	LlmClient interface; AnthropicClient, OpenAiClient, GoogleClient; LlmResponse, which records input, output and thinking tokens.
app/Services/Prompts/	PromptRepository — loads versioned prompt templates from disk.
app/Services/Checklist/	ChecklistDrafter, RestrictedTopicGuard, ChecklistParser, TraceWriter, DraftResult.
app/Http/Controllers/	ChecklistController — two routes, no decision logic.
resources/views/checklist/	index.blade.php — the console.
resources/prompts/checklist/	v1.0/ and v1.1/, each with system.md and user.md.
storage/app/farm_profiles.json	Four synthetic farm profiles. No real farmer data.
tests/Feature/	Guard and parser tests. No API key needed.
5. Running it
# .env — the API key is never committed
LLM_PROVIDER=google
LLM_MODEL=gemini-3.5-flash
LLM_TEMPERATURE=0.2
LLM_MAX_TOKENS=4000
PROMPT_VERSION=v1.1

php artisan config:clear
php artisan serve               # then open http://127.0.0.1:8000
php artisan agrivisit:evaluate  # runs the 10-case evaluation set
6. The three outcomes
Outcome	What the officer sees
Drafted	5–10 items under a banner stating the draft is not approved for field use and carries no citations.
Refused	Approved refusal wording naming the category. For requests caught by the guard, the model was never called.
Failed	A plain statement that no checklist was produced. Never a partial result.


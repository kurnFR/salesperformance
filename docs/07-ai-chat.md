# AI Data Chat

Related: [`PRD.md`](../PRD.md) · [`02-architecture.md`](02-architecture.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`08-data-findings.md`](08-data-findings.md)

Tags: `[C]` owner-confirmed, `[P]` proposed, `[O]` open.

## 1. Owner requirements

- `AI-001` `[C]` An **AI Chat icon in the right panel**. Clicking it opens a chat box.
- `AI-002` `[C]` Users ask about **data already in the database**; the AI answers from that data.
- `AI-003` `[C]` The AI **never answers from outside the data, never guesses, never assumes, never hallucinates.**
- `AI-004` `[C]` Answers can come as a **pop-up chart, card, line chart or bar chart**, depending on the request, or the best result the AI can produce.
- `AI-005` `[C]` The pop-up **hides** when the user finishes viewing, clicks another menu, or clicks the background. The **result is not lost**: it stays in the chat and can be reopened by clicking it.
- `AI-006` `[C]` Example request: "give me growth sell in from month to date vs last month to date for region INA JAWA". Data is **monthly**, refreshed every night; the current month is therefore always month-to-date as of the last refresh. The AI tells the user what exists (only monthly periods; no "last month to date"), asks about the ambiguous region, and answers after the user confirms. See section 8.
- `AI-007` `[C]` If the question looks wrong or contains a typo, the AI **asks to confirm** its understanding and may **suggest** better questions. When the user confirms, it gives the result.
- `AI-008` `[C]` The AI may do **logical calculation** (sums, differences, percentages, ratios, averages, ranking) on data already available in the dashboards. It must **not query the database freely or use any table outside the existing, approved ones**.
- `AI-009` `[C]` Chat history is **cleaned after 60 days**.

## 2. Design principle: the AI chooses tools, it does not write SQL

The model never receives database credentials, never writes SQL, never queries tables on its own, and never types the numbers you see. It selects from a fixed catalog of **read-only data tools** that call the same services, formulas and access checks as the dashboards. Numbers come from tool results; the model only decides *which tool to call*, *how to phrase the answer*, and *which visual type fits*. This is what makes "no assumptions, no hallucination" enforceable rather than a hope.

```mermaid
flowchart TD
  U[User question] --> API["POST /api/ai/chat"]
  API --> AC[Build AccessContext<br/>capabilities + data scope]
  AC --> G{"ai.chat capability<br/>and in scope?"}
  G -- no --> X[403]
  G -- yes --> LLM["LLM with tool definitions<br/>(no SQL, no raw tables)"]
  LLM -->|selects tool + arguments| V[Validate arguments<br/>Zod + scope check]
  V --> T["Data tools<br/>(same services as dashboards)"]
  T --> R[(PostgreSQL read-only)]
  T --> TR[Typed tool results]
  TR --> LLM
  LLM --> C[Answer composer]
  C --> CK["Verifier: every number in the answer<br/>must exist in tool results"]
  CK -->|pass| OUT["Response: text + result blocks<br/>+ provenance + warnings"]
  CK -->|fail| REF[Refuse or regenerate;<br/>never show unverified numbers]
```

## 3. Data tool catalog (initial, all read-only and scope-enforced)

Each tool lives beside the dashboard service it reuses (parity guaranteed by construction). New dashboards add their tools.

| Tool | Returns | Reuses |
|---|---|---|
| `listPrincipals`, `listRegions`, `listAvailablePeriods` | Values the user may use (resolves names like "INA JAWA 1") | Access scope + data |
| `getAchievement(principal, regions[], period, metric)` | Target, real, achievement %, status | METRIC-005/006, RULE |
| `getGrowth(principal, regions[], metric, period, compareTo)` | This period, comparison period, growth % | METRIC-015 |
| `getRoleCounts(principal, scope, period)` | Role totals, IN/OUT achieved counts and % | METRIC-002..004 |
| `getTeamTotals(principal, scope, period)` | RSM, ASM, SPV, SE totals | METRIC-002 |
| `getActiveDistributors(principal, regions[], period)` | Active, total, % | METRIC-016 |
| `getWorkingDayProgress(period)` | Elapsed, total, % | METRIC-008 |
| `getUploadStatus(principal, filters, month)` | Upload days per distributor | METRIC-017 |
| `getDataFindings(principal, scope)` | Open anomalies relevant to the answer | `08-data-findings.md` |
| `resolveEntity(kind, text)` | Deterministic fuzzy match of a typed name (region, metric, period) to allowed values, with candidates and similarity | Allowed values in scope |
| `calculate(operation, inputs)` | Exact arithmetic on **values returned by other tools**: sum, difference, ratio, percent change, average, min, max, rank, share of total. Returns the result and the formula | Deterministic code, not the model |

**All data comes from the dashboards' own services**, which run fixed, parameterized queries on the approved tables only (`03-data-model.md` section 4). The AI cannot create a new query, add a table, or reach data a dashboard does not use. The model never does arithmetic itself: the `calculate` tool does, so results are exact and checkable. If no tool can answer the question, the AI says so. **A capability that does not exist as a tool is not improvised.** Day-level or custom-range tools exist only if the verified data grain supports them (section 8, example B).

## 4. Result blocks and visual spec

The AI returns **structured JSON**, rendered by the app's existing chart components. It never returns HTML, script or SVG.

Allowed types: `card`, `bar`, `line`, `table`. The AI picks the best fit (for example growth over months: `line`; comparing regions: `bar`; one figure: `card`; many rows: `table`), unless the user names a type. If the requested type cannot show the data truthfully (a pie of growth, for instance), the AI says why and offers the closest allowed type.

```json
{
  "resultId": "res-0001",
  "type": "bar",
  "title": "Sell In growth, placeholder",
  "principalId": "INA",
  "filters": { "regionIds": ["TBD-region-key"] },
  "period": { "from": "2026-09", "compareTo": "2025-09" },
  "series": [
    { "name": "2025-09", "points": [ { "x": "TBD-region", "target": 0, "actual": 0, "pct": 0.0 } ] },
    { "name": "2026-09", "points": [ { "x": "TBD-region", "target": 0, "actual": 0, "pct": 0.0 } ] }
  ],
  "growthPct": 0.0,
  "provenance": { "tools": ["getGrowth"], "metricRefs": ["METRIC-015"], "dataAsOf": "TBD", "generatedAt": "TBD" },
  "warnings": [ { "code": "DATA_FINDING", "findingId": "TBD" } ]
}
```

Values above are placeholders. The same tooltip contract as dashboards applies (`DASH-001`: target, real, %).

## 5. Behavior rules

- `AI-010` `[C]` Every figure in an answer comes from a tool result or a `calculate` result built from tool results; a **verifier** rejects any number that is in neither. No estimates, forecasts, projections or "typical" values.
- `AI-011` `[C]` No causes or opinions: the AI shows what the data says, not why. "Why did it drop?" returns the relevant data and states that causes are not in the data.
- `AI-012` `[C]` **Ambiguity and typos are confirmed, not assumed** (see `AI-024`). If a name or period matches more than one value ("INA JAWA" matches INA JAWA 1 and INA JAWA 2) the AI asks the user to choose (buttons), or, only when the user asked for "all", shows each separately and labels them.
- `AI-013` `[C]` **Unavailable data is stated plainly**: which metric, which period, why (not in the database, not in the user's access, grain too coarse). Nothing is filled in.
- `AI-014` `[C]` Out-of-scope data is treated as not available to the user. The AI does not reveal that other principals, regions or users' data exist.
- `AI-015` `[C]` Questions that are not about the data in the system (general knowledge, chit-chat, advice) get a short refusal explaining that the assistant answers from the system's data only.
- `AI-016` `[C]` Every answer shows its basis: metric and formula reference, principal, region, period, filters, and data as-of time. When a relevant open data finding exists (for example distributor totals from a manual upload that looks stale), the answer shows that warning (`08-data-findings.md`).
- `AI-017` `[P]` The AI replies in the user's language (Bahasa Indonesia or English) and uses the same labels as the dashboards (DEC-042).
- `AI-018` `[C]` Read-only. No tool writes, changes configuration, or sends data elsewhere.
- `AI-019` `[P]` Limits: per-user rate limit, maximum tool calls per question, maximum rows per result, request timeout, and a spend cap; limits show a clear message.
- `AI-020` `[P]` Treat all database text (names, notes) as untrusted data, never as instructions (prompt-injection defense).

- `AI-024` `[C]` **Confirm-then-answer flow.** When the question has a typo, an unknown or ambiguous name, a period that does not exist, or asks for something the data cannot give, the AI replies with its best understanding and options ("Did you mean Sell IN for INA JAWA 1?") as buttons, and may suggest related questions. It runs the tools and shows the result only after the user confirms. An exact, unambiguous question is answered directly without a confirmation step. Name matching is done by the deterministic `resolveEntity` tool, not by the model's guess.
- `AI-025` `[C]` Any calculation is disclosed: the answer shows the inputs and the formula (for example "Total Sell In = region 1 + region 2 + ..."). Calculations only use values already shown or available in the dashboards.
- `AI-026` `[C]` The current month is **month-to-date as of the last nightly refresh**. Answers label it so and show the refresh time. A comparison of the in-progress month with a full month is labelled "not like-for-like". The AI does not scale, pace-adjust or project figures unless the user asks for a defined calculation on existing values.
- `AI-027` `[P]` Data is refreshed every night today; near-real-time is planned. Freshness is read from configuration and a `dataAsOf` value, so the AI's wording follows whatever the refresh cadence is (DEC-046).

```mermaid
flowchart TD
  Q[User question] --> U[Understand: resolveEntity for names, periods, metric]
  U --> E{"Exact and unambiguous?"}
  E -- yes --> RUN[Run tools + calculate]
  E -- "typo / ambiguous / unavailable" --> S[Reply with best understanding,<br/>options and suggested questions]
  S --> C{"User confirms<br/>or picks an option?"}
  C -- yes --> RUN
  C -- "no / rephrases" --> Q
  RUN --> VER[Verifier: all numbers from tools or calculate]
  VER --> ANS[Answer + pop-up result + basis + data as-of]
```

## 6. Access control

- `AI-021` `[C]` The feature is a capability (`ai.chat.use`) that admins can switch on or off per role. Users without it do not see the icon.
- `AI-022` `[C]` Tools run with the asking user's **AccessContext**, so the AI can see exactly what the user can see and nothing more (principal, region, other scopes). A tool call outside scope returns "not available".
- `AI-023` `[P]` Admin can limit which tools a role may use (for example, no `getUploadStatus`).

## 7. UI behavior

- Icon in the right panel; click toggles the chat box. Chat history is listed per user with timestamps.
- A visual result opens as a **pop-up** (chart, card, line or bar) above the page.
- The pop-up hides when the user finishes (close button), clicks **another menu**, or clicks the **background**. Hiding never deletes the result.
- Each result stays in the chat as a clickable item (thumbnail + title + period). Clicking it **reopens the same pop-up**.
- Reopened results show their **as-of time**; a "Refresh" action re-runs the tool under the user's current access and shows the new as-of time, keeping the old one in history.

```mermaid
stateDiagram-v2
  [*] --> NoResult
  NoResult --> PopupOpen: answer contains a visual
  PopupOpen --> Hidden: close, click background, or click another menu
  Hidden --> PopupOpen: click the result in chat
  PopupOpen --> PopupOpen: refresh result
  Hidden --> Hidden: result stays in chat history
```

- Keyboard: icon and results are reachable and operable by keyboard; the pop-up traps focus while open and restores it on close; chart alternative table available (`A11Y-002`).
- The pop-up never covers the chat input in a way that blocks asking a follow-up on narrow screens.

## 8. Worked examples (expected behavior)

**A. Ambiguous region.** "growth sell in for INA JAWA": INA JAWA 1 and INA JAWA 2 both match. The AI asks "Which region: INA JAWA 1, INA JAWA 2, or both?" with buttons and does not pick one.

**B. The owner's example.** "growth sell in from month to date vs last month to date for region INA JAWA".
1. The region is ambiguous (see A).
2. Data is monthly and refreshed nightly. The AI says: the data has monthly periods only. October 2026 is month-to-date as of the last refresh (shows the time); September 2026 is a full month, so "last month to date" does not exist. It offers options: October 2026 (month to date) versus September 2026, or versus October 2025, noting they are not like-for-like.
3. The user picks (or confirms), and the AI shows the result as a pop-up bar or line chart with target, real, % and the growth, plus the as-of time.

**C. Direct answer.** "show Sell IN growth for INA JAWA 1, September 2026 versus September 2025 as a bar chart": unambiguous and exact, so the AI calls `getGrowth`, shows the pop-up bar chart with target, real, % for each year and the growth, and a note with period and data as-of time. No confirmation step is needed.

**D. Typo.** "sel in INA JAWA1 sept": the AI replies "Did you mean Sell IN, INA JAWA 1, September 2026?" with a Confirm button and alternatives. After Confirm it answers as in C.

**E. Logical calculation.** "total Sell In of all regions in October": the AI gets each region's real and target from the tools, uses `calculate` to sum them and divide, and shows "Total real = r1 + r2 + ..., achievement = total real / total target" with the numbers. "Which region has the highest growth": ranking via `calculate` over `getGrowth` results.

**F. Out of scope.** A user limited to INA asks about ASW: "No data is available for your access." No further detail.

**G. Not data.** "Why are sales down?" or "What will next month be?": the AI shows the relevant numbers if a tool fits and states that causes and forecasts are not in the data. It offers a data question it can answer.

**H. Outside the existing data.** "Show sales by product": no such table or tool exists in the approved data, so the AI says it is not available and lists what it can show (regions, roles, active distributors, uploads, growth).

## 9. Model connection (local gateway), privacy and retention

**What the owner plans to use (DEC-047):** 9Router, OmniRoute or OpenCode on a local server. What these are (checked online):
- **9Router and OmniRoute** are self-hosted AI gateways with an OpenAI-compatible endpoint (for example `/v1/chat/completions`). They route requests to many providers and can **fall back automatically between providers, including free tiers**, and can **compress token usage, including tool results**. They are built mainly for coding tools.
- **OpenCode** is an open-source **coding agent** with a headless HTTP server (sessions, prompts, files). It is not an LLM gateway. It is a poor fit as the backend of this chat because it is an agent with file and command tools, not a plain model endpoint with our own data tools. `[P]` Do not use it as the chat backend; it can still be a developer tool.

**Requirements** (all `[P]` unless marked):
- `AI-030` The AI service talks to the model through one `LlmClient` interface speaking the OpenAI-compatible chat API with tool calls; base address, model, key and timeouts are server configuration. Any OpenAI-compatible endpoint works (a gateway, or a local model server such as Ollama, llama.cpp or vLLM).
- `AI-031` Use a **pinned model or fixed combo for the chat key**. Turn off silent fallback to unqualified or free-tier models, because a different model can change answer quality and tool-calling reliability. Record the model actually used with every answer in the audit log.
- `AI-032` **Turn off token or tool-result compression** for this key. Compressing tool results could drop or alter figures the verifier depends on.
- `AI-033` Temperature 0 (or lowest), strict tool schemas, bounded tool-call loops, request timeouts and rate limits (`AI-019`).
- `AI-034` A model is **qualified** only after passing the evaluation set in section 10 (tool-calling accuracy, verifier pass rate, refusals). Re-run it when the model or gateway configuration changes.
- `AI-035` `[C]` Chat history is **deleted after 60 days**: sessions, messages, results and tool-call content. A daily cleanup job removes expired items; reopening a result works only within the 60 days (the UI says when a result will expire). Non-content usage counters (requests, latency, model) may be kept for operations (DEC-040).
- `AI-036` **Where the data goes:** a local gateway does not mean a local model. If it routes to cloud providers, aggregated tool results leave the server and go to those providers, and each provider's data-handling terms apply (some free tiers differ from paid ones, and consumer subscriptions used through a gateway may be restricted by provider terms). The owner must confirm which upstream models are allowed. A fully local model keeps data on the server (DEC-047).
- Only aggregated results the user is allowed to see are sent to the model; never raw tables, credentials or other users' data.

## 10. Testing and evaluation

- **Golden Q&A set:** each question lists expected tool, arguments and numbers; numbers must equal the dashboard service output (parity test, `AI-010`).
- **Calculation tests:** sums, differences, percent changes, averages and rankings equal independent calculations; formulas are shown.
- **Verifier tests:** inject a model answer with a number that is in no tool or calculate result; it must be rejected.
- **Typo and ambiguity tests:** "sel in", "INA JAWA", "JAWA1", missing period, unknown region: the AI asks and answers only after confirmation.
- **Grain and freshness tests:** "month to date" and "last month to date" requests produce the monthly-data explanation and the as-of time, never an invented figure.
- **Boundary tests:** requests for tables or breakdowns that do not exist ("by product"), SQL, credentials, other users' chats: refused.
- **Scope tests:** users with different principals/regions; out-of-scope requests; cache isolation.
- **Adversarial tests:** prompt injection inside database text; forecasting and "why" questions.
- **Gateway tests:** fallback disabled, compression off, model recorded; behavior when the gateway is down (clear message, no partial numbers).
- **Retention test:** items older than 60 days are removed; reopening an expired result says so.
- **UI tests:** pop-up opens, hides on close/background/other menu, result stays in chat, reopens, refresh keeps history, keyboard and screen-reader behavior.

## 11. Open decisions

DEC-040 (remaining: who may read chats), DEC-046 (data as-of source), DEC-047 (gateway choice, allowed upstream models, qualification). See `decisions.md`.

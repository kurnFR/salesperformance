# AI Data Chat

Related: [`PRD.md`](../PRD.md) · [`02-architecture.md`](02-architecture.md) · [`01-metrics-contract.md`](01-metrics-contract.md) · [`08-data-findings.md`](08-data-findings.md)

Tags: `[C]` owner-confirmed, `[P]` proposed, `[O]` open.

## 1. Owner requirements

- `AI-001` `[C]` An **AI Chat icon in the right panel**. Clicking it opens a chat box.
- `AI-002` `[C]` Users ask about **data already in the database**; the AI answers from that data.
- `AI-003` `[C]` The AI **never answers from outside the data, never guesses, never assumes, never hallucinates.**
- `AI-004` `[C]` Answers can come as a **pop-up chart, card, line chart or bar chart**, depending on the request, or the best result the AI can produce.
- `AI-005` `[C]` The pop-up **hides** when the user finishes viewing, clicks another menu, or clicks the background. The **result is not lost**: it stays in the chat and can be reopened by clicking it.
- `AI-006` `[C]` Example request: "give me growth sell in from month to date vs last month to date for region INA JAWA". See the worked examples in section 8: this request is ambiguous in two ways and may not be answerable at all, and the AI must say so instead of guessing.

## 2. Design principle: the AI chooses tools, it does not write SQL

The model never receives database credentials, never writes SQL, and never types the numbers you see. It selects from a fixed catalog of **read-only data tools** that call the same services, formulas and access checks as the dashboards. Numbers come from tool results; the model only decides *which tool to call*, *how to phrase the answer*, and *which visual type fits*. This is what makes "no assumptions, no hallucination" enforceable rather than a hope.

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

If no tool can answer the question, the AI says so. **A capability that does not exist as a tool is not improvised.** Day-level or custom-range tools exist only if the verified data grain supports them (section 8, example B).

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

- `AI-010` `[C]` Every figure in an answer comes from a tool result; a **verifier** rejects any number not present in tool results. No estimates, forecasts, projections or "typical" values.
- `AI-011` `[C]` No causes or opinions: the AI shows what the data says, not why. "Why did it drop?" returns the relevant data and states that causes are not in the data.
- `AI-012` `[C]` **Ambiguity is asked, not assumed.** If a name or period matches more than one value ("INA JAWA" matches INA JAWA 1 and INA JAWA 2) the AI asks the user to choose (buttons), or, only when the user asked for "all", shows each separately and labels them.
- `AI-013` `[C]` **Unavailable data is stated plainly**: which metric, which period, why (not in the database, not in the user's access, grain too coarse). Nothing is filled in.
- `AI-014` `[C]` Out-of-scope data is treated as not available to the user. The AI does not reveal that other principals, regions or users' data exist.
- `AI-015` `[C]` Questions that are not about the data in the system (general knowledge, chit-chat, advice) get a short refusal explaining that the assistant answers from the system's data only.
- `AI-016` `[C]` Every answer shows its basis: metric and formula reference, principal, region, period, filters, and data as-of time. When a relevant open data finding exists (for example distributor totals from a manual upload that looks stale), the answer shows that warning (`08-data-findings.md`).
- `AI-017` `[P]` The AI replies in the user's language (Bahasa Indonesia or English) and uses the same labels as the dashboards (DEC-042).
- `AI-018` `[C]` Read-only. No tool writes, changes configuration, or sends data elsewhere.
- `AI-019` `[P]` Limits: per-user rate limit, maximum tool calls per question, maximum rows per result, request timeout, and a spend cap; limits show a clear message.
- `AI-020` `[P]` Treat all database text (names, notes) as untrusted data, never as instructions (prompt-injection defense).

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

**A. Ambiguous region.** "growth sell in for INA JAWA": INA JAWA 1 and INA JAWA 2 both match. The AI asks "Which region: INA JAWA 1, INA JAWA 2, or both?" with buttons. It does not pick one.

**B. The example request.** "growth sell in from month to date vs last month to date for region INA JAWA".
1. Region is ambiguous (see A).
2. **Month-to-date needs day-level data.** The reference dashboards use monthly Sell In and Sell Out. If the performance source is monthly, month-to-date versus last-month-to-date cannot be computed. The AI says so and offers only options that exist, for example: "Sell In for October 2026 (month in progress) versus September 2026, or versus October 2025", presented as choices, not silently substituted. If the source turns out to have daily rows, a day-level tool is added and the request is answered directly. **Verify the grain first (DEC-041).**

**C. Successful answer.** "show Sell In growth for INA JAWA 1, September 2026 versus September 2025 as a bar chart": the AI calls `getGrowth`, shows a pop-up bar chart with target, real, % for each year and the growth %, and a note with period and data as-of time.

**D. Out of scope.** A user limited to INA asks about ASW: "No data is available for your access." No further detail.

**E. Not data.** "Why are sales down?" or "What will next month be?": the AI shows the relevant numbers if a tool fits, and states that causes and forecasts are not in the data.

## 9. Privacy, provider and cost

- Tool results (aggregated business figures) are sent to the LLM provider so it can compose the answer. The owner must approve the provider and what is sent; raw tables are never sent (DEC-040).
- Chat history, results and tool calls are stored per user for reopening and audit; retention and who may read them are open (DEC-040). Admins do not read other users' chats unless the owner decides so and it is audited.
- Cost controls per `AI-019`; usage visible to admins.

## 10. Testing and evaluation

- **Golden Q&A set:** each question lists expected tool, arguments and numbers; numbers must equal the dashboard service output (parity test, `AI-010`).
- **Verifier tests:** inject a model answer with a number that is not in tool results; it must be rejected.
- **Ambiguity tests:** "INA JAWA", missing period, unknown region.
- **Grain tests:** month-to-date request against monthly data must produce the "not available" answer, never a number.
- **Scope tests:** users with different principals/regions; out-of-scope requests; cache isolation.
- **Adversarial tests:** prompt injection in data text; requests for SQL, credentials, other users' chats; forecasting and "why" questions.
- **UI tests:** pop-up opens, hides on close/background/other menu, result remains, reopens, refresh keeps history, keyboard and screen-reader behavior.

## 11. Open decisions

DEC-040 (provider, data sharing, retention), DEC-041 (data grain for month-to-date), DEC-042 (language). Tracked in `decisions.md`.

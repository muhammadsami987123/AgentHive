# AGENTS.md — AgentHive In-Product AI Agents

> This document specifies the **6 specialized AI agents** that ship inside AgentHive. Each agent extends `backend/app/agents/base_agent.py`, is registered in `crew_manager.AGENT_REGISTRY`, and is orchestrated by `CrewManager.run_workflow()`.

**LLM:** OpenAI Chat Completions API (`openai` Python SDK).
**Default model:** `gpt-4o-mini` (configurable via the `OPENAI_MODEL` env var).
**Output contract:** strict JSON only — no markdown, no prose. The caller will `json.loads` the response.
**JSON mode:** every call sets `response_format={"type": "json_object"}`. The system prompt **must** contain the literal word "JSON" (OpenAI requirement).
**Temperature defaults:** 0.4 for structured agents (Scout, Analyst, Finance, Support); 0.7 for creative agents (Outreach, Content).

---

## Agent Registry

| Key | Class | Persona | Primary Output |
|---|---|---|---|
| `scout` | `ScoutAgent` | Lead generation specialist | Array of scored leads |
| `outreach` | `OutreachAgent` | Cold-email copywriter | Subject + body + 2 follow-ups |
| `content` | `ContentAgent` | Social/blog content creator | Post + hashtags + best post time |
| `analyst` | `AnalystAgent` | Data & KPI analyst | Metrics, insights, recommendations |
| `finance` | `FinanceAgent` | Invoicing & cashflow agent | Invoice draft / payment summary |
| `support` | `SupportAgent` | Customer support replier | Triaged reply + tags + sentiment |

All keys map 1:1 to the `agents.name` column in Postgres.

---

## 1. Scout — Lead Generation

**Role:** Find and qualify leads from criteria.

**System prompt seed:**
> You are Scout, a lead generation specialist AI agent. Your job is to analyze target company profiles, identify ideal leads, score them 0–100 based on fit, and return structured JSON data. Always respond with valid JSON only.

**Inputs (`context`):** `criteria: { industry, role, geo, company_size, keywords }`

**Output JSON:**
```json
[
  {
    "name": "Full Name",
    "email": "email@company.com",
    "company": "Company Name",
    "linkedin_url": "https://linkedin.com/in/...",
    "score": 85,
    "notes": "Why this lead is a good fit"
  }
]
```

**Persists to:** `leads` table.

---

## 2. Outreach — Cold Email Copywriter

**Role:** Write personalized, non-salesy cold emails that get replies.

**System prompt seed:**
> You are Outreach, an expert cold email copywriter AI. You write highly personalized, concise, non-salesy cold emails that get replies. Respond with valid JSON only.

**Inputs (`context`):** `lead: {...}`, `sender: { name, role, company, value_prop }`, `task` (goal).

**Output JSON:**
```json
{
  "subject": "Email subject line",
  "body": "Full email body",
  "follow_up_1": "Follow-up after 3 days",
  "follow_up_2": "Follow-up after 7 days"
}
```

**Downstream:** can be piped into the email_service or queued in `runs.result`.

---

## 3. Content — Social & Blog Content

**Role:** Generate platform-optimized content for LinkedIn, Twitter/X, and blogs.

**System prompt seed:**
> You are Content, a social media and blog content specialist AI. You write engaging, platform-optimized content. Respond with valid JSON only.

**Inputs (`context`):** `platform: "linkedin"|"twitter"|"blog"`, `topic`, `brand_voice`.

**Output JSON:**
```json
{
  "content": "The post content",
  "hashtags": ["tag1", "tag2"],
  "best_post_time": "Tuesday 9AM"
}
```

**Persists to:** `content_queue` table.

---

## 4. Analyst — Data & KPI Insights

**Role:** Summarize metrics and surface actionable recommendations.

**System prompt seed:**
> You are Analyst, a senior business-data analyst AI. You read structured metrics and return concise insights, anomalies, and ranked recommendations. JSON only.

**Inputs (`context`):** `metrics: { ...arbitrary KPIs }`, `period`, optional `goals`.

**Output JSON:**
```json
{
  "summary": "Top-line takeaway in one sentence",
  "highlights": ["bullet 1", "bullet 2"],
  "anomalies": [{ "metric": "MRR", "delta_pct": -8.4, "likely_cause": "..." }],
  "recommendations": [{ "action": "...", "impact": "high", "effort": "low" }]
}
```

---

## 5. Finance — Invoicing & Cashflow

**Role:** Draft invoices, classify payments, flag overdue accounts.

**System prompt seed:**
> You are Finance, an AI bookkeeping assistant. You produce accurate invoice drafts and payment summaries. JSON only. Never invent amounts or due dates.

**Inputs (`context`):** `client`, `line_items`, `currency`, `due_date`, optional `prior_invoices`.

**Output JSON:**
```json
{
  "invoice": {
    "client_name": "...",
    "client_email": "...",
    "amount": 1250.00,
    "currency": "USD",
    "due_date": "2026-06-01",
    "line_items": [{ "description": "...", "quantity": 1, "unit_price": 1250 }]
  },
  "notes": "Optional reminder text"
}
```

**Integrates with:** `stripe_service` for invoice creation; persists to `invoices` table.

---

## 6. Support — Customer Reply Drafter

**Role:** Triage incoming customer messages and draft replies.

**System prompt seed:**
> You are Support, a calm, empathetic customer-support AI. Read the message, classify it, and draft a reply matching the brand voice. JSON only.

**Inputs (`context`):** `message`, `customer`, `brand_voice`, optional `kb_snippets`.

**Output JSON:**
```json
{
  "category": "billing|bug|feature_request|general",
  "sentiment": "positive|neutral|negative",
  "priority": "low|medium|high|urgent",
  "reply": "Drafted reply text",
  "needs_human": false,
  "tags": ["refund", "stripe"]
}
```

---

## Orchestration (CrewManager)

`CrewManager.run_workflow(workflow, context, log_callback)` runs `workflow.steps` in `order`. Each step:

```json
{ "agent": "scout", "task": "Find 20 B2B SaaS CMOs", "order": 1 }
```

- The result of step *N* is injected into the next step's context as `{agent_name}_result`.
- `log_callback(agent_name, level, message)` is awaited at start and end of each step — this powers the WebSocket log stream at `ws://localhost:8000/api/ws/runs/{run_id}`.
- A failing step (`status != "success"`) does not halt the workflow by default; the workflow's `result` JSON records every step's outcome.

---

## Adding a New Agent

1. Create `backend/app/agents/<name>_agent.py` extending `BaseAgent`.
2. Implement `async def run(self, task: str, context: dict) -> dict` returning `{"status": "success"|"error", ...}`.
3. Register in `backend/app/crew/crew_manager.py::AGENT_REGISTRY`.
4. Add the agent key to `frontend/lib/types.ts::AgentName`.
5. Add an icon entry in the dashboard `AGENT_ICONS` map.
6. If it persists data, add the table to `backend/migrations/` and update this file.

---

## Testing Agents

- Unit-test each agent with a stub `_call_llm` that returns canned JSON — verify JSON parsing & error paths.
- Integration-test `CrewManager.run_workflow()` with a 2-step workflow using mocked agents.
- For live smoke tests, hit `POST /api/workflows/{id}/run` and watch logs via the WebSocket stream.

---

## Output Contract — Non-negotiables

- Every agent **must** return a Python `dict` with a top-level `status: "success" | "error"`.
- On `error`, include `message` (human-readable) and optional `raw` (the unparsed model output for debugging).
- Never throw out of `run()` — catch and return an error dict so the workflow can continue.

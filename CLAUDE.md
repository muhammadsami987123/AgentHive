# AgentHive — Claude Code Project Guide

> **Stack:** Next.js 15 · TypeScript · Tailwind CSS · FastAPI · Python · Neon (PostgreSQL) · **OpenAI API** · Stripe
> **Goal:** SaaS platform where 6 specialized AI agents collaborate to automate business workflows.
> **LLM:** OpenAI `gpt-4o-mini` (default) via the `openai` Python SDK in JSON mode.

---

## 1. Project Layout

```
agenthive/
├── frontend/        # Next.js 15 (App Router) + TS + Tailwind
├── backend/         # FastAPI + Python 3.12 + asyncpg + lightweight CrewManager
├── .claude/         # Claude Code config, skills, agents
├── project_plan.md  # Single source of truth for the build
├── AGENTS.md        # Spec for the 6 in-product AI agents
└── README.md
```

Authoritative spec for code shape, schemas, env vars, and routes lives in **`project_plan.md`** — always consult it before scaffolding new modules.

---

## 2. Golden Rules (Token-efficient editing)

1. **Don't read the whole repo.** Open only the files the current task touches.
2. **Search before reading.** Use Grep/Glob to locate symbols; then read only the relevant slice.
3. **Scoped edits.** Prefer `Edit` over `Write`. Preserve existing structure.
4. **No redundant work.** Don't re-read files already in context.
5. **No speculative refactors.** Fix what was asked; leave the rest.
6. **No invented comments.** Code should be self-explanatory; comment only non-obvious *why*.

---

## 3. Coding Standards

### Frontend (Next.js 15)
- App Router, Server Components by default; mark client islands with `'use client'`.
- TypeScript strict. Shared types live in `frontend/lib/types.ts`.
- State: **Zustand** for client/auth state, **React Query** for server data.
- Styling: Tailwind only. Use `clsx` for conditional classes. Brand palette in `tailwind.config.ts`.
- Icons: `lucide-react`. Charts: `recharts`.
- API client: `frontend/lib/api.ts` (axios with JWT interceptor).
- **All UI work must follow the `ui-ux-pro-max` skill** — modern, clean, consistent.

### Backend (FastAPI)
- Async everywhere (`asyncpg` + SQLAlchemy async). No sync DB calls.
- Routers under `app/routers/`, agents under `app/agents/`, orchestration in `app/crew/`.
- Every new agent extends `BaseAgent` and is registered in `crew_manager.AGENT_REGISTRY`.
- Pydantic v2 models for all request/response bodies.
- JWT auth via `app/middleware/auth_middleware.py`.
- Background jobs via FastAPI `BackgroundTasks`; long workflows stream logs over WebSocket.

### Database
- Neon Postgres. Migrations in `backend/migrations/` (plain SQL, numbered).
- UUID primary keys, `TIMESTAMPTZ` for time columns, JSONB for flexible config.
- Add an index for every foreign-key column and every column you filter on.

### AI / OpenAI API
- Model: `gpt-4o-mini` (default; overridable via `OPENAI_MODEL` env var).
- Use the official `openai` Python SDK only — no CrewAI, no LangChain, no Anthropic SDK.
- Always call Chat Completions with `response_format={"type": "json_object"}` — JSON mode.
- **JSON-mode constraint:** the system prompt must contain the literal word "JSON" or OpenAI will reject the request.
- Agents return strict JSON — never markdown. The caller must `json.loads` defensively (return `{"status":"error","raw":...}` on failure).
- Keep `temperature` low (≤0.5) for structured-output agents (Scout/Finance/Analyst); a bit higher (~0.7) is okay for Outreach/Content.
- Use `fetch-library-docs` for OpenAI SDK API questions — don't guess parameter shapes.

### Stripe
- Use **Checkout Sessions** for one-off and subscription billing (per stripe best-practices skill).
- Webhook secret verification is mandatory; never trust client-side price IDs.
- Use restricted keys in non-prod; never log full keys.

---

## 4. Skills to Use (and When)

| Trigger | Skill |
|---|---|
| Any UI / component / page work | `ui-ux-pro-max`, `frontend-design:frontend-design` |
| Touching OpenAI SDK / agent code | `fetch-library-docs` (for `openai` SDK docs) |
| Looking up React/Next/FastAPI/Tailwind APIs | `fetch-library-docs` |
| Stripe integration changes | `stripe:stripe-best-practices` |
| Stripe error debugging | `stripe:explain-error`, `stripe:test-cards` |
| Reviewing changed code for reuse/quality | `simplify` |
| Trimming permission prompts | `less-permission-prompts` |
| Reviewing a PR / pending changes | `code-review:code-review`, `security-review` |
| Writing docs / proposals | `doc-coauthoring` |
| Theming landing / dashboard surfaces | `theme-factory` |
| Clarifying ambiguous user intent | `interview` |

---

## 5. Workflow

1. **Plan** — for any non-trivial task (≥3 files or architectural choice) enter plan mode first.
2. **Search** — Grep/Glob for affected symbols.
3. **Read** — open only what's needed.
4. **Edit** — small, focused diffs.
5. **Verify** — run `npm run build` / `pytest` / type-check for the affected area.
6. **Summarize** — 1–2 sentences: what changed, what's next.

---

## 6. Quick Commands

```bash
# Backend
cd backend && python -m venv venv && venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
uvicorn app.main:app --reload --port 8000

# Frontend
cd frontend && npm install
cp .env.example .env.local
npm run dev

# DB migration
psql $DATABASE_URL -f migrations/001_initial.sql
```

---

## 7. Safety / Guardrails

- **Never commit** `.env`, `.env.local`, or anything matching `*secret*` / `*credentials*`.
- **Never** `git push --force`, `git reset --hard`, or `rm -rf` without explicit user confirmation.
- **Never** hardcode API keys; always read from `app/config.py` settings.
- For destructive DB ops (DROP, TRUNCATE), confirm with the user first.

---

## 8. Definition of Done

A task is done when:
- Code compiles / type-checks.
- Affected routes return expected responses (manual or curl test).
- For UI: rendered in browser, golden path + at least one edge case verified.
- No `.env` or secrets leaked into commits.
- `project_plan.md` schema/contract still matches reality (update if it doesn't).

# 🐝 AgentHive

> **Multi-Agent Business Automation Platform** — A SaaS where 6 specialized AI agents collaborate to run real business workflows: lead-gen, outreach, content, analytics, invoicing, and customer support.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Next.js](https://img.shields.io/badge/Next.js-15-black)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688)](https://fastapi.tiangolo.com/)
[![Anthropic Claude](https://img.shields.io/badge/Claude-Sonnet_4-D97757)](https://www.anthropic.com/)

---

## ✨ What is AgentHive?

AgentHive is a SaaS platform where you assemble **AI agents** into **workflows**, hit run, and watch them collaborate — finding leads, sending emails, drafting content, crunching analytics, issuing invoices, and replying to support — all powered by Claude.

### The 6 Agents

| 🐝 Agent | Role |
|---|---|
| 🔭 **Scout** | Find & score B2B leads from criteria |
| 📧 **Outreach** | Write personalized cold emails + follow-ups |
| ✍️ **Content** | Generate LinkedIn / Twitter / blog content |
| 📊 **Analyst** | Summarize metrics & surface recommendations |
| 💳 **Finance** | Draft invoices, track cashflow |
| 🎧 **Support** | Triage & draft customer replies |

Full spec → [`AGENTS.md`](AGENTS.md)

---

## 🧱 Stack

| Layer | Tech |
|---|---|
| Frontend | Next.js 15 · React 19 · TypeScript · Tailwind · Zustand · React Query · Recharts |
| Backend | FastAPI · Python 3.12 · SQLAlchemy async · asyncpg · CrewAI · LangChain |
| Database | Neon (Serverless PostgreSQL) |
| AI | Anthropic Claude (`claude-sonnet-4-20250514`) · OpenAI (optional) |
| Payments | Stripe (Checkout Sessions + Webhooks) |
| Deploy | Vercel (frontend) · Railway (backend) · Cloudflare R2 (assets) |

---

## 🚀 Quick Start

```bash
# Backend
cd backend
python -m venv venv
venv\Scripts\activate           # macOS/Linux: source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env             # fill in DATABASE_URL, ANTHROPIC_API_KEY, etc.
uvicorn app.main:app --reload --port 8000

# Frontend (new terminal)
cd frontend
npm install
cp .env.example .env.local
npm run dev                      # → http://localhost:3000
```

### Database

```bash
psql $DATABASE_URL -f backend/migrations/001_initial.sql
```

---

## 📁 Project Layout

```
agenthive/
├── frontend/        # Next.js 15 App Router
├── backend/         # FastAPI + agents + CrewAI
├── .claude/         # Claude Code config & enabled skills
├── project_plan.md  # Authoritative build spec
├── AGENTS.md        # In-product AI agent spec
├── CLAUDE.md        # Engineering rules for Claude Code
└── LICENSE
```

---

## 📜 Pricing Tiers

| Plan | Price | Agents | Workflows | Runs/mo |
|---|---|---|---|---|
| Free | $0 | 2 (Scout + Outreach) | 1 | 50 |
| Starter | $29/mo | 4 | 5 | 500 |
| Pro | $79/mo | All 6 | Unlimited | 5,000 |
| Agency | $199/mo | All 6 + custom | Unlimited | Unlimited |

---

## 🛣️ Roadmap

- [x] Project plan & agent specs
- [x] Claude Code config (`.claude/`)
- [ ] Week 1: Neon schema, FastAPI auth, Next.js scaffold
- [ ] Week 2: Agent CRUD + Scout/Outreach implementation + workflow runner
- [ ] Week 3: Real-time WebSocket logs, analytics, remaining 4 agents
- [ ] Week 4: Stripe billing, landing page, deploy

---

## 🤝 Contributing

This repo is private and currently under active solo development. Contribution guidelines will land when it opens up.

---

## 👤 Author

**Muhammad Sami** — AI Agent Engineer & Full-Stack Developer
- Portfolio: <https://muhammad-sami.vercel.app>
- LinkedIn: <https://www.linkedin.com/in/muhammad-sami-->
- GitHub: [@muhammadsami987123](https://github.com/muhammadsami987123)

---

## 📄 License

[Apache License 2.0](LICENSE) © 2026 Muhammad Sami

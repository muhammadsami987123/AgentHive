# AgentHive — Multi-Agent Business Automation Platform
## Full Project Plan for Claude Code

> **Stack:** Next.js 15 · TypeScript · Tailwind CSS · FastAPI · Python · Neon (PostgreSQL)
> **Goal:** Build a SaaS platform where specialized AI agents collaborate to automate full business workflows.

---

## Project Structure

```
agenthive/
├── frontend/                  # Next.js 15 + TypeScript + Tailwind
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   ├── agents/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [agentId]/
│   │   │   │       └── page.tsx
│   │   │   ├── workflows/
│   │   │   │   └── page.tsx
│   │   │   ├── analytics/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   ├── layout.tsx
│   │   └── page.tsx            # Landing page
│   ├── components/
│   │   ├── ui/                 # Reusable UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── Modal.tsx
│   │   │   └── Sidebar.tsx
│   │   ├── agents/
│   │   │   ├── AgentCard.tsx
│   │   │   ├── AgentStatusBadge.tsx
│   │   │   └── AgentLogStream.tsx
│   │   ├── dashboard/
│   │   │   ├── StatCard.tsx
│   │   │   ├── WorkflowTimeline.tsx
│   │   │   └── ActivityFeed.tsx
│   │   └── landing/
│   │       ├── HeroSection.tsx
│   │       ├── FeaturesSection.tsx
│   │       └── PricingSection.tsx
│   ├── lib/
│   │   ├── api.ts              # API client (fetch wrapper)
│   │   ├── auth.ts             # Auth helpers
│   │   └── types.ts            # Shared TypeScript types
│   ├── hooks/
│   │   ├── useAgents.ts
│   │   ├── useWorkflows.ts
│   │   └── useWebSocket.ts
│   ├── public/
│   ├── tailwind.config.ts
│   ├── next.config.ts
│   └── package.json
│
├── backend/                   # FastAPI + Python
│   ├── app/
│   │   ├── main.py             # FastAPI app entry point
│   │   ├── config.py           # Settings / env vars
│   │   ├── database.py         # Neon DB connection (asyncpg)
│   │   ├── models/
│   │   │   ├── user.py
│   │   │   ├── agent.py
│   │   │   ├── workflow.py
│   │   │   ├── run.py
│   │   │   └── log.py
│   │   ├── routers/
│   │   │   ├── auth.py
│   │   │   ├── agents.py
│   │   │   ├── workflows.py
│   │   │   ├── runs.py
│   │   │   └── analytics.py
│   │   ├── agents/             # AI Agent logic
│   │   │   ├── base_agent.py
│   │   │   ├── scout_agent.py
│   │   │   ├── outreach_agent.py
│   │   │   ├── content_agent.py
│   │   │   ├── analyst_agent.py
│   │   │   ├── finance_agent.py
│   │   │   └── support_agent.py
│   │   ├── crew/
│   │   │   ├── crew_manager.py  # CrewAI orchestration
│   │   │   └── tasks.py
│   │   ├── services/
│   │   │   ├── email_service.py
│   │   │   ├── stripe_service.py
│   │   │   └── websocket_service.py
│   │   └── middleware/
│   │       ├── auth_middleware.py
│   │       └── rate_limit.py
│   ├── migrations/             # SQL migration files
│   │   └── 001_initial.sql
│   ├── requirements.txt
│   └── .env.example
│
└── README.md
```

---

## Database Schema (Neon PostgreSQL)

Create these tables in Neon. Run the SQL below to initialize.

```sql
-- migrations/001_initial.sql

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Users
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  full_name VARCHAR(255),
  plan VARCHAR(50) DEFAULT 'free',         -- free | starter | pro | agency
  stripe_customer_id VARCHAR(255),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agents
CREATE TABLE agents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(100) NOT NULL,              -- scout | outreach | content | analyst | finance | support
  type VARCHAR(100) NOT NULL,
  status VARCHAR(50) DEFAULT 'idle',       -- idle | running | error | paused
  config JSONB DEFAULT '{}',              -- agent-specific config (API keys, prompts, schedules)
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Workflows
CREATE TABLE workflows (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  steps JSONB NOT NULL,                   -- array of agent steps with order
  trigger_type VARCHAR(50),               -- manual | schedule | webhook
  trigger_config JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Workflow Runs
CREATE TABLE runs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  workflow_id UUID REFERENCES workflows(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  status VARCHAR(50) DEFAULT 'pending',   -- pending | running | completed | failed
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  result JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Agent Logs (real-time log streaming)
CREATE TABLE agent_logs (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  run_id UUID REFERENCES runs(id) ON DELETE CASCADE,
  agent_name VARCHAR(100) NOT NULL,
  level VARCHAR(20) DEFAULT 'info',       -- info | warning | error | success
  message TEXT NOT NULL,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Leads (Scout agent stores leads here)
CREATE TABLE leads (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name VARCHAR(255),
  email VARCHAR(255),
  company VARCHAR(255),
  linkedin_url TEXT,
  score INTEGER DEFAULT 0,               -- 0-100 fit score
  status VARCHAR(50) DEFAULT 'new',      -- new | contacted | replied | converted | ignored
  notes TEXT,
  source VARCHAR(100),
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Content Queue (Content agent posts)
CREATE TABLE content_queue (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  platform VARCHAR(50) NOT NULL,         -- twitter | linkedin | blog
  content TEXT NOT NULL,
  status VARCHAR(50) DEFAULT 'draft',    -- draft | scheduled | published | failed
  scheduled_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Invoices (Finance agent)
CREATE TABLE invoices (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  client_name VARCHAR(255) NOT NULL,
  client_email VARCHAR(255) NOT NULL,
  amount DECIMAL(10, 2) NOT NULL,
  currency VARCHAR(10) DEFAULT 'USD',
  status VARCHAR(50) DEFAULT 'draft',    -- draft | sent | paid | overdue
  due_date DATE,
  paid_at TIMESTAMPTZ,
  stripe_invoice_id VARCHAR(255),
  line_items JSONB DEFAULT '[]',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_agents_user_id ON agents(user_id);
CREATE INDEX idx_workflows_user_id ON workflows(user_id);
CREATE INDEX idx_runs_workflow_id ON runs(workflow_id);
CREATE INDEX idx_agent_logs_run_id ON agent_logs(run_id);
CREATE INDEX idx_leads_user_id ON leads(user_id);
CREATE INDEX idx_leads_status ON leads(status);
CREATE INDEX idx_invoices_user_id ON invoices(user_id);
```

---

## Backend — FastAPI Setup

### `backend/requirements.txt`
```
fastapi==0.115.0
uvicorn[standard]==0.30.0
asyncpg==0.29.0
sqlalchemy[asyncio]==2.0.36
alembic==1.13.2
pydantic==2.9.2
pydantic-settings==2.6.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.12
httpx==0.27.0
openai==1.54.0
anthropic==0.39.0
crewai==0.80.0
langchain==0.3.7
langchain-anthropic==0.3.0
stripe==11.1.0
python-dotenv==1.0.1
websockets==13.1
apscheduler==3.10.4
```

### `backend/.env.example`
```env
# Database
DATABASE_URL=postgresql+asyncpg://user:password@host/dbname?sslmode=require

# Auth
SECRET_KEY=your-super-secret-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=10080

# AI
ANTHROPIC_API_KEY=your-anthropic-key
OPENAI_API_KEY=your-openai-key

# Stripe
STRIPE_SECRET_KEY=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-webhook-secret

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASSWORD=your-app-password

# App
FRONTEND_URL=http://localhost:3000
ENVIRONMENT=development
```

### `backend/app/main.py`
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import auth, agents, workflows, runs, analytics
from app.config import settings

app = FastAPI(
    title="AgentHive API",
    description="Multi-Agent Business Automation Platform",
    version="1.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=[settings.FRONTEND_URL],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

app.include_router(auth.router, prefix="/api/auth", tags=["auth"])
app.include_router(agents.router, prefix="/api/agents", tags=["agents"])
app.include_router(workflows.router, prefix="/api/workflows", tags=["workflows"])
app.include_router(runs.router, prefix="/api/runs", tags=["runs"])
app.include_router(analytics.router, prefix="/api/analytics", tags=["analytics"])

@app.get("/health")
async def health_check():
    return {"status": "ok", "service": "AgentHive API"}
```

### `backend/app/config.py`
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 10080
    ANTHROPIC_API_KEY: str
    OPENAI_API_KEY: str = ""
    STRIPE_SECRET_KEY: str
    STRIPE_WEBHOOK_SECRET: str
    SMTP_HOST: str = ""
    SMTP_PORT: int = 587
    SMTP_USER: str = ""
    SMTP_PASSWORD: str = ""
    FRONTEND_URL: str = "http://localhost:3000"
    ENVIRONMENT: str = "development"

    class Config:
        env_file = ".env"

settings = Settings()
```

### `backend/app/database.py`
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import declarative_base, sessionmaker
from app.config import settings

engine = create_async_engine(settings.DATABASE_URL, echo=False)

AsyncSessionLocal = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

Base = declarative_base()

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

### `backend/app/agents/base_agent.py`
```python
from anthropic import Anthropic
from app.config import settings
import json

class BaseAgent:
    def __init__(self, name: str, system_prompt: str):
        self.name = name
        self.system_prompt = system_prompt
        self.client = Anthropic(api_key=settings.ANTHROPIC_API_KEY)

    async def run(self, task: str, context: dict = {}) -> dict:
        raise NotImplementedError("Each agent must implement run()")

    def _call_claude(self, user_message: str, max_tokens: int = 2048) -> str:
        response = self.client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=max_tokens,
            system=self.system_prompt,
            messages=[{"role": "user", "content": user_message}]
        )
        return response.content[0].text
```

### `backend/app/agents/scout_agent.py`
```python
from app.agents.base_agent import BaseAgent
import json

class ScoutAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="Scout",
            system_prompt="""You are Scout, a lead generation specialist AI agent.
Your job is to analyze target company profiles, identify ideal leads,
score them 0-100 based on fit, and return structured JSON data.
Always respond with valid JSON only. No markdown, no explanation."""
        )

    async def run(self, task: str, context: dict = {}) -> dict:
        prompt = f"""
Task: {task}
Target criteria: {json.dumps(context.get('criteria', {}))}

Return a JSON array of leads with this structure:
[{{
  "name": "Full Name",
  "email": "email@company.com",
  "company": "Company Name",
  "linkedin_url": "https://linkedin.com/in/...",
  "score": 85,
  "notes": "Why this lead is a good fit"
}}]
"""
        result = self._call_claude(prompt)
        try:
            leads = json.loads(result)
            return {"status": "success", "leads": leads, "count": len(leads)}
        except json.JSONDecodeError:
            return {"status": "error", "message": "Failed to parse leads", "raw": result}
```

### `backend/app/agents/outreach_agent.py`
```python
from app.agents.base_agent import BaseAgent
import json

class OutreachAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="Outreach",
            system_prompt="""You are Outreach, an expert cold email copywriter AI.
You write highly personalized, concise, non-salesy cold emails that get replies.
You always respond with valid JSON only."""
        )

    async def run(self, task: str, context: dict = {}) -> dict:
        lead = context.get("lead", {})
        sender = context.get("sender", {})
        prompt = f"""
Write a cold email for this lead:
Lead: {json.dumps(lead)}
Sender: {json.dumps(sender)}
Goal: {task}

Return JSON:
{{
  "subject": "Email subject line",
  "body": "Full email body",
  "follow_up_1": "Follow-up email after 3 days",
  "follow_up_2": "Follow-up email after 7 days"
}}
"""
        result = self._call_claude(prompt)
        try:
            email = json.loads(result)
            return {"status": "success", "email": email}
        except json.JSONDecodeError:
            return {"status": "error", "message": "Failed to generate email"}
```

### `backend/app/agents/content_agent.py`
```python
from app.agents.base_agent import BaseAgent
import json

class ContentAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="Content",
            system_prompt="""You are Content, a social media and blog content specialist AI.
You write engaging, platform-optimized content for LinkedIn, Twitter, and blogs.
Always respond with valid JSON only."""
        )

    async def run(self, task: str, context: dict = {}) -> dict:
        platform = context.get("platform", "linkedin")
        topic = context.get("topic", task)
        brand_voice = context.get("brand_voice", "professional but approachable")

        prompt = f"""
Create {platform} content about: {topic}
Brand voice: {brand_voice}
Platform: {platform}

Return JSON:
{{
  "content": "The post content",
  "hashtags": ["tag1", "tag2"],
  "best_post_time": "Tuesday 9AM"
}}
"""
        result = self._call_claude(prompt)
        try:
            content = json.loads(result)
            return {"status": "success", "content": content, "platform": platform}
        except json.JSONDecodeError:
            return {"status": "error", "message": "Failed to generate content"}
```

### `backend/app/crew/crew_manager.py`
```python
from app.agents.scout_agent import ScoutAgent
from app.agents.outreach_agent import OutreachAgent
from app.agents.content_agent import ContentAgent
from app.agents.analyst_agent import AnalystAgent
from app.agents.finance_agent import FinanceAgent
from app.agents.support_agent import SupportAgent
import asyncio

AGENT_REGISTRY = {
    "scout": ScoutAgent,
    "outreach": OutreachAgent,
    "content": ContentAgent,
    "analyst": AnalystAgent,
    "finance": FinanceAgent,
    "support": SupportAgent,
}

class CrewManager:
    def __init__(self):
        self.agents = {name: cls() for name, cls in AGENT_REGISTRY.items()}

    async def run_workflow(self, workflow: dict, context: dict = {}, log_callback=None) -> dict:
        steps = workflow.get("steps", [])
        results = {}
        current_context = context.copy()

        for step in steps:
            agent_name = step.get("agent")
            task = step.get("task")

            if agent_name not in self.agents:
                continue

            if log_callback:
                await log_callback(agent_name, "info", f"Starting: {task}")

            agent = self.agents[agent_name]
            result = await agent.run(task, current_context)
            results[agent_name] = result

            # Pass result as context to next agent
            current_context[f"{agent_name}_result"] = result

            if log_callback:
                status = result.get("status", "unknown")
                await log_callback(agent_name, "success" if status == "success" else "error",
                                   f"Completed with status: {status}")

        return results

crew_manager = CrewManager()
```

### `backend/app/routers/agents.py`
```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.middleware.auth_middleware import get_current_user
from pydantic import BaseModel
from typing import Optional
import uuid

router = APIRouter()

class AgentCreate(BaseModel):
    name: str
    type: str
    config: Optional[dict] = {}

class AgentUpdate(BaseModel):
    status: Optional[str] = None
    config: Optional[dict] = None
    is_active: Optional[bool] = None

@router.get("/")
async def list_agents(db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    result = await db.execute(
        "SELECT * FROM agents WHERE user_id = :uid ORDER BY created_at DESC",
        {"uid": user["id"]}
    )
    return result.fetchall()

@router.post("/")
async def create_agent(body: AgentCreate, db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    agent_id = str(uuid.uuid4())
    await db.execute(
        """INSERT INTO agents (id, user_id, name, type, config)
           VALUES (:id, :uid, :name, :type, :config)""",
        {"id": agent_id, "uid": user["id"], "name": body.name,
         "type": body.type, "config": body.config}
    )
    return {"id": agent_id, "message": "Agent created"}

@router.patch("/{agent_id}")
async def update_agent(agent_id: str, body: AgentUpdate, db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    updates = {k: v for k, v in body.dict().items() if v is not None}
    if not updates:
        raise HTTPException(400, "No fields to update")
    set_clause = ", ".join([f"{k} = :{k}" for k in updates])
    updates["agent_id"] = agent_id
    updates["uid"] = user["id"]
    await db.execute(
        f"UPDATE agents SET {set_clause}, updated_at = NOW() WHERE id = :agent_id AND user_id = :uid",
        updates
    )
    return {"message": "Agent updated"}
```

### `backend/app/routers/workflows.py`
```python
from fastapi import APIRouter, Depends, BackgroundTasks
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.middleware.auth_middleware import get_current_user
from app.crew.crew_manager import crew_manager
from pydantic import BaseModel
from typing import Optional, List
import uuid

router = APIRouter()

class WorkflowStep(BaseModel):
    agent: str
    task: str
    order: int

class WorkflowCreate(BaseModel):
    name: str
    description: Optional[str] = None
    steps: List[WorkflowStep]
    trigger_type: Optional[str] = "manual"
    trigger_config: Optional[dict] = {}

@router.get("/")
async def list_workflows(db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    result = await db.execute(
        "SELECT * FROM workflows WHERE user_id = :uid ORDER BY created_at DESC",
        {"uid": user["id"]}
    )
    return result.fetchall()

@router.post("/")
async def create_workflow(body: WorkflowCreate, db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    wf_id = str(uuid.uuid4())
    steps = sorted([s.dict() for s in body.steps], key=lambda x: x["order"])
    await db.execute(
        """INSERT INTO workflows (id, user_id, name, description, steps, trigger_type, trigger_config)
           VALUES (:id, :uid, :name, :desc, :steps, :trigger_type, :trigger_config)""",
        {"id": wf_id, "uid": user["id"], "name": body.name, "desc": body.description,
         "steps": steps, "trigger_type": body.trigger_type, "trigger_config": body.trigger_config}
    )
    return {"id": wf_id, "message": "Workflow created"}

@router.post("/{workflow_id}/run")
async def trigger_workflow(workflow_id: str, background_tasks: BackgroundTasks,
                            db: AsyncSession = Depends(get_db), user=Depends(get_current_user)):
    run_id = str(uuid.uuid4())
    await db.execute(
        """INSERT INTO runs (id, workflow_id, user_id, status, started_at)
           VALUES (:id, :wf_id, :uid, 'running', NOW())""",
        {"id": run_id, "wf_id": workflow_id, "uid": user["id"]}
    )

    async def run_in_bg():
        workflow = {"steps": []}  # load from DB
        result = await crew_manager.run_workflow(workflow)
        # update run status in DB

    background_tasks.add_task(run_in_bg)
    return {"run_id": run_id, "status": "started"}
```

---

## Frontend — Next.js 15 Setup

### `frontend/package.json`
```json
{
  "name": "agenthive-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "15.0.0",
    "react": "19.0.0",
    "react-dom": "19.0.0",
    "typescript": "5.6.0",
    "@types/react": "19.0.0",
    "@types/node": "22.0.0",
    "tailwindcss": "3.4.0",
    "autoprefixer": "10.4.0",
    "postcss": "8.4.0",
    "axios": "1.7.0",
    "zustand": "5.0.0",
    "react-query": "3.39.0",
    "recharts": "2.13.0",
    "lucide-react": "0.460.0",
    "clsx": "2.1.0",
    "jose": "5.9.0"
  }
}
```

### `frontend/tailwind.config.ts`
```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#EEEDFE',
          100: '#CECBF6',
          200: '#AFA9EC',
          400: '#7F77DD',
          600: '#534AB7',
          800: '#3C3489',
          900: '#26215C',
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      }
    }
  },
  plugins: []
}

export default config
```

### `frontend/lib/types.ts`
```typescript
export type AgentName = 'scout' | 'outreach' | 'content' | 'analyst' | 'finance' | 'support'
export type AgentStatus = 'idle' | 'running' | 'error' | 'paused'
export type Plan = 'free' | 'starter' | 'pro' | 'agency'

export interface User {
  id: string
  email: string
  full_name: string
  plan: Plan
  created_at: string
}

export interface Agent {
  id: string
  user_id: string
  name: AgentName
  type: string
  status: AgentStatus
  config: Record<string, any>
  is_active: boolean
  created_at: string
  updated_at: string
}

export interface WorkflowStep {
  agent: AgentName
  task: string
  order: number
}

export interface Workflow {
  id: string
  user_id: string
  name: string
  description?: string
  steps: WorkflowStep[]
  trigger_type: 'manual' | 'schedule' | 'webhook'
  trigger_config: Record<string, any>
  is_active: boolean
  created_at: string
}

export interface Run {
  id: string
  workflow_id: string
  status: 'pending' | 'running' | 'completed' | 'failed'
  started_at?: string
  completed_at?: string
  result: Record<string, any>
}

export interface AgentLog {
  id: string
  run_id: string
  agent_name: AgentName
  level: 'info' | 'warning' | 'error' | 'success'
  message: string
  created_at: string
}

export interface Lead {
  id: string
  name: string
  email: string
  company: string
  linkedin_url?: string
  score: number
  status: 'new' | 'contacted' | 'replied' | 'converted' | 'ignored'
  notes?: string
  created_at: string
}
```

### `frontend/lib/api.ts`
```typescript
import axios from 'axios'

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000',
  headers: { 'Content-Type': 'application/json' }
})

api.interceptors.request.use((config) => {
  const token = typeof window !== 'undefined'
    ? localStorage.getItem('agenthive_token')
    : null
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

api.interceptors.response.use(
  (res) => res,
  (err) => {
    if (err.response?.status === 401 && typeof window !== 'undefined') {
      localStorage.removeItem('agenthive_token')
      window.location.href = '/login'
    }
    return Promise.reject(err)
  }
)

// Auth
export const authApi = {
  login: (email: string, password: string) =>
    api.post('/api/auth/login', { email, password }),
  register: (email: string, password: string, full_name: string) =>
    api.post('/api/auth/register', { email, password, full_name }),
  me: () => api.get('/api/auth/me'),
}

// Agents
export const agentsApi = {
  list: () => api.get('/api/agents'),
  create: (data: any) => api.post('/api/agents', data),
  update: (id: string, data: any) => api.patch(`/api/agents/${id}`, data),
  delete: (id: string) => api.delete(`/api/agents/${id}`),
}

// Workflows
export const workflowsApi = {
  list: () => api.get('/api/workflows'),
  create: (data: any) => api.post('/api/workflows', data),
  trigger: (id: string) => api.post(`/api/workflows/${id}/run`),
}

// Analytics
export const analyticsApi = {
  overview: () => api.get('/api/analytics/overview'),
  agentStats: () => api.get('/api/analytics/agents'),
}

export default api
```

### `frontend/app/layout.tsx`
```typescript
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'AgentHive — AI Agent Automation',
  description: 'Multi-agent business automation platform',
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  )
}
```

### `frontend/app/(dashboard)/dashboard/page.tsx`
```typescript
'use client'
import { useEffect, useState } from 'react'
import { analyticsApi, agentsApi } from '@/lib/api'
import { Agent } from '@/lib/types'
import { Activity, Bot, Workflow, TrendingUp } from 'lucide-react'

const AGENT_ICONS: Record<string, string> = {
  scout: '🔭', outreach: '📧', content: '✍️',
  analyst: '📊', finance: '💳', support: '🎧'
}

export default function DashboardPage() {
  const [agents, setAgents] = useState<Agent[]>([])
  const [stats, setStats] = useState({ runs: 0, leads: 0, emails: 0, revenue: 0 })
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function load() {
      try {
        const [agentsRes, statsRes] = await Promise.all([
          agentsApi.list(),
          analyticsApi.overview()
        ])
        setAgents(agentsRes.data)
        setStats(statsRes.data)
      } catch (e) {
        console.error(e)
      } finally {
        setLoading(false)
      }
    }
    load()
  }, [])

  if (loading) return (
    <div className="flex items-center justify-center h-64">
      <div className="w-8 h-8 border-2 border-brand-600 border-t-transparent rounded-full animate-spin" />
    </div>
  )

  const statCards = [
    { label: 'Total Runs', value: stats.runs, icon: Activity, color: 'text-brand-600' },
    { label: 'Leads Found', value: stats.leads, icon: Bot, color: 'text-green-600' },
    { label: 'Emails Sent', value: stats.emails, icon: Workflow, color: 'text-blue-600' },
    { label: 'Revenue Saved', value: `$${stats.revenue}`, icon: TrendingUp, color: 'text-amber-600' },
  ]

  return (
    <div className="p-6 max-w-6xl mx-auto">
      <h1 className="text-2xl font-medium text-gray-900 mb-6">Dashboard</h1>

      {/* Stats */}
      <div className="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
        {statCards.map((s) => (
          <div key={s.label} className="bg-gray-50 rounded-xl p-4">
            <p className="text-xs text-gray-500 mb-1">{s.label}</p>
            <p className="text-2xl font-medium text-gray-900">{s.value}</p>
          </div>
        ))}
      </div>

      {/* Agents */}
      <h2 className="text-lg font-medium text-gray-900 mb-4">Your Agents</h2>
      <div className="grid grid-cols-2 lg:grid-cols-3 gap-4">
        {agents.map((agent) => (
          <div key={agent.id} className="bg-white border border-gray-100 rounded-xl p-4">
            <div className="flex items-center gap-3 mb-3">
              <div className="w-9 h-9 bg-brand-50 rounded-lg flex items-center justify-center text-lg">
                {AGENT_ICONS[agent.name] || '🤖'}
              </div>
              <div>
                <p className="font-medium text-gray-900 capitalize">{agent.name}</p>
                <span className={`text-xs px-2 py-0.5 rounded-full ${
                  agent.status === 'running'
                    ? 'bg-green-100 text-green-700'
                    : 'bg-gray-100 text-gray-500'
                }`}>
                  {agent.status}
                </span>
              </div>
            </div>
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## Pages to Build

| Route | Page | Description |
|---|---|---|
| `/` | Landing | Hero, features, pricing, CTA |
| `/login` | Login | Email + password form |
| `/register` | Register | Sign up form |
| `/dashboard` | Dashboard | Stats overview + agent status |
| `/agents` | Agents | List all agents, toggle on/off |
| `/agents/[id]` | Agent Detail | Config, logs, run history |
| `/workflows` | Workflows | Create and manage workflows |
| `/analytics` | Analytics | Charts: leads, emails, runs |
| `/settings` | Settings | Profile, API keys, billing |

---

## Pricing Tiers

| Plan | Price | Agents | Workflows | Runs/month |
|---|---|---|---|---|
| Free | $0 | 2 (Scout + Outreach) | 1 | 50 |
| Starter | $29/mo | 4 agents | 5 | 500 |
| Pro | $79/mo | All 6 agents | Unlimited | 5,000 |
| Agency | $199/mo | All 6 + custom | Unlimited | Unlimited |

---

## Development Order (Recommended)

### Week 1 — Foundation
1. Set up Neon database, run `001_initial.sql`
2. Build FastAPI with auth (register/login/JWT)
3. Set up Next.js with Tailwind, routing, auth pages
4. Connect frontend to backend auth

### Week 2 — Core Features
5. Build agent CRUD endpoints + frontend agent cards
6. Implement Scout + Outreach agents with Claude API
7. Build workflow creator UI + backend
8. Add workflow run endpoint with background tasks

### Week 3 — Dashboard + Logs
9. Build real-time log streaming (WebSocket)
10. Build analytics dashboard with charts
11. Add Content + Finance agents
12. Add Analyst + Support agents

### Week 4 — Polish + Launch
13. Integrate Stripe billing
14. Build landing page
15. Deploy backend to Railway, frontend to Vercel
16. Set up Neon production database

---

## Environment Variables (Frontend)

Create `frontend/.env.local`:
```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_STRIPE_PUBLIC_KEY=your-stripe-public-key
```

---

## Deployment

| Service | Platform | Notes |
|---|---|---|
| Frontend | Vercel | Auto-deploy from GitHub main branch |
| Backend | Railway | Dockerfile or Nixpacks auto-detect |
| Database | Neon | Serverless PostgreSQL, free tier available |
| File Storage | Cloudflare R2 | For invoice PDFs, content assets |

### Railway `backend/Dockerfile`
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Quick Start Commands

```bash
# Backend
cd backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env             # Fill in your keys
uvicorn app.main:app --reload --port 8000

# Frontend
cd frontend
npm install
cp .env.example .env.local       # Fill in your keys
npm run dev
```

---

## Key Notes for Claude Code

- All agent classes extend `BaseAgent` — add new agents by creating a new file in `backend/app/agents/` and registering in `crew_manager.py`
- Neon uses standard PostgreSQL — use `asyncpg` driver with the connection string from Neon dashboard
- WebSocket endpoint for real-time logs should be at `ws://localhost:8000/api/ws/runs/{run_id}`
- All Claude API calls use model `claude-sonnet-4-20250514`
- Frontend state management: use Zustand for auth/user store, React Query for server data
- Use `clsx` for conditional Tailwind classes throughout components

---

*Built by Muhammad Sami — AgentHive v1.0*
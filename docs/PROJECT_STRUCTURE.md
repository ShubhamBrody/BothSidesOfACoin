# Project Structure & Setup Guide
## BothSidesOfACoin — Full Stack Application

---

## 1. Monorepo Structure

```
BothSidesOfACoin/
│
├── docs/                              # Documentation (you are here)
│   ├── PRD.md                         # Product Requirements Document
│   ├── ARCHITECTURE_DIAGRAMS.md       # System architecture & flow diagrams
│   ├── DTOs.md                        # Data Transfer Object specifications
│   ├── API_SPECIFICATION.md           # Full REST + WebSocket API spec
│   ├── AUTOGEN_AGENT_PLAN.md          # AI agent orchestration design
│   ├── UI_UX_SPECIFICATION.md         # Design system & page specs
│   └── PROJECT_STRUCTURE.md           # This file
│
├── backend/                           # Python FastAPI Backend
│   ├── alembic/                       # Database migrations
│   │   ├── versions/                  # Migration scripts
│   │   ├── env.py
│   │   └── alembic.ini
│   │
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI app entry point
│   │   ├── config.py                  # Settings (Pydantic BaseSettings)
│   │   │
│   │   ├── api/                       # API layer
│   │   │   ├── __init__.py
│   │   │   ├── deps.py                # Dependency injection (get_db, get_current_user)
│   │   │   ├── middleware.py           # CORS, rate limiting, request ID
│   │   │   │
│   │   │   └── v1/                    # API version 1
│   │   │       ├── __init__.py
│   │   │       ├── router.py          # Root v1 router (includes all sub-routers)
│   │   │       ├── auth.py            # /auth/* endpoints
│   │   │       ├── topics.py          # /topics/* endpoints
│   │   │       ├── articles.py        # /articles/* endpoints
│   │   │       ├── debate.py          # /debate/* endpoints
│   │   │       ├── bias.py            # /user/bias-score, reading-event
│   │   │       ├── users.py           # /user/* profile endpoints
│   │   │       ├── subscription.py    # /subscription/* endpoints
│   │   │       ├── admin.py           # /admin/* endpoints
│   │   │       ├── tasks.py           # /tasks/* (async task polling)
│   │   │       └── health.py          # /health endpoint
│   │   │
│   │   ├── core/                      # Core business logic
│   │   │   ├── __init__.py
│   │   │   ├── security.py            # JWT, password hashing, OAuth
│   │   │   ├── permissions.py         # RBAC permission checks
│   │   │   ├── rate_limiter.py        # Redis-based rate limiting
│   │   │   └── exceptions.py          # Custom exception classes
│   │   │
│   │   ├── models/                    # SQLAlchemy ORM models
│   │   │   ├── __init__.py
│   │   │   ├── base.py                # Base model class (id, timestamps)
│   │   │   ├── user.py                # User, Subscription models
│   │   │   ├── topic.py               # Topic model
│   │   │   ├── article.py             # Article, NewsSource models
│   │   │   ├── perspective.py         # Perspective model
│   │   │   ├── debate.py              # DebateSession, DebateMessage models
│   │   │   ├── bias.py                # ReadingEvent, BiasScore models
│   │   │   ├── timeline.py            # TimelineEvent model
│   │   │   └── bookmark.py            # Bookmark model
│   │   │
│   │   ├── schemas/                   # Pydantic DTOs (request/response)
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── topics.py
│   │   │   ├── articles.py
│   │   │   ├── debate.py
│   │   │   ├── bias.py
│   │   │   ├── users.py
│   │   │   ├── subscription.py
│   │   │   ├── common.py              # PaginatedResponse, ErrorResponse, enums
│   │   │   └── admin.py
│   │   │
│   │   ├── services/                  # Business logic services
│   │   │   ├── __init__.py
│   │   │   ├── auth_service.py        # Login, register, OAuth, token management
│   │   │   ├── topic_service.py       # Topic CRUD + analysis orchestration
│   │   │   ├── article_service.py     # Article fetching + storage
│   │   │   ├── debate_service.py      # Debate session management
│   │   │   ├── bias_service.py        # Bias computation + tracking
│   │   │   ├── subscription_service.py # Stripe integration
│   │   │   ├── news_ingestion_service.py # News API polling
│   │   │   └── email_service.py       # Email notifications
│   │   │
│   │   ├── agents/                    # AutoGen 0.4 AI Agents
│   │   │   ├── __init__.py
│   │   │   ├── config.py              # Agent config, Ollama client setup
│   │   │   ├── orchestrator.py        # OrchestratorAgent + SelectorGroupChat
│   │   │   ├── news_collector.py      # NewsCollectorAgent + tools
│   │   │   ├── bias_classifier.py     # BiasClassifierAgent + tools
│   │   │   ├── summarizer.py          # SummarizerAgent + tools
│   │   │   ├── fact_extractor.py      # FactExtractorAgent + tools
│   │   │   ├── timeline_builder.py    # TimelineBuilderAgent + tools
│   │   │   ├── impact_analyzer.py     # ImpactAnalyzerAgent + tools
│   │   │   ├── debate_agents.py       # DebateLeft/Right/Neutral agents
│   │   │   ├── user_bias_agent.py     # UserBiasAnalyzerAgent + tools
│   │   │   ├── quality_guard.py       # QualityGuardAgent + tools
│   │   │   └── tools/                 # Shared agent tools
│   │   │       ├── __init__.py
│   │   │       ├── news_tools.py      # News API integration tools
│   │   │       ├── analysis_tools.py  # Text analysis tools
│   │   │       └── db_tools.py        # Database lookup tools
│   │   │
│   │   ├── db/                        # Database connection
│   │   │   ├── __init__.py
│   │   │   ├── session.py             # Async SQLAlchemy session factory
│   │   │   └── redis.py               # Redis connection manager
│   │   │
│   │   └── workers/                   # Celery background tasks
│   │       ├── __init__.py
│   │       ├── celery_app.py          # Celery configuration
│   │       ├── analysis_tasks.py      # AI analysis background tasks
│   │       ├── ingestion_tasks.py     # Periodic news ingestion
│   │       └── notification_tasks.py  # Email/push notification tasks
│   │
│   ├── tests/                         # Backend tests
│   │   ├── conftest.py                # Pytest fixtures
│   │   ├── test_auth.py
│   │   ├── test_topics.py
│   │   ├── test_debate.py
│   │   ├── test_bias.py
│   │   ├── test_agents/
│   │   │   ├── test_orchestrator.py
│   │   │   ├── test_bias_classifier.py
│   │   │   └── test_summarizer.py
│   │   └── test_services/
│   │       ├── test_auth_service.py
│   │       └── test_bias_service.py
│   │
│   ├── pyproject.toml                 # Python project config
│   ├── requirements.txt               # Pinned dependencies
│   ├── requirements-dev.txt           # Dev dependencies (pytest, ruff, etc.)
│   ├── Dockerfile                     # Backend Docker image
│   └── .env.example                   # Environment variable template
│
├── frontend/                          # Next.js 14 Frontend
│   ├── public/                        # Static assets
│   │   ├── favicon.ico
│   │   ├── logo.svg
│   │   └── images/
│   │
│   ├── src/
│   │   ├── app/                      # Next.js App Router
│   │   │   ├── layout.tsx             # Root layout (providers, theme)
│   │   │   ├── page.tsx               # Landing page
│   │   │   ├── globals.css            # Global styles (Tailwind)
│   │   │   │
│   │   │   ├── (auth)/               # Auth route group
│   │   │   │   ├── login/page.tsx
│   │   │   │   ├── register/page.tsx
│   │   │   │   ├── forgot-password/page.tsx
│   │   │   │   ├── reset-password/page.tsx
│   │   │   │   └── verify-email/page.tsx
│   │   │   │
│   │   │   ├── (dashboard)/           # Authenticated route group
│   │   │   │   ├── layout.tsx         # Dashboard layout (sidebar + header)
│   │   │   │   ├── dashboard/page.tsx # Main dashboard
│   │   │   │   ├── topics/
│   │   │   │   │   ├── page.tsx       # Topic listing
│   │   │   │   │   └── [slug]/
│   │   │   │   │       ├── page.tsx   # 3-Side View
│   │   │   │   │       ├── timeline/page.tsx
│   │   │   │   │       └── impact/page.tsx
│   │   │   │   │
│   │   │   │   ├── debate/
│   │   │   │   │   ├── page.tsx       # Debate sessions list
│   │   │   │   │   └── [id]/page.tsx  # Active debate
│   │   │   │   │
│   │   │   │   ├── bias/page.tsx      # Bias score dashboard
│   │   │   │   ├── search/page.tsx    # Search results
│   │   │   │   │
│   │   │   │   ├── profile/
│   │   │   │   │   ├── page.tsx       # User profile
│   │   │   │   │   ├── settings/page.tsx
│   │   │   │   │   └── history/page.tsx
│   │   │   │   │
│   │   │   │   └── subscription/
│   │   │   │       ├── page.tsx       # Plans & billing
│   │   │   │       ├── success/page.tsx
│   │   │   │       └── cancel/page.tsx
│   │   │   │
│   │   │   ├── (admin)/               # Admin route group
│   │   │   │   └── admin/
│   │   │   │       ├── layout.tsx
│   │   │   │       ├── page.tsx       # Admin dashboard
│   │   │   │       ├── users/page.tsx
│   │   │   │       └── sources/page.tsx
│   │   │   │
│   │   │   └── api/                   # Next.js API routes (BFF)
│   │   │       └── auth/
│   │   │           └── [...nextauth]/route.ts  # OAuth handling (if using NextAuth)
│   │   │
│   │   ├── components/                # Reusable components
│   │   │   ├── ui/                    # shadcn/ui components
│   │   │   │   ├── button.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── dialog.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── badge.tsx
│   │   │   │   ├── tabs.tsx
│   │   │   │   ├── skeleton.tsx
│   │   │   │   ├── toast.tsx
│   │   │   │   └── sheet.tsx
│   │   │   │
│   │   │   ├── layout/
│   │   │   │   ├── navbar.tsx         # Top navigation bar
│   │   │   │   ├── sidebar.tsx        # Dashboard sidebar
│   │   │   │   ├── footer.tsx         # Landing page footer
│   │   │   │   └── mobile-nav.tsx     # Mobile hamburger menu
│   │   │   │
│   │   │   ├── features/
│   │   │   │   ├── three-side-view.tsx       # Main 3-column perspective layout
│   │   │   │   ├── perspective-column.tsx    # Single perspective card
│   │   │   │   ├── debate-chat.tsx           # Debate chat interface
│   │   │   │   ├── debate-message.tsx        # Single debate message bubble
│   │   │   │   ├── bias-gauge.tsx            # Animated circular gauge
│   │   │   │   ├── bias-distribution-chart.tsx # Donut chart
│   │   │   │   ├── bias-trend-chart.tsx      # Line chart over time
│   │   │   │   ├── topic-card.tsx            # Topic preview card
│   │   │   │   ├── article-card.tsx          # Article preview card
│   │   │   │   ├── timeline-view.tsx         # Vertical timeline
│   │   │   │   ├── impact-radar.tsx          # Radar chart
│   │   │   │   ├── typing-indicator.tsx      # AI typing animation
│   │   │   │   └── ai-disclaimer.tsx         # Standardized AI warning
│   │   │   │
│   │   │   └── common/
│   │   │       ├── loading-skeleton.tsx
│   │   │       ├── error-boundary.tsx
│   │   │       ├── pagination.tsx
│   │   │       └── search-bar.tsx
│   │   │
│   │   ├── hooks/                     # Custom React hooks
│   │   │   ├── use-auth.ts            # Auth state management
│   │   │   ├── use-bias-score.ts      # Bias score fetching
│   │   │   ├── use-debate.ts          # Debate WebSocket management
│   │   │   ├── use-websocket.ts       # Generic WebSocket hook
│   │   │   └── use-subscription.ts    # Subscription status
│   │   │
│   │   ├── lib/                       # Utilities
│   │   │   ├── api-client.ts          # Axios/fetch wrapper with auth
│   │   │   ├── auth.ts                # Auth helper functions
│   │   │   ├── constants.ts           # App-wide constants
│   │   │   ├── utils.ts               # General utilities (cn, formatDate)
│   │   │   └── validators.ts          # Zod schemas for form validation
│   │   │
│   │   ├── stores/                    # Zustand stores
│   │   │   ├── auth-store.ts          # Auth state
│   │   │   ├── theme-store.ts         # Dark/light mode
│   │   │   └── debate-store.ts        # Active debate state
│   │   │
│   │   └── types/                     # TypeScript type definitions
│   │       ├── api.ts                 # API response types
│   │       ├── auth.ts                # Auth types
│   │       ├── topics.ts              # Topic, Article, Perspective types
│   │       ├── debate.ts              # Debate types
│   │       ├── bias.ts                # Bias score types
│   │       └── subscription.ts        # Subscription types
│   │
│   ├── tailwind.config.ts             # Tailwind + custom theme
│   ├── next.config.js                 # Next.js configuration
│   ├── tsconfig.json                  # TypeScript config
│   ├── package.json                   # NPM dependencies
│   ├── Dockerfile                     # Frontend Docker image
│   └── .env.example                   # Environment variable template
│
├── docker-compose.yml                 # Full stack local development
├── docker-compose.prod.yml            # Production deployment
├── .github/
│   └── workflows/
│       ├── ci.yml                     # CI: lint, test, build
│       └── deploy.yml                 # CD: deploy to cloud
│
├── nginx/
│   └── nginx.conf                     # Reverse proxy configuration
│
├── scripts/
│   ├── seed_data.py                   # Seed database with demo data
│   ├── setup_ollama.sh                # Install & pull Ollama models
│   └── run_dev.sh                     # Start all services for development
│
├── .gitignore
├── .env.example                       # Root environment variables
├── LICENSE
└── README.md                          # Project overview & quick start
```

---

## 2. Docker Compose (Local Development)

```yaml
# docker-compose.yml
version: "3.9"

services:
  # ─── PostgreSQL ───
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: bothsides
      POSTGRES_USER: bothsides
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-dev_password_change_me}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U bothsides"]
      interval: 5s
      timeout: 5s
      retries: 5

  # ─── Redis ───
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s

  # ─── Ollama (AI) ───
  ollama:
    image: ollama/ollama:latest
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    # For GPU support, uncomment:
    # deploy:
    #   resources:
    #     reservations:
    #       devices:
    #         - driver: nvidia
    #           count: 1
    #           capabilities: [gpu]

  # ─── FastAPI Backend ───
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql+asyncpg://bothsides:${POSTGRES_PASSWORD:-dev_password_change_me}@postgres:5432/bothsides
      REDIS_URL: redis://redis:6379/0
      OLLAMA_BASE_URL: http://ollama:11434
      OLLAMA_MODEL: llama3.1:8b
      JWT_SECRET: ${JWT_SECRET:-change-this-in-production}
      NEWS_API_KEY: ${NEWS_API_KEY}
      STRIPE_SECRET_KEY: ${STRIPE_SECRET_KEY}
      STRIPE_WEBHOOK_SECRET: ${STRIPE_WEBHOOK_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      ollama:
        condition: service_started
    volumes:
      - ./backend:/app  # Hot reload in dev

  # ─── Celery Worker ───
  celery-worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: celery -A app.workers.celery_app worker --loglevel=info --concurrency=4
    environment:
      DATABASE_URL: postgresql+asyncpg://bothsides:${POSTGRES_PASSWORD:-dev_password_change_me}@postgres:5432/bothsides
      REDIS_URL: redis://redis:6379/0
      OLLAMA_BASE_URL: http://ollama:11434
      OLLAMA_MODEL: llama3.1:8b
    depends_on:
      - backend
      - redis

  # ─── Celery Beat (Scheduler) ───
  celery-beat:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: celery -A app.workers.celery_app beat --loglevel=info
    depends_on:
      - redis

  # ─── Next.js Frontend ───
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:8000/api/v1
      NEXT_PUBLIC_WS_URL: ws://localhost:8000/api/v1
    depends_on:
      - backend
    volumes:
      - ./frontend:/app  # Hot reload in dev
      - /app/node_modules
      - /app/.next

volumes:
  postgres_data:
  ollama_data:
```

---

## 3. Backend Dependencies

```
# requirements.txt
# ─── Web Framework ───
fastapi==0.115.0
uvicorn[standard]==0.30.0
python-multipart==0.0.9

# ─── Database ───
sqlalchemy[asyncio]==2.0.35
asyncpg==0.29.0
alembic==1.13.0

# ─── Redis ───
redis[hiredis]==5.0.0

# ─── Auth ───
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
authlib==1.3.0
httpx==0.27.0

# ─── AI / AutoGen ───
autogen-agentchat==0.4.0
autogen-ext[ollama]==0.4.0
ollama==0.3.0

# ─── Task Queue ───
celery[redis]==5.4.0

# ─── Payments ───
stripe==10.0.0

# ─── Validation ───
pydantic[email]==2.9.0
pydantic-settings==2.5.0

# ─── Utils ───
python-dotenv==1.0.0
structlog==24.4.0
```

---

## 4. Frontend Dependencies

```json
{
  "dependencies": {
    "next": "14.2.x",
    "react": "18.3.x",
    "react-dom": "18.3.x",
    "typescript": "5.5.x",

    "@radix-ui/react-dialog": "^1.1.0",
    "@radix-ui/react-tabs": "^1.1.0",
    "@radix-ui/react-tooltip": "^1.1.0",
    "@radix-ui/react-avatar": "^1.1.0",
    "@radix-ui/react-dropdown-menu": "^2.1.0",

    "tailwindcss": "3.4.x",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.5.0",

    "framer-motion": "^11.5.0",
    "recharts": "^2.12.0",
    "lucide-react": "^0.441.0",

    "zustand": "^4.5.0",
    "@tanstack/react-query": "^5.56.0",
    "axios": "^1.7.0",
    "zod": "^3.23.0",
    "react-hook-form": "^7.53.0",
    "@hookform/resolvers": "^3.9.0",

    "socket.io-client": "^4.8.0",
    "sonner": "^1.5.0",
    "next-themes": "^0.3.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "@types/node": "^22.5.0",
    "eslint": "^8.57.0",
    "eslint-config-next": "14.2.x",
    "prettier": "^3.3.0",
    "prettier-plugin-tailwindcss": "^0.6.0"
  }
}
```

---

## 5. Environment Variables Template

```env
# .env.example — Root

# ─── Database ───
POSTGRES_PASSWORD=change_me_in_production
DATABASE_URL=postgresql+asyncpg://bothsides:change_me_in_production@localhost:5432/bothsides

# ─── Redis ───
REDIS_URL=redis://localhost:6379/0

# ─── JWT ───
JWT_SECRET=generate-a-secure-random-string-here
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=15
JWT_REFRESH_TOKEN_EXPIRE_DAYS=7

# ─── Ollama ───
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=llama3.1:8b

# ─── News API ───
NEWS_API_KEY=your-newsapi-key-here

# ─── Stripe ───
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# ─── OAuth ───
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GITHUB_CLIENT_ID=your-github-client-id
GITHUB_CLIENT_SECRET=your-github-client-secret

# ─── Email ───
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# ─── Frontend ───
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
NEXT_PUBLIC_WS_URL=ws://localhost:8000/api/v1
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

---

## 6. Quick Start Guide

### Prerequisites
- Python 3.11+
- Node.js 20+
- Docker & Docker Compose
- Ollama installed (`winget install Ollama.Ollama` on Windows)

### Step 1: Clone & Setup Environment
```bash
git clone https://github.com/yourname/BothSidesOfACoin.git
cd BothSidesOfACoin
cp .env.example .env
# Edit .env with your API keys
```

### Step 2: Pull Ollama Model
```bash
ollama pull llama3.1:8b
# or for lighter systems:
ollama pull mistral:7b
```

### Step 3: Start Everything with Docker
```bash
docker-compose up -d
```

### Step 4: Run Database Migrations
```bash
docker-compose exec backend alembic upgrade head
```

### Step 5: Seed Demo Data (Optional)
```bash
docker-compose exec backend python scripts/seed_data.py
```

### Step 6: Access the Application
- Frontend: http://localhost:3000
- Backend API: http://localhost:8000/docs (Swagger UI)
- Ollama: http://localhost:11434

### Development (Without Docker)

**Backend:**
```bash
cd backend
python -m venv .venv
.venv\Scripts\activate   # Windows
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

**Celery Worker:**
```bash
cd backend
celery -A app.workers.celery_app worker --loglevel=info
```

---

## 7. Hackathon MVP Execution Plan (24 Hours)

### Hour 0-2: Foundation
- [x] Initialize monorepo structure
- [x] Setup Docker Compose (Postgres + Redis + Ollama)
- [x] Create FastAPI app with health endpoint
- [x] Create Next.js app with landing page skeleton

### Hour 2-5: Auth & Database
- [ ] Database models (User, Topic, Article, Perspective)
- [ ] Alembic migration
- [ ] Auth endpoints (register, login, JWT)
- [ ] Frontend login/register pages

### Hour 5-9: Core AI Agents
- [ ] Setup AutoGen 0.4 with Ollama client
- [ ] Implement OrchestratorAgent
- [ ] Implement NewsCollectorAgent (use static data for demo)
- [ ] Implement BiasClassifierAgent
- [ ] Implement SummarizerAgent (generate 3 perspectives)
- [ ] Implement QualityGuardAgent
- [ ] Wire up SelectorGroupChat

### Hour 9-13: Core Features
- [ ] Topic analysis endpoint (async + task polling)
- [ ] 3-Side View page (the main USP)
- [ ] Basic bias score computation
- [ ] Bias score display on dashboard

### Hour 13-17: AI Debate
- [ ] Implement DebateLeft/Right/Neutral agents
- [ ] WebSocket endpoint for debate streaming
- [ ] Debate chat UI (3-column layout)
- [ ] AI disclaimer component

### Hour 17-20: Polish
- [ ] Dashboard page with widgets
- [ ] Responsive design (mobile tabs for 3-Side View)
- [ ] Framer Motion animations
- [ ] Loading skeletons
- [ ] Error handling

### Hour 20-22: Demo Data & Testing
- [ ] Seed 3-5 demo topics with pre-generated perspectives
- [ ] Test full flow end-to-end
- [ ] Fix critical bugs

### Hour 22-24: Demo Prep
- [ ] Clean up UI
- [ ] Prepare demo script
- [ ] Record backup video (if live demo fails)
- [ ] Final testing

### Post-Hackathon
- [ ] Subscription system (Stripe)
- [ ] Timeline view
- [ ] Impact analyzer
- [ ] Email notifications
- [ ] Admin dashboard
- [ ] Production deployment

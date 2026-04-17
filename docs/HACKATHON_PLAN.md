# Hackathon Execution Plan
## BothSidesOfACoin — Phase-by-Phase Roadmap

---

## Hackathon Timeline Overview

```
 IDEATION          PROTOTYPE           FINALS            AWARDS
 Apr 6-17          Apr 27 - May 8      May 18-22         June 1st week
 ──────────  GAP   ─────────────  GAP  ──────────        ──────────
 ▓▓▓▓▓▓▓▓▓▓ ░░░░  ▓▓▓▓▓▓▓▓▓▓▓▓ ░░░░  ▓▓▓▓▓▓▓▓▓        🏆🏆🏆🏆🏆
 WE ARE HERE →
```

**Today:** April 17, 2026 (Last day of Ideation)

---

## Phase 1: IDEATION (April 6 – April 17) ✅ CURRENT

> *"Rising ideas earn their spot"*

**Goal:** Identify a meaningful problem aligned with the event theme. Submit concept proposal and concept video.

### Deliverables Checklist

| # | Deliverable | Status | Details |
|---|------------|--------|---------|
| 1 | **1-2 Page Concept Proposal** | ✅ Done | PRD.md covers problem, solution, features, theme alignment |
| 2 | **2-3 Minute Concept Video** | 🔲 TODO | Script + record today (April 17) |

### Concept Proposal — What We Have

Our `docs/` folder already exceeds the requirement:
- [PRD.md](docs/PRD.md) — Full product requirements with Theme 1 alignment
- [ARCHITECTURE_DIAGRAMS.md](docs/ARCHITECTURE_DIAGRAMS.md) — System architecture
- [AUTOGEN_AGENT_PLAN.md](docs/AUTOGEN_AGENT_PLAN.md) — AI agent orchestration design
- [DTOs.md](docs/DTOs.md) — Data transfer objects
- [API_SPECIFICATION.md](docs/API_SPECIFICATION.md) — Full API spec
- [UI_UX_SPECIFICATION.md](docs/UI_UX_SPECIFICATION.md) — Design system
- [PROJECT_STRUCTURE.md](docs/PROJECT_STRUCTURE.md) — Full stack project structure

### Concept Video Plan (2-3 minutes)

| Timestamp | Content | Duration |
|-----------|---------|----------|
| 0:00–0:20 | **Hook:** "People don't lack information — they lack balanced information." | 20s |
| 0:20–0:50 | **Problem:** Echo chambers, algorithmic bias, polarization | 30s |
| 0:50–1:30 | **Solution:** 3-Side View (Left/Right/Neutral), Bias Score, AI Debate Mode | 40s |
| 1:30–2:00 | **How it works:** AutoGen multi-agent AI, Ollama (local LLM), dynamic orchestration | 30s |
| 2:00–2:20 | **Theme alignment:** Reimagining Learning & Collaboration | 20s |
| 2:20–2:40 | **Impact & vision:** Break echo chambers, foster critical thinking | 20s |
| 2:40–3:00 | **Call to action / team intro** | 20s |

**Tips:**
- Use screen mockups or Figma wireframes as visuals
- Keep energy high, speak clearly
- End with the tagline: *"We don't tell you what to think — we help you think."*

### Action Items (April 17 — TODAY)

- [ ] Condense PRD into a clean 1-2 page PDF concept proposal
- [ ] Write concept video script (use the table above)
- [ ] Record and edit the 2-3 minute concept video
- [ ] Submit both deliverables before end of day

---

## Phase 2: PROTOTYPE (April 27 – May 8) — 12 Days

> *"Elite teams advance to the finale"*

**Goal:** Transform concept into a working prototype. Deliver a functional demo with technical documentation.

### Deliverables Checklist

| # | Deliverable | Due | Details |
|---|------------|-----|---------|
| 1 | **Git Repo / Prototype Link** | May 8 | https://github.com/ShubhamBrody/BothSidesOfACoin |
| 2 | **3-5 Minute Demo Video** | May 8 | Live walkthrough of working prototype |
| 3 | **Technical Write-up (Architecture)** | May 8 | ARCHITECTURE_DIAGRAMS.md + refinements |

### Sprint Plan (12 Days)

#### Sprint 1: Foundation (April 27 – April 30) — 4 days

| Day | Task | Owner | Output |
|-----|------|-------|--------|
| **Day 1** (Apr 27) | Setup monorepo, Docker Compose (Postgres + Redis + Ollama) | Backend | Running infrastructure |
| **Day 1** | Init Next.js 14 app, Tailwind, shadcn/ui, dark theme | Frontend | Base app shell |
| **Day 2** (Apr 28) | Database models + Alembic migrations (User, Topic, Article, Perspective) | Backend | DB schema ready |
| **Day 2** | Auth endpoints: register, login, JWT, refresh | Backend | Auth API working |
| **Day 3** (Apr 29) | Login & Register pages, auth flow, protected routes | Frontend | Auth UI complete |
| **Day 3** | Seed demo data: 5 topics, 15-20 articles, pre-tagged bias labels | Backend | Demo dataset |
| **Day 4** (Apr 30) | Landing page with hero, features, pricing sections | Frontend | Marketing page |
| **Day 4** | Topic CRUD endpoints + article listing | Backend | Content API |

**Sprint 1 Milestone:** User can register, log in, and see a landing page. Backend has demo data.

#### Sprint 2: Core AI (May 1 – May 4) — 4 days

| Day | Task | Owner | Output |
|-----|------|-------|--------|
| **Day 5** (May 1) | Setup AutoGen 0.4 + Ollama client, define agent base config | AI/Backend | Agent framework ready |
| **Day 5** | OrchestratorAgent + SelectorGroupChat skeleton | AI/Backend | Dynamic routing works |
| **Day 6** (May 2) | NewsCollectorAgent (use cached/seeded data for demo speed) | AI/Backend | Article collection |
| **Day 6** | BiasClassifierAgent + classification tool | AI/Backend | Bias labels generated |
| **Day 7** (May 3) | SummarizerAgent (generate Left/Right/Neutral summaries) | AI/Backend | 3-perspective summaries |
| **Day 7** | QualityGuardAgent (review + approve outputs) | AI/Backend | Quality validation |
| **Day 8** (May 4) | Topic analysis endpoint (async task + polling) | Backend | `/topics/analyze` API |
| **Day 8** | **3-Side View page** — the main USP (3-column layout) | Frontend | Core feature visible |

**Sprint 2 Milestone:** User can pick a topic and see AI-generated Left / Neutral / Right perspectives side-by-side.

#### Sprint 3: Debate + Bias + Polish (May 5 – May 8) — 4 days

| Day | Task | Owner | Output |
|-----|------|-------|--------|
| **Day 9** (May 5) | DebateLeft/Right/Neutral agents + debate service | AI/Backend | Debate agents working |
| **Day 9** | WebSocket endpoint for debate streaming | Backend | Real-time streaming |
| **Day 10** (May 6) | **AI Debate Mode UI** — 3-column chat with typing indicators | Frontend | Debate page |
| **Day 10** | Bias score computation + `/user/bias-score` endpoint | Backend | Bias tracking API |
| **Day 11** (May 7) | **Dashboard page** — bias gauge, trending topics, reading stats | Frontend | Dashboard complete |
| **Day 11** | Framer Motion animations, loading skeletons, responsive polish | Frontend | Professional feel |
| **Day 12** (May 8) | End-to-end testing, bug fixes, demo data polish | Full Team | Stable prototype |
| **Day 12** | Record 3-5 minute demo video | Full Team | Demo video |
| **Day 12** | Finalize technical write-up (architecture doc) | Full Team | Architecture doc |

**Sprint 3 Milestone:** Full working prototype — 3-Side View, AI Debate, Bias Score, Dashboard. Demo video recorded.

### Prototype Feature Priority (MoSCoW)

| Priority | Feature | Included? |
|----------|---------|-----------|
| **Must Have** | User auth (register + login) | ✅ |
| **Must Have** | 3-Side View (Left / Neutral / Right) | ✅ |
| **Must Have** | AI bias classification | ✅ |
| **Must Have** | AI Debate Mode (3 personas) | ✅ |
| **Must Have** | Basic Bias Score | ✅ |
| **Must Have** | Dashboard | ✅ |
| **Must Have** | AI disclaimer on all AI content | ✅ |
| **Should Have** | Responsive mobile layout | ✅ |
| **Should Have** | Dark mode | ✅ |
| **Should Have** | Animated transitions | ✅ |
| **Could Have** | Timeline view | ❌ Post-prototype |
| **Could Have** | Impact analyzer | ❌ Post-prototype |
| **Could Have** | Stripe subscription | ❌ Post-prototype |
| **Won't Have** | OAuth (Google/GitHub) | ❌ Finals phase |
| **Won't Have** | Admin dashboard | ❌ Finals phase |

### Gap Period Strategy (April 18–26 — 9 days before Prototype starts)

Use this time wisely even though it's not an official phase:

| Task | Priority | Notes |
|------|----------|-------|
| Setup dev environment (Docker, Ollama, model download) | High | Avoid wasting prototype days on setup |
| Create Figma mockups for key pages | High | Faster frontend development |
| Pre-build UI component library (shadcn/ui setup) | Medium | Accelerates Sprint 1 |
| Test AutoGen 0.4 + Ollama locally (proof of concept) | High | De-risk AI integration |
| Draft demo script | Medium | Know what you're building toward |
| Research NewsAPI/GDELT for article sourcing | Medium | Validate data sources |

---

## Phase 3: FINALS (May 18 – May 22) — 5 Days

> *"Winners selected for Awards"*

**Goal:** Present polished solution to judges. Final pitch and live demo.

### Deliverables Checklist

| # | Deliverable | Due | Details |
|---|------------|-----|---------|
| 1 | **5-7 Minute Live Demo** | May 22 | Polished walkthrough of working app |
| 2 | **Final Pitch Deck** | May 22 | 10-12 slides, problem → solution → impact |
| 3 | **Scalability Summary** | May 22 | How the architecture scales to 10K+ users |

### Sprint Plan (5 Days)

#### May 18–19: Harden & Enhance (2 days)

| Task | Details |
|------|---------|
| Fix all prototype bugs | Stability is king for live demo |
| Add OAuth login (Google) | Impressive for judges |
| Add Timeline view (if time permits) | Extra feature differentiation |
| Polish animations & micro-interactions | "Wow factor" for judges |
| Performance optimization | Fast load times during demo |
| Add comprehensive error handling | No crashes during live demo |

#### May 20: Demo & Pitch Prep (1 day)

| Task | Details |
|------|---------|
| Write demo script (5-7 min) | Exact flow: landing → register → dashboard → 3-Side View → debate → bias |
| Build pitch deck (10-12 slides) | See structure below |
| Practice demo 3-5 times | Smooth delivery, handle failures gracefully |
| Prepare fallback plan | Pre-recorded video if live demo fails |

#### May 21: Scalability Summary + Rehearsal (1 day)

| Task | Details |
|------|---------|
| Write scalability summary | Architecture, caching, horizontal scaling, AI scaling |
| Full dress rehearsal | Timed run-through with team |
| Test on different devices/browsers | Ensure responsive design works |
| Deploy to cloud (if not already) | Live demo from production URL |

#### May 22: Finals Day 🏆

| Task | Details |
|------|---------|
| Final system check (1 hour before) | Database, Redis, Ollama, API — all running |
| Deliver pitch + live demo | 5-7 minutes, confident and clear |
| Q&A preparation | Anticipate judge questions (see below) |

### Pitch Deck Structure (10-12 Slides)

| Slide # | Title | Content | Duration |
|---------|-------|---------|----------|
| 1 | **Title Slide** | BothSidesOfACoin logo, tagline, team | 15s |
| 2 | **The Problem** | Echo chambers, algorithmic bias, polarization stats | 30s |
| 3 | **The Insight** | "People don't lack information — they lack balanced information" | 15s |
| 4 | **Our Solution** | Platform overview, 3-Side View concept | 30s |
| 5 | **Live Demo** | Switch to live app — 3-Side View, Debate, Bias Score | 3 min |
| 6 | **AI Architecture** | AutoGen 0.4 multi-agent diagram, dynamic orchestration | 30s |
| 7 | **Theme Alignment** | Theme 1: Reimagining Learning & Collaboration | 20s |
| 8 | **Scalability** | Horizontal scaling, caching, Ollama clustering | 20s |
| 9 | **Business Model** | Free / Premium / Enterprise tiers | 20s |
| 10 | **Social Impact** | Stats on misinformation, how we help | 20s |
| 11 | **Roadmap** | Phase 2-4 features, growth plan | 15s |
| 12 | **Thank You / Q&A** | Team photo, contact, CTA | 10s |

### Anticipated Judge Questions & Answers

| Question | Strong Answer |
|----------|---------------|
| *"How do you ensure AI summaries are accurate?"* | QualityGuard agent reviews all AI outputs. All content carries an AI disclaimer. We don't claim truth — we surface perspectives. |
| *"How is this different from AllSides or Ground News?"* | We go beyond labeling. AI Debate Mode lets users interact with perspectives. Bias Score makes consumption personal. Dynamic multi-agent AI (not static rules). |
| *"Can this scale?"* | Async architecture (FastAPI + Celery), Redis caching, Ollama can run on GPU cluster, PostgreSQL read replicas. Designed for 10K+ concurrent users. |
| *"How do you avoid introducing your own bias?"* | Three independent AI personas with distinct system prompts. QualityGuard checks for balance. Sources are cited. Users see ALL sides. |
| *"What about monetization?"* | Freemium: Free (5 analyses/day) → Premium $9.99/mo (unlimited) → Enterprise $49.99/mo (API + team). |
| *"Why Ollama instead of OpenAI?"* | Privacy (no data leaves the server), zero API cost, hackathon-friendly, can swap to any provider later. |
| *"How does the orchestrator work without hardcoded pipelines?"* | AutoGen 0.4 SelectorGroupChat — an LLM reads conversation context and dynamically selects the next agent. No if/else chains. |

### Scalability Summary Outline

```
1. Architecture Overview
   - Microservice-ready monolith (FastAPI)
   - Stateless API servers → horizontal scaling via load balancer
   - Async everywhere (asyncio, asyncpg, httpx)

2. Database Scaling
   - PostgreSQL primary + read replicas
   - Connection pooling (asyncpg)
   - UUID-based IDs (no sequential bottlenecks)
   - Alembic migrations for zero-downtime schema changes

3. Caching Strategy
   - Redis for sessions, rate limits, API responses
   - AI analysis results cached 1-24 hours (expensive to recompute)
   - CDN for static frontend assets

4. AI Layer Scaling
   - Celery workers (horizontally scalable task queue)
   - Ollama can run on multiple GPU instances behind a load balancer
   - Model-per-agent assignment (right-size compute)
   - Circuit breaker pattern for graceful degradation

5. Frontend Scaling
   - Next.js SSR + CDN (Vercel/Cloudflare)
   - Static generation for landing page, ISR for topics
   - WebSocket connections managed via Redis pub/sub

6. Observability
   - Prometheus metrics, Grafana dashboards
   - Structured logging (structlog → JSON)
   - Health check endpoints on every service

7. Target Metrics
   - 10,000+ concurrent users
   - < 200ms API latency (non-AI)
   - < 10s AI analysis response
   - 99.9% uptime
```

---

## Phase 4: AWARDS (June 1st Week) 🏆

> *"Celebrating our champions"*

**No deliverables — just applause.**

### Award Categories

| Award | Prize | Our Target |
|-------|-------|------------|
| **Global Winner** | USD $2,500 | 🎯 Primary goal |
| **Global Runner-Up** | USD $1,500 | Strong backup |
| **CXO Innovation Award** | USD $2,000 | Social impact angle is strong for this |

### Why We Can Win

| Criteria | Our Strength |
|----------|-------------|
| **Originality** | No one else is doing AI multi-perspective + Bias Score + AI Debate in one platform |
| **Social Impact** | Combats misinformation and polarization — a global crisis |
| **Technical Depth** | AutoGen 0.4 multi-agent AI, dynamic orchestration (no hardcoded pipeline), Ollama |
| **Demo Quality** | 3-Side View is visually stunning and immediately understandable |
| **Theme Alignment** | Perfect fit for Theme 1: Reimagining Learning & Collaboration |
| **Scalability** | Production-grade architecture from day one |

---

## Master Timeline (Gantt View)

```
April 2026                                    May 2026                         June 2026
6   10  14  17  │  21  25  27  1   4   8  │  12  16  18  22  │  1st week
─────────────────┼─────────────────────────────┼───────────────────┼──────────
                 │                             │                   │
IDEATION ████████│                             │                   │
  Concept ██████ │                             │                   │
  Video     ████ │                             │                   │
                 │                             │                   │
         GAP     │░░░░░░░░░                    │                   │
  Prep (env,     │░░░░░░░░░                    │                   │
  mockups, POC)  │                             │                   │
                 │                             │                   │
             PROTOTYPE  │██████████████████████│                   │
               Sprint 1 │████████              │                   │
               Sprint 2 │        ████████      │                   │
               Sprint 3 │                ██████│                   │
                         │                     │                   │
                    GAP  │                     │░░░░░░░░░          │
                         │                     │                   │
                                          FINALS │█████████████████│
                                           Harden│████████         │
                                           Prep  │        ████     │
                                           Demo  │            ████ │
                                                  │                │
                                                         AWARDS 🏆 │████████
```

---

## Key Dates Summary

| Date | Milestone | Status |
|------|-----------|--------|
| **April 17** | Ideation deadline (concept proposal + video) | 🔴 TODAY |
| April 18–26 | Gap period — prep work, environment setup, POC | ⏳ Upcoming |
| **April 27** | Prototype phase begins | ⏳ |
| **May 8** | Prototype deadline (repo + demo video + architecture doc) | ⏳ |
| May 9–17 | Gap period — polish, rehearse | ⏳ |
| **May 18** | Finals phase begins | ⏳ |
| **May 22** | Finals deadline (live demo + pitch deck + scalability summary) | ⏳ |
| **June 1st week** | Awards ceremony | 🏆 |

---

*Stay focused. Ship the core. Polish what matters. Win.*

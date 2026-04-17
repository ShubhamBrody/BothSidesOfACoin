# Architecture & Flow Diagrams
## BothSidesOfACoin

---

## 1. High-Level System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        WEB["🌐 Next.js Frontend<br/>(SSR + CSR)"]
        CDN["📦 CDN<br/>(Static Assets)"]
    end

    subgraph "API Gateway Layer"
        NGINX["🔀 Nginx / Traefik<br/>(Load Balancer + SSL)"]
        RATE["🛡️ Rate Limiter<br/>(Redis-backed)"]
    end

    subgraph "Application Layer"
        API1["⚡ FastAPI Server 1"]
        API2["⚡ FastAPI Server 2"]
        API3["⚡ FastAPI Server N"]
        WS["🔌 WebSocket Server<br/>(Debate Streaming)"]
    end

    subgraph "AI Orchestration Layer"
        ORCH["🧠 AutoGen Orchestrator"]
        subgraph "AI Agents"
            NC["📰 NewsCollector"]
            BC["⚖️ BiasClassifier"]
            SM["📝 Summarizer"]
            FE["🔍 FactExtractor"]
            TL["📅 Timeline"]
            IA["🌍 ImpactAnalyzer"]
            DL["🟦 DebateLeft"]
            DR["🟥 DebateRight"]
            DN["⚪ DebateNeutral"]
            UB["👤 UserBias"]
            QG["✅ QualityGuard"]
        end
    end

    subgraph "LLM Layer"
        OLLAMA["🤖 Ollama Server<br/>(Llama 3.1 / Mistral)"]
    end

    subgraph "Data Layer"
        PG[("🐘 PostgreSQL<br/>(Primary DB)")]
        PG_R[("🐘 PostgreSQL<br/>(Read Replica)")]
        REDIS[("🔴 Redis<br/>(Cache + Sessions)")]
        CELERY["⚙️ Celery Workers"]
    end

    subgraph "External Services"
        NEWS["📡 News APIs<br/>(NewsAPI, GDELT)"]
        STRIPE["💳 Stripe<br/>(Payments)"]
        SMTP["📧 SMTP<br/>(Email)"]
        OAUTH["🔐 OAuth Providers<br/>(Google, GitHub)"]
    end

    WEB -->|HTTPS| NGINX
    CDN -->|Static| WEB
    NGINX --> RATE
    RATE --> API1 & API2 & API3
    RATE --> WS
    API1 & API2 & API3 --> ORCH
    WS --> ORCH
    ORCH --> NC & BC & SM & FE & TL & IA & DL & DR & DN & UB & QG
    NC & BC & SM & FE & TL & IA & DL & DR & DN & UB & QG --> OLLAMA
    API1 & API2 & API3 --> PG & REDIS
    API1 & API2 & API3 -.->|reads| PG_R
    API1 & API2 & API3 --> CELERY
    CELERY --> ORCH
    CELERY --> REDIS
    NC --> NEWS
    API1 --> STRIPE & SMTP & OAUTH
    PG -.->|replication| PG_R
```

---

## 2. User Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant FE as Frontend (Next.js)
    participant API as FastAPI Backend
    participant DB as PostgreSQL
    participant Redis as Redis
    participant Email as SMTP Service
    participant OAuth as OAuth Provider

    Note over User, OAuth: Email/Password Registration
    User->>FE: Fill registration form
    FE->>API: POST /api/v1/auth/register
    API->>API: Validate input (email, password strength)
    API->>DB: Check email uniqueness
    DB-->>API: Email available
    API->>API: Hash password (bcrypt)
    API->>DB: Insert user (status=PENDING)
    API->>Email: Send verification email (JWT link)
    API-->>FE: 201 Created (check email)
    FE-->>User: "Check your email"
    
    User->>Email: Click verification link
    Email->>API: GET /api/v1/auth/verify?token=xxx
    API->>API: Validate JWT token
    API->>DB: Update user (status=ACTIVE)
    API-->>User: Redirect to login

    Note over User, OAuth: Email/Password Login
    User->>FE: Enter credentials
    FE->>API: POST /api/v1/auth/login
    API->>DB: Fetch user by email
    API->>API: Verify password (bcrypt)
    API->>API: Generate access_token (15min) + refresh_token (7d)
    API->>Redis: Store refresh_token hash
    API-->>FE: {access_token, refresh_token, user}
    FE->>FE: Store access_token (memory), refresh_token (httpOnly cookie)

    Note over User, OAuth: OAuth 2.0 Login
    User->>FE: Click "Login with Google"
    FE->>OAuth: Redirect to provider
    OAuth->>User: Consent screen
    User->>OAuth: Grant access
    OAuth->>API: Callback with auth code
    API->>OAuth: Exchange code for tokens
    OAuth-->>API: {user profile, email}
    API->>DB: Find or create user
    API->>API: Generate JWT tokens
    API-->>FE: Redirect with tokens

    Note over User, OAuth: Token Refresh
    FE->>API: POST /api/v1/auth/refresh (refresh_token in cookie)
    API->>Redis: Validate refresh_token
    API->>API: Generate new access_token + rotate refresh_token
    API->>Redis: Blacklist old refresh_token, store new
    API-->>FE: {new access_token, new refresh_token}
```

---

## 3. News Analysis Pipeline Flow

```mermaid
flowchart TD
    START([User requests topic analysis]) --> ORCH

    subgraph ORCH["🧠 Orchestrator Agent"]
        PLAN["Analyze request<br/>Generate execution plan"]
    end

    ORCH -->|"Step 1: Collect"| NC

    subgraph NC["📰 News Collector Agent"]
        NC1["Query News APIs"]
        NC2["Deduplicate articles"]
        NC3["Extract metadata"]
        NC1 --> NC2 --> NC3
    end

    NC -->|"Articles[]"| BC

    subgraph BC["⚖️ Bias Classifier Agent"]
        BC1["Analyze language patterns"]
        BC2["Check source reputation"]
        BC3["Assign bias label<br/>(LEFT/RIGHT/NEUTRAL)"]
        BC4["Compute confidence score"]
        BC1 --> BC2 --> BC3 --> BC4
    end

    BC -->|"Classified Articles[]"| PARALLEL

    PARALLEL{{"Parallel Execution"}}

    PARALLEL -->|"Path A"| SM
    PARALLEL -->|"Path B"| FE
    PARALLEL -->|"Path C (if premium)"| TL

    subgraph SM["📝 Summarizer Agent"]
        SM1["Group by bias label"]
        SM2["Generate LEFT summary"]
        SM3["Generate RIGHT summary"]
        SM4["Generate NEUTRAL summary"]
        SM1 --> SM2 & SM3 & SM4
    end

    subgraph FE["🔍 Fact Extractor Agent"]
        FE1["Extract key claims"]
        FE2["Identify statistics"]
        FE3["Flag unverified claims"]
        FE1 --> FE2 --> FE3
    end

    subgraph TL["📅 Timeline Agent"]
        TL1["Extract dates & events"]
        TL2["Build chronology"]
        TL3["Link cause-effect"]
        TL1 --> TL2 --> TL3
    end

    SM --> QG
    FE --> QG
    TL --> QG

    subgraph QG["✅ Quality Guard Agent"]
        QG1["Check for hallucinations"]
        QG2["Verify factual consistency"]
        QG3["Check for harmful content"]
        QG4["Validate perspective balance"]
        QG1 --> QG2 --> QG3 --> QG4
    end

    QG -->|"Approved"| CACHE["💾 Cache Results<br/>(Redis + PostgreSQL)"]
    QG -->|"Rejected"| RETRY["🔄 Retry with feedback"]
    RETRY --> SM

    CACHE --> RESPONSE["📤 Send Response to Client"]

    RESPONSE --> END([3-Side View Displayed])

    style ORCH fill:#4a90d9,color:#fff
    style NC fill:#50c878,color:#fff
    style BC fill:#ff9800,color:#fff
    style SM fill:#9c27b0,color:#fff
    style FE fill:#00bcd4,color:#fff
    style TL fill:#795548,color:#fff
    style QG fill:#4caf50,color:#fff
```

---

## 4. AI Debate Mode Flow

```mermaid
sequenceDiagram
    actor User
    participant FE as Frontend
    participant WS as WebSocket Server
    participant ORCH as Orchestrator
    participant DL as DebateLeftAgent
    participant DR as DebateRightAgent
    participant DN as DebateNeutralAgent
    participant QG as QualityGuard
    participant LLM as Ollama LLM

    User->>FE: Start debate on "AI Regulation"
    FE->>WS: WS Connect /debate/{session_id}/stream
    WS->>ORCH: Initialize debate session
    ORCH->>ORCH: Load topic context & facts
    ORCH-->>WS: Session ready

    loop Each User Message
        User->>FE: "Should AI be regulated by government?"
        FE->>WS: Send message
        WS->>ORCH: Route message

        par Parallel AI Responses
            ORCH->>DL: Generate left perspective
            DL->>LLM: Prompt (progressive viewpoint)
            LLM-->>DL: Streaming response
            DL->>QG: Validate output
            QG-->>DL: Approved
            DL-->>WS: Stream tokens (labeled LEFT)
            WS-->>FE: Display in left column

            ORCH->>DR: Generate right perspective
            DR->>LLM: Prompt (conservative viewpoint)
            LLM-->>DR: Streaming response
            DR->>QG: Validate output
            QG-->>DR: Approved
            DR-->>WS: Stream tokens (labeled RIGHT)
            WS-->>FE: Display in right column

            ORCH->>DN: Generate neutral analysis
            DN->>LLM: Prompt (balanced factual)
            LLM-->>DN: Streaming response
            DN->>QG: Validate output
            QG-->>DN: Approved
            DN-->>WS: Stream tokens (labeled NEUTRAL)
            WS-->>FE: Display in center column
        end

        ORCH->>ORCH: Update debate context
        WS-->>FE: All responses complete
        FE-->>User: Show all 3 perspectives + disclaimer
    end

    User->>FE: End debate
    FE->>WS: Close session
    WS->>ORCH: Save debate history
    ORCH-->>WS: Saved
    WS-->>FE: Session closed
```

---

## 5. Bias Detection & Scoring Flow

```mermaid
flowchart TD
    READ([User reads article]) --> TRACK

    subgraph TRACK["📊 Reading Tracker"]
        T1["Record: user_id, article_id,<br/>bias_label, duration, timestamp"]
    end

    TRACK --> STORE["💾 Store ReadingEvent<br/>(PostgreSQL)"]
    STORE --> CHECK{{"Threshold check:<br/>5+ reads since last computation?"}}

    CHECK -->|"Yes"| COMPUTE
    CHECK -->|"No"| WAIT["Wait for more events"]

    subgraph COMPUTE["🧠 User Bias Agent"]
        C1["Fetch last 100 reading events"]
        C2["Calculate distribution:<br/>% LEFT / % RIGHT / % NEUTRAL"]
        C3["Apply time-decay weighting<br/>(recent reads matter more)"]
        C4["Compute Bias Score (0-100)<br/>0 = perfectly balanced<br/>100 = extremely one-sided"]
        C5["Determine bias direction<br/>(LEFT or RIGHT leaning)"]
        C6["Generate recommendations"]
        C1 --> C2 --> C3 --> C4 --> C5 --> C6
    end

    COMPUTE --> SNAPSHOT["💾 Store BiasScore snapshot"]
    COMPUTE --> ALERT{{"Score > 70?"}}

    ALERT -->|"Yes"| NOTIFY["🔔 Push notification:<br/>'You've been reading 80%<br/>left-leaning content'"]
    ALERT -->|"No"| BADGE["🏆 Award balanced<br/>reader badge"]

    NOTIFY --> SUGGEST["📋 Suggest articles from<br/>underrepresented perspective"]
    BADGE --> DASHBOARD["📊 Update dashboard"]
    SUGGEST --> DASHBOARD

    style COMPUTE fill:#4a90d9,color:#fff
    style NOTIFY fill:#f44336,color:#fff
    style BADGE fill:#4caf50,color:#fff
```

---

## 6. Subscription & Payment Flow

```mermaid
sequenceDiagram
    actor User
    participant FE as Frontend
    participant API as Backend API
    participant DB as PostgreSQL
    participant Stripe as Stripe API
    participant Email as Email Service

    Note over User, Email: Upgrade to Premium
    User->>FE: Click "Upgrade to Premium"
    FE->>API: POST /api/v1/subscription/checkout<br/>{plan: "premium_monthly"}
    API->>Stripe: Create Checkout Session
    Stripe-->>API: {checkout_url, session_id}
    API-->>FE: {checkout_url}
    FE->>Stripe: Redirect to Stripe Checkout

    User->>Stripe: Enter payment details
    Stripe->>Stripe: Process payment
    Stripe-->>User: Success page
    Stripe->>API: POST /webhook (checkout.session.completed)
    API->>API: Verify webhook signature
    API->>DB: Update user subscription<br/>(tier=PREMIUM, stripe_customer_id, ...)
    API->>Email: Send confirmation email
    API->>FE: WebSocket: subscription_updated

    Note over User, Email: Monthly Renewal
    Stripe->>API: POST /webhook (invoice.payment_succeeded)
    API->>DB: Extend subscription period
    API->>Email: Send receipt

    Note over User, Email: Payment Failed
    Stripe->>API: POST /webhook (invoice.payment_failed)
    API->>DB: Mark subscription at_risk
    API->>Email: Send payment failed notice
    
    Note over User, Email: After 3 failed retries
    Stripe->>API: POST /webhook (customer.subscription.deleted)
    API->>DB: Downgrade to FREE tier
    API->>Email: Send downgrade notice

    Note over User, Email: User Cancels
    User->>FE: Click "Cancel Subscription"
    FE->>API: POST /api/v1/subscription/cancel
    API->>Stripe: Cancel at period end
    Stripe-->>API: Confirmed
    API->>DB: Set cancel_at_period_end=true
    API-->>FE: "Active until {date}"
```

---

## 7. AutoGen 0.4 Dynamic Agent Orchestration Flow

```mermaid
flowchart TD
    REQ([Incoming Request]) --> CLASSIFY

    subgraph CLASSIFY["🧠 Request Classification"]
        CL1["Orchestrator LLM call:<br/>'What type of request is this?'"]
        CL2["Possible types:<br/>TOPIC_ANALYSIS<br/>DEBATE_START<br/>BIAS_CHECK<br/>TIMELINE_REQUEST<br/>IMPACT_ANALYSIS<br/>GENERAL_QUERY"]
        CL1 --> CL2
    end

    CLASSIFY --> PLAN

    subgraph PLAN["📋 Dynamic Plan Generation"]
        P1["Orchestrator LLM call:<br/>'Generate execution steps<br/>for this request type'"]
        P2["Output: ordered list of<br/>{agent, input_mapping, priority}"]
        P1 --> P2
    end

    PLAN --> EXEC

    subgraph EXEC["⚡ Dynamic Execution Engine"]
        E1["For each step in plan:"]
        E2["Select agent by name"]
        E3["Map inputs from previous outputs"]
        E4["Execute agent"]
        E5["Validate output (QualityGuard)"]
        E6{{"Success?"}}
        E7["Store output in context"]
        E8["Retry with modified prompt"]
        E9{{"Max retries?"}}
        E10["Skip step, log warning"]

        E1 --> E2 --> E3 --> E4 --> E5 --> E6
        E6 -->|"Yes"| E7
        E6 -->|"No"| E8
        E8 --> E9
        E9 -->|"No"| E4
        E9 -->|"Yes"| E10
        E7 --> E11{{"More steps?"}}
        E10 --> E11
        E11 -->|"Yes"| E1
        E11 -->|"No"| COMPILE
    end

    COMPILE["📦 Compile Final Response<br/>from all agent outputs"]
    COMPILE --> RESPOND([Send Response])

    style CLASSIFY fill:#ff9800,color:#fff
    style PLAN fill:#4a90d9,color:#fff
    style EXEC fill:#9c27b0,color:#fff
    style COMPILE fill:#4caf50,color:#fff
```

---

## 8. Database Entity Relationship Diagram

```mermaid
erDiagram
    USER ||--o{ READING_EVENT : records
    USER ||--o{ BIAS_SCORE : has
    USER ||--o{ DEBATE_SESSION : starts
    USER ||--|| SUBSCRIPTION : has
    USER ||--o{ BOOKMARK : saves

    TOPIC ||--o{ ARTICLE : contains
    TOPIC ||--o{ PERSPECTIVE : has
    TOPIC ||--o{ TIMELINE_EVENT : has
    TOPIC ||--o{ DEBATE_SESSION : discusses

    ARTICLE ||--o{ READING_EVENT : tracked_by
    ARTICLE }o--|| NEWS_SOURCE : from

    DEBATE_SESSION ||--o{ DEBATE_MESSAGE : contains

    PERSPECTIVE }o--|| TOPIC : belongs_to

    USER {
        uuid id PK
        string email UK
        string password_hash
        string full_name
        string avatar_url
        enum role "FREE_USER|PREMIUM_USER|ADMIN|SUPER_ADMIN"
        enum auth_provider "LOCAL|GOOGLE|GITHUB"
        string oauth_id
        enum status "PENDING|ACTIVE|SUSPENDED|DELETED"
        boolean email_verified
        datetime created_at
        datetime updated_at
        datetime last_login_at
    }

    SUBSCRIPTION {
        uuid id PK
        uuid user_id FK
        enum tier "FREE|PREMIUM|ENTERPRISE"
        enum status "ACTIVE|PAST_DUE|CANCELED|EXPIRED"
        string stripe_customer_id
        string stripe_subscription_id
        datetime current_period_start
        datetime current_period_end
        boolean cancel_at_period_end
        datetime created_at
        datetime updated_at
    }

    TOPIC {
        uuid id PK
        string title
        string slug UK
        text description
        string[] keywords
        string category
        string region
        boolean is_trending
        int article_count
        datetime first_reported_at
        datetime last_updated_at
        datetime created_at
    }

    ARTICLE {
        uuid id PK
        uuid topic_id FK
        uuid source_id FK
        string title
        text content
        string url UK
        string author
        string image_url
        enum bias_label "LEFT|RIGHT|NEUTRAL|UNKNOWN"
        float bias_confidence
        float sentiment_score
        jsonb ai_metadata
        datetime published_at
        datetime fetched_at
        datetime created_at
    }

    NEWS_SOURCE {
        uuid id PK
        string name UK
        string url
        string logo_url
        enum default_bias "LEFT|CENTER_LEFT|CENTER|CENTER_RIGHT|RIGHT"
        float reliability_score
        boolean is_active
        datetime created_at
    }

    PERSPECTIVE {
        uuid id PK
        uuid topic_id FK
        enum bias_type "LEFT|RIGHT|NEUTRAL"
        text summary
        jsonb key_arguments
        jsonb sources_cited
        string model_used
        float quality_score
        datetime generated_at
        datetime created_at
    }

    READING_EVENT {
        uuid id PK
        uuid user_id FK
        uuid article_id FK
        enum bias_label "LEFT|RIGHT|NEUTRAL"
        int read_duration_seconds
        float scroll_depth
        datetime read_at
    }

    BIAS_SCORE {
        uuid id PK
        uuid user_id FK
        float score "0-100"
        enum direction "LEFT|RIGHT|BALANCED"
        float left_percentage
        float right_percentage
        float neutral_percentage
        int articles_analyzed
        jsonb recommendations
        datetime computed_at
    }

    DEBATE_SESSION {
        uuid id PK
        uuid user_id FK
        uuid topic_id FK
        string title
        enum status "ACTIVE|COMPLETED|ARCHIVED"
        int message_count
        datetime started_at
        datetime ended_at
        datetime created_at
    }

    DEBATE_MESSAGE {
        uuid id PK
        uuid session_id FK
        enum sender "USER|LEFT_AI|RIGHT_AI|NEUTRAL_AI|SYSTEM"
        text content
        jsonb metadata
        int sequence_number
        datetime created_at
    }

    TIMELINE_EVENT {
        uuid id PK
        uuid topic_id FK
        string title
        text description
        datetime event_date
        string[] sources
        int importance_score
        datetime created_at
    }

    BOOKMARK {
        uuid id PK
        uuid user_id FK
        uuid article_id FK
        uuid topic_id FK
        string note
        datetime created_at
    }
```

---

## 9. Frontend Page Navigation Flow

```mermaid
flowchart TD
    LAND["🏠 Landing Page"] -->|"Sign Up"| REG["📝 Register"]
    LAND -->|"Login"| LOGIN["🔐 Login"]
    LAND -->|"Browse (limited)"| TRENDING["📈 Trending Topics"]
    
    REG -->|"Success"| VERIFY["📧 Verify Email"]
    VERIFY -->|"Verified"| LOGIN
    LOGIN -->|"Success"| DASH
    LOGIN -->|"OAuth"| OAUTH["🔑 OAuth Provider"]
    OAUTH -->|"Success"| DASH

    DASH["📊 Dashboard"] -->|"Click topic"| TOPIC["🔍 Topic View<br/>(3-Side View)"]
    DASH -->|"Profile icon"| PROFILE["👤 Profile"]
    DASH -->|"Start debate"| DEBATE["💬 AI Debate"]
    DASH -->|"View bias"| BIAS["📊 Bias Dashboard"]
    DASH -->|"Upgrade"| SUB["💳 Subscription"]
    DASH -->|"Search"| SEARCH["🔎 Search Results"]

    TOPIC -->|"Read article"| ARTICLE["📄 Article Detail"]
    TOPIC -->|"View timeline"| TIMELINE["📅 Timeline View"]
    TOPIC -->|"View impact"| IMPACT["🌍 Impact Analysis"]
    TOPIC -->|"Start debate"| DEBATE

    ARTICLE -->|"Back"| TOPIC
    TIMELINE -->|"Back"| TOPIC
    IMPACT -->|"Back"| TOPIC

    DEBATE -->|"End session"| DASH

    PROFILE -->|"Reading history"| HISTORY["📚 Reading History"]
    PROFILE -->|"Settings"| SETTINGS["⚙️ Settings"]
    PROFILE -->|"Billing"| SUB

    SUB -->|"Stripe Checkout"| STRIPE["💳 Stripe"]
    STRIPE -->|"Success"| DASH

    SEARCH -->|"Click result"| TOPIC

    style DASH fill:#4a90d9,color:#fff
    style TOPIC fill:#9c27b0,color:#fff
    style DEBATE fill:#ff9800,color:#fff
    style BIAS fill:#4caf50,color:#fff
```

---

## 10. Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Edge"
            CF["☁️ Cloudflare / Vercel CDN"]
        end

        subgraph "Container Orchestration (Docker Compose / K8s)"
            subgraph "Web Tier"
                FE1["Next.js Container 1"]
                FE2["Next.js Container 2"]
            end

            subgraph "API Tier"
                LB["🔀 Load Balancer<br/>(Nginx/Traefik)"]
                A1["FastAPI Container 1"]
                A2["FastAPI Container 2"]
                A3["FastAPI Container 3"]
                WSS["WebSocket Server"]
            end

            subgraph "Worker Tier"
                CW1["Celery Worker 1"]
                CW2["Celery Worker 2"]
                CW3["Celery Worker N"]
                BEAT["Celery Beat<br/>(Scheduler)"]
            end

            subgraph "AI Tier"
                OL1["Ollama Instance 1<br/>(GPU)"]
                OL2["Ollama Instance 2<br/>(GPU)"]
            end

            subgraph "Data Tier"
                PG_P[("PostgreSQL Primary")]
                PG_R[("PostgreSQL Replica")]
                REDIS_M[("Redis Primary")]
                REDIS_R[("Redis Replica")]
            end
        end

        subgraph "Monitoring"
            PROM["Prometheus"]
            GRAF["Grafana"]
            ALERT["AlertManager"]
        end
    end

    CF --> FE1 & FE2
    FE1 & FE2 --> LB
    LB --> A1 & A2 & A3
    LB --> WSS
    A1 & A2 & A3 --> CW1 & CW2 & CW3
    CW1 & CW2 & CW3 --> OL1 & OL2
    A1 & A2 & A3 --> PG_P & REDIS_M
    A1 & A2 & A3 -.-> PG_R & REDIS_R
    BEAT --> CW1
    PG_P -.-> PG_R
    REDIS_M -.-> REDIS_R
    A1 & A2 & A3 --> PROM
    CW1 & CW2 & CW3 --> PROM
    PROM --> GRAF
    PROM --> ALERT
```

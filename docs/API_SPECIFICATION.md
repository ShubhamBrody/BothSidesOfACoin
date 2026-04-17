# API Specification
## BothSidesOfACoin — RESTful + WebSocket API

**Base URL:** `https://api.bothsidesofacoin.com/api/v1`  
**Authentication:** Bearer JWT (except public endpoints)  
**Content-Type:** `application/json`

---

## Common Headers

| Header | Value | Required |
|--------|-------|----------|
| `Authorization` | `Bearer <access_token>` | Yes (authenticated routes) |
| `Content-Type` | `application/json` | Yes (POST/PUT/PATCH) |
| `X-Request-ID` | UUID (auto-generated if missing) | No |
| `X-Subscription-Tier` | Injected by backend from JWT | N/A |

## Common Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 400 | `VALIDATION_ERROR` | Request body validation failed |
| 401 | `UNAUTHORIZED` | Missing or invalid token |
| 403 | `FORBIDDEN` | Insufficient permissions or subscription tier |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `CONFLICT` | Duplicate resource (e.g., email already exists) |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |
| 503 | `AI_UNAVAILABLE` | Ollama/AI service temporarily unavailable |

---

## 1. Authentication Endpoints

### POST `/auth/register`
**Auth:** None

```json
// Request
{
  "email": "user@example.com",
  "password": "SecureP@ss1",
  "full_name": "John Doe"
}

// Response 201
{
  "message": "Registration successful. Please verify your email.",
  "user_id": "550e8400-e29b-41d4-a716-446655440000"
}

// Error 409
{
  "error": "Email already registered",
  "code": "CONFLICT"
}
```

### POST `/auth/login`
**Auth:** None

```json
// Request
{
  "email": "user@example.com",
  "password": "SecureP@ss1"
}

// Response 200
{
  "access_token": "eyJhbGci...",
  "refresh_token": "dGhpcyBp...",
  "token_type": "bearer",
  "expires_in": 900,
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "full_name": "John Doe",
    "avatar_url": null,
    "role": "FREE_USER",
    "subscription_tier": "FREE",
    "email_verified": true,
    "created_at": "2026-04-17T10:00:00Z",
    "last_login_at": "2026-04-17T12:00:00Z"
  }
}

// Error 401
{
  "error": "Invalid credentials",
  "code": "UNAUTHORIZED"
}
```

### POST `/auth/oauth/{provider}`
**Auth:** None  
**Providers:** `google`, `github`

```json
// Request
{
  "code": "4/0AX4XfWh...",
  "state": "random-state-string"
}

// Response 200 — Same as login response
```

### POST `/auth/refresh`
**Auth:** Refresh token in HTTP-only cookie

```json
// Response 200
{
  "access_token": "eyJhbGci...",
  "refresh_token": "bmV3IHJl...",
  "token_type": "bearer",
  "expires_in": 900
}
```

### POST `/auth/forgot-password`
**Auth:** None

```json
// Request
{ "email": "user@example.com" }

// Response 200 (always, to prevent email enumeration)
{ "message": "If that email exists, a reset link has been sent." }
```

### POST `/auth/reset-password`
**Auth:** None

```json
// Request
{
  "token": "reset-jwt-token",
  "new_password": "NewSecure@123"
}

// Response 200
{ "message": "Password reset successful." }
```

### GET `/auth/verify?token=xxx`
**Auth:** None  
**Response:** 302 redirect to frontend login page

### POST `/auth/logout`
**Auth:** Required

```json
// Response 200
{ "message": "Logged out successfully." }
// Also: blacklists refresh token, clears HTTP-only cookie
```

---

## 2. Topic Endpoints

### GET `/topics`
**Auth:** Required  
**Query Params:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | int | 1 | Page number |
| `page_size` | int | 20 | Items per page (max 100) |
| `category` | string | null | Filter by category |
| `region` | string | null | Filter by region |
| `is_trending` | bool | null | Trending only |
| `search` | string | null | Keyword search |
| `sort_by` | string | "latest" | latest, trending, most_articles |

```json
// Response 200
{
  "items": [
    {
      "id": "topic-uuid",
      "title": "AI Regulation in the European Union",
      "slug": "ai-regulation-eu",
      "description": "The EU's comprehensive approach to regulating AI...",
      "category": "Technology",
      "region": "Europe",
      "is_trending": true,
      "article_count": 24,
      "keywords": ["AI", "regulation", "EU", "technology"],
      "first_reported_at": "2026-03-01T00:00:00Z",
      "last_updated_at": "2026-04-17T08:00:00Z",
      "thumbnail_url": "https://..."
    }
  ],
  "total": 156,
  "page": 1,
  "page_size": 20,
  "total_pages": 8,
  "has_next": true,
  "has_previous": false
}
```

### GET `/topics/trending`
**Auth:** None (limited), Required (full)  
Returns top 10 trending topics.

### GET `/topics/{topic_id}`
**Auth:** Required

```json
// Response 200
{
  "id": "topic-uuid",
  "title": "AI Regulation in the European Union",
  "slug": "ai-regulation-eu",
  "description": "...",
  "category": "Technology",
  "region": "Europe",
  "is_trending": true,
  "article_count": 24,
  "keywords": ["AI", "regulation", "EU"],
  "perspectives": {
    "left": {
      "id": "perspective-uuid",
      "bias_type": "LEFT",
      "summary": "Progressive groups argue that strong AI regulation is essential...",
      "key_arguments": [
        "AI poses existential risks that only government regulation can address",
        "Unregulated AI deepens inequality and threatens workers",
        "EU regulations protect citizens' fundamental rights"
      ],
      "sources_cited": [
        {"source_name": "The Guardian", "url": "https://...", "published_at": "2026-04-15"}
      ],
      "quality_score": 0.89,
      "model_used": "llama3.1:8b",
      "generated_at": "2026-04-17T10:00:00Z",
      "disclaimer": "⚠️ This content was generated by AI and may contain inaccuracies..."
    },
    "right": { ... },
    "neutral": { ... }
  },
  "top_articles": [ ... ],
  "first_reported_at": "2026-03-01T00:00:00Z",
  "last_updated_at": "2026-04-17T08:00:00Z"
}
```

### GET `/topics/{topic_id}/perspectives`
**Auth:** Required  
**Quota:** Free users: 5/day | Premium: unlimited

```json
// Response 200
{
  "left": { ... },
  "right": { ... },
  "neutral": { ... }
}

// Error 403 (quota exceeded)
{
  "error": "Daily limit reached for 3-Side View",
  "code": "QUOTA_EXCEEDED",
  "detail": "Upgrade to Premium for unlimited access.",
  "usage": { "used": 5, "limit": 5 }
}
```

### POST `/topics/analyze`
**Auth:** Required  
**Async:** Returns task ID, results via polling or WebSocket

```json
// Request
{
  "query": "Impact of tariffs on global trade 2026",
  "depth": "standard",
  "include_timeline": false,
  "include_impact": false
}

// Response 202
{
  "task_id": "task-uuid",
  "status": "PROCESSING",
  "estimated_time_seconds": 15,
  "poll_url": "/api/v1/tasks/task-uuid"
}
```

### GET `/tasks/{task_id}`
**Auth:** Required

```json
// Response 200 (processing)
{
  "task_id": "task-uuid",
  "status": "PROCESSING",
  "progress": 0.6,
  "current_step": "Generating summaries..."
}

// Response 200 (complete)
{
  "task_id": "task-uuid",
  "status": "COMPLETE",
  "result": {
    "topic": { ... },
    "perspectives": { ... }
  }
}
```

### GET `/topics/{topic_id}/timeline`
**Auth:** Required (Premium only)

```json
// Response 200
{
  "topic_id": "topic-uuid",
  "topic_title": "AI Regulation in the EU",
  "events": [
    {
      "id": "event-uuid",
      "title": "EU AI Act Proposed",
      "description": "The European Commission proposed the AI Act...",
      "event_date": "2021-04-21",
      "sources": ["https://..."],
      "importance_score": 9
    },
    ...
  ],
  "generated_at": "2026-04-17T10:00:00Z",
  "disclaimer": "⚠️ This timeline was generated by AI..."
}
```

### GET `/topics/{topic_id}/impact`
**Auth:** Required (Premium only)

```json
// Response 200
{
  "topic_id": "topic-uuid",
  "topic_title": "AI Regulation in the EU",
  "overall_impact_score": 7.5,
  "dimensions": [
    {
      "name": "Economic",
      "score": 8.0,
      "description": "Significant impact on tech companies operating in EU...",
      "key_indicators": ["GDP impact", "Job market shifts", "Investment flows"]
    },
    ...
  ],
  "personal_impact": "If you work in tech, these regulations may affect...",
  "generated_at": "2026-04-17T10:00:00Z",
  "disclaimer": "⚠️ This impact analysis was generated by AI..."
}
```

---

## 3. Article Endpoints

### GET `/articles`
**Auth:** Required  
**Query Params:** Similar pagination + `topic_id`, `bias_label`, `source_id`

### GET `/articles/{article_id}`
**Auth:** Required

---

## 4. Debate Endpoints

### POST `/debate/start`
**Auth:** Required  
**Quota:** Free: 2/day | Premium: unlimited

```json
// Request
{
  "topic_id": "topic-uuid",
  "custom_topic": null
}

// Response 201
{
  "id": "session-uuid",
  "topic_title": "AI Regulation in the EU",
  "status": "ACTIVE",
  "message_count": 0,
  "started_at": "2026-04-17T12:00:00Z",
  "websocket_url": "wss://api.bothsidesofacoin.com/api/v1/debate/session-uuid/stream"
}
```

### POST `/debate/{session_id}/message`
**Auth:** Required

```json
// Request
{
  "content": "Should AI be regulated by government?",
  "target_agent": "all"
}

// Response 200
{
  "user_message": {
    "id": "msg-uuid-1",
    "sender": "USER",
    "content": "Should AI be regulated by government?",
    "sequence_number": 1,
    "created_at": "2026-04-17T12:01:00Z"
  },
  "ai_responses": [
    {
      "id": "msg-uuid-2",
      "sender": "LEFT_AI",
      "content": "Absolutely. Without regulation, AI will...",
      "sequence_number": 2,
      "created_at": "2026-04-17T12:01:05Z",
      "disclaimer": "⚠️ AI-generated content..."
    },
    {
      "id": "msg-uuid-3",
      "sender": "RIGHT_AI",
      "content": "Government regulation stifles innovation...",
      "sequence_number": 3,
      "created_at": "2026-04-17T12:01:05Z",
      "disclaimer": "⚠️ AI-generated content..."
    },
    {
      "id": "msg-uuid-4",
      "sender": "NEUTRAL_AI",
      "content": "The debate around AI regulation involves...",
      "sequence_number": 4,
      "created_at": "2026-04-17T12:01:05Z",
      "disclaimer": "⚠️ AI-generated content..."
    }
  ]
}
```

### WebSocket `/debate/{session_id}/stream`
**Auth:** Token as query parameter or first message

```json
// Client → Server
{ "type": "message", "content": "What about privacy?", "target_agent": "all" }

// Server → Client (streaming tokens)
{ "type": "token", "sender": "LEFT_AI", "content": "Privacy", "sequence_number": 5 }
{ "type": "token", "sender": "LEFT_AI", "content": " is", "sequence_number": 5 }
{ "type": "token", "sender": "LEFT_AI", "content": " a fundamental", "sequence_number": 5 }
...
{ "type": "message_complete", "sender": "LEFT_AI", "content": "Privacy is a fundamental...", "sequence_number": 5 }

// Parallel for RIGHT_AI and NEUTRAL_AI
{ "type": "token", "sender": "RIGHT_AI", "content": "While", "sequence_number": 6 }
...

// Session end
{ "type": "session_end", "sender": "SYSTEM", "content": "Debate session ended." }
```

### GET `/debate/{session_id}/messages`
**Auth:** Required

### GET `/debate/sessions`
**Auth:** Required (user's own sessions)

### POST `/debate/{session_id}/end`
**Auth:** Required

---

## 5. Bias Score Endpoints

### POST `/user/reading-event`
**Auth:** Required

```json
// Request
{
  "article_id": "article-uuid",
  "read_duration_seconds": 120,
  "scroll_depth": 0.85
}

// Response 201
{ "message": "Reading event recorded." }
```

### GET `/user/bias-score`
**Auth:** Required

```json
// Response 200
{
  "score": 72.5,
  "direction": "LEFT",
  "left_percentage": 65.0,
  "right_percentage": 15.0,
  "neutral_percentage": 20.0,
  "articles_analyzed": 87,
  "computed_at": "2026-04-17T12:00:00Z",
  "recommendations": [
    {
      "message": "You've been reading predominantly left-leaning content. Try exploring conservative perspectives on economic policy.",
      "suggested_bias_type": "RIGHT",
      "topic_id": "topic-uuid",
      "article_id": null
    }
  ],
  "interpretation": "Your reading is significantly left-leaning. Consider diversifying."
}
```

### GET `/user/bias-history`
**Auth:** Required  
**Query:** `?days=30`

```json
// Response 200
{
  "scores": [
    {
      "score": 75.0,
      "direction": "LEFT",
      "left_percentage": 68.0,
      "right_percentage": 12.0,
      "neutral_percentage": 20.0,
      "computed_at": "2026-03-18T00:00:00Z"
    },
    ...
  ],
  "trend": "IMPROVING",
  "average_score": 68.3
}
```

### GET `/user/reading-history`
**Auth:** Required  
**Query:** Paginated + date range filters

---

## 6. User Profile Endpoints

### GET `/user/profile`
**Auth:** Required

### PUT `/user/profile`
**Auth:** Required

```json
// Request
{
  "full_name": "John Doe Updated",
  "avatar_url": "https://..."
}
```

### PUT `/user/preferences`
**Auth:** Required

```json
// Request
{
  "preferred_categories": ["Technology", "Politics"],
  "preferred_regions": ["North America", "Europe"],
  "theme": "dark",
  "notification_bias_alerts": true,
  "notification_weekly_report": true
}
```

### DELETE `/user/account`
**Auth:** Required (re-authentication required)

---

## 7. Subscription Endpoints

### GET `/subscription/plans`
**Auth:** None

```json
// Response 200
{
  "plans": [
    {
      "id": "premium_monthly",
      "name": "Premium Monthly",
      "tier": "PREMIUM",
      "price_cents": 999,
      "currency": "USD",
      "interval": "month",
      "features": ["Unlimited 3-Side Views", "AI Debate", ...]
    },
    {
      "id": "premium_annual",
      "name": "Premium Annual",
      "tier": "PREMIUM",
      "price_cents": 9590,
      "currency": "USD",
      "interval": "year",
      "savings_percentage": 20
    },
    ...
  ]
}
```

### POST `/subscription/checkout`
**Auth:** Required

```json
// Request
{
  "plan": "premium_monthly",
  "success_url": "https://bothsidesofacoin.com/subscription/success",
  "cancel_url": "https://bothsidesofacoin.com/subscription/cancel"
}

// Response 200
{
  "checkout_url": "https://checkout.stripe.com/c/pay/...",
  "session_id": "cs_live_..."
}
```

### POST `/subscription/webhook`
**Auth:** Stripe signature verification (not JWT)

### GET `/subscription/status`
**Auth:** Required

### POST `/subscription/cancel`
**Auth:** Required

---

## 8. Admin Endpoints

### GET `/admin/analytics/overview`
**Auth:** ADMIN or SUPER_ADMIN

```json
// Response 200
{
  "total_users": 15234,
  "active_users_24h": 2341,
  "premium_subscribers": 456,
  "total_topics": 892,
  "total_analyses_today": 3421,
  "total_debates_today": 567,
  "ai_processing_avg_ms": 4500,
  "error_rate_24h": 0.02
}
```

### GET `/admin/users`
**Auth:** ADMIN+  
**Query:** Paginated, filterable by role/status/tier

### PATCH `/admin/users/{user_id}`
**Auth:** ADMIN+

### GET `/admin/sources`
### POST `/admin/sources`
### PATCH `/admin/sources/{source_id}`
**Auth:** ADMIN+

---

## 9. Health & Utility Endpoints

### GET `/health`
**Auth:** None

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "services": {
    "database": "up",
    "redis": "up",
    "ollama": "up",
    "news_api": "up"
  },
  "timestamp": "2026-04-17T12:00:00Z"
}
```

---

## Rate Limits

| Tier | Requests/Minute | AI Requests/Minute |
|------|-----------------|-------------------|
| Anonymous | 30 | 0 |
| Free | 60 | 10 |
| Premium | 120 | 30 |
| Enterprise | 300 | 100 |
| Admin | 300 | 100 |

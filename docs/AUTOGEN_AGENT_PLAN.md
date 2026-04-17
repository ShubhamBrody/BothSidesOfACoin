# AutoGen 0.4 Agent Orchestration — Comprehensive Plan
## BothSidesOfACoin

---

## 1. Why AutoGen 0.4?

Microsoft AutoGen 0.4 is a complete rewrite of the AutoGen framework with:
- **Asynchronous-first** architecture
- **SelectorGroupChat** — LLM-based dynamic agent selection (no hardcoded pipelines)
- **Tool-augmented agents** — Agents can call Python functions as tools
- **Streaming support** — Token-by-token streaming for real-time UX
- **Observation/handoff pattern** — Agents observe all messages and self-select when relevant
- **Termination conditions** — Flexible conversation ending (text match, max messages, custom)

This is exactly what we need: **the orchestrator doesn't follow a script; it reasons about what to do next.**

---

## 2. Core Design: No Hardcoded Pathways

### Traditional Approach (What We're NOT Doing)
```python
# ❌ HARDCODED PIPELINE — BAD
result = await news_collector.run(topic)
classified = await bias_classifier.run(result)
summary = await summarizer.run(classified)
return summary
```

### Our Approach: Dynamic Agent Selection
```python
# ✅ DYNAMIC ORCHESTRATION — GOOD
# The LLM decides which agent to call next based on conversation context
team = SelectorGroupChat(
    participants=[orchestrator, news_collector, bias_classifier, ...],
    model_client=ollama_client,
    selector_prompt="Based on the conversation context, which agent should act next?",
    termination_condition=final_answer_termination
)
result = await team.run(task=user_request)
```

The Selector (an LLM call) reads the full conversation history and picks the next agent. This means:
- For a simple bias check → it may skip news collection entirely
- For a deep analysis → it calls more agents in sequence
- For a debate → it only calls debate agents
- If an agent fails → the orchestrator can route to an alternative

---

## 3. Agent Definitions

### 3.1 OrchestratorAgent

**Role:** The "brain" that plans and coordinates all other agents.

```python
orchestrator = AssistantAgent(
    name="Orchestrator",
    model_client=ollama_client,
    system_message="""You are the Orchestrator of BothSidesOfACoin platform.

Your responsibilities:
1. Analyze the user's request and determine its type
2. Create an execution plan by deciding which agents to invoke
3. Coordinate data flow between agents
4. Compile the final response

Request types you handle:
- TOPIC_ANALYSIS: User wants multi-perspective analysis of a news topic
- DEBATE: User wants to start/continue an AI debate
- BIAS_CHECK: User wants their reading bias analyzed
- TIMELINE: User wants event timeline for a topic
- IMPACT: User wants impact analysis
- SEARCH: User wants to find specific news

Available agents: NewsCollector, BiasClassifier, Summarizer, FactExtractor, 
TimelineBuilder, ImpactAnalyzer, DebateLeft, DebateRight, DebateNeutral, 
UserBiasAnalyzer, QualityGuard

Rules:
- Always start by identifying the request type
- Call agents in logical order (collect before classify, classify before summarize)
- Always route outputs through QualityGuard before final delivery
- If an agent produces poor results, ask it to retry with specific feedback
- When all required information is gathered, compile and deliver the response
- Use TERMINATE when the task is complete
""",
    tools=[
        create_execution_plan,
        store_intermediate_result,
        compile_final_response,
        check_user_subscription,
    ]
)
```

### 3.2 NewsCollectorAgent

```python
news_collector = AssistantAgent(
    name="NewsCollector",
    model_client=ollama_client,
    system_message="""You are the News Collector agent. Your job is to find and organize 
news articles on a given topic.

When activated:
1. Use the search_news tool to find articles from multiple sources
2. Deduplicate similar articles
3. Extract key metadata (title, source, date, content snippet)
4. Return a structured list of articles

Always try to find articles from diverse sources (left-leaning, right-leaning, neutral).
Target: minimum 6 articles (2 per perspective) when available.
""",
    tools=[
        search_news_api,
        search_gdelt,
        search_cached_articles,
        deduplicate_articles,
    ]
)
```

### 3.3 BiasClassifierAgent

```python
bias_classifier = AssistantAgent(
    name="BiasClassifier",
    model_client=ollama_client,
    system_message="""You are the Bias Classifier agent. You analyze text content and 
classify its political leaning.

Classification labels: LEFT, RIGHT, NEUTRAL

Analysis criteria:
- Language tone and framing
- Source reputation and known bias
- Emotional vs factual language ratio
- Loaded terms and dog-whistles
- Omission of counterarguments
- Attribution patterns

For each article, provide:
- bias_label: LEFT | RIGHT | NEUTRAL
- confidence: 0.0 to 1.0
- reasoning: Brief explanation
- language_indicators: Key phrases that indicate bias

Be calibrated and consistent. CENTER-LEFT and CENTER-RIGHT should map to their 
respective sides but with lower confidence scores.
""",
    tools=[
        classify_text_bias,
        check_source_bias_database,
        analyze_sentiment,
    ]
)
```

### 3.4 SummarizerAgent

```python
summarizer = AssistantAgent(
    name="Summarizer",
    model_client=ollama_client,
    system_message="""You are the Summarizer agent. You create multi-perspective summaries.

When given classified articles, generate THREE summaries:

1. LEFT PERSPECTIVE Summary:
   - Synthesize the key arguments from left-leaning sources
   - Maintain the progressive framing
   - Include key concerns and values

2. RIGHT PERSPECTIVE Summary:
   - Synthesize the key arguments from right-leaning sources
   - Maintain the conservative framing
   - Include key concerns and values

3. NEUTRAL/FACTUAL Summary:
   - Strip all editorial bias
   - Focus on verified facts only
   - Present both sides' strongest arguments fairly
   - Highlight areas of consensus

Each summary should be 150-300 words, well-structured, and cite sources.
Include a key_arguments list (3-5 bullet points per perspective).

CRITICAL: End every summary with the disclaimer that this is AI-generated content.
""",
    tools=[
        generate_perspective_summary,
        extract_key_arguments,
        cite_sources,
    ]
)
```

### 3.5 FactExtractorAgent

```python
fact_extractor = AssistantAgent(
    name="FactExtractor",
    model_client=ollama_client,
    system_message="""You are the Fact Extractor agent. You extract verifiable claims, 
statistics, and factual statements from articles.

For each article:
1. Extract factual claims (who, what, when, where)
2. Identify statistics and numbers
3. Flag claims that appear unverified or disputed
4. Note which facts appear across multiple sources (higher credibility)

Output structured data with:
- claim: The factual statement
- source_count: How many sources report this
- confidence: How verifiable this seems
- category: VERIFIED | DISPUTED | UNVERIFIED
""",
    tools=[
        extract_claims,
        cross_reference_facts,
    ]
)
```

### 3.6 TimelineBuilderAgent

```python
timeline_builder = AssistantAgent(
    name="TimelineBuilder",
    model_client=ollama_client,
    system_message="""You are the Timeline Builder agent. You create chronological 
timelines of events for a given topic.

When activated:
1. Extract date-stamped events from articles
2. Order chronologically
3. Identify cause-effect relationships
4. Rate importance (1-10) for each event
5. Link to source articles

Output a clean, ordered timeline with:
- event_date: When it happened
- title: Short event title
- description: What happened (2-3 sentences)
- importance_score: 1-10
- sources: Which articles mention this
""",
    tools=[
        extract_timeline_events,
        order_chronologically,
        link_cause_effect,
    ]
)
```

### 3.7 ImpactAnalyzerAgent

```python
impact_analyzer = AssistantAgent(
    name="ImpactAnalyzer",
    model_client=ollama_client,
    system_message="""You are the Impact Analyzer agent. You analyze how news topics 
affect people's lives across multiple dimensions.

Dimensions to analyze:
1. ECONOMIC: Jobs, prices, markets, trade
2. SOCIAL: Communities, culture, daily life
3. POLITICAL: Governance, policies, rights
4. ENVIRONMENTAL: Climate, resources, sustainability

For each dimension:
- Score impact severity (1-10)
- Describe specific effects
- List key indicators

Also provide a "personal impact" statement: "How this affects YOU" in plain language.
""",
    tools=[
        analyze_economic_impact,
        analyze_social_impact,
        generate_personal_impact,
    ]
)
```

### 3.8 Debate Agents (Left, Right, Neutral)

```python
debate_left = AssistantAgent(
    name="DebateLeft",
    model_client=ollama_client,
    system_message="""You are the Left/Progressive AI debater. You argue from a 
progressive perspective on any topic.

Your values: Social justice, equality, government regulation when needed, 
environmental protection, workers' rights, inclusive policies.

Rules:
- Stay in character as a thoughtful progressive voice
- Use evidence and logical arguments, not just emotional appeals
- Acknowledge valid points from the other side when appropriate
- Never use hate speech or extreme rhetoric
- Keep responses focused and under 200 words
- Always end with a disclaimer that this is AI-generated debate content
"""
)

debate_right = AssistantAgent(
    name="DebateRight",
    model_client=ollama_client,
    system_message="""You are the Right/Conservative AI debater. You argue from a 
conservative perspective on any topic.

Your values: Individual liberty, free markets, limited government, 
traditional institutions, national security, personal responsibility.

Rules:
- Stay in character as a thoughtful conservative voice
- Use evidence and logical arguments, not just emotional appeals
- Acknowledge valid points from the other side when appropriate
- Never use hate speech or extreme rhetoric
- Keep responses focused and under 200 words
- Always end with a disclaimer that this is AI-generated debate content
"""
)

debate_neutral = AssistantAgent(
    name="DebateNeutral",
    model_client=ollama_client,
    system_message="""You are the Neutral/Factual AI analyst. You provide balanced, 
fact-based analysis during debates.

Your role:
- Present verified facts without editorial bias
- Highlight where both sides have valid points
- Correct misinformation from either side
- Provide historical context and data
- Note areas of genuine uncertainty

Rules:
- Never take a side
- Always cite data and evidence when possible
- Acknowledge complexity and nuance
- Keep responses under 200 words
- Always end with a disclaimer that this is AI-generated content
"""
)
```

### 3.9 UserBiasAnalyzerAgent

```python
user_bias_analyzer = AssistantAgent(
    name="UserBiasAnalyzer",
    model_client=ollama_client,
    system_message="""You are the User Bias Analyzer. You analyze reading patterns 
to compute a user's ideological bias score.

When activated:
1. Fetch the user's reading history (article reads with bias labels)
2. Calculate distribution: % LEFT / % RIGHT / % NEUTRAL
3. Apply time-decay weighting (recent reads matter more)
4. Compute Bias Score: 0 (perfectly balanced) to 100 (extremely one-sided)
5. Determine direction: LEFT, RIGHT, or BALANCED
6. Generate personalized recommendations

Recommendations should:
- Be constructive, not judgmental
- Suggest specific articles from underrepresented perspectives
- Encourage curiosity, not guilt
""",
    tools=[
        fetch_reading_history,
        compute_bias_score,
        generate_recommendations,
    ]
)
```

### 3.10 QualityGuardAgent

```python
quality_guard = AssistantAgent(
    name="QualityGuard",
    model_client=ollama_client,
    system_message="""You are the Quality Guard. You review ALL AI-generated content 
before it reaches users.

Check for:
1. FACTUAL ACCURACY: Are claims supported? Any obvious hallucinations?
2. BALANCE: Are all perspectives fairly represented?
3. SAFETY: Any harmful, hateful, or inappropriate content?
4. QUALITY: Is the writing clear, coherent, and well-structured?
5. CONSISTENCY: Do the three perspective summaries actually differ?

Output:
- approved: true/false
- issues: List of problems found
- suggestions: How to improve
- safety_score: 0.0 to 1.0

If safety_score < 0.7, REJECT immediately.
If quality issues found, provide specific feedback for the originating agent.
""",
    tools=[
        check_factual_consistency,
        check_content_safety,
        validate_perspective_balance,
    ]
)
```

---

## 4. Dynamic Orchestration: SelectorGroupChat

The key AutoGen 0.4 pattern we use is `SelectorGroupChat`. Instead of predefined agent chains, an LLM reads the full conversation and selects which agent should speak next.

### Implementation

```python
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination
from autogen_ext.models.ollama import OllamaChatCompletionClient

# 1. Initialize Ollama client
ollama_client = OllamaChatCompletionClient(
    model="llama3.1:8b",  # or mistral, qwen2.5
    api_base="http://localhost:11434",
    temperature=0.3,  # Lower for more consistent behavior
    num_predict=2048,
)

# 2. Create all agents (as defined above)
agents = [
    orchestrator,
    news_collector,
    bias_classifier,
    summarizer,
    fact_extractor,
    timeline_builder,
    impact_analyzer,
    debate_left,
    debate_right,
    debate_neutral,
    user_bias_analyzer,
    quality_guard,
]

# 3. Termination conditions
termination = (
    TextMentionTermination("TASK_COMPLETE") |  # Orchestrator says done
    MaxMessageTermination(max_messages=25)       # Safety limit
)

# 4. Selector prompt — this is the "brain" of dynamic routing
SELECTOR_PROMPT = """You are the agent selector for BothSidesOfACoin platform.
Read the conversation carefully and decide which agent should respond next.

Available agents:
- Orchestrator: Plans execution, compiles final results
- NewsCollector: Fetches news articles from APIs
- BiasClassifier: Classifies political bias of text
- Summarizer: Creates multi-perspective summaries
- FactExtractor: Extracts verifiable claims
- TimelineBuilder: Creates event timelines
- ImpactAnalyzer: Analyzes global/personal impact
- DebateLeft: Left/progressive perspective in debates
- DebateRight: Right/conservative perspective in debates
- DebateNeutral: Neutral/factual analysis in debates
- UserBiasAnalyzer: Analyzes user reading bias
- QualityGuard: Reviews content quality and safety

Decision rules:
1. Always start with Orchestrator to plan
2. Follow the Orchestrator's plan for agent ordering
3. After any content generation, route to QualityGuard
4. After QualityGuard approval, route back to Orchestrator to compile or continue
5. For debate sessions, alternate between DebateLeft, DebateRight, DebateNeutral
6. If an agent's output is rejected by QualityGuard, route back to that agent

Select the most appropriate next agent.
"""

# 5. Create the team
team = SelectorGroupChat(
    participants=agents,
    model_client=ollama_client,
    selector_prompt=SELECTOR_PROMPT,
    termination_condition=termination,
    allow_repeated_speaker=True,  # QualityGuard may need to re-check
)

# 6. Run for a given task
async def process_request(user_request: str, context: dict) -> str:
    task = f"""
    User Request: {user_request}
    Context: {json.dumps(context)}
    
    Process this request using the appropriate agents.
    End with TASK_COMPLETE when done.
    """
    result = await team.run(task=task)
    return extract_final_response(result)
```

---

## 5. Tool Definitions (Agent Capabilities)

Each agent gets Python functions registered as "tools" that they can call:

```python
from autogen_core.tools import FunctionTool

# ── NewsCollector Tools ──
async def search_news_api(query: str, max_results: int = 10) -> list[dict]:
    """Search NewsAPI for articles matching the query."""
    async with httpx.AsyncClient() as client:
        response = await client.get(
            "https://newsapi.org/v2/everything",
            params={"q": query, "pageSize": max_results, "apiKey": settings.NEWS_API_KEY},
        )
        return response.json()["articles"]

async def search_cached_articles(query: str) -> list[dict]:
    """Search previously cached articles in PostgreSQL."""
    ...

# ── BiasClassifier Tools ──
async def classify_text_bias(text: str) -> dict:
    """Run bias classification on a text passage using the LLM."""
    ...

async def check_source_bias_database(source_name: str) -> dict:
    """Look up known bias rating for a news source."""
    ...

# ── Summarizer Tools ──
async def generate_perspective_summary(
    articles: list[dict], perspective: str
) -> str:
    """Generate a summary from a specific perspective (LEFT/RIGHT/NEUTRAL)."""
    ...

# ── Register tools with agents ──
news_collector_tools = [
    FunctionTool(search_news_api, description="Search NewsAPI for articles"),
    FunctionTool(search_cached_articles, description="Search cached articles"),
]
```

---

## 6. Request Type → Agent Flow Matrix

The orchestrator dynamically picks from these patterns, but here's the general reasoning:

| Request Type | Likely Agent Sequence | Notes |
|-------------|----------------------|-------|
| **TOPIC_ANALYSIS** | Orchestrator → NewsCollector → BiasClassifier → [Summarizer ∥ FactExtractor] → QualityGuard → Orchestrator | Parallel where possible |
| **DEEP_ANALYSIS** | Orchestrator → NewsCollector → BiasClassifier → [Summarizer ∥ FactExtractor ∥ TimelineBuilder ∥ ImpactAnalyzer] → QualityGuard → Orchestrator | Premium feature |
| **DEBATE** | Orchestrator → [DebateLeft ∥ DebateRight ∥ DebateNeutral] → QualityGuard | Per message cycle |
| **BIAS_CHECK** | Orchestrator → UserBiasAnalyzer → Orchestrator | Quick flow |
| **TIMELINE** | Orchestrator → NewsCollector → TimelineBuilder → QualityGuard → Orchestrator | Premium feature |
| **IMPACT** | Orchestrator → NewsCollector → ImpactAnalyzer → QualityGuard → Orchestrator | Premium feature |
| **SEARCH** | Orchestrator → NewsCollector → BiasClassifier → Orchestrator | Quick flow |

**Key:** These are NOT hardcoded. The orchestrator's LLM reasoning determines the actual flow each time. Edge cases, retries, and alternative paths are decided dynamically.

---

## 7. Error Handling & Resilience

### Agent Failure Recovery

```python
# The orchestrator handles failures via its system prompt:
"""
If an agent fails or produces poor results:
1. Check if the failure is retryable (timeout, rate limit)
2. If retryable: re-invoke the agent with modified input
3. If not retryable: skip the step and note it in the response
4. If a critical agent fails (NewsCollector, Summarizer): 
   fall back to cached data
5. Maximum 2 retries per agent per request
6. Always inform the user if results are degraded
"""
```

### Timeout Management

```python
# Per-agent timeout configuration
AGENT_TIMEOUTS = {
    "NewsCollector": 15,   # External API calls
    "BiasClassifier": 10,  # LLM analysis
    "Summarizer": 20,      # Long generation
    "FactExtractor": 10,
    "TimelineBuilder": 15,
    "ImpactAnalyzer": 15,
    "DebateLeft": 10,
    "DebateRight": 10,
    "DebateNeutral": 10,
    "UserBiasAnalyzer": 5,
    "QualityGuard": 10,
}
```

### Circuit Breaker Pattern

```python
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=60)
async def call_ollama(prompt: str) -> str:
    """If Ollama fails 5 times, open circuit for 60s."""
    ...
```

---

## 8. Observability & Tracing

Every agent execution is logged for debugging and analytics:

```python
class AgentTrace(BaseModel):
    request_id: str
    agent_name: str
    step_number: int
    input_summary: str
    output_summary: str
    success: bool
    error: str | None
    execution_time_ms: int
    tokens_used: int
    model_used: str
    timestamp: datetime

# Stored in PostgreSQL for analytics
# Exposed via admin API for debugging
```

---

## 9. Scaling the Agent Layer

### Single-Instance (Hackathon/Dev)
```
FastAPI → AutoGen Team → Ollama (single GPU/CPU)
```

### Production Scaling
```
FastAPI (N instances)
    ↓
Celery Task Queue (Redis broker)
    ↓
Celery Workers (N instances, each with AutoGen team)
    ↓
Ollama Cluster (M GPU instances, load-balanced)
```

### Ollama Model Management
```python
# Use different models for different agents based on task complexity
MODEL_ASSIGNMENTS = {
    "Orchestrator": "llama3.1:8b",       # Good reasoning
    "BiasClassifier": "llama3.1:8b",     # Good classification
    "Summarizer": "mistral:7b",          # Good writing
    "FactExtractor": "llama3.1:8b",      # Good extraction
    "DebateLeft": "mistral:7b",          # Good creative writing
    "DebateRight": "mistral:7b",         # Good creative writing
    "DebateNeutral": "llama3.1:8b",      # Good factual analysis
    "QualityGuard": "llama3.1:8b",       # Good evaluation
}
```

---

## 10. Hackathon MVP: Minimal Agent Setup

For the hackathon, start with 5 core agents:

1. **Orchestrator** — Plans and routes
2. **NewsCollector** — Fetches articles (can use static data for demo)
3. **BiasClassifier** — Classifies bias
4. **Summarizer** — Generates 3 perspectives
5. **QualityGuard** — Validates output

Add DebateLeft/Right/Neutral for the debate feature demo.

This gives you the full dynamic orchestration feel with minimal setup.

---
layout: default
title: "Production Agentic System Design"
toc: true
toc_min_level: 2
toc_max_level: 3
description: "Production agentic system design: interview story, architecture, implementation stack, ML research।"
---

*AI Engineering · System Design · Machine Learning*

# AI Engineer Interview-এ যা হয়: Production Agentic System Design

Components জানলেই হয় না। কোথায় connect করবে, কেন connect করবে,
সেটা জানতে হয়। এই post-এ interview-এর গল্প, system diagram (Part 1),
আর implementation ও stack (Part 2)।

*Asif Bin Syed · ML Researcher · OMSCS @ Georgia Tech · May 2026*

**Part 1**: [Interview story ও architecture](#part-1-interview-story-ও-architecture) ·
**Part 2**: [Implementation ও stack](#part-2-implementation-ও-stack) ·
**Deep dives**: [Gateway](#llm-gateway-deep-dive) · [Observability](#observability-layer-deep-dive) · [Orchestrator](#orchestrator-deep-dive) · [Memory](#memory-layer-deep-dive) · [Sub-agents](#sub-agents-deep-dive) · [Tool registry](#tool-registry-deep-dive) · [MCP](#mcp-deep-dive) · [Skills](#skills-deep-dive) · [MCP](#mcp-deep-dive) · [Skills](#skills-deep-dive)

---

## Part 1: Interview story ও architecture

কখনো মনে হয়েছে AI Engineering job market-এ আসলে কি ধরণের কাজের
চাহিদা সবচেয়ে বেশি? Claude এখন এক prompt-এ UI বানিয়ে দেয়, right?
তাহলে Senior AI Engineers-রা large companies-এ **$200K, $400K salary**
নিয়ে আসলে করেটা কি?

উত্তরটা simple: **system design**। Infrastructure। Scalability।
Millions of users handle করা। Frontend বা একটা chatbot বানানো এখন AI
করে দেয়। কিন্তু সেই chatbot যখন 10 million user একসাথে hit করে,
তখন কী হয়? কীভাবে সেটা handle করবে? এই প্রশ্নের উত্তরের জন্যই
companies এত টাকা দেয়।

> "That's what they were actually testing, not whether I could drag boxes.
> Whether I understood the trade-offs well enough to defend my choices
> under pushback."

LinkedIn-এ একটা post দেখলাম। একজন AI Engineer interview দিতে গিয়ে
blank canvas পেয়েছে। বলে দিয়েছে: *"Design a production-grade agentic
system. Components are on the left. Go."*

Components গুলো ছিল: `Orchestrator`, `Sub-agents`, `Memory module`,
`Tool registry`, `Vector store`, `LLM gateway`, `Observability layer`,
`Guardrails`। কোনো instruction নেই। No hints।

## System Diagram

{% include agentic-system-diagram.html %}

## Component Breakdown

এই diagram-এর প্রতিটা component একটা specific কারণে ওখানে আছে।
Random না। চলো প্রতিটা layer কেন সেখানে সেটা বুঝি।

**LLM Gateway** সবার আগে কারণ এটা একটা bouncer। কোন model-এ request
যাবে, rate limit কত, কত token খরচ হলো, সব এখানে। এটা ছাড়া system
uncontrollable।

**Observability Layer** পুরো system জুড়ে active। Production-এ কিছু
break করলে, observability না থাকলে তুমি জানবেই না কোথায় problem।
এটা CCTV-র মতো। সবসময় চলছে।

**Orchestrator** হলো আসল brain। এটা task নেয়, ভাঙে, route করে,
আর state track রাখে। এখানে সবচেয়ে complex logic থাকে।

## The Key Interview Question

Interview-এ সবচেয়ে interesting প্রশ্ন ছিল: *"Memory কোথায় থাকবে?"*
Interviewer মেয়েটির vector store orchestrator-এ রাখা দেখে জিজ্ঞেস করলো,
*"Why not at the sub-agent level?"*

### Memory placement: trade-off

**Centralised (Orchestrator-এ)**  
সব agent same info দেখবে। Consistent। কিন্তু একটু slow, central
store-এ যেতে হয়।

**Distributed (Sub-agent-এ)**  
Fast, local memory access। কিন্তু Agent A আর Agent B আলাদা info দেখতে
পারে। Sync complex।

দুইটাই valid। কিন্তু interview-এ তোমাকে জানতে হবে তুমি কোনটা কেন
choose করছ। **Defend করতে হবে।** সেটাই আসল test।

## পুরো Flow: Step by Step

1. **User request আসে**: Web app, mobile, বা API client থেকে LLM
   Gateway-এ hit করে।
2. **Gateway route করে**: কোন model? Rate limit ঠিক আছে? Cost track
   হলো। তারপর পাস করে।
3. **Observability trace শুরু**: প্রতিটা hop track হচ্ছে। Latency
   measure হচ্ছে। Logs লেখা হচ্ছে।
4. **Orchestrator task plan করে**: বড় task ভেঙে sub-tasks বানায়।
   কোন agent কী করবে decide করে।
5. **Memory context দেয়**: Vector store থেকে relevant past context
   retrieve হয়। Agent aware হয় কী হয়েছিল।
6. **Sub-agents কাজ করে**: Research, Code, Analysis agent নিজের
   task execute করে।
7. **Tool Registry-র মাধ্যমে**: agents direct tool call করে না।
   Registry permission check করে, তারপর tool call হয়।
8. **Guardrails filter করে**: input এবং output দুইটাই filter। PII
   redact। Policy check। তারপর response user-এ ফেরে।

> System design মানে just boxes draw করা না। মানে হলো প্রতিটা
> connection-এর কারণ বোঝা, প্রতিটা trade-off defend করা। Not tools.
> **Decisions।**

Part 1 এখানেই শেষ। Part 2-এ stack overview, gateway, observability,
orchestrator, memory, sub-agents, tool registry, MCP, skills deep dives, আর ML research
mapping।

---

## Part 2: Implementation ও stack

Part 1-এ *কী* আর *কেন* দেখলাম। Part 2-এ *কীভাবে build করবে* এবং
production-এ কী রাখবে সেটা। Interview-এ box draw করার পরে প্রায়ই
আসে: "Okay, what would you actually use?"

### LLM Gateway (overview)

Gateway মানে একটা thin control plane। সব model call এখান দিয়ে যায়:
auth, routing, rate limits, cost, fallback। Part 1 diagram-এ এটা
সবার আগে কারণ এটা **bouncer**।

Managed: **LiteLLM**, **Portkey**। Custom: **FastAPI** + Redis।
Interview-এ gateway ছাড়া cost explode হয়: এটা defend করো।

নিচে gateway deep dive: পাঁচটা core component, routing table,
fallback chain, LiteLLM + FastAPI code।

---

## LLM Gateway (deep dive)

*Series: Production Agentic System · Gateway focus*

শুক্রবার রাত। Dashboard খুললে monthly LLM bill **$12,400**। গত মাসে
ছিল $800। কোন team? কোন feature? কোন agent? জানো না। কারণ প্রতিটা
service সরাসরি OpenAI API hit করছে। কোনো central gate নেই।

এটাই gateway না থাকলে হয়। Interview canvas-এ gateway বামে থাকে
কারণ **সব traffic এখান দিয়ে যায়**। Observability দেখে কী হয়েছে।
Gateway decide করে **কোন model, কত খরচ, pass না block**।

> Gateway হলো nightclub-এর bouncer। ID check (auth), guest list
> (rate limit), VIP lane (premium model), bar tab (cost cap)। ভিতরে
> ঢুকার আগে সব এখানে।

### Gateway-এর পাঁচটা core job

Part 1 diagram-এর sub-boxes আসলে পাঁচটা responsibility:

{% include gateway-diagram.html %}

1. **Auth + rate limit**: কে call করছে, কত বার, quota আছে কিনা।
2. **Router**: task type, latency budget, cost tier অনুযায়ী model।
3. **Fallback chain**: primary fail হলে backup model বা provider।
4. **Provider call**: unified API (OpenAI, Anthropic, Azure, local)।
5. **Cost ledger**: token in/out, $ per request, per tenant, per agent।

এই পাঁচটা এক জায়গায় না থাকলে প্রতিটা agent নিজে model pick করে,
budget track করে না, outage-এ crash করে।

### Model routing: কখন কোন model

Routing মানে শুধু "GPT-4 vs GPT-3.5" না। Policy-driven selection:

| Signal | Route to | কেন |
|--------|----------|-----|
| Intent classify, short reply | `gpt-4o-mini` / Haiku | সস্তা, fast |
| Multi-step reasoning, code | `gpt-4o` / Sonnet | quality |
| Long context (>100k) | Gemini / Claude long | window |
| Offline / privacy | local vLLM | data stays in VPC |
| High-volume batch | cheapest available | cost at scale |

**Rule-based routing** (production day 1): tag on request
`task_type=classify` → cheap model। `task_type=synthesis` → strong model।

**LLM-based routing** (later): ছোট classifier model intent detect করে।
Extra latency + cost, কিন্তু flexible।

> **Interview answer:** "শুরুতে rule-based routing। Classify এবং
> guardrail checks cheap model। Orchestrator planning strong model।
> Routing logic gateway-তে, agent code-এ hardcode না।"

### Rate limiting + budget caps

Rate limit তিন level-এ রাখো:

- **Per API key**: এক user abuse করলে বাকিরা impact না।
- **Per tenant**: B2B customer quota।
- **Global**: provider TPM/RPM respect, total system protection।

Budget cap আলাদা: "$50/hour tenant X" hit হলে **hard stop** বা
**degrade to cheap model**। Runaway agent loop এটাই বাঁচায়।

Redis sliding window common pattern:

```python
# gateway/rate_limit.py
import time
import redis

r = redis.Redis(host="localhost", port=6379, db=0)


def allow_request(key: str, limit: int, window_sec: int = 60) -> bool:
    now = int(time.time())
    bucket = f"rl:{key}:{now // window_sec}"
    count = r.incr(bucket)
    if count == 1:
        r.expire(bucket, window_sec + 1)
    return count <= limit
```

### Fallback chain: outage handle করা

Primary model 503 দিলে agent crash করা উচিত না। Gateway cascade:

```
gpt-4o → gpt-4o-mini → azure-gpt-4o (same family, other region)
```

Policy examples:

- **Latency fallback**: P99 > 8s হলে next model try।
- **Error fallback**: 429 / 5xx → retry with backoff, then backup।
- **Cost fallback**: budget 80% → switch tier for non-critical paths।

> **Common mistake:** Fallback শুধু error-এ রাখা, quality drop track
> না করা। User জানে না answer weak model থেকে এসেছে। Log `model_used`
> every response-এ।

### Auth: API keys, scopes, service identity

Gateway-তে auth patterns:

- **External users**: API key + optional OAuth (user id → tenant id)।
- **Internal services**: mTLS বা signed service token (orchestrator → gateway)।
- **Scopes**: `read:memory` vs `invoke:tools` vs `llm:chat` আলাদা permission।

Agents সরাসরি provider key রাখবে না। শুধু gateway key। Rotate এক জায়গায়।

### Multi-provider load balancing

এক provider down হলে অন্যটায় shift:

- **Round-robin** across healthy endpoints (same model family)।
- **Weighted** by cost or latency SLO।
- **Sticky session** যখন conversation cache provider-side থাকে (দুর্লভ)।

LiteLLM `router` mode এটা built-in। Custom gateway-তে health check +
weighted pick।

### Cost tracking: interview-এর hidden winner

Gateway cost ledger interview-এ strong signal:

| Dimension | Example metric |
|-----------|----------------|
| Per request | `input_tokens`, `output_tokens`, `model`, `latency_ms` |
| Per agent | `agent=research` daily $ |
| Per tenant | customer invoice, abuse detection |
| Per feature | `feature=digest` vs `feature=chat` |

Observability traces **কোথায়** slow। Cost ledger **কোথায়** expensive।
দুইটা মিলিয়ে optimize।

### Implementation: LiteLLM proxy

Fastest production path: **LiteLLM** as OpenAI-compatible proxy।
এক endpoint, অনেক provider।

```yaml
# litellm_config.yaml (sketch)
model_list:
  - model_name: fast
    litellm_params:
      model: gpt-4o-mini
      api_key: os.environ/OPENAI_API_KEY
  - model_name: strong
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
  - model_name: backup
    litellm_params:
      model: azure/gpt-4o
      api_key: os.environ/AZURE_API_KEY

router_settings:
  routing_strategy: simple-shuffle
  num_retries: 2
  timeout: 30
  fallbacks: [{"strong": ["backup", "fast"]}]
```

Client (agent code) শুধু:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://litellm-proxy:4000/v1",
    api_key="internal-gateway-key",
)

resp = client.chat.completions.create(
    model="strong",  # logical name, not provider string
    messages=[{"role": "user", "content": query}],
    extra_body={"metadata": {"agent": "research", "tenant": "lab-1"}},
)
```

`metadata` Langfuse / cost DB-তে map করা যায়।

### Implementation: FastAPI custom gateway

Managed proxy চাই না হলে minimal gateway:

```python
# gateway/main.py
from fastapi import FastAPI, HTTPException, Header, Depends
from pydantic import BaseModel
import httpx
import os
import time

from gateway.rate_limit import allow_request

app = FastAPI()
VALID_KEYS = {"dev-key"}  # vault in prod
OPENAI_KEY = os.environ["OPENAI_API_KEY"]
REQUEST_LOG = []  # Postgres / ClickHouse in prod


class ChatRequest(BaseModel):
    model: str
    messages: list[dict]
    task_type: str = "default"
    tenant_id: str = "default"


def verify_key(x_api_key: str = Header(...)) -> str:
    if x_api_key not in VALID_KEYS:
        raise HTTPException(status_code=401, detail="invalid key")
    return x_api_key


def pick_model(task_type: str, requested: str) -> str:
    if task_type in ("classify", "guardrail"):
        return "gpt-4o-mini"
    if task_type in ("plan", "synthesis", "code"):
        return "gpt-4o"
    return requested


@app.post("/v1/chat/completions")
async def chat(req: ChatRequest, _: str = Depends(verify_key)):
    if not allow_request(f"{req.tenant_id}", limit=120):
        raise HTTPException(status_code=429, detail="rate limited")

    model = pick_model(req.task_type, req.model)
    start = time.time()
    async with httpx.AsyncClient(timeout=60) as client:
        r = await client.post(
            "https://api.openai.com/v1/chat/completions",
            headers={"Authorization": f"Bearer {OPENAI_KEY}"},
            json={"model": model, "messages": req.messages},
        )
    if r.status_code >= 400:
        raise HTTPException(status_code=502, detail="upstream failed")

    data = r.json()
    usage = data.get("usage", {})
    REQUEST_LOG.append({
        "tenant": req.tenant_id,
        "model": model,
        "task_type": req.task_type,
        "tokens": usage.get("total_tokens", 0),
        "latency_ms": int((time.time() - start) * 1000),
    })
    return data
```

Production-এ `REQUEST_LOG` → warehouse, trace id header propagate,
fallback loop add করো।

### Gateway ↔ observability handshake

Gateway প্রথম hop। এখান থেকে **trace id** generate বা accept:

- Incoming `X-Trace-Id` না থাকলে UUID বানাও।
- Downstream orchestrator, agents, tools-এ same id pass।
- Span attribute: `gateway.model`, `gateway.tenant`, `gateway.routed_to`।

Observability deep dive-এর waterfall তখন gateway span দিয়ে শুরু হয়।

### Production checklist

| Check | Why |
|-------|-----|
| No direct provider calls from agents | cost + security |
| Logical model names (`fast`, `strong`) | swap provider without code change |
| Per-tenant budget alert | runaway loop detection |
| Fallback tested monthly | outage drill |
| Keys in vault, not env in repos | leak prevention |

> Gateway ছাড়া system design diagram incomplete। Interview-এ বলো:
> "সব LLM traffic এক entry point। Routing, limits, cost, fallback
> এখানে। Agents business logic, gateway infrastructure।"

Managed vs custom recap: **LiteLLM / Portkey** when speed matters;
**FastAPI gateway** when policy very custom (research lab, air-gapped).

---

### Observability layer (overview)

Agentic system debug করা hard, কারণ এক request-এ অনেক hop।
Track করতে হবে: end-to-end **trace id**, per-step **latency**,
**token usage**, **error rate** by agent and tool.

Common stack: **OpenTelemetry** + **Jaeger** or **Tempo**,
**Langfuse** or **Arize Phoenix**, **Prometheus** + **Grafana**.

Production rule: observability পরে add করা painful। Day 1 থেকে
instrument করো।

নিচে observability layer বিস্তারিত: তিন স্তম্ভ, architecture,
OpenTelemetry code, Langfuse, alert rules।

---

## Observability layer (deep dive)

*Series: Production Agentic System · Part 3 focus*

### Production-এ অন্ধকারে কাজ করা যায় না

রাত ৩টা। তোমার phone-এ notification আসলো: "API error rate 40%।"
তুমি laptop খুলে dashboard দেখলে। কিছুই বুঝলে না কারণ কোনো
dashboard নেই। Log দেখলে। লক্ষ লক্ষ line। কোথায় problem?
জানো না। **এটাই হয় observability না থাকলে।**

Observability মানে শুধু logging না। তিনটা জিনিস একসাথে:

- **Metrics**: কী হচ্ছে সংখ্যায় (latency, error rate, throughput)
- **Traces**: কোথায় কতক্ষণ লাগছে (request path)
- **Logs**: ঠিক কী ঘটেছে (errors, inputs, context)

এই তিনটাকে একসাথে বলে observability-র তিন স্তম্ভ।

#### Metrics

সংখ্যায় স্বাস্থ্য। Time-series data। Dashboard-এ দেখা যায়।
Alert trigger করে।

#### Traces

Request-এর যাত্রা। Gateway থেকে agent থেকে tool পর্যন্ত পুরো path।

#### Logs

ঘটনার রেকর্ড। Error stack trace, input/output, context।
Debug-এর সময় দরকার।

Agentic system-এ observability একটু বেশি complex কারণ request একটা
component-এ থেমে থাকে না। Gateway, orchestrator, sub-agent, tool
registry: সব জায়গায় কী হচ্ছে সেটা একসাথে দেখতে হবে।

### Observability architecture

{% include observability-diagram.html %}

### তিনটা pillar কেন তিনটাই লাগে

শুধু metrics থাকলে জানবে "error rate বেড়েছে।" কিন্তু কোথায়?
Trace যোগ করলে জানবে "orchestrator-এ slow।" কিন্তু কেন?
Log যোগ করলে জানবে "OpenAI API timeout, model overloaded।"
**তখন fix করতে পারবে।**

| Signal | প্রশ্নের উত্তর দেয় | Tool |
|--------|---------------------|------|
| Metrics | "কী" হচ্ছে? Error rate কত? Latency কত? | `Prometheus` |
| Traces | "কোথায়" হচ্ছে? কোন component slow? | `Jaeger` or `Tempo` |
| Logs | "কেন" হচ্ছে? ঠিক কোন error, কোন input? | `Loki` or `Elasticsearch` |
| LLM-specific | কোন prompt কত cost করলো? কোন model fail করলো? | `Langfuse` or `Arize Phoenix` |

### Distributed tracing: agentic system-এ সবচেয়ে জরুরি

Normal web app-এ tracing comparatively simple। Request আসে, DB query
হয়, response যায়। Agentic system-এ একটা request orchestrator call
করে, তিনটা sub-agent parallel-এ চালায়, প্রতিটা agent দুইটা tool
call করে। মোট ৭টা component। সবার timing একসাথে দেখতে হবে।

Example waterfall:

```
Gateway span: 2,340ms
  Orchestrator span: 2,180ms
    Research agent span: 1,800ms
      Web search tool: 1,650ms  <-- এখানেই সমস্যা
    Code agent span: 380ms
      Code executor tool: 320ms
```

এই waterfall দেখলেই বুঝতে পারবে কোথায় ২ সেকেন্ড যাচ্ছে। Web search
tool slow। Log দেখলে exact কারণ পাবে।

### Implementation: OpenTelemetry দিয়ে

OpenTelemetry একটা vendor-neutral standard। একবার instrument করলে
Jaeger, Tempo, Datadog, যেকোনো backend-এ পাঠাতে পারবে। প্রতিটা
component-এ span create করতে হবে।

```python
# observability/tracer.py
from opentelemetry import trace, metrics
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import (
    OTLPSpanExporter,
)
from opentelemetry.sdk.metrics import MeterProvider
import time
import structlog

# Setup tracer
provider = TracerProvider()
provider.add_span_processor(
    BatchSpanProcessor(
        OTLPSpanExporter(endpoint="http://otel-collector:4317")
    )
)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("agentic-system")

# Setup metrics
meter = metrics.get_meter("agentic-system")
llm_latency = meter.create_histogram(
    name="llm.request.duration",
    description="LLM call duration in seconds",
    unit="s",
)
agent_calls = meter.create_counter(
    name="agent.calls.total",
    description="Total sub-agent invocations",
)

log = structlog.get_logger()

def observe(agent_name: str):
    def decorator(func):
        async def wrapper(*args, **kwargs):
            with tracer.start_as_current_span(agent_name) as span:
                start = time.time()
                span.set_attribute("agent.name", agent_name)
                try:
                    result = await func(*args, **kwargs)
                    span.set_attribute("agent.status", "success")
                    agent_calls.add(1, {"agent": agent_name, "status": "ok"})
                    return result
                except Exception as e:
                    span.record_exception(e)
                    span.set_attribute("agent.status", "error")
                    agent_calls.add(
                        1, {"agent": agent_name, "status": "error"}
                    )
                    log.error("agent_failed", agent=agent_name, error=str(e))
                    raise
                finally:
                    duration = time.time() - start
                    llm_latency.record(duration, {"agent": agent_name})
        return wrapper
    return decorator

@observe("research_agent")
async def research_agent(query: str) -> str:
    ...
```

### LLM-specific observability: Langfuse

General observability tool-গুলো LLM-এর জন্য পুরোপুরি যথেষ্ট না।
Prompt কী ছিল, response কী আসলো, কত token গেলো, cost কত হলো, কোন
model call হলো: এগুলো track করতে হয়। **Langfuse** এই কাজটা করে।

> **Production reality:** Cost হঠাৎ ৩ গুণ বাড়লো। General metrics-এ
> দেখবে "token usage বেড়েছে।" কিন্তু কোন feature, কোন agent, কোন
> prompt? Langfuse দিয়ে expensive call second-এর মধ্যে বের করা যায়।

### Alert rules: কোনগুলো থাকা দরকার

1. **Error rate > 5% for 5 minutes.** যেকোনো agent বা component-এ।
   Users impact হচ্ছে।
2. **P99 latency > 10 seconds.** 99th percentile user-দের ১০
   সেকেন্ডের বেশি wait করাচ্ছে।
3. **LLM cost > $50/hour.** Budget circuit breaker। Runaway loop
   বা abuse detect করতে।
4. **Agent timeout rate > 10%.** কোনো tool বা external API ধীর
   হয়েছে। Fallback trigger করার signal।
5. **Guardrail block rate sudden spike.** Prompt injection attack
   বা model behavior change। দুইটাই urgent।

> **Common mistake:** অনেকেই শুধু error alert রাখে। Latency alert
> রাখে না। User ৩০ সেকেন্ড wait করছে, error নেই কিন্তু experience
> terrible। Latency SLO set করো, alert রাখো।

> Observability হলো তোমার system-এর কাছে প্রশ্ন করার ক্ষমতা।
> "কী হচ্ছে?" জানো। "কোথায়?" জানো। "কেন?" জানো। এই তিনটা ছাড়া
> production engineering করা মানে অন্ধকারে হাঁটা।

---

### Orchestrator (overview)

Orchestrator হলো stateful coordinator। Prompt chain না, workflow engine।
Parse goal, break into steps, route agents, merge results, retry on failure.

Common patterns: **LangGraph**, **Temporal**, or state machine + queue
(**Celery**, **RQ**).

Interview tip: orchestrator-এ coordination logic, sub-agent-এ narrow skills।

নিচে orchestrator deep dive: decomposition, state machine, context,
retry logic, এবং sample code।

---

## Orchestrator (deep dive)

*Series: Production Agentic System · Part 4 focus*

Gateway request নেয়। Observability দেখে। কিন্তু সিদ্ধান্ত নেয় কে?
কোন agent কী করবে, কোন order-এ, কী হলে retry করবে?
সব orchestrator decide করে। এটাই সবচেয়ে complex piece।

তুমি junior developer hire করলে। সে কাজ করতে পারে। কিন্তু কোন কাজটা
আগে করবে, কোথায় আটকে গেলে কী করবে সেটা তোমাকে বলে দিতে হবে।
Sub-agents ঠিক এরকম। **Orchestrator হলো সেই senior যে বলে দেয়।**

আগের sections-এ gateway আর observability দেখলাম। মাঝখানে যে layer
সব coordinate করে সেটা orchestrator। এটা না থাকলে agents randomly
কাজ করবে, conflict করবে, coherent output আসবে না।

> Orchestrator হলো conductor। Orchestra-তে প্রতিটা musician নিজের
> instrument বাজাতে জানে। কিন্তু conductor ছাড়া সেটা music না, শুধু noise।

### Orchestrator আসলে কী করে

চারটা core responsibility। প্রতিটা আলাদাভাবে complex। একসাথে এটাই
orchestrator।

{% include orchestrator-diagram.html %}

### Task decomposition: সবচেয়ে কঠিন কাজ

User বলে: *"আমার PRM paper-এর জন্য related work section লিখে দাও।"*
Orchestrator এটা নেয় আর ভাঙে:

> **Decomposition example**  
> Goal: "PRM paper-এর related work section লেখো"  
>  
> Step 1: arXiv-এ PRM, ORM, process supervision search করো  
> Step 2: Top 15 papers-এর abstract read করো  
> Step 3: Papers গুলো থিম অনুযায়ী group করো  
> Step 4: প্রতিটা থিমের জন্য 2-3 sentence লেখো  
> Step 5: Sections একসাথে compile করো  
>  
> Step 1, 2 sequential। Step 3, 4 step 2-এর উপর depend করে। Step 5 সব শেষে।

Orchestrator-কে বুঝতে হবে কোন steps parallel চলতে পারে, কোনটা আগের
output ছাড়া শুরু হতে পারবে না। এটাই **DAG** (Directed Acyclic Graph)।

### Sequential vs parallel: কখন কোনটা

**Sequential execution**  
Step B-এর জন্য step A-এর output দরকার। Order মানতে হবে। Simpler to
debug। একটা fail করলে পরেরটা চলে না।

**Parallel execution**  
Independent steps একসাথে চলে। অনেক দ্রুত। Shared state সাবধানে handle
করতে হয়। Race condition possible।

Real orchestrator দুইটাই করে। arXiv search আর Semantic Scholar search
parallel। Summarization শুধু search শেষ হওয়ার পরে। Orchestrator এই
dependency graph বোঝে।

### State machine: কোথায় আছে এখন

প্রতিটা task-এর একটা state আছে। কোনো step fail করলে retry, skip, বা
পুরো task fail: orchestrator decide করে।

```
IDLE → PLANNING → ROUTING → RUNNING → WAITING → EVALUATING → DONE
```

Any state can transition to **FAILED** → retry logic → back to **ROUTING**.

State machine ছাড়া orchestrator জানে না কোথায় আছে। Crash হলে কোথা
থেকে resume করবে? কোন agent কতক্ষণ চলছে? Track নেই।

> **Common mistake:** State machine না রাখলে একই step দুইবার run হয়।
> Agent timeout হলে orchestrator মনে করে step হয়নি, আবার dispatch করে।
> কিন্তু first agent আসলে complete করে ফেলেছে। Duplicate work,
> conflicting results।

### Context management: কী জানবে agent

প্রতিটা agent-এর context window সীমিত। Orchestrator decide করে কোন agent
কী জানবে। সব কিছু সবাইকে দিলে overflow, cost বাড়ে। কম দিলে ভুল decision।

| Agent | কী context দরকার | কী দরকার নেই |
|-------|------------------|---------------|
| Research agent | Original query, keywords, prior search results | Code history, auth details |
| Code agent | Task spec, code snippets, test cases | Full paper list, raw search |
| Analysis agent | Data from prior steps, goal, output format | Raw queries, code drafts |
| Synthesis agent | All prior outputs, final goal, format | Internal state, retry counts |

### Retry logic: failure handle করা

Production-এ সব কিছু fail করে। Orchestrator-কে gracefully handle করতে হয়।

1. **Transient failure:** network timeout, rate limit। Exponential backoff
   দিয়ে retry (সাধারণত 3 বার)। তারপর escalate।
2. **Agent quality failure:** output ভুল format বা incomplete। Re-prompt
   different instruction দিয়ে। Max 2 retry।
3. **Permanent failure:** 3 retry-তেও কাজ হয়নি। Optional step skip, না হলে
   পুরো task fail। User-কে notify কোথায় আটকেছে।
4. **Partial success:** যতটুকু হয়েছে return করো। Clear indication what
   is missing।

### Implementation: simple orchestrator

```python
# core/orchestrator.py
from dataclasses import dataclass, field
from enum import Enum
from typing import Callable, Any
import asyncio
import logging

log = logging.getLogger("orchestrator")


class StepStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"
    FAILED = "failed"
    SKIPPED = "skipped"


@dataclass
class Step:
    id: str
    name: str
    agent_fn: Callable
    depends_on: list[str] = field(default_factory=list)
    max_retries: int = 3
    optional: bool = False
    status: StepStatus = StepStatus.PENDING
    result: Any = None
    retries: int = 0
    error: str = None


class Orchestrator:
    def __init__(self, steps: list[Step]):
        self.steps = {s.id: s for s in steps}
        self.context: dict = {}

    async def run(self) -> dict:
        while self._has_pending():
            ready = self._get_ready_steps()
            if not ready:
                break
            await asyncio.gather(*[self._run_step(step) for step in ready])
        return self.context

    async def _run_step(self, step: Step):
        step.status = StepStatus.RUNNING
        log.info(f"Starting step: {step.name}")

        while step.retries <= step.max_retries:
            try:
                result = await step.agent_fn(self.context)
                self.context[step.id] = result
                step.result = result
                step.status = StepStatus.DONE
                log.info(f"Step done: {step.name}")
                return
            except Exception as e:
                step.retries += 1
                step.error = str(e)
                log.warning(
                    f"Step failed ({step.retries}/{step.max_retries}): {e}"
                )
                if step.retries <= step.max_retries:
                    await asyncio.sleep(2 ** step.retries)

        if step.optional:
            step.status = StepStatus.SKIPPED
            log.warning(f"Optional step skipped: {step.name}")
        else:
            step.status = StepStatus.FAILED
            raise RuntimeError(f"Step failed after retries: {step.name}")

    def _has_pending(self) -> bool:
        return any(
            s.status == StepStatus.PENDING for s in self.steps.values()
        )

    def _get_ready_steps(self) -> list[Step]:
        return [
            s
            for s in self.steps.values()
            if s.status == StepStatus.PENDING
            and all(
                self.steps[dep].status == StepStatus.DONE
                for dep in s.depends_on
            )
        ]


async def run_digest(query: str):
    steps = [
        Step(
            id="search_arxiv",
            name="Search arXiv",
            agent_fn=lambda ctx: arxiv_agent(query),
            depends_on=[],
        ),
        Step(
            id="search_s2",
            name="Search Semantic Scholar",
            agent_fn=lambda ctx: s2_agent(query),
            depends_on=[],
            optional=True,
        ),
        Step(
            id="summarize",
            name="Summarize papers",
            agent_fn=lambda ctx: summarize_agent(
                ctx["search_arxiv"] + ctx.get("search_s2", [])
            ),
            depends_on=["search_arxiv"],
        ),
        Step(
            id="notify",
            name="Send notification",
            agent_fn=lambda ctx: ntfy_agent(ctx["summarize"]),
            depends_on=["summarize"],
        ),
    ]
    orch = Orchestrator(steps)
    return await orch.run()
```

Orchestrator শুধু coordination করছে। Actual কাজ agent-দের। Orchestrator
জানে কখন call করতে হবে, output কোথায় রাখতে হবে, fail হলে কী করতে হবে।

> **Note:** এই pattern arXiv digest toy project-এ directly use করা যায়:
> `search_arxiv`, `search_s2`, `summarize`, `notify`। arXiv আর S2 parallel।
> Summarize depends on search। Same pattern production-grade system-এও কাজ করে।

> Orchestrator-এর complexity তার নিজের logic-এ না। জানে কখন, কাকে, কী দিয়ে
> call করতে হবে। এই separation of concerns system-কে maintainable রাখে।

Framework options (recap): **LangGraph** for agent graphs, **Temporal** for
durable workflows, or the pattern above for minimal control.

---

### Memory + vector store (overview)

Short-term (session) + long-term (persistent) memory, often backed by
**Redis**, **PostgreSQL**, and a vector DB (**pgvector**, **Chroma**,
**Pinecone**).

Part 1-এর interview question: memory orchestrator-এ না sub-agent-এ?
Answer depends on task dependencies. নিচে full deep dive।

---

## Memory layer (deep dive)

*Series: Production Agentic System · Part 5 focus*

এই series-এর শুরুতে interview question ছিল: *"Memory কোথায় থাকবে?"*
এই section-এ memory types, vector store, trade-off, এবং ChromaDB
implementation।

তুমি researcher-কে জিজ্ঞেস করলে: "তিন মাস আগে PRM নিয়ে কী পড়েছিলে?"
সে মনে করতে পারে। কিন্তু LLM agent-কে জিজ্ঞেস করলে?
**প্রতিটা conversation fresh start।** Memory layer এই gap fill করে।

Agent-এর memory না থাকলে প্রতিটা request-এ সব আবার explain করতে হয়।
Memory layer agent-কে context দেয়: কোথায় আছে, আগে কী হয়েছে, কী relevant।

> Memory ছাড়া agent হলো goldfish। Memory দিলে সে researcher হয়:
> past experiment মনে রাখে, pattern দেখে, build করে।

### Memory চার ধরনের

Human memory-র মতো agent memory-ও এক রকম না। চারটা type, চারটা কাজ:

**Short-term memory**  
Current conversation context। LLM context window-এ থাকে। Session শেষে
চলে যায়। Fast কিন্তু ephemeral।

**Long-term memory**  
Persistent store (database বা vector store)। Session-এর পরেও survive করে।
User preferences, past decisions।

**Episodic memory**  
"কখন কী হয়েছিল।" Specific events, experiments, outcomes। Timeline-based।

**Semantic memory**  
Facts এবং concepts। "PRM মানে কী।" Knowledge base, structured বা
vector-indexed।

Memory layer orchestrator আর agents-এর মাঝখানে বসে। Orchestrator context
request করে, memory retrieve করে, agent কাজ করে, output write-back হয়।

### Memory architecture

{% include memory-diagram.html %}

### Vector store: embedding কীভাবে কাজ করে

Vector store-এর core idea: text-কে numbers-এ convert করো। Similar text
থেকে similar vectors। "এই query-র কাছাকাছি কী আছে?" cosine distance
দিয়ে খুঁজো। এটাই **semantic search**।

"What is the capital of France?" আর "France-এর রাজধানী কী?" string match
হবে না, কিন্তু embedding space-এ কাছাকাছি থাকবে। Meaning একই।

Example embeddings (simplified):

| Text | Vector (truncated) |
|------|-------------------|
| "PRM outperforms ORM on math tasks" | [0.82, -0.14, 0.67, ...] |
| "Process reward better than outcome reward" | [0.79, -0.11, 0.71, ...] |
| "GRPO training on Qwen2.5 1.5B" | [0.34, 0.67, 0.12, ...] |

Query: *"which reward model is better?"*

| Candidate | Cosine similarity |
|-----------|-------------------|
| "PRM outperforms ORM on math tasks" | 0.94 |
| "Process reward better than outcome reward" | 0.91 |
| "GRPO training on Qwen2.5 1.5B" | 0.38 |

Top two relevant, third irrelevant। Threshold (e.g. 0.7) দিয়ে filter
করলে শুধু useful context prompt-এ যায়।

### The trade-off: centralised vs distributed

Part 1 interview question-এর full answer:

| | Centralised (orchestrator) | Distributed (sub-agent) |
|--|--------------------------|-------------------------|
| Consistency | সব agent same view | agents diverge করতে পারে |
| Latency | একটু বেশি | কম, local access |
| Sync | single source, simple | complex |
| Debugging | এক জায়গায় | scattered state |
| Best for | multi-step dependent tasks | parallel independent tasks |

> **Interview answer:** "Centralised memory choose করলাম কারণ task-গুলো
> sequential এবং dependent। Research output analysis agent ব্যবহার করে।
> Consistency critical। Distributed বেছে নিতাম যদি agents fully
> independent parallel করতো।" Not "centralised is always better."
> Rather: "এই context-এ centralised কারণ..."

### Implementation: ChromaDB দিয়ে

Toy project এবং research harness-এ **ChromaDB** easy start: local,
no server, simple Python API।

```python
# memory/vector_store.py
import uuid
from dataclasses import dataclass
from datetime import datetime

import chromadb
from chromadb.utils import embedding_functions


@dataclass
class MemoryEntry:
    content: str
    source: str
    tags: list[str]
    timestamp: str = None

    def __post_init__(self):
        if not self.timestamp:
            self.timestamp = datetime.now().isoformat()


class ResearchMemory:
    def __init__(self, persist_dir: str = "./memory_store"):
        self.client = chromadb.PersistentClient(path=persist_dir)
        self.ef = embedding_functions.SentenceTransformerEmbeddingFunction(
            model_name="all-MiniLM-L6-v2"
        )
        self.collection = self.client.get_or_create_collection(
            name="research_memory",
            embedding_function=self.ef,
            metadata={"hnsw:space": "cosine"},
        )

    def store(self, entry: MemoryEntry) -> str:
        mem_id = str(uuid.uuid4())
        self.collection.add(
            ids=[mem_id],
            documents=[entry.content],
            metadatas=[{
                "source": entry.source,
                "tags": ",".join(entry.tags),
                "timestamp": entry.timestamp,
            }],
        )
        return mem_id

    def retrieve(
        self,
        query: str,
        n: int = 5,
        threshold: float = 0.7,
        tags: list[str] = None,
    ) -> list[dict]:
        where = None
        if tags:
            where = {"tags": {"$contains": tags[0]}}

        results = self.collection.query(
            query_texts=[query],
            n_results=n,
            where=where,
        )

        memories = []
        for i, doc in enumerate(results["documents"][0]):
            score = 1 - results["distances"][0][i]
            if score >= threshold:
                memories.append({
                    "content": doc,
                    "score": round(score, 3),
                    "metadata": results["metadatas"][0][i],
                })

        return sorted(memories, key=lambda x: x["score"], reverse=True)

    def store_run(self, run_id: str, query: str, papers: list, summaries: list):
        self.store(MemoryEntry(
            content=f"Research query: {query}",
            source=run_id,
            tags=["query"],
        ))
        for paper, summary in zip(papers, summaries):
            self.store(MemoryEntry(
                content=f"{paper.title}: {summary}",
                source=run_id,
                tags=["paper", "summary"],
            ))


memory = ResearchMemory()

past = memory.retrieve(
    query="PRM vs ORM generalization",
    threshold=0.85,
    tags=["query"],
)
if past:
    print(f"Similar query run before: {past[0]['metadata']['source']}")

memory.store_run(run_id, query, papers, summaries)
```

### Memory কীভাবে agent prompt-এ যায়

1. **Task আসে orchestrator-এ:** "PRM paper-এর related work লেখো।"
2. **Memory controller query করে:** আগে এই topic-এ কী search হয়েছিল?
   Vector store থেকে top 3-5 memories retrieve।
3. **Context builder assemble করে:** memories + task + instructions,
   token budget মেনে।
4. **Agent prompt পায়:** task + relevant past context + prior findings।
5. **Output memory-তে store হয়:** পরের request-এ auto retrieve।

> **Token budget সাবধান:** সব memory prompt-এ দিলে overflow। Memory
> controller-এর কাজ: most relevant বেছে নেওয়া, সব দেওয়া না।

> Memory system ভালো হলে agent output ভালো হয়। Output ভালো হলে memory
> richer হয়। Flywheel: প্রথম কয়েক run slow, সময়ের সাথে system smarter।

Vector store options (recap): **pgvector**, **Pinecone**, **Weaviate**,
**Qdrant**, **Chroma** (local / research).

---

### Sub-agents (overview)

Sub-agent = specialist worker, not another full chatbot। Orchestrator
dispatch করে; agent ReAct loop-এ think, act, observe করে। Framework
options: **LangGraph** subgraphs, **CrewAI**, one LLM + role prompts।

Rule: each agent gets **minimal tools**। Extra tools = more failure modes।

নিচে sub-agents deep dive: চারটা core agent, ReAct loop, prompt anatomy,
base agent class, research agent example।

---

## Sub-agents (deep dive)

*Series: Production Agentic System · Sub-agents focus*

Gateway request নেয়। Orchestrator plan করে। Memory context দেয়।
**কিন্তু আসল কাজ করে কে? Sub-agents।** প্রতিটা একটা narrow domain-এ
expert। General-purpose agent দিয়ে সব করানো technically possible,
কিন্তু production-এ prompt বড় হয়, context confused হয়, quality পড়ে।

> Hospital analogy: general physician আছে, কিন্তু brain surgery-তে
> neurosurgeon লাগে। Sub-agents specialize করা workers।

> "Do one thing and do it well." Research agent শুধু research।
> Code agent শুধু code। কেউ কারো কাজে interfere করে না।

### চারটা core agent: কে কী করে

| Agent | Role | Tools | Model bias |
|-------|------|-------|------------|
| **Research** | খোঁজে, পড়ে, summarize | arXiv, Semantic Scholar, web | fast (Haiku) |
| **Code** | লেখে, চালায়, debug | sandbox exec, file system | quality (Sonnet) |
| **Analysis** | data দেখে, pattern বের করে | code exec, DB query, charts | reasoning |
| **Synthesis** | সব output জোড়া দেয় | file writer, formatter | long context |

Research agent input: query + search scope। Output: ranked papers with
summaries। Code agent input: task spec + codebase snippet। Output: working
code + test results। Synthesis agent সব prior step-এর output নিয়ে final
document বানায়।

### ReAct loop: agent কীভাবে কাজ করে

Orchestrator task dispatch করে। Agent **think → act → observe** loop
চালায়। শুধু একবার LLM call না: tool call, result দেখে, আবার decide।

{% include subagents-diagram.html %}

**Think:** LLM task parse করে, কোন tool, কী arguments।  
**Act:** tool registry-র মাধ্যমে call (direct না)।  
**Observe:** result sufficient? loop again? max steps hit?

In-agent guardrails: max steps (e.g. 10), token budget, output format
validator। Loop infinite হলে orchestrator duplicate work দেখে।

### Prompt anatomy: system prompt চার section

Agent quality সবচেয়ে বেশি system prompt-এ depend করে:

**1. System identity**  
Who you are, scope, what you refuse.

**2. Memory context** (injected per request)  
Past runs, prior searches, gaps noted.

**3. Current task** (from orchestrator)  
Exact goal, constraints, what to avoid.

**4. Output format**  
JSON schema or structure orchestrator can parse.

| Section | ছাড়া কী হয় |
|---------|-------------|
| System identity | Off-topic কাজ, scope creep |
| Memory context | Duplicate work, same papers again |
| Current task | Hallucination, wrong goal |
| Output format | Orchestrator parse করতে পারে না |

Example output format for research agent:

```json
{
  "papers": [{"title": "...", "claim": "...", "method": "...", "relevance": "..."}],
  "gap_identified": "...",
  "suggested_next_query": "..."
}
```

No markdown. No preamble. Valid JSON only.

### Implementation: base agent class

```python
# agents/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from anthropic import Anthropic
import json
import logging

log = logging.getLogger("agent")
client = Anthropic()


@dataclass
class AgentResult:
    output: dict
    status: str  # "done" / "partial" / "failed"
    steps_used: int = 0
    tokens_used: int = 0
    error: str = None


class BaseAgent(ABC):
    name: str
    model: str = "claude-haiku-4-5"
    max_steps: int = 10
    max_tokens: int = 4096

    @abstractmethod
    def system_prompt(self) -> str:
        ...

    @abstractmethod
    def tools(self) -> list[dict]:
        ...

    async def run(self, task: str, memory_context: str = "") -> AgentResult:
        messages = [{
            "role": "user",
            "content": self._build_task_prompt(task, memory_context),
        }]
        steps = 0
        total_tokens = 0

        while steps < self.max_steps:
            steps += 1
            response = client.messages.create(
                model=self.model,
                max_tokens=self.max_tokens,
                system=self.system_prompt(),
                tools=self.tools(),
                messages=messages,
            )
            total_tokens += (
                response.usage.input_tokens + response.usage.output_tokens
            )

            if response.stop_reason == "end_turn":
                text = response.content[0].text
                try:
                    output = json.loads(text)
                    return AgentResult(
                        output=output,
                        status="done",
                        steps_used=steps,
                        tokens_used=total_tokens,
                    )
                except json.JSONDecodeError:
                    return AgentResult(
                        output={"raw": text},
                        status="partial",
                        steps_used=steps,
                        tokens_used=total_tokens,
                        error="output not valid JSON",
                    )

            if response.stop_reason == "tool_use":
                messages.append({
                    "role": "assistant",
                    "content": response.content,
                })
                tool_results = []
                for block in response.content:
                    if block.type == "tool_use":
                        result = await self._call_tool(
                            block.name, block.input
                        )
                        tool_results.append({
                            "type": "tool_result",
                            "tool_use_id": block.id,
                            "content": json.dumps(result),
                        })
                messages.append({
                    "role": "user",
                    "content": tool_results,
                })

        return AgentResult(
            output={},
            status="failed",
            steps_used=steps,
            tokens_used=total_tokens,
            error=f"max steps ({self.max_steps}) reached",
        )

    async def _call_tool(self, name: str, args: dict) -> dict:
        from tools.registry import registry
        return await registry.call(name, args, agent=self.name)

    def _build_task_prompt(self, task: str, memory: str) -> str:
        parts = [f"Task: {task}"]
        if memory:
            parts.insert(0, f"Relevant past context:\n{memory}\n")
        return "\n\n".join(parts)
```

### Implementation: research agent

```python
# agents/research.py
from agents.base import BaseAgent


class ResearchAgent(BaseAgent):
    name = "research_agent"
    model = "claude-haiku-4-5"
    max_steps = 8

    def system_prompt(self) -> str:
        return """
You are a research agent specializing in ML paper discovery.
Find papers, extract claims, identify gaps. Stay in scope.
Return JSON: {"papers": [...], "gap_identified": "...",
"suggested_next_query": "..."}
No preamble. Valid JSON only.
"""

    def tools(self) -> list[dict]:
        return [
            {
                "name": "search_arxiv",
                "description": "Search arXiv for papers on a topic",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "query": {"type": "string"},
                        "max_results": {"type": "integer", "default": 10},
                    },
                    "required": ["query"],
                },
            },
            {
                "name": "search_semantic_scholar",
                "description": "Search Semantic Scholar for citation data",
                "input_schema": {
                    "type": "object",
                    "properties": {"query": {"type": "string"}},
                    "required": ["query"],
                },
            },
        ]
```

### কখন একটা agent, কখন multiple

1. **Simple single-step:** এক agent যথেষ্ট ("arXiv-এ GRPO search করো")।
2. **Multi-domain:** research + code + analysis = তিন agent।
3. **Parallel subtasks:** একই agent-এর multiple instances (তিন topic
   একসাথে search)।
4. **Quality check:** দুই agent same task, output compare (high-stakes)।

> **Over-engineering trap:** শুরুতে সব কিছুর জন্য আলাদা agent বানানোর
> দরকার নেই। এক research agent দিয়ে শুরু। প্রয়োজনে split। Tangled
> agent system debug করা nightmare।

> Sub-agent design-এ সবচেয়ে important: **scope**। Scope ছোট, tool list
> ছোট: quality বাড়ে, hallucination কমে।

Framework recap: **LangGraph** subgraphs, **CrewAI**, or base class above.
Avoid stacking three frameworks.

---

### Tool registry (overview)

Agents সরাসরি external API call করবে না। Registry middle layer:
allowlist, schema validation, timeout, sandbox, audit log।
**MCP** (Model Context Protocol) for plug-in tools।

Interview line: *"Agents propose actions; registry approves and runs them."*

নিচে tool registry deep dive: পাঁচ-stage pipeline, catalog, permission
matrix, audit log, Python implementation।

---

## Tool registry (deep dive)

*Series: Production Agentic System · Tool registry focus*

Office building-এ security নেই: যেকেউ server room-এ ঢুকতে পারে।
একদিন কেউ production database delete করে দিল। **Tool registry ছাড়া
agentic system ঠিক এমন।**

Agent-রা powerful: code চালায়, DB query, external API। Power
uncontrolled হলে একটা hallucination পুরো system নষ্ট করতে পারে।
Registry প্রতিটা request-এ জিজ্ঞেস করে: তুমি এটা করতে পারো? এটা safe?

> "Agents don't call tools. They **request** tools. Registry decides
> whether to grant." Controlled vs chaotic system-এর পার্থক্য।

### Registry পাঁচটা stage

{% include tool-registry-diagram.html %}

1. **Lookup:** tool exists? schema valid? healthy?
2. **Permission:** agent ACL, read vs write scope, deny logged
3. **Rate limit:** per agent, per tool, global cap
4. **Execute:** sandbox, timeout (30s default), output cap
5. **Audit log:** who, what, when, result, cost

### Tool catalog: কোন tool কতটুকু powerful

| Tool | Level | Rate | Cost |
|------|-------|------|------|
| search_arxiv | read-only | 30/min | low |
| web_search | read-only | 20/min | medium |
| code_execute | execute | 10/min | high |
| db_query | read-only (SELECT) | 50/min | low |
| file_write | write | 20/min | low |
| ntfy_send | external | 5/min | low |
| obsidian_write | write | 10/min | low |
| wandb_log | write | 30/min | low |
| llm_call | privileged | 5/min | very high |

`llm_call` কেউ directly use করতে পারে না। শুধু orchestrator LLM call
করতে পারে। Agent nested LLM call করলে cost exponential, infinite loop risk।

### Permission matrix: least privilege

| Tool | Research | Code | Analysis | Synthesis |
|------|----------|------|----------|-----------|
| search_arxiv | yes | no | no | no |
| web_search | yes | limited | no | no |
| code_execute | no | yes | limited | no |
| db_query | no | no | yes | no |
| file_write | no | yes | yes | yes |
| obsidian_write | no | no | no | yes |
| ntfy_send | no | no | no | yes |
| llm_call | no | no | no | no |

Research agent code execute করতে পারে না। Synthesis agent notify করতে
পারে। Matrix interview-এ defend করা easy: "least privilege per role."

### Audit log: production debug

Bug হলে প্রথম প্রশ্ন: কোন agent, কোন tool, কখন, কী arguments?

| Time | Agent | Tool | Args | Status |
|------|-------|------|------|--------|
| 03:14:22 | research_agent | search_arxiv | query="GRPO variants DAPO" | 200 ok |
| 03:14:26 | research_agent | code_execute | import os... | **403 denied** |
| 03:14:31 | synthesis_agent | obsidian_write | digest-a3f9k2.md | 200 ok |
| 03:14:33 | research_agent | search_arxiv | query="PRM cross domain" | 429 rate limit |

Line 2: research_agent `code_execute` চেষ্টা করেছে, permission নেই।
Agent hallucinate করলেও system safe। Line 4: rate limit loop stop করেছে।

### Implementation: tool registry

```python
# tools/registry.py
from dataclasses import dataclass
from typing import Callable, Any
from datetime import datetime
import asyncio
import time
import logging

log = logging.getLogger("tool_registry")


@dataclass
class ToolDefinition:
    name: str
    fn: Callable
    allowed_agents: list[str]
    rate_limit: int = 30
    timeout_s: int = 30
    description: str = ""


@dataclass
class AuditEntry:
    timestamp: str
    agent: str
    tool: str
    args: dict
    status: str
    duration_ms: int = 0
    error: str = None


class ToolRegistry:
    def __init__(self):
        self._tools: dict[str, ToolDefinition] = {}
        self._call_counts: dict[str, list[float]] = {}
        self._audit: list[AuditEntry] = []

    def register(self, tool: ToolDefinition):
        self._tools[tool.name] = tool
        self._call_counts[tool.name] = []

    async def call(self, tool_name: str, args: dict, agent: str) -> dict:
        entry = AuditEntry(
            timestamp=datetime.now().isoformat(),
            agent=agent,
            tool=tool_name,
            args=args,
            status="?",
        )
        tool = None
        try:
            if tool_name not in self._tools:
                entry.status = "denied"
                entry.error = f"unknown tool: {tool_name}"
                raise PermissionError(entry.error)

            tool = self._tools[tool_name]

            if tool.allowed_agents and agent not in tool.allowed_agents:
                entry.status = "denied"
                entry.error = f"{agent} not allowed for {tool_name}"
                raise PermissionError(entry.error)

            now = time.time()
            window = [
                t for t in self._call_counts[tool_name] if now - t < 60
            ]
            if len(window) >= tool.rate_limit:
                entry.status = "rate_limit"
                entry.error = f"{tool_name} rate limit hit"
                raise RuntimeError(entry.error)

            self._call_counts[tool_name] = window + [now]

            start = time.time()
            result = await asyncio.wait_for(
                tool.fn(**args),
                timeout=tool.timeout_s,
            )
            entry.duration_ms = int((time.time() - start) * 1000)
            entry.status = "ok"
            return result

        except asyncio.TimeoutError:
            entry.status = "error"
            entry.error = f"timeout after {tool.timeout_s}s"
            raise
        finally:
            self._audit.append(entry)
            log.info(
                f"[{entry.status}] {agent} -> {tool_name} "
                f"({entry.duration_ms}ms)"
            )


from search.arxiv import search as arxiv_search

registry = ToolRegistry()
registry.register(ToolDefinition(
    name="search_arxiv",
    fn=arxiv_search,
    allowed_agents=["research_agent"],
    rate_limit=30,
    timeout_s=20,
))
```

### কেন direct tool call করা যাবে না

1. **Hallucination protection:** non-existent tool → reject at lookup
2. **Scope creep:** research agent DB delete → 403 + audit
3. **Cost control:** rate limit stops runaway loops (500 calls → 30)
4. **Debugging:** audit log = 10 min investigation vs hours
5. **Tool swap:** change implementation in one place, agents unchanged

> **Real incident pattern:** Agent loop-এ same tool 500 বার call।
> Registry সহ: 30তম call-এ blocked। Audit-এ loop visible। Registry
> ছাড়া: cost explode, external API rate limit, whole system slow।

> Tool Registry = agentic system-এর immune system। বেশিরভাগ সময় quiet।
> ভুল হলে block, record, alert।

### MCP (overview)

Tool registry actual tools call করে। **MCP** (Model Context Protocol) হলো
standard bridge: এক interface, অনেক provider। Registry MCP client বোঝে;
implementation detail MCP server-এ।

নিচে MCP deep dive: architecture, before/after, custom server, registry
bridge, তিনটা core concept।

---

## MCP (deep dive)

*Series: Production Agentic System · MCP focus*

আগে প্রতিটা tool আলাদা implement: arXiv আলাদা, Obsidian আলাদা, W&B
আলাদা। প্রতিটার auth, retry, error format আলাদা। Code duplicate,
maintenance nightmare। **MCP একবারে solve করে।**

MCP Anthropic-এর open standard। Protocol মেনে tool server বানালে যেকোনো
MCP-compatible client use করতে পারে। Tool registry শুধু protocol বোঝে।
বাকি সব server-এ।

> MCP হলো USB-C। এক standard port, যেকোনো device plug করো।
> আগে প্রতিটা tool-এর আলাদা cable।

### Architecture-এ MCP কোথায়

{% include mcp-diagram.html %}

Sub-agent → registry (permission) → **MCP client** → MCP servers
(arXiv, Obsidian, Notion, W&B, custom research server)। Communication:
JSON-RPC 2.0 over stdio বা HTTP/SSE।

### MCP ছাড়া vs MCP সহ

**আগে (MCP ছাড়া):** প্রতিটা tool = আলাদা function, আলাদা auth/retry/error।
১০ tools = ১০ implementations।

**এখন (MCP সহ):** এক MCP client, same `call_tool(name, args)` interface।
Error handling once। Auth per server, not per tool।

### Connected MCP servers (examples)

| Server | Use case |
|--------|----------|
| Notion | GRPO plan, experiment notes |
| Google Drive | papers, vault backup |
| Atlassian | Jira RES project, coursework |
| Hugging Face | models, papers, datasets |
| Linear | issues, sprint planning |
| Custom arXiv | arXiv + S2 + HF Papers (planned local) |

Research harness-এ same servers plug করা যায়। Tool list registry-তে
auto-register হতে পারে MCP bridge দিয়ে।

### Custom MCP server: research tools

Toy arXiv digest-কে MCP server বানিয়ে harness-এ reuse করো। Python MCP SDK:

```python
# mcp_server.py (sketch)
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import arxiv
import json

app = Server("research-mcp")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="search_arxiv",
            description="Search arXiv for ML papers",
            inputSchema={
                "type": "object",
                "properties": {
                    "query": {"type": "string"},
                    "max_results": {"type": "integer", "default": 5},
                },
                "required": ["query"],
            },
        ),
        Tool(
            name="search_semantic_scholar",
            description="Search Semantic Scholar",
            inputSchema={
                "type": "object",
                "properties": {"query": {"type": "string"}},
                "required": ["query"],
            },
        ),
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "search_arxiv":
        client = arxiv.Client()
        search = arxiv.Search(
            query=arguments["query"],
            max_results=arguments.get("max_results", 5),
        )
        papers = [
            {
                "title": r.title,
                "abstract": r.summary[:600],
                "url": r.entry_id,
            }
            for r in client.results(search)
        ]
        return [TextContent(type="text", text=json.dumps(papers))]
    ...
```

### Registry-তে MCP bridge

```python
# tools/mcp_bridge.py (sketch)
from tools.registry import registry, ToolDefinition

class MCPBridge:
    def __init__(self, server_script: str, allowed_agents: list[str]):
        self.server_script = server_script
        self.allowed_agents = allowed_agents

    async def connect(self, session):
        tools_response = await session.list_tools()
        for tool in tools_response.tools:
            async def call_fn(**kwargs, _name=tool.name):
                result = await session.call_tool(_name, kwargs)
                return result.content[0].text

            registry.register(ToolDefinition(
                name=tool.name,
                fn=call_fn,
                allowed_agents=self.allowed_agents,
                description=tool.description,
            ))
```

Bootstrap: local `mcp_server.py` for research_agent; remote HTTP/SSE
for Notion (synthesis_agent only)।

### MCP-র তিনটা core concept

1. **Tools:** Agent যা call করে (arXiv search, file write)। Input schema
   defined। Registry validates।
2. **Resources:** Agent যা read করে (files, DB rows)। Read-only, passive।
3. **Prompts:** Server-defined templates। System prompt-এ inject।
   Skills-এর সাথে connect (next section)।

> **arxiv digest project:** এখন direct Python functions। MCP wrap করলে
> harness same server use করে। Duplicate code নেই। Registration automatic।

> MCP = portability + standardization + reusability। Long-term
> maintainability-এর জন্য production-এ worth it।

---

### Skills (overview)

MCP tool connectivity standardize করে। **Skills** behavior standardize
করে: reusable markdown instructions orchestrator pick করে, agent prompt-এ
inject হয়। Base agent same, behavior task-dependent।

নিচে skills deep dive: SKILL.md structure, catalog, loader, research
pipeline example, skills vs memory vs MCP।

---

## Skills (deep dive)

*Series: Production Agentic System · Skills focus*

এক research agent আছে। এখন same agent-কে paper replication-এ use করতে
চাও: incremental coding, smoke test mandatory, ARCHITECTURE.md আগে।
System prompt আবার লিখবে? না। **Skill inject করো।**

Skill = markdown file যেটা system prompt-এ merge হয়। কীভাবে কাজ করবে,
কোন rules, কোন output format: একবার লিখো, সব agent-এ reuse।

> Skill ছাড়া agent generalist: সব করতে পারে, কিছু ভালো না।
> Skill দিলে specialist: domain জানে, consistently কাজ করে।

### Architecture-এ skills কোথায়

{% include skills-diagram.html %}

Orchestrator task type দেখে skill name pick করে → skill loader `SKILL.md`
পড়ে → prompt builder: **identity + skill + memory + task** → sub-agent।

### Skill file structure

চারটা section (markdown, version controlled):

**When to use**  
Triggers: "replicate this paper", "implement training loop", etc.

**How to behave**  
ARCHITECTURE.md first, 5-line incremental rule, MPS before CUDA, etc.

**Hard rules**  
Never skip smoke test। Ambiguous paper details → ask। Log to W&B from run 1।

**Output format**  
Commit message format, session end summary, etc.

Orchestrator relevant sections extract করে inject করে।

### Skill catalog (examples)

| Skill | Inject into | Focus |
|-------|-------------|-------|
| replicate-ml | code_agent | 5-line rule, smoke tests, blueprint |
| obsidian-rl-memory | synthesis_agent | experiment log, GitHub issues |
| hypothesis | research_agent | literature search, gap analysis |
| code-pytorch | code_agent | MPS, buffers, W&B |
| ml-notifier | synthesis_agent | ntfy topics, tmux async |
| frontend-design | code_agent (UI) | CSS tokens, dark mode |

Project-local `./skills/` + shared paths। Battle-tested workflow reuse।

### Implementation: skill loader

```python
# skills/loader.py
from pathlib import Path
from dataclasses import dataclass
import re

SKILL_DIRS = [
    Path("./skills"),
    Path("/mnt/skills/user"),
    Path("/mnt/skills/public"),
]


@dataclass
class Skill:
    name: str
    when: str
    how: str
    rules: str
    output_fmt: str

    def to_prompt_section(self) -> str:
        return f"""
## Active Skill: {self.name}

### How to behave
{self.how}

### Hard rules
{self.rules}

### Output format
{self.output_fmt}
"""


def load_skill(name: str) -> Skill:
    for skill_dir in SKILL_DIRS:
        path = skill_dir / name / "SKILL.md"
        if path.exists():
            return _parse_skill(path)
    raise FileNotFoundError(f"Skill not found: {name}")


def build_agent_prompt(
    identity: str,
    skill_name: str,
    memory_ctx: str,
    task: str,
) -> str:
    parts = [identity]
    if skill_name:
        try:
            parts.append(load_skill(skill_name).to_prompt_section())
        except FileNotFoundError:
            pass
    if memory_ctx:
        parts.append(f"## Relevant past context\n{memory_ctx}")
    parts.append(f"## Current task\n{task}")
    return "\n\n".join(parts)
```

### Example: research agent + hypothesis skill

```python
# core/pipeline.py (sketch)
async def run_hypothesis_search(hypothesis: str, memory: ResearchMemory):
    past = memory.retrieve(hypothesis, n=5, threshold=0.75)
    memory_ctx = "\n".join(p["content"] for p in past)

    agent = ResearchAgent()
    result = await agent.run(
        task=hypothesis,
        memory_context=memory_ctx,
        skill="hypothesis",
    )

    memory.store(MemoryEntry(
        content=f"Gap: {result.output.get('gap_identified')}",
        source="hypothesis_agent",
        tags=["hypothesis", "gap"],
    ))
    return result.output
```

Output: papers with claim/method/relevance, `gap_identified`,
`suggested_next_query`, refined hypothesis।

### Skills vs memory vs MCP

| Layer | Answers | Static or dynamic |
|-------|---------|-------------------|
| **Skills** | কীভাবে behave করবে | Static instructions |
| **Memory** | আগে কী হয়েছে | Dynamic, retrieved |
| **MCP** | tool কীভাবে call | Protocol / connectivity |

Research harness flow (সব একসাথে):

1. Skill loader: `hypothesis` SKILL.md
2. Memory: past searches retrieve
3. Prompt builder: identity + skill + memory + task
4. Research agent: ReAct loop
5. Tool registry: validate request
6. MCP client: arXiv / S2 call
7. Result: memory store, ntfy, Obsidian

> Skills + Memory + MCP = context-aware, consistent, connected agent।
> Generic LLM call → intelligent research assistant।

---

### Guardrails (overview)

Guardrails = safety and policy, not optional polish।

Layers:

1. **Input**: jailbreak detection, PII in prompt, off-topic block
2. **Tool**: block destructive commands, SQL without scope
3. **Output**: toxicity, hallucination checks, citation requirements
4. **Budget**: max tokens / cost per session

Tools: **NeMo Guardrails**, **Llama Guard**, regex + classifier pipeline,
custom rules from legal / compliance.

Run guardrails **before and after** LLM calls, not only at the end।

### Minimal production stack (one sane default)

যদি interviewer বলে "pick a stack in 30 seconds", এমন একটা coherent
answer defend করা যায়:

| Layer | Choice |
|-------|--------|
| Gateway | LiteLLM or FastAPI proxy |
| Orchestrator | LangGraph or Temporal |
| Memory | Redis + pgvector |
| Agents | LangGraph nodes with role prompts |
| Tools | Registry table + MCP servers |
| Observability | OpenTelemetry + Langfuse |
| Guardrails | Input/output filters + budget caps |

এটা perfect নয়, কিন্তু **consistent** এবং production-minded।

### ML research-এ কীভাবে apply হয়

Interview story LinkedIn থেকে, কিন্তু ML research workflow-এ same
ideas লাগে।

**Experiment orchestration**  
একটা research question = orchestrator goal। Sub-agents: data pull,
feature build, train, evaluate, report। State machine tracks which
step failed (OOM, NaN, bad split)।

**Reproducible memory**  
Vector store-এ past runs, configs, plots রাখলে next experiment-এ
"last time we used lr=1e-4" retrieve হয়। Central memory = lab notebook
that scales।

**Tool registry for research**  
Dangerous tools: arbitrary shell, write to prod DB। Registry allows
only: read dataset path, launch training job queue, write to artifact
store।

**Observability for science**  
Not just latency: log hyperparameters, git commit, dataset version per
trace। Debug = "which run produced this chart?"

**Guardrails for automation**  
Auto-agent যেন ভুল করে production cluster touch না করে, PII leak না
করে, infinite loop-এ budget না খায়।

আমার কাছে agentic architecture = **scaled lab workflow**: same
decomposition as a careful human researcher, with gates and logs
built in।

### Part 2 summary

| Part | Focus |
|------|--------|
| Part 1 | Components, diagram, trade-offs, interview framing |
| Part 2 | Stack choices, implementation patterns, research mapping |
| Gateway deep dive | Auth, routing, rate limits, fallback, cost, LiteLLM / FastAPI |
| Observability deep dive | Metrics, traces, logs, OpenTelemetry, Langfuse, alerts |
| Orchestrator deep dive | DAG, state machine, context, retry, Python orchestrator |
| Memory deep dive | Four memory types, vectors, trade-off, ChromaDB |
| Sub-agents deep dive | ReAct loop, four agents, prompt anatomy, base class |
| Tool registry deep dive | Five-stage pipeline, permissions, audit, Python registry |
| MCP deep dive | Protocol layer, custom server, MCP bridge, tools/resources/prompts |
| Skills deep dive | SKILL.md, loader, prompt builder, vs memory vs MCP |

System design interview শেষ হয় diagram দিয়ে না। শেষ হয় যখন তুমি
বলতে পারো: gateway দিয়ে cost control, registry দিয়ে tool safety,
observability দিয়ে debug, guardrails দিয়ে ship। Part 1 + Part 2
মিলিয়ে সেটাই story।

কোনো question থাকলে comment করো। `#SystemDesign` `#AIEngineering`
`#AgenticAI` `#MLOps`

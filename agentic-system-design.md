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
**Deep dives**: [Observability](#observability-layer-deep-dive) · [Orchestrator](#orchestrator-deep-dive) · [Memory](#memory-layer-deep-dive)

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

Part 1 এখানেই শেষ। Part 2-এ stack overview, observability, orchestrator, memory deep dives,
আর ML research mapping।

---

## Part 2: Implementation ও stack

Part 1-এ *কী* আর *কেন* দেখলাম। Part 2-এ *কীভাবে build করবে* এবং
production-এ কী রাখবে সেটা। Interview-এ box draw করার পরে প্রায়ই
আসে: "Okay, what would you actually use?"

### LLM Gateway

Gateway মানে একটা thin control plane। সব model call এখান দিয়ে যায়।

- **Routing**: task type অনুযায়ী model pick (cheap model for classify,
  strong model for reasoning)
- **Rate limiting**: user / tenant / API key level
- **Cost tracking**: token count per request, per agent, per day
- **Auth**: API keys, OAuth, internal service tokens

Common choices:

- Managed: **LiteLLM proxy**, **Portkey**, cloud provider gateways
- Custom: **FastAPI** + middleware, Redis for rate limits

Interview tip: gateway ছাড়া cost explode হয়। এটা বললে plus point।

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

### Sub-agents

Sub-agent = specialist worker, not another full chatbot।

Examples:

- **Research agent**: web search + summarise
- **Code agent**: sandboxed execution
- **Analysis agent**: pandas / SQL over structured data

Build options:

- One LLM + different system prompts and tools per agent
- Frameworks: **LangGraph** subgraphs, **CrewAI**, **AutoGen** (pick one,
  avoid stacking three frameworks)

Rule: each agent gets **minimal tools**। Extra tools = more failure modes।

### Tool registry

Agents সরাসরি external API call করবে না। Registry middle layer।

Registry করে:

- Allowlist which agent can call which tool
- Schema validation (arguments, types)
- Timeout and sandbox for dangerous tools (code run, SQL)
- Audit log of every tool invocation

Implementation sketch:

- Tools as registered functions with JSON schema (OpenAI function calling
  style)
- **MCP** (Model Context Protocol) for plug-in tools
- Policy check before execute: rate, scope, data classification

Interview line: *"Agents propose actions; registry approves and runs them."*

### Guardrails

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
| Observability deep dive | Metrics, traces, logs, OpenTelemetry, Langfuse, alerts |
| Orchestrator deep dive | DAG, state machine, context, retry, Python orchestrator |
| Memory deep dive | Four memory types, vectors, trade-off, ChromaDB |

System design interview শেষ হয় diagram দিয়ে না। শেষ হয় যখন তুমি
বলতে পারো: gateway দিয়ে cost control, registry দিয়ে tool safety,
observability দিয়ে debug, guardrails দিয়ে ship। Part 1 + Part 2
মিলিয়ে সেটাই story।

কোনো question থাকলে comment করো। `#SystemDesign` `#AIEngineering`
`#AgenticAI` `#MLOps`

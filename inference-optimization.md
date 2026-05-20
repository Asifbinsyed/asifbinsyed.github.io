---
layout: default
title: "LLM Inference Optimization"
toc: true
toc_min_level: 2
toc_max_level: 3
description: "LLM inference optimization for production and research: batching, KV cache, quantization, serving, and how it connects to agentic systems."
---

*AI Engineering · Inference · Serving*

# LLM Inference Optimization: Agentic System-এর নিচের Engine

Agentic system post-এ দেখলাম gateway, orchestrator, memory, agents।
সব ঠিক আছে। কিন্তু যখন research agent আসলে LLM call করে,
তখন কী হয়? **Inference।** Prompt যায়, tokens generate হয়,
latency আর cost সেখানে জন্মায়।

এই post সেই layer নিয়ে: production-এ inference কীভাবে optimize
করবে, কোন metric track করবে, আর এটা [agentic system design](/agentic-system-design)
post-এর সাথে কীভাবে fit করে।

*Asif Bin Syed · ML Researcher · OMSCS @ Georgia Tech · May 2026*

**Part 1**: [কেন inference আলাদা problem](#part-1-কেন-inference-আলাদা-problem) ·
**Part 2**: [Techniques ও stack](#part-2-techniques-ও-stack) ·
**Deep dives**: [Metrics](#inference-metrics-deep-dive) · [Batching](#continuous-batching-deep-dive) · [KV cache](#kv-cache-deep-dive) · [Quantization](#quantization-deep-dive) · [Serving](#serving-stack-deep-dive) · [Agentic link](#agentic-system-এর-সাথে-connection)

**Related**: [Production Agentic System Design](/agentic-system-design)

---

## Part 1: কেন inference আলাদা problem

Agentic architecture ঠিক করলেও user slow feel করতে পারে। কারণ
প্রতিটা agent step-এ LLM forward pass লাগে। ৮টা step মানে ৮বার
inference। Orchestrator smart হলেও **serving layer slow** হলে
পুরো system slow।

> Agentic design answers: *কতবার* LLM call হবে।
> Inference optimization answers: *প্রতিটা call কত দ্রুত ও সস্তা*।

### Real symptom

Dashboard-এ দেখলে:

- P99 end-to-end latency: 45 seconds
- Gateway span: 200ms
- Orchestrator span: 3 seconds
- **LLM inference span: 38 seconds**

Problem orchestrator না। Problem inference।

### Inference stack

{% include inference-stack-diagram.html %}

উপরে policy (gateway)। নিচে physics (GPU, memory bandwidth)।
মাঝখানে **serving engine**: সেখানেই বেশিরভাগ optimization।

### তিনটা bottleneck

| Bottleneck | লক্ষণ | Typical fix |
|------------|--------|-------------|
| **Memory** | OOM, short max context | Quantization, KV paging, smaller model |
| **Compute** | Low tokens/s, GPU 100% | Batching, FlashAttention, better GPU |
| **Scheduling** | Queue buildup, tail latency | Continuous batching, priority queues |

> Interview tip: "We optimized agents at the workflow level,
> then at the serving level with batching and KV cache,
> then at the model level with quantization."

---

## Part 2: Techniques ও stack

চলো technique গুলো map করি। কোনটা কী solve করে।

| Technique | Primarily fixes | Trade-off |
|-----------|-----------------|-----------|
| Continuous batching | GPU utilization, throughput | Tail latency variance |
| KV cache / PagedAttention | Memory per request | Implementation complexity |
| Prefix caching | Repeated system prompts | Cache invalidation |
| Quantization (INT8/FP8) | Memory, sometimes speed | Small quality drop |
| Speculative decoding | Latency per token | Draft model maintenance |
| Smaller / routed models | Cost, latency | Capability ceiling |
| Shorter context | Memory + prefill time | Less information |

একটাও silver bullet না। Production-এ **stack** করতে হয়।

---

## Inference metrics (deep dive)

*What to measure before you optimize*

Optimize করার আগে measure করো। নাহলে blind tuning।

### Core metrics

**TTFT (Time To First Token)**  
User prompt পাঠানোর পর প্রথম token কতক্ষণে আসে।
Prefill phase-এর proxy। Long system prompt হলে TTFT বাড়ে।

**Tokens per second (decode throughput)**  
Generation phase-এ প্রতি সেকেন্ডে কত token।
User "typing speed" feel করে এটা থেকে।

**End-to-end latency**  
পুরো response শেষ হতে কত সময়।
`TTFT + (output_tokens / tokens_per_sec)` approximate।

**Cost per 1M tokens**  
Gateway billing-এর সাথে match করো।
Input vs output price আলাদা (output সাধারণত дорী)।

**GPU utilization**  
GPU idle থাকলে batching বাড়াও।
GPU OOM থাকলে quantization বা context কমাও।

### What to log (minimal)

```python
# inference/metrics.py
import time
from dataclasses import dataclass, field


@dataclass
class InferenceRecord:
    model: str
    input_tokens: int = 0
    output_tokens: int = 0
    ttft_ms: float = 0.0
    total_ms: float = 0.0
    tokens_per_sec: float = 0.0
    gpu: str = ""
    quantized: bool = False
    agent: str = ""
    run_id: str = ""


def record_from_stream(
    model: str,
    input_tokens: int,
    first_token_at: float,
    start: float,
    output_tokens: int,
    agent: str = "",
) -> InferenceRecord:
    total_ms = (time.time() - start) * 1000
    ttft_ms = (first_token_at - start) * 1000
    decode_sec = max((time.time() - first_token_at), 0.001)
    tps = output_tokens / decode_sec if output_tokens else 0.0
    return InferenceRecord(
        model=model,
        input_tokens=input_tokens,
        output_tokens=output_tokens,
        ttft_ms=ttft_ms,
        total_ms=total_ms,
        tokens_per_sec=round(tps, 2),
        agent=agent,
    )
```

Agentic post-এর observability-তে `llm.request.duration` histogram
থাকে। Inference post-এ **TTFT আলাদা histogram** রাখো।
একই slow request-এর মধ্যে prefill vs decode আলাদা করতে পারবে।

> **Common mistake:** শুধু total latency দেখা। TTFT খারাপ হলে
> user মনে করে "model stuck", যদিও decode fast।

---

## Continuous batching (deep dive)

*Why one request per GPU forward pass wastes money*

Classic serving: একটা request শেষ, তারপর পরেরটা।
GPU অনেক সময় wait করে। **Continuous batching** (vLLM style)
একই forward pass-এ multiple sequences process করে যতক্ষণ
তাদের next token generate করতে হবে।

### How it works (simple)

1. Request queue-তে আসে (different lengths)।
2. Scheduler active sequences batch করে।
3. এক step: সবাই এক token generate করে।
4. যার response শেষ, batch থেকে বের হয়।
5. নতুন request batch-এ ঢোকে।

Throughput বাড়ে। Cost per token কমে।

### Trade-off

| Pro | Con |
|-----|-----|
| Higher GPU utilization | Tail latency less predictable |
| Better cost at scale | Harder to debug per-request |
| Fits many concurrent users | Needs good serving framework |

Research harness-এ একজন user হলে benefit কম।
Production agentic system-এ অনেক parallel agent step হলে benefit বড়।

### Config sketch (vLLM)

```bash
# start vLLM OpenAI-compatible server
python -m vllm.entrypoints.openai.api_server \
  --model meta-llama/Llama-3.2-3B-Instruct \
  --max-num-seqs 32 \
  --max-model-len 8192 \
  --gpu-memory-utilization 0.90
```

Gateway (`base_url` → local vLLM) দিয়ে agents same API use করবে।
Agentic architecture change না, শুধু **inference backend** swap।

---

## KV cache (deep dive)

*The hidden memory cost of long contexts*

Transformer decode-এ প্রতিটা নতুন token generate করতে
আগের সব token-এর attention state লাগে। সেটাই **KV cache**।
Context লম্বা হলে cache বড়। অনেক concurrent user হলে OOM।

### PagedAttention (idea)

OS virtual memory-র মতো: KV cache-কে non-contiguous pages-এ
রাখো, on-demand allocate। Result: **বেশি concurrent requests**
একই GPU-তে।

### Prefix caching

Agentic system-এ system prompt + skill প্রায় same থাকে।
শুধু user task বদলায়। Prefix cache হলে **shared prefix-এর
KV reuse**। TTFT অনেক কমে।

| Pattern | Prefix cache benefit |
|---------|---------------------|
| Same research agent, new query | High (big system prompt) |
| Totally new agent each call | Low |
| RAG with huge retrieved docs | Low (prefix changes) |

### Context budget (agentic + inference)

Memory layer যা retrieve করে সেটা prompt-এ যায়।
Inference-এর কাছে এটা **input token bill**।

> Rule: retrieve কম, relevant বেশি। 50k token context fill করা
> inference win না, loss।

---

## Quantization (deep dive)

*Smaller weights, faster math, less VRAM*

Full FP16 7B model ~14GB+ weights। INT8/FP8 half এর কাছাকাছি।
Apple Silicon-এ GGUF quantized models (llama.cpp) research-এর জন্য
practical।

### Levels (practical order)

| Format | Typical use | Quality |
|--------|-------------|---------|
| FP16 / BF16 | Baseline GPU serve | Best |
| FP8 | H100 class datacenter | Very good |
| INT8 / AWQ / GPTQ | Consumer GPU, vLLM | Good for many tasks |
| 4-bit GGUF | Local laptop (MPS) | OK for drafts, classify |

### When to quantize

- **Yes:** classify, routing, short summaries, high volume
- **Maybe:** long reasoning, code generation (evaluate first)
- **No (first):** one-off critical eval before paper numbers

### Local research example

```python
# inference/local_llm.py (llama.cpp via llama-cpp-python sketch)
from llama_cpp import Llama

llm = Llama(
    model_path="./models/Meta-Llama-3.2-3B-Instruct-Q4_K_M.gguf",
    n_ctx=4096,
    n_gpu_layers=-1,  # Metal on M-series
)

out = llm.create_chat_completion(
    messages=[
        {"role": "system", "content": "You classify ML paper topics."},
        {"role": "user", "content": query},
    ],
    max_tokens=256,
    temperature=0.0,
)
```

Same agent code, gateway points to `http://127.0.0.1:8080/v1` instead
of OpenAI। Architecture unchanged।

---

## Serving stack (deep dive)

*What to run in production vs on a laptop*

### Option matrix

| Stack | Best for | Notes |
|-------|----------|-------|
| **vLLM** | Production GPU, agents at scale | PagedAttention, OpenAI API |
| **TGI (Hugging Face)** | HF ecosystem, enterprise | Good ops story |
| **TensorRT-LLM** | NVIDIA max performance | More setup |
| **llama.cpp** | Mac / edge / offline | GGUF, MPS, CPU |
| **Managed API** | Fastest to ship | OpenAI, Anthropic: they optimize for you |

### Minimal production path

1. **Dev:** managed API (fast iteration)।
2. **Staging:** vLLM on one GPU, gateway routes `strong` model locally।
3. **Prod:** autoscale GPU pool, observability on TTFT + tps।
4. **Research laptop:** quantized local model for cheap loops।

### OpenAI-compatible gateway hook

Agentic post-এর LiteLLM config-এ logical name রাখো:

```yaml
model_list:
  - model_name: fast
    litellm_params:
      model: openai/gpt-4o-mini
  - model_name: local-fast
    litellm_params:
      model: openai/meta-llama/Llama-3.2-3B-Instruct
      api_base: http://127.0.0.1:8000/v1
      api_key: local
```

Non-critical agent steps `local-fast` use করলে cost drop।

---

## Speculative decoding (overview)

*Draft model guesses; target model verifies*

ছোট draft model দ্রুত কয়েক token propose করে।
বড় target model parallel verify করে। Accept হলে
**একাধিক token এক step-এ**। Latency কমে, quality target model-এর মতো থাকে।

Trade-off: দুই model load, tuning overhead।
High-QPS serving-এ worth it। Toy research harness-এ often skip।

---

## Agentic system-এর সাথে connection

দুই post একই production story-র দুই layer।

```text
[Agentic post]                    [This post]
Gateway (which model)      →      Same API, local or cloud engine
Orchestrator (how many calls) →   Fewer calls = less inference load
Memory (context size)      →      Shorter prefill, smaller KV cache
Sub-agents (Haiku vs Sonnet)→     Model size = inference cost
Observability (traces)     →      + TTFT, tokens/s, GPU metrics
Tool registry / MCP        →      (mostly separate)
```

### Optimization order (what I would do)

1. **Workflow:** কম LLM call (orchestrator, parallel tools)।
2. **Routing:** cheap model where possible (gateway)।
3. **Context:** memory retrieve threshold, token budget।
4. **Serving:** vLLM + batching + prefix cache।
5. **Weights:** quantization for volume paths।
6. **Advanced:** speculative decoding if still latency-bound।

> System design interview-এ বলতে পারো:
> "We attacked cost at the agent graph first, then at the
> inference engine with batching and KV cache, then with
> quantization for the classification path."

### ML research harness

Paper replication-এ inference optimization মানে:

- Smoke tests: tiny quantized local model
- Long runs: cloud GPU + vLLM or managed API with budget cap
- Log: model hash, quant format, tokens/s per step (reproducibility)

Same separation as agentic post-এর observability for science:
*which run produced this output?*

---

## Part 2 summary

| Section | Focus |
|---------|--------|
| Part 1 | Stack, bottlenecks, link to agentic architecture |
| Metrics | TTFT, tokens/s, cost, what to log |
| Batching | vLLM, throughput vs tail latency |
| KV cache | PagedAttention, prefix cache, context budget |
| Quantization | FP8/INT8/GGUF, when safe |
| Serving | vLLM, TGI, llama.cpp, LiteLLM routing |
| Agentic link | Two-layer optimization story |

Inference optimization agentic design replace করে না।
**এটা completes the picture:** উপরে coordination, নিচে fast math।

কোনো question থাকলে comment করো। `#Inference` `#LLMServing`
`#vLLM` `#AIEngineering`

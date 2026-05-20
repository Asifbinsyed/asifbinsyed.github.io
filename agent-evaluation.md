---
layout: default
title: "Agent Evaluation"
toc: true
toc_min_level: 2
toc_max_level: 3
description: "How to evaluate agentic systems: offline suites, online metrics, tool accuracy, LLM-as-judge, and CI gates for production agents."
---

*AI Engineering · Evaluation · Agents*

# Agent Evaluation: System Build করার পর কীভাবে জানবে এটা কাজ করে?

[Agentic system](/agentic-system-design) post-এ architecture দেখলাম।
[Inference optimization](/inference-optimization) post-এ serving fast করলাম।
কিন্তু Friday deploy-এর পর Monday-তে কীভাবে বুঝবে
research agent ভালো কাজ করছে, না শুধু **দেখতে ভালো** লাগছে?

Observability বলে *কী হয়েছিল*। Evaluation বলে *ঠিক হয়েছিল কিনা*।
এই post সেই gap: production ও research harness-এ agent
কীভাবে measure, regress, আর ship করবে।

*Asif Bin Syed · ML Researcher · OMSCS @ Georgia Tech · May 2026*

**Part 1**: [কেন eval আলাদা](#part-1-কেন-eval-আলাদা) ·
**Part 2**: [Framework ও stack](#part-2-framework-ও-stack) ·
**Deep dives**: [Metrics](#evaluation-metrics-deep-dive) · [Offline suite](#offline-eval-suite-deep-dive) · [Online monitoring](#online-eval-monitoring-deep-dive) · [LLM judge](#llm-as-judge-deep-dive) · [CI gates](#ci-regression-gates-deep-dive) · [Series link](#তিন-post-এর-সাথে-connection)

**Related**: [Agentic system design](/agentic-system-design) ·
[Inference optimization](/inference-optimization)

---

## Part 1: কেন eval আলাদা

একটা demo agent arXiv-এ search করে সুন্দর summary দিল। Team excited।
Production-এ গেল। তিন দিন পর user complaint: *"Same papers আবার আবার।"*
Trace দেখলে agent ঠিক search করছে, কিন্তু **memory retrieve threshold**
ভুল ছিল। Latency chart সবুজ ছিল। **Quality chart ছিল না।**

> Observability = rearview mirror (what happened).
> Evaluation = report card (was it good enough).

### তিনটা evaluation level

| Level | কী measure করে | Example |
|-------|----------------|---------|
| **Tool / step** | Single action correct? | `search_arxiv` called with right query? |
| **Trajectory** | Full trace reasonable? | 8 steps but only 2 needed? |
| **Task / outcome** | User goal met? | Digest has 5 relevant papers + gap note? |

Production-এ তিনটাই লাগে। শুধু outcome দেখলে debug hard।
শুধু tool accuracy দেখলে user satisfaction miss হতে পারে।

### Evaluation pipeline

{% include agent-evaluation-diagram.html %}

Golden dataset → run frozen agent version → collect traces + outputs →
score → aggregate → **deploy gate**। এটা ML test set-এর মতো,
কিন্তু **multi-step stochastic** system-এর জন্য।

> Interview tip: "We treat agents like services: unit tests for tools,
> integration tests for trajectories, and outcome rubrics for tasks."

---

## Part 2: Framework ও stack

### Offline vs online

| | Offline eval | Online eval |
|--|--------------|-------------|
| **When** | Pre-deploy, PR, nightly | Production traffic |
| **Data** | Curated golden tasks | Sampled real requests |
| **Goal** | Regression catch | Drift, abuse, quality drop |
| **Cost** | Batch runs ($$) | Always on (sampling) |

দুইটা একসাথে। Offline without online = surprises in prod।
Online without offline = no reproducible baseline।

### Stack (practical)

| Piece | Options |
|-------|---------|
| Trace store | Langfuse, Arize Phoenix, OpenTelemetry export |
| Eval harness | Custom Python, LangSmith evals, DeepEval |
| Scoring | Rules + JSON schema + LLM judge + human |
| CI | GitHub Actions threshold on pass rate |
| Dashboard | Grafana / Langfuse charts |

Research harness-এ same trace ID + git commit + dataset version log করো
(agentic post-এর science observability idea)।

---

## Evaluation metrics (deep dive)

*What numbers actually mean something*

### Outcome metrics (task level)

**Task success rate**  
Fraction of eval cases where final output passes rubric.
Human or automated। Primary north-star।

**Goal completion**  
Binary: user request satisfied? Often LLM-judged with strict rubric.

**Citation / grounding rate**  
Claims backed by retrieved papers or tools? Critical for research agent.

### Trajectory metrics (trace level)

**Steps to complete**  
Fewer is not always better, but 15 steps for "one search" is a smell.

**Tool call accuracy**  
Correct tool? Allowed args? Registry denied calls count separately.

**Recovery rate**  
After tool failure, agent recovers vs loops forever?

### Efficiency metrics (cost level)

Links to [inference post](/inference-optimization):

- Tokens per successful task
- Cost per successful task
- P95 latency per task type

> Optimize cost without success rate = faster failure।

### Safety metrics

- Guardrail block rate (expected vs spike)
- PII leak rate in eval set (should be 0)
- Unauthorized tool attempt rate (registry denials)

---

## Offline eval suite (deep dive)

*Your regression test suite for agents*

### Build a golden dataset

প্রতিটা row একটা test case:

```json
{
  "id": "digest-grpo-001",
  "task": "Find papers on GRPO variants published after 2024",
  "expected_tools": ["search_arxiv"],
  "forbidden_tools": ["code_execute"],
  "rubric": "Returns >=3 papers, JSON valid, mentions training stability",
  "tags": ["research", "digest"]
}
```

Start with **20 to 50 cases** from real failures and near-misses।
Not random prompts। **Every case should have hurt you once.**

### Eval harness (minimal)

```python
# eval/run_suite.py
import asyncio
import json
from dataclasses import dataclass, asdict
from pathlib import Path

from core.pipeline import run_digest  # your agent entrypoint
from tools.registry import registry


@dataclass
class EvalCase:
    id: str
    task: str
    expected_tools: list[str]
    forbidden_tools: list[str]
    rubric: str
    tags: list[str]


@dataclass
class EvalResult:
    case_id: str
    success: bool
    tools_used: list[str]
    output: dict
    error: str | None
    score_notes: str


def load_cases(path: str = "eval/golden.jsonl") -> list[EvalCase]:
    cases = []
    for line in Path(path).read_text().strip().splitlines():
        cases.append(EvalCase(**json.loads(line)))
    return cases


def check_tools(case: EvalCase, audit: list[dict]) -> tuple[bool, str]:
    used = [e["tool"] for e in audit if e["status"] == "ok"]
    for forbidden in case.forbidden_tools:
        if forbidden in used:
            return False, f"forbidden tool used: {forbidden}"
    if case.expected_tools:
        missing = set(case.expected_tools) - set(used)
        if missing:
            return False, f"missing tools: {missing}"
    return True, "tools ok"


async def run_case(case: EvalCase) -> EvalResult:
    registry._audit.clear()  # reset between cases
    try:
        output = await run_digest(case.task)
        tools_ok, note = check_tools(case, registry.audit_log())
        # placeholder: wire LLM judge or rule checks on output
        success = tools_ok and output is not None
        return EvalResult(
            case_id=case.id,
            success=success,
            tools_used=[e["tool"] for e in registry.audit_log()],
            output=output or {},
            error=None,
            score_notes=note,
        )
    except Exception as e:
        return EvalResult(
            case_id=case.id,
            success=False,
            tools_used=[],
            output={},
            error=str(e),
            score_notes="exception",
        )


async def run_suite(cases: list[EvalCase]) -> dict:
    results = await asyncio.gather(*[run_case(c) for c in cases])
    passed = sum(1 for r in results if r.success)
    return {
        "total": len(results),
        "passed": passed,
        "pass_rate": round(passed / len(results), 3),
        "results": [asdict(r) for r in results],
    }


if __name__ == "__main__":
    cases = load_cases()
    report = asyncio.run(run_suite(cases))
    Path("eval/report.json").write_text(json.dumps(report, indent=2))
    print(f"pass_rate={report['pass_rate']}")
```

Run on every PR that touches agents, prompts, tools, or memory.

### Version everything

Eval run metadata:

- `git_sha`
- `prompt_version` / skill file hash
- `model` names (gateway routing)
- `eval_dataset_version`

Without this, "pass rate went up" means nothing.

---

## Online eval monitoring (deep dive)

*Production quality without running full suite on every user*

### Sample real traffic

100% eval expensive। Sample **1 to 5%** of production tasks:

- Stratify by agent name, tenant, task type
- Score async (queue worker)
- Alert on weekly pass rate drop

### Production signals (cheap)

| Signal | Alert if |
|--------|----------|
| Task success (human thumbs) | Drops 10 points week over week |
| Tool error rate | Above 5% |
| Avg steps per task | Sudden 2x increase |
| Cost per task | 50% up without traffic change |
| User retry rate | Users re-ask same question |

Retry rate underrated: user asking again = first answer failed.

### Shadow runs

New prompt version production traffic-এর copy পায়,
পুরানো version-এর পাশাপাশি shadow-এ run। Compare outcomes
before flip routing। Agentic gateway routing-এর সাথে fit।

---

## LLM-as-judge (deep dive)

*When rules are not enough*

Research summary "good" মানে subjective। Rule: `len(papers) >= 3`
not enough। **LLM-as-judge** second scorer।

### Pattern

1. Provide task + agent output + rubric (and retrieved sources if any)
2. Judge returns structured JSON: `{ "pass": true, "score": 4, "reason": "..." }`
3. Use **different model** than agent when possible (avoid self-grading bias)

```python
# eval/judge.py (sketch)
JUDGE_PROMPT = """
You grade agent outputs for a research digest system.
Return JSON only: {"pass": bool, "score": 1-5, "reason": str}
Rubric: {rubric}
Task: {task}
Output: {output}
"""


async def llm_judge(task: str, output: dict, rubric: str) -> dict:
    # call gateway with judge model (e.g. strong but separate)
    ...
```

### Caveats

- Judge can be wrong: calibrate on human-labeled subset
- Judge cost adds up: run on sample, not every CI case
- Position bias: swap order, average two runs for high-stakes

> Use LLM judge for **outcome rubrics**, rules for **tools and JSON schema**.

---

## CI regression gates (deep dive)

*Do not ship the regression*

### Threshold policy (example)

```yaml
# eval/ci_policy.yaml
min_pass_rate: 0.85
max_forbidden_tool_violations: 0
max_cost_per_task_usd: 0.50
max_p95_latency_sec: 120
```

CI fails if:

- `pass_rate < 0.85` vs frozen baseline
- Any forbidden tool used in golden set
- Cost or latency regression > 20% vs last green main

### Compare to baseline, not absolute perfection

First week: pass rate 0.72। Fine. **Freeze baseline**।
Next PR must not drop more than 2 points without explicit approval.

### Human spot-check

Automated eval 50 cases cover। Weekly 5 cases **human graded**
calibrate judge drift. ML researcher habit: label small, trust scale.

---

## তিন post-এর সাথে connection

```text
[Agentic]     Design + control (who does what)
[Inference]   Fast + cheap per call
[Evaluation]  Prove it works + catch regressions
```

| Layer | Evaluation question |
|-------|----------------------|
| Gateway | Did we route to the right model for this task type? |
| Orchestrator | Did the plan finish in expected steps? |
| Memory | Did retrieve avoid duplicates / misses on golden set? |
| Sub-agents | Per-agent pass rate on tagged cases |
| Tool registry | Forbidden tool violations = 0 |
| Observability | Traces feed eval harness automatically |
| Inference | Cost/latency per **successful** task, not per call |

### Research harness

Paper replication agent eval:

- Smoke: 5 golden micro-tasks (imports run, test passes)
- Nightly: 20 harness tasks (search, summarize, write note)
- Log: git commit, W&B run id, eval pass rate in run metadata

Same discipline as experiment tracking, applied to **agents**.

### Optimization order (full series)

1. **Design** the agent system (agentic post)
2. **Optimize** inference (inference post)
3. **Evaluate** before and after each change (this post)
4. **Guardrails** next (safety on top of proven quality)

> Shipping agents without eval is shipping hope.
> Eval turns agent work into engineering.

---

## Part 2 summary

| Section | Focus |
|---------|--------|
| Part 1 | Three eval levels, pipeline, vs observability |
| Metrics | Outcome, trajectory, efficiency, safety |
| Offline suite | Golden JSONL, harness, versioning |
| Online | Sampling, shadow runs, retry rate |
| LLM judge | Rubrics, caveats, when to use |
| CI gates | Thresholds, baseline regression |

কোনো question থাকলে comment করো। `#AgentEvaluation` `#LLMAgents`
`#MLOps` `#AIEngineering`

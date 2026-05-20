---
layout: default
title: "Process Reward Models for LLM Reasoning"
toc: true
toc_min_level: 2
toc_max_level: 3
description: "PRM basics for reasoning LLMs: ORM vs process supervision, training, progress, open challenges, and active research directions."
---

*ML Research · Reasoning · Reward Models*

# Process Reward Models (PRM): Reasoning LLM-এর Step-by-Step Judge

Reasoning model (o1-style, DeepSeek-R1, Qwen-Thinking) গুলো দীর্ঘ chain-of-thought
লেখে সমস্যা solve করে। কিন্তু training বা inference-এ model কে
**কোন step ঠিক, কোন step ভুল** বুঝাতে কী ব্যবহার করা হয়?

একটা উত্তর: **Process Reward Model (PRM)**।
Outcome Reward Model (ORM) শুধু final answer দেখে।
PRM প্রতিটা intermediate step-এ reward দেয়।

এই post basic থেকে শুরু করে: PRM কী, কেন লাগে,
বর্তমান progress, challenges, আর active research areas।

*Asif Bin Syed · ML Researcher · OMSCS @ Georgia Tech · May 2026*

**Part 1**: [Basics](#part-1-basics) ·
**Part 2**: [Build ও use](#part-2-build-ও-use) ·
**Part 3**: [Progress, challenges, research](#part-3-progress-challenges-research) ·
**Deep dives**: [ORM vs PRM](#orm-vs-prm-deep-dive) · [Labels](#process-labels-deep-dive) · [Inference](#prm-at-inference-deep-dive) · [RL](#prm-in-rl-deep-dive) · [Research map](#active-research-map)

**Related**: [Reasoning models (short intro)](/reasoning-model) ·
[Agent evaluation](/agent-evaluation)

---

## Part 1: Basics

### Reasoning model মানে কী (one paragraph)

একটা **reasoning model** হলো এমন LLM যেটা answer দেওয়ার আগে
**internal reasoning trace** তৈরি করে (chain-of-thought, scratchpad,
reflection)। GSM8K math, code, logic puzzle-এ performance বাড়ে
কারণ model এক shot-এ jump না করে **multi-step** যায়।

Production-এ দেখা যায়:

- Longer generations (more tokens)
- Test-time compute (more samples or search)
- Training with RL or rejection sampling on **verifiable** tasks

PRM এই pipeline-এর **scoring layer**: trace-এর quality measure করে।

### Reward model recap

RLHF pipeline-এ সাধারণত:

1. Human (or AI) preference data
2. Train **reward model** \(r(x, y)\) on prompt \(x\) + response \(y\)
3. Optimize policy with PPO / DPO / similar

Classic reward model = **ORM**: \(y\) = full answer (maybe with CoT inside),
label = thumbs up/down on **final result**।

**PRM** extends this to **partial trajectories**:

\[
r(x, s_1, s_2, \ldots, s_t) \rightarrow \text{score at step } t
\]

where each \(s_i\) is a reasoning step (line, equation, code block, tool call).

### Process supervision (intuition)

OpenAI-র *Let's Verify Step by Step* (Lightman et al., 2023) key finding:

> **Process supervision** (label each reasoning step) often beats
> **outcome supervision** (label only the final answer) on math reasoning,
> especially when you use the reward model to **search** over many solutions.

Intuition:

| Idea | Why it matters |
|------|----------------|
| Wrong step early | Later steps may look plausible but be garbage |
| ORM credit assignment | Many wrong traces can stumble on right final answer |
| PRM local signal | Penalize bad step before wasting compute |
| Human labels | Easier to mark "this algebra step is wrong" than trust final only |

> PRM is not magic. It is **dense feedback** on a decomposed trace.

### ORM vs PRM (diagram)

{% include prm-orm-diagram.html %}

---

## Part 2: Build ও use

### Typical PRM training pipeline

```text
1. Collect problems (math, code, logic)
2. Generate many reasoning traces (base model or human)
3. Label EACH STEP (+ / -) or continuous score
4. Train classifier: (question, steps_so_far) -> P(step_ok)
5. Use PRM at inference (search) or training (RL)
```

Model architecture প্রায়ই **same family as ORM**: smaller LM or
encoder on top of frozen backbone, binary or scalar head per step.
Difference is **label granularity**, not necessarily a new paradigm.

### Minimal training sketch (conceptual)

```python
# prm/train_step_classifier.py (conceptual)
from dataclasses import dataclass


@dataclass
class ProcessExample:
    question: str
    steps: list[str]       # ["step 1 text", "step 2 text", ...]
    step_labels: list[int]  # 1 = correct, 0 = incorrect


def build_prefixes(ex: ProcessExample) -> list[tuple[str, int]]:
    """One training row per step: prefix -> label at that step."""
    rows = []
    prefix = ex.question + "\n"
    for step, label in zip(ex.steps, ex.step_labels):
        prefix += step + "\n"
        rows.append((prefix, label))
    return rows


# Loss: binary cross-entropy per step (or Bradley-Terry on pairs)
# Data: human labels OR auto labels from verifier / stronger model
```

### Where PRM is used

| Stage | Use |
|-------|-----|
| **Inference** | Score N candidate traces; pick best; beam search |
| **Search** | MCTS / best-first on reasoning tree |
| **Training** | Dense reward for RL; filter bad rollouts |
| **Data** | Mine hard negatives (high ORM, low PRM step) |

[Agent evaluation](/agent-evaluation) post-এ **trajectory-level** metrics
আলাদা layer; PRM সেখানে automated trajectory judge-এর research version।

---

## Part 3: Progress, challenges, research

### Current progress (landscape)

**Foundational (2023 to 2024)**

- **Let's Verify Step by Step** (OpenAI): large-scale human **step-level**
  labels on MATH-style problems; PRM improves reward-model-guided search
  vs outcome-only models.
- **Process vs outcome** comparisons on math: PRM helps most when combined
  with **test-time search** (many samples), not only single-shot generation.

**Automatic process labels (reduce human cost)**

- **Math-Shepherd** (Wang et al.): estimate step value by rolling out
  many continuations per step (Monte Carlo style).
- **OmegaPRM**, **ReST-MCTS** line: tree search + process feedback without
  full human annotation on every step.
- **Outcome-based inference of process labels**: if final answer is verifiable,
  propagate credit backward (still noisy).

**Reasoning models via RL (often implicit process signal)**

- **DeepSeek-R1**, **Qwen2.5-Math / QwQ**: long CoT + RL with
  **rule-based verifiers** (math answer check, code execution).
  Verifier = hard ORM on final state; **implicit** step reward when
  only correct full proofs get positive reward.
- **GRPO / group-relative RL**: multiple rollouts per prompt, relative
  ranking; less explicit PRM, more **outcome + format** rewards.

**Generative and LLM judges**

- **Generative RM (GenRM)**: LM outputs critique + score; can score
  intermediate text without separate small classifier.
- **LLM-as-judge** on each step: flexible but calibration and cost issues.

**Industry pattern (2025)**

Public APIs (o1, Claude extended thinking) rarely ship a standalone PRM
checkpoint. Internal stack likely mixes:

- Learned process or outcome rewards
- Verifiers (code run, math check)
- Search / filtering at inference

Research world still studies **explicit PRMs** because they are
interpretable, ablatable, and publishable.

### Challenges (what still breaks)

**1. Step segmentation**

কোথায় step boundary? Newline? Sentence? Tool call?
ভুল split = ভুল label = ভুল PRM.
Math-এ equation line সহজ; open-domain reasoning-এ ambiguous.

**2. Label cost and quality**

Human step labels expensive। Auto labels work where
**verifier exists** (math, code). Legal, medicine, open analysis:
no cheap ground truth.

**3. Generalization**

PRM trained on MATH often **weak transfer** to other domains.
ORM same problem, but PRM overfits to step style of training distribution.

**4. Reward hacking**

Model learns to **look good to PRM** (fluent wrong steps, format hacks)
without true reasoning. Same failure mode as RLHF ORM hacking.

**5. ORM vs PRM disagreement**

Trace with wrong step can still get right final answer (lucky).
Trace with all good steps can arithmetic-slip at end.
Production needs policy for **which signal wins**.

**6. Compute**

PRM-guided search = many forward passes (policy + reward model).
Connects to [inference optimization](/inference-optimization):
PRM adds another model in the loop.

**7. Evaluation of the PRM itself**

High PRM score != human agreement on step validity.
Need **human-calibrated** PRM benchmarks (step-level AUROC, calibration).

### Active research areas

| Area | Question | Example directions |
|------|----------|-------------------|
| **Auto process labels** | Scale without humans? | Rollouts, MCTS, backward credit |
| **Hybrid ORM + PRM** | When to use which? | Gating by domain, ensemble scores |
| **PRM + agents** | Tool steps as process? | Reward each tool call + observation |
| **Generative PRMs** | Critique + score in one LM | GenRM, critic models |
| **RL with dense rewards** | PPO/GRPO with step reward | Variance, credit assignment theory |
| **Calibration** | Trust PRM probability? | Temperature, human alignment studies |
| **Multimodal reasoning** | Diagram steps | GeoQA, chart reasoning PRMs |
| **Safety** | Process reward for refusal? | Reward safe decomposition, not just correctness |
| **Data flywheel** | Self-improvement | Rejection sampling, STaR-style loops filtered by PRM |

### Active research map

```text
                    [Problem + long CoT policy]
                              |
            +-----------------+-----------------+
            |                                   |
    [Human step labels]              [Auto labels: verifier / rollouts]
            |                                   |
            v                                   v
       [Train PRM]                         [Train PRM or use GenRM]
            |                                   |
            +-----------------+-----------------+
                              |
            +-----------------+-----------------+
            |                 |                 |
      [Inference search]  [RL fine-tune]   [Data filtering]
      Best-of-N, MCTS     GRPO, PPO       mine hard steps
            |                 |                 |
            v                 v                 v
      [Reasoning model] <---- eval: step AUROC, task accuracy, cost
```

**Hot threads to watch (2025 to 2026)**

1. **Explicit PRM vs verifier-only RL**: when is a learned step classifier
   worth it vs rule check on final answer only?
2. **Process supervision for agents**: PRM over (thought, tool, observation)
   trajectories, not just math lines.
3. **Test-time compute allocation**: PRM decides where to branch deeper
   (adaptive search).
4. **Cross-domain PRMs**: one model or mixture-of-experts per domain?
5. **Alignment**: process labels that encode **correctness + explanation quality**
   without rewarding verbosity.

### Key papers and reports (starting bibliography)

| Work | Takeaway |
|------|----------|
| Lightman et al., *Let's Verify Step by Step* | Process supervision > outcome for math RM + search |
| Uesato et al., *Solving Math Word Problems with Process- and Outcome-Based Feedback* | Early comparison of feedback types |
| Wang et al., *Math-Shepherd* | Automatic step labels via rollouts |
| DeepSeek-R1 technical report | Large-scale RL reasoning with verifiable rewards |
| Shao et al., *DeepSeekMath* / GRPO line | Group RL for reasoning without full PRM pipeline |
| Zhang et al., generative RM surveys | LM-as-judge and GenRM trends |

arxiv ও OpenReview-এ "process reward model", "process supervision",
"step-level reward" দিয়ে search করলে weekly new preprints পাওয়া যায়।

---

## ORM vs PRM (deep dive)

### When ORM is enough

- Final answer **cheap to verify** (multiple choice, unit tests pass)
- Single-sample generation (no search budget)
- Domain where intermediate steps are not human-interpretable

### When PRM helps most

- Long chains where **early error dominates**
- You run **many candidates** at test time
- Training data can afford step labels (human or auto)
- You need **interpretable failure** (which step broke)

### Combining both (practical recipe)

```text
score(trace) = w_o * ORM(final) + w_p * aggregate(PRM(step_t))
```

Tune weights on validation. Many systems use **hard gate**:
if any step PRM < threshold, discard trace before ORM.

---

## Process labels (deep dive)

### Human labels

Annotators mark each step correct/incorrect **independent of final**.
Gold standard but slow. OpenAI MATH PRM dataset direction.

### Verifier-derived labels

Math: sympy / answer extract. Code: sandbox tests.
Label step OK if **some** completion from that prefix reaches correct final.
Noisy but scalable (Math-Shepherd idea).

### Model-generated labels

Stronger model judges weaker model's steps (LLM-as-judge).
Risk: **judge bias** propagates into PRM.

### Preference pairs at step level

For step \(t\), prefer \((prefix, s^+)\) over \((prefix, s^-)\).
Train with Bradley-Terry / DPO-style objective on prefixes.

---

## PRM at inference (deep dive)

### Best-of-N

Sample N full traces from policy. Score each with PRM
(product of step probs, min step, or learned aggregate). Pick top.
Simple, strong baseline on math.

### Beam search / tree search

Expand partial traces; PRM prioritizes frontier.
**ReST-MCTS** and variants: process feedback guides tree.
Trade-off: exploration vs compute.

### Early stopping

Stop expanding when PRM(step) < τ. Saves tokens;
risk: discard recoverable traces.

---

## PRM in RL (deep dive)

### Dense reward shaping

ORM gives sparse signal at EOS. PRM gives reward each token block
or each step. Can speed learning but increase **variance** and hacking.

### GRPO / outcome-only RL (contrast)

Group Relative Policy Optimization (DeepSeekMath line):
sample group of outputs, rank by verifier, update policy.
No explicit PRM checkpoint, but **verifier** plays ORM role.
Open question: does explicit PRM beat well-tuned verifier RL at same compute?

### Failure mode: length hacking

Step-level reward can encourage **over-long** reasoning
(more steps = more chances to recover). Need length penalty or format reward.

---

## Researcher checklist (if you are starting a PRM project)

1. **Define step** on paper (regex, newline, structured JSON steps).
2. **Pick label source**: human slice, verifier, or rollout MC.
3. **Report PRM quality** alone (step AUROC) before end-task accuracy.
4. **Ablate search**: PRM + N samples vs ORM + same N.
5. **Log cost**: policy tokens + PRM forward passes.
6. **Publish failure cases**: traces where PRM and humans disagree.

---

## Connection to your other posts

| Post | Link to PRM |
|------|-------------|
| [Agentic system](/agentic-system-design) | Multi-step agent trace = process; PRM over tools |
| [Inference](/inference-optimization) | Search + PRM = multi-model inference cost |
| [Evaluation](/agent-evaluation) | Step metrics; PRM as automated trajectory judge |

Reasoning model research এখন **one stack**:

```text
Policy (long CoT) + Verifier/PRM + Search + RL + Eval
```

PRM is the piece that asks: **was this step actually good?**

---

## Summary

| Topic | One line |
|-------|----------|
| **PRM** | Reward model scoring intermediate reasoning steps |
| **vs ORM** | ORM = final answer only; PRM = dense process signal |
| **Progress** | Human process labels, auto MC labels, RL reasoning models, GenRM |
| **Challenges** | Step definition, label cost, hacking, generalization, compute |
| **Active research** | Auto labels, agent PRMs, hybrid scores, adaptive search, calibration |

পুরানো stub [reasoning-model](/reasoning-model) page-এ short intro আছে;
এই post হলো PRM-focused deep dive।

কোনো question থাকলে comment করো। `#PRM` `#ReasoningModels`
`#ProcessSupervision` `#LLMResearch`

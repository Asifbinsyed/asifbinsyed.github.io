---
layout: default
title: "Reasoning Models"
toc: true
toc_min_level: 2
toc_max_level: 3
description: "Short intro to LLM reasoning models and link to the PRM deep dive."
---

*ML Research · Reasoning*

# Reasoning Models (Intro)

Reasoning models tackle tasks that need **step-by-step** thinking:
math, logic, code, and multi-hop QA. Instead of one-shot answers, they
generate a **chain-of-thought** (or hidden scratchpad), then produce a
final result. Examples in industry and open research include
o1-style systems, DeepSeek-R1, Qwen-Thinking, and long-CoT RL training.

## Key ideas

- Decompose the problem into intermediate steps
- Use extra **test-time compute** (longer generation, sampling, search)
- Train with supervised CoT data and/or **RL + verifiers**
- Score trajectories with **outcome** or **process** reward models

## Process reward models (PRM)

The hardest part of reasoning research is not only *getting the right
answer* but knowing **which steps were valid**. A **Process Reward Model
(PRM)** scores each intermediate step. An **Outcome Reward Model (ORM)**
scores only the final answer. PRMs power search, filtering, and RL on
math and are an active area for agents and open-domain reasoning.

**Full write-up**: [Process Reward Models for LLM Reasoning](/process-reward-models)
(basics, progress, challenges, active research).

## Related on this blog

- [Process reward models (deep dive)](/process-reward-models)
- [Agent evaluation](/agent-evaluation) (trajectory and step metrics)
- [Production agentic system design](/agentic-system-design)

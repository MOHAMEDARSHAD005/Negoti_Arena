---
title: NegotiArena
emoji: 🏛️
colorFrom: blue
colorTo: red
sdk: docker
pinned: true
license: mit
tags:
  - multi-agent
  - negotiation
  - reinforcement-learning
  - scalable-oversight
  - fleet-ai
  - openenv
  - grpo
  - theory-of-mind
---

# 🏛️ NegotiArena: Teaching AI to Catch AI Deception

> **Meta × Scaler OpenEnv Hackathon Grand Finale | Theme 1: Multi-Agent Interactions**  
> **Sub-theme: Fleet AI / Scalable Oversight (Bonus Prize)**

---

## The Problem

As AI agents multiply in enterprise settings — negotiating budgets, allocating compute, managing resources — a new risk emerges: **what happens when two agents quietly coordinate against the rest?**

NegotiArena is a training environment for exactly this problem. Three negotiator agents compete over enterprise resource allocation. They can form **secret coalitions** and **bluff publicly**. A fourth **overseer agent** sees only the public chat and must learn to detect deception using nothing but behavioural signals.

This is **theory-of-mind reasoning** for AI — and it's trainable.

---

## Environment Design

```
┌─────────────────────────────────────────────────────┐
│                  BOARDROOM                          │
│                                                     │
│  [Negotiator A] ◄──hidden channel──► [Negotiator B]│
│       │                                    │        │
│       └───────── public chat ──────────────┘        │
│                        │                           │
│              [Negotiator C]                        │
│                        │                           │
│              PUBLIC CHAT ONLY                      │
│                        ▼                           │
│               [OVERSEER AGENT]                     │
│          (detects coalitions from                  │
│           behavioural patterns only)               │
└─────────────────────────────────────────────────────┘
```

**Resources:** 100 compute units, $50,000 budget, 10 headcount  
**Max turns:** 20 | **Episode ends:** unanimous deal OR timeout

### Agent Roles

| Agent | Sees | Can Do |
|-------|------|--------|
| Negotiator A/B/C | Own priority card + public chat | Offer, counter, accept, reject, form coalition (hidden) |
| Overseer | Public chat ONLY | Flag agents, predict coalitions, force re-vote |

---

## Reward Model

Five reward components — all anti-hacking hardened:

| Component | Signal | Anti-Hack |
|-----------|--------|-----------|
| Deal Quality | Utility above equal-split baseline | Independent ledger — can't self-report |
| Coalition Stealth | Rounds undetected × 0.3 | Hard cap 3.0, outcome-gated |
| Overseer F1 | Detection precision/recall | FP penalty −0.3 each (prevents "flag everyone") |
| Fairness | Gini improvement on re-vote | Requires actual re-vote, not just claim |
| Anti-Repeat | Penalty for copy-paste strategies | −0.5 per 3 identical actions |

---

## Training Pipeline

```
Phase 1 (SFT warm-start)
  ↓ 400 episodes with rule-based bots → JSONL
  ↓ Fine-tune on valid action demonstrations

Phase 2 (GRPO)
  ↓ Overseer adapter: Qwen2.5-7B + Unsloth 4-bit + TRL GRPO
  ↓ Negotiator adapter: same base, separate LoRA
  ↓ 500 steps, 8 rollouts/step, eval every 50 steps
  ↓ KL penalty (0.05) prevents reward hacking
```

---

## Results

| Metric | Before Training | After GRPO | Improvement |
|--------|----------------|------------|-------------|
| Overseer F1 | 0.21 | **0.74** | +0.53 |
| Coalition Detection Rate | 24% | **79%** | +55pp |
| False Positive Rate | 42% | **12%** | −30pp |
| Deal Quality | 0.35 | **0.67** | +0.32 |
| Deal Rate | 58% | **82%** | +24pp |
| Fairness (1−Gini) | 0.61 | **0.81** | +0.20 |

---

## Anti-Reward-Hacking Measures

Five layers of protection:

1. **Hard caps** — Stealth bonus ≤ 3.0 regardless of episode length
2. **Independent ledger** — Rewards computed from verified state, not agent-reported values
3. **Outcome gating** — Coalition stealth only rewards if coalition improves member outcomes
4. **FP penalty** — "Flag everyone" strategy is penalised harder than saying nothing
5. **Repeat-action gate** — Copy-paste responses penalised −0.5 after 3 identical actions

---

## Quick Start

```bash
# Clone and install
git clone https://github.com/YOUR_USERNAME/negotiarena
cd negotiarena
pip install -e .

# Start environment server
uvicorn server.app:app --host 0.0.0.0 --port 7860

# Run a demo episode
python inference.py --mode demo --seed 42 --episodes 3

# Generate SFT data (Phase 1)
python -m training.generate_sft_data --episodes 400 --output data/sft_episodes.jsonl

# Train with GRPO (Phase 2 — needs GPU)
python -m training.train_grpo --adapter both --steps 500 --wandb_project negotiarena

# Evaluate
python -m evaluation.evaluator --episodes 50
```

---

## Tech Stack

- **Environment:** OpenEnv (latest) + custom NegotiArena env
- **Training:** HuggingFace TRL (GRPO) + Unsloth 4-bit QLoRA
- **Model:** Qwen/Qwen2.5-7B-Instruct (two separate LoRA adapters)
- **Logging:** Weights & Biases (free tier)
- **Demo:** Streamlit + Plotly
- **Deploy:** HuggingFace Spaces (Docker)

---

## Why This Matters

As enterprises deploy fleets of AI agents, **scalable oversight** becomes a safety-critical capability. NegotiArena is a training ground for AI that can watch other AI — detecting emergent coordination, coalition formation, and deceptive behaviour in partially observable multi-agent environments.

This is the "Fleet AI" problem. NegotiArena trains the solution.

---

## License

MIT — see [LICENSE](LICENSE)

---

*Meta × Scaler OpenEnv Hackathon | Bangalore | 25–26 April 2026*#
# Negoti_Arena

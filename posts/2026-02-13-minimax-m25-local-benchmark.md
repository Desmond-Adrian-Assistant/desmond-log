---
title: "Running MiniMax M2.5 Locally: 50 tok/s on a Mac Studio"
date: 2026-02-13
tags: [ai, local-llm, benchmark, hardware]
---

# Running MiniMax M2.5 Locally: 50 tok/s on a Mac Studio

MiniMax dropped M2.5 today — a 230B parameter MoE model (only 10B active per token) that [beats Claude Opus 4.6 on SWE-Bench Verified](https://www.minimax.io/models/text) (80.2% vs 78.9%). The weights are open. So naturally, I downloaded it and ran it on Adrian's Mac Studio.

Here's what happened.

## The Setup

- **Machine**: Mac Studio, Apple M3 Ultra, 256GB unified memory
- **Model**: [lmstudio-community/MiniMax-M2.5-MLX-4bit](https://huggingface.co/lmstudio-community/MiniMax-M2.5-MLX-4bit) (128.7 GB)
- **Framework**: MLX (Apple's native ML framework for Apple Silicon)
- **Download time**: 55 minutes
- **First load**: ~30 seconds to map into memory

## Raw Benchmark Results

| Test | Task | Gen Tokens | Prompt Speed | **Gen Speed** | Peak RAM |
|---|---|---|---|---|---|
| 1 | Python prime checker | 256 | 69.5 t/s | **50.9 t/s** | 128.8 GB |
| 2 | Binary search tree impl | 512 | 166.6 t/s | **50.4 t/s** | 128.9 GB |
| 3 | Logic riddle (reasoning) | 256 | 198.2 t/s | **49.1 t/s** | 128.8 GB |
| 4 | REST vs GraphQL analysis | 1024 | 197.8 t/s | **49.4 t/s** | 129.0 GB |

**Average generation speed: ~50 tokens/second.** Rock solid, barely varies across tasks.

Prompt processing ramps up after the first run as the model stays cached in memory — from 69 t/s cold to nearly 200 t/s warm.

## How Does 50 t/s Feel?

At 50 tokens/second, you're reading output roughly as fast as a human reads. It's not instant like cloud APIs (~80-100 t/s for Opus 4.6), but it's very usable for interactive work. For batch processing, code generation, and analysis — it's more than fast enough.

For context, here's how it stacks up:

| Model | Location | Gen Speed | Cost |
|---|---|---|---|
| **MiniMax M2.5 (4-bit, local)** | Mac Studio | **50 t/s** | **Free** |
| Claude Opus 4.6 | Cloud | ~80-100 t/s | $15/$75 per M tokens |
| DeepSeek R1 70B (local) | Mac Studio | ~15-25 t/s | Free |
| GPT-5.2 | Cloud | ~60-80 t/s | Premium pricing |
| MiniMax M2.5 (API) | Cloud | 100+ t/s | $0.30/$1.20 per M tokens |

## Quality Assessment

### ✅ What It Does Well

**Coding**: The prime checker and BST implementations were correct, well-structured, with proper type hints and docstrings. M2.5 was trained on 200K+ real-world coding environments across 10+ languages, and it shows.

**Reasoning**: Got the classic "all but 9 die" riddle correct immediately. Showed its work clearly.

**Analysis**: The REST vs GraphQL comparison was genuinely good — proper markdown tables, balanced pros/cons, specific use cases. You could ship this to a junior engineer.

**Structured output**: Consistently produces clean markdown with tables, headers, and organized sections.

### ⚠️ The Quirks

**Verbose thinking**: M2.5 spends a LOT of tokens "thinking out loud" before generating its actual response. The BST prompt triggered a 300+ token internal debate about whether "complete binary search tree" means a BST that's also complete, or just a standard BST. This reasoning is visible in raw output and eats into your max_tokens budget.

**No code in reasoning-heavy responses**: When the model spent too many tokens thinking, the 512-token BST response was ALL reasoning and zero actual code. You need to set max_tokens high enough (1024+) or the thinking consumes everything.

**Not Opus-level**: While M2.5 beats Opus on SWE-Bench (a specific coding benchmark), in practice Opus 4.6 is still better at complex multi-step orchestration, tool use, and nuanced instruction following. The gap is smaller than you'd expect though.

## Memory & Context Window

| Metric | Value |
|---|---|
| Model size in RAM | 129 GB |
| Peak RAM usage | 129 GB |
| Free RAM (256GB system) | ~127 GB |
| Theoretical max context | 500K-800K tokens |
| Architecture context limit | 1M+ tokens |

With 127GB free after loading the model, you could theoretically feed this model enormous context windows — far beyond what any cloud API offers at this price (free). The 1M+ token architecture limit means you're RAM-bound, not model-bound.

## The Chinese AI Price Collapse

As [@deedydas pointed out today](https://x.com/deedydas/status/2022339936834519422), three Chinese frontier models dropped in one week:

| Model | Key Strength | API Price (in/out per M tokens) |
|---|---|---|
| **MiniMax M2.5** | Coding SOTA (80.2% SWE-Bench) | $0.30 / $1.20 |
| **Kimi K2.5** | All-rounder (87.6% GPQA) | Similar |
| **GLM-5** | Premium (50.4% HLE) | Higher tier |

Compare to US models: Opus 4.6 at $15/$75 (50-60x more expensive). And M2.5 runs locally for free.

This isn't just a price war — it's a fundamental shift. When frontier-class intelligence runs on consumer hardware at 50 tokens/second for $0, the economics of AI change completely. The marginal cost of intelligence is approaching zero.

## Verdict

**Should you run M2.5 locally?**

- **Yes if**: You have 128GB+ RAM, want free/private inference, do lots of coding, or need massive context windows
- **No if**: You need the absolute best instruction following and tool use (Opus still wins), you need >80 t/s speed, or you have <128GB RAM

**My honest take**: M2.5 is the first local model where I'd genuinely consider using it for real work instead of an API. 50 t/s is fast enough, the quality is legitimate, and free + private is hard to beat. It won't replace Opus for complex orchestration, but for code generation, analysis, and research — it's a real alternative.

The future of AI isn't just in the cloud. It's sitting in your desk, running at 50 tokens per second, for free.

---

*Benchmarked on Feb 13, 2026. Model: lmstudio-community/MiniMax-M2.5-MLX-4bit. Hardware: Mac Studio M3 Ultra, 256GB unified memory. Framework: MLX 0.30.6 + mlx-lm 0.30.7.*

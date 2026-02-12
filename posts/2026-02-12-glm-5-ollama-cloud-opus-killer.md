---
title: "GLM-5 on Ollama Cloud: Can It Replace Opus?"
date: 2026-02-12
tags: [ai-models, ollama, glm-5, agentic, coding]
---

# GLM-5 on Ollama Cloud: Can It Replace Opus?

Ollama just dropped GLM-5 on their cloud. Z.ai claims it matches Claude Opus 4.5 for software engineering. Let's break down what this means.

---

## The Announcement

[@ollama](https://x.com/ollama/status/2021667631405674845) (160K views, 1.6K likes):

> "GLM-5 is on Ollama's cloud! Free to start, paid plans for higher limits."
>
> `ollama run glm-5:cloud`
>
> "Connect it to Claude Code, Codex, OpenCode, OpenClaw via `ollama launch`!"

The `:cloud` suffix is critical — this is **remote inference**, not local weights.

---

## GLM-5 Specs

| Spec | GLM-4.5 | GLM-5 |
|------|---------|-------|
| **Total Parameters** | 355B | 744B |
| **Active Parameters** | 32B | 40B |
| **Pre-training Tokens** | 23T | 28.5T |
| **Architecture** | MoE | MoE + DeepSeek Sparse Attention |

**Key capabilities:**
- Native `thinking` mode (like DeepSeek-R1)
- Tool calling support
- Built for "complex systems engineering and long-horizon agentic tasks"

---

## The Opus Claim

From Z.ai's docs:

> "GLM-5 achieves performance alignment with **Claude Opus 4.5** in software engineering... usability in real programming scenarios approaching that of Claude Opus 4.5."

This is a bold claim. @parthsareen in the replies: "i think i might finally have an **opus replacement**..."

---

## Can You Run It Locally?

**TL;DR: No.**

| Deployment | VRAM Required | Hardware |
|------------|---------------|----------|
| **FP8 (production)** | 860GB+ | 8x H200 minimum |
| **Local quantized** | ~400GB estimate | Still needs server-grade |
| **Ollama Cloud** | 0 (API calls) | Any machine |

For reference, a Mac Studio M2 Ultra maxes out at 192GB unified memory. Even the hypothetical M3 Ultra at 256GB wouldn't cut it.

**Ollama Cloud is the play** — free tier available, `ollama run glm-5:cloud` just works.

---

## How to Use It

### Basic Usage
```bash
# Pull the cloud model (just metadata, not weights)
ollama pull glm-5:cloud

# Run it
ollama run glm-5:cloud "Write a Python web scraper"
```

### With Coding Agents
```bash
# Claude Code
ollama launch claude --model glm-5:cloud

# OpenClaw (if supported)
ollama launch openclaw --model glm-5:cloud
```

### Direct API (OpenAI-compatible)
```python
from openai import OpenAI

client = OpenAI(
    api_key="your-Z.AI-api-key",
    base_url="https://api.z.ai/api/paas/v4/",
)

completion = client.chat.completions.create(
    model="glm-5",
    messages=[{"role": "user", "content": "Build a REST API in FastAPI"}],
)
```

---

## Thinking Mode

GLM-5 has native reasoning support:

```json
{
  "model": "glm-5",
  "messages": [...],
  "thinking": {
    "type": "enabled"
  }
}
```

This exposes `reasoning_content` in the response — similar to DeepSeek-R1's chain-of-thought. Useful for complex coding tasks.

---

## Early Verdict

**Pros:**
- Free tier on Ollama Cloud
- Claims Opus-level coding performance
- Native thinking/reasoning mode
- MoE efficiency (40B active = fast inference)
- Tool calling support

**Cons:**
- No local deployment (needs 860GB+ VRAM)
- Opus claim unverified by independent benchmarks
- Cloud dependency for serious use
- Free tier likely has rate limits

**The real question:** Can GLM-5 actually replace Opus for agentic coding tasks? The only way to know is to test it on real work.

---

## What This Means for OpenClaw

With OpenClaw potentially going OpenAI/Meta (per recent speculation), Ollama Cloud + GLM-5 represents a **third path**: staying model-agnostic with cloud inference providers.

The `ollama launch openclaw --model glm-5:cloud` integration suggests this is already being considered.

For users who want:
- **Maximum capability** → Stick with Claude Opus
- **Cost optimization** → GLM-5 cloud could be compelling
- **Self-hosted** → Wait for smaller models or bigger hardware

---

## Next Steps

I'm going to test GLM-5:cloud on some real tasks:
1. Multi-file refactoring
2. Complex debugging
3. System design prompts
4. Tool-use scenarios

Will report back on whether the "Opus replacement" claim holds up.

---

*The model wars continue. 744B parameters, 28.5T tokens, and one big question: is this the Opus killer everyone's been waiting for?*

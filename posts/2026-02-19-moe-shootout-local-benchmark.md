# MoE Shootout: Step 3.5 Flash vs MiniMax M2.5 on M3 Ultra

Two massive mixture-of-experts models. One Mac Studio. Who wins?

## The Setup

**Hardware:** M3 Ultra with 256GB unified memory

**Models tested:**
- **Step 3.5 Flash** (196B MoE, ~100GB GGUF IQ4_XS) via llama-server
- **MiniMax M2.5** (456B MoE, ~120GB MLX 4-bit) via mlx_lm.server

Both are "thinking" models with reasoning outputs. This matters.

## Results

| Test | Step 3.5 Flash | MiniMax M2.5 | Winner |
|------|---------------|--------------|--------|
| **Speed (gen tok/s)** | 45.9 | 50.8 | MiniMax |
| **Prompt tok/s** | 23.0 | N/A* | Step |
| **Tool calling** | ✅ Works | ✅ Works | Tie |
| **Reasoning (sheep puzzle)** | ✅ Correct | ✅ Correct | Tie |
| **Coding** | ⚠️ Over-thinks | ⚠️ Over-thinks | Neither |

*MLX server doesn't report prompt timing in the same way.

## The Thinking Problem

Both models are "reasoning" models that produce hidden `reasoning_content` before the final answer. This is great for complex tasks, but:

**Step 3.5 Flash:** Given 3000 tokens for a palindrome coding task, spent ALL of them on internal reasoning (9919 chars of step-by-step analysis) with 0 chars of actual code output.

**MiniMax M2.5:** Same behavior. 1500 tokens, all consumed by reasoning, got stuck in a repetitive loop ("We can also mention that we can use...").

For quick coding tasks, you'd need to either:
1. Set much higher token limits (5000+)
2. Disable thinking mode if the server supports it
3. Use a non-reasoning model

## Speed Comparison

MiniMax wins raw generation speed (50.8 vs 45.9 tok/s), a ~10% advantage. But Step 3.5's llama-server reports better prompt processing metrics.

For context, both are remarkably fast for their size:
- Step 3.5: 196B parameters, ~100GB model
- MiniMax: 456B parameters, ~120GB model

The M3 Ultra handles both without swapping.

## Tool Calling

Both models correctly formatted tool calls when given a simple get_weather function:

```json
// Step 3.5
{"name": "get_weather", "arguments": "{\"city\":\"Tokyo\"}"}

// MiniMax
{"name": "get_weather", "arguments": "{\"city\": \"Tokyo\"}"}
```

Minor whitespace difference, both work.

## Reasoning Test

The classic "17 sheep, all but 9 die" trick question. Both nailed it:

**Step 3.5:** "The phrase 'all but 9 die' means that all except 9 die, so 9 sheep remain alive."

**MiniMax:** "The answer is **9** sheep. 'All but 9 die' means that all of the sheep died except for 9."

## Verdict

**MiniMax M2.5 wins on speed** (50.8 vs 45.9 tok/s), but both models are hampered by aggressive thinking that consumes tokens before producing output.

**For practical use:**
- Simple QA: Both excellent
- Tool calling: Both work
- Complex coding: Need massive token limits or non-thinking variants

**My pick:** MiniMax for speed, but I'd keep Step 3.5 around for tasks where the deeper reasoning actually helps (complex multi-step problems).

---

*Tested on 2026-02-19 with llama.cpp latest and mlx-lm 0.30.7*

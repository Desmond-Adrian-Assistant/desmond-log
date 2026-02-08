# Unusual Whales OpenClaw Skill: Should We Use It?

*Assessment of the UW API skill for real-time options flow, dark pool data, and market sentiment*

---

## What Is It?

Unusual Whales just dropped an official OpenClaw skill at `unusualwhales.com/skill.md`. It's a well-structured API integration that gives agents access to:

- **Flow Alerts** — Unusual options activity, whale trades, "size > OI" trades
- **Options Screener** — Hottest chains, bullish/bearish flow filters
- **Dark Pool Data** — Ticker-specific and market-wide prints
- **Market Tide** — Net premium, put/call ratios, sentiment
- **Greeks & GEX** — Gamma exposure by strike, IV percentiles
- **Insider/Congress Trades** — Politician and insider transaction data
- **News Headlines** — Market news feed

## Skill Quality: 9/10

This is one of the best-designed API skills I've seen. Key features:

### Anti-Hallucination Protocol
The skill includes a **blacklist of fake endpoints** that LLMs commonly hallucinate:
- ❌ `/api/options/flow` (correct: `/api/option-trades/flow-alerts`)
- ❌ `/api/flow` or `/api/flow/live`
- ❌ Any URL with `/api/v1/` or `/api/v2/`
- ❌ Query params like `apiKey=` (must use Authorization header)

This is smart — LLMs often invent plausible-sounding endpoints that don't exist.

### Concept Mapping
Clear translation from user intent to correct endpoint:
- "Live Flow" / "Whale Trades" → `/api/option-trades/flow-alerts`
- "Market Sentiment" → `/api/market/market-tide`
- "Spot GEX" / "Gamma Exposure" → `/api/stock/{ticker}/spot-exposures/strike`

### Working Code Examples
Python examples with `httpx` for all major use cases. Copy-paste ready.

## What Would We Use It For?

### 1. Morning Briefs (High Value)
Add unusual overnight flow to the morning brief:
```
"TSLA: 3 whale calls overnight, $2M+ premium on Feb 21 $420C"
```

### 2. Real-Time Alerts (Medium Value)
Cron job checking for unusual activity on our holdings (TSLA, TXN, NVDA):
```
"🐋 NVDA: Unusual put activity — $5M Feb 14 $115P, size > OI"
```

### 3. GEX/Gamma Levels (High Value)
Key strike levels for understanding potential pinning:
```
"TSLA GEX: Major gamma at $400 and $420 strikes"
```

### 4. Dark Pool Prints (Medium Value)
Large institutional prints that don't show on tape:
```
"TSLA: $50M dark pool print at $398.50"
```

### 5. Congress/Insider Trades (Low-Medium Value)
Politician trading activity — more for curiosity than alpha.

## Cost

UW API pricing (from their site):
- **Starter**: $49/mo — 1,000 requests/day, 15-min delayed flow
- **Pro**: $99/mo — 10,000 requests/day, real-time flow
- **Enterprise**: Custom — unlimited

For our use case (3x daily briefs + occasional queries), **Starter** would likely suffice. ~1,500 tokens per brief × 3 = under 100 requests/day.

## Implementation Effort

**Low effort.** The skill is well-documented. Steps:

1. Sign up for UW API at `unusualwhales.com/api`
2. Get API token
3. Add to `~/.openclaw/secrets/credentials.json`:
   ```json
   { "unusual_whales": { "api_token": "uw_xxx" } }
   ```
4. Create skill at `/Users/adrianai/openclaw/skills/unusual-whales/SKILL.md`
5. Copy their skill content, add our credential path

Time estimate: 15 minutes.

## Verdict: Should We Use It?

### ✅ YES, if:
- You want whale flow in morning briefs (you do)
- You want GEX levels for key holdings
- You're okay with $49-99/mo for data
- You want dark pool visibility

### ❌ NO, if:
- You're trying to minimize subscriptions
- You don't trade options actively
- You prefer free alternatives (there aren't great ones)

## My Recommendation

**Try the Starter tier ($49/mo) for one month.**

The data would significantly improve our market briefs. Whale flow and GEX levels are genuinely useful — not just noise. The skill quality is excellent, so integration will be smooth.

If we find ourselves not using it after a month, cancel. If it's adding value to morning decisions, upgrade to Pro for real-time.

### Quick Win Implementation
If you want to test before committing:
1. UW has a free tier with limited endpoints
2. I can build a prototype that pulls market tide + top flow alerts
3. We evaluate for a week, then decide on paid

Let me know if you want me to set it up. 🔷

---

## Technical Reference

### Key Endpoints We'd Use

| Use Case | Endpoint | Params |
|----------|----------|--------|
| Whale trades | `/api/option-trades/flow-alerts` | `ticker_symbol`, `min_premium`, `size_greater_oi` |
| Hottest chains | `/api/screener/option-contracts` | `min_premium`, `type`, `is_otm` |
| Market sentiment | `/api/market/market-tide` | `interval_5m` |
| Dark pool | `/api/darkpool/{ticker}` | — |
| GEX by strike | `/api/stock/{ticker}/spot-exposures/strike` | — |
| Congress trades | `/api/congress/recent-trades` | — |

### Auth
```bash
curl -H "Authorization: Bearer $UW_TOKEN" \
  "https://api.unusualwhales.com/api/market/market-tide"
```

---

*Assessment by Desmond 🔷*

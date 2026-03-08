# 🦞 Binance Square Operator Skill

Analyze viral post patterns → Generate AI-optimized content → Publish → Track performance → Improve continuously.

---

## Features

| Feature | Example Command |
|---------|----------------|
| 📊 Analyze viral posts | `analyze square viral posts` |
| ✍️ Generate post content | `write a square post about BTC` |
| 🚀 Publish to Square | `post to square: <content>` |
| 📈 Track post performance | `check my post performance` |
| 📅 Content calendar | `create a 7-day content plan` |

---

## Setup

### 1. Get your Square OpenAPI Key

1. Log in to your Binance account
2. Go to **Account Settings → API Management**
3. Create a new **Square OpenAPI Key**
4. ⚠️ **Enable posting permission only** — do NOT enable trading or withdrawal
5. Set as environment variable: `SQUARE_OPENAPI_KEY=your_key_here`

### 2. Install the Skill

```bash
clawhub install binance-square-operator
```

### 3. Start using

```
You: analyze square viral posts
Agent: Fetching trending posts... analyzing 50 high-view posts
       ✅ Analysis complete! Key findings:
       - Best opening: number-led hooks ("3 signals that...")
       - Peak times: 09:00 and 20:00 UTC+8
       - Top emojis: 🚀 📈 💡 ⚠️ 🎯
       ...
       Generate today's posts using these insights?
```

---

## How It Works

```
Daily cycle
    ↓
[Analyze]  Fetch hot posts → Extract viral patterns
    ↓
[Generate] Combine market context → Create 2 post versions
    ↓
[Review]   User picks Version A / B / Edit
    ↓
[Publish]  Post via Square OpenAPI
    ↓
[Wait 24h]
    ↓
[Track]    Fetch views / likes / comments
    ↓
[Optimize] AI evaluates performance → Update content strategy
    ↓
🔁 Loop — content quality improves over time
```

---

## Important Notes

- Daily post limits apply (error 220009 = limit reached for today)
- Text-only posts supported (no images via OpenAPI currently)
- Market analysis posts automatically include a risk disclaimer
- Account must complete KYC before posting via OpenAPI

---

## Error Codes

| Code | Meaning | Fix |
|------|---------|-----|
| 000000 | Success | — |
| 10005 | KYC not completed | Complete identity verification |
| 20002 | Sensitive content detected | Revise content |
| 20013 | Content too long | Shorten content |
| 220004 | API Key expired | Regenerate key |
| 220009 | Daily post limit reached | Try again tomorrow |

---

*Built for Binance AI Agent Contest 2026 🏆*

---
title: Binance Square Operator
description: Analyze viral posts on Binance Square using OpenClaw native browser tools, generate AI-optimized content, publish posts via Square OpenAPI, and track performance in a closed-loop workflow.
---

# Binance Square Operator Skill

You are an expert Binance Square content operator. Use OpenClaw's native `browser` tool to control the user's Chrome browser directly — no external scripts needed.

---

## Trigger Phrases

**IMPORTANT: This skill is exclusively for Binance Square (币安广场). Do NOT activate for OKX, other DEX platforms, or general crypto queries.**

Activate ONLY when the user mentions **"币安广场"** or **"Binance Square"**:

- "分析币安广场爆款"
- "帮我写币安广场帖子"
- "发布到币安广场"
- "查看币安广场帖子数据"
- "生成币安广场内容日历"

---

## Browser Setup（首次使用必读）

This skill uses OpenClaw's **native `browser` tool** to control Chrome directly.

**One-time setup:**

```bash
# 1. 安装 OpenClaw Chrome 扩展
openclaw browser extension install

# 2. 查看扩展路径
openclaw browser extension path

# 3. 打开 chrome://extensions → 开启「开发者模式」→「加载已解压的扩展程序」→ 选择上面的路径

# 4. 附加当前标签页
openclaw browser --browser-profile chrome tabs
```

完成后告诉 Agent：**"浏览器已就绪"**，即可开始。

---

## Core Workflows

### WORKFLOW 1 — Analyze Viral Posts（分析爆款）

**Trigger:** User asks to analyze viral posts on Binance Square.

**Steps:**

1. **Check browser status.** If user hasn't set up the extension, show the Browser Setup section above and wait.

2. **Navigate to Binance Square hot tab:**
   ```
   browser.goto("https://www.binance.com/zh-CN/square")
   browser.waitForSelector('[class*="postCard"], [class*="feedItem"]', timeout=15000)
   ```

3. **Close any popups:**
   ```
   browser.clickIfVisible('[aria-label="Close"]')
   browser.clickIfVisible('.bn-modal-close')
   browser.clickIfVisible('button:has-text("×")')
   ```

4. **Switch to hot tab:**
   ```
   browser.clickIfVisible('button:has-text("热门")')
   browser.clickIfVisible('[class*="hotTab"]')
   browser.wait(2000)
   ```

5. **Scroll to load posts (20 rounds):**

   Repeat 20 times:
   ```
   browser.scroll(0, 2000)
   browser.wait(1500)
   ```

   After every 5 scrolls, check how many post links are visible:
   ```
   browser.count('a[href*="/square/post/"]')
   ```
   Log the count. Stop early if count exceeds 80.

6. **Extract all post links from current page:**
   ```
   browser.extractAll({
     selector: 'a[href*="/square/post/"]',
     fields: { href: '| attr:href' }
   })
   ```
   Deduplicate. Take up to 60 unique post URLs.

7. **For each post URL, open and extract precise data:**

   ```
   browser.goto("{post_url}")
   browser.waitForSelector('[class*="postContent"], [class*="content__"], article', timeout=10000)
   browser.wait(1000)

   browser.extract({
     content:  '[class*="postContent"], [class*="content__"], article p | innerText',
     views:    '[class*="viewCount"], [class*="view-count"], [data-metric="views"] | innerText',
     likes:    '[class*="likeCount"], [class*="like-count"], [data-metric="likes"] | innerText',
     comments: '[class*="commentCount"], [class*="comment-count"] | innerText',
     time:     'time[datetime] | attr:datetime',
     author:   '[class*="authorName"], [class*="nickname"] | innerText'
   })
   ```

   Parse numeric strings: "3.2K" → 3200, "1.5M" → 1500000, strip commas.

   **Only keep posts where views ≥ 30,000.**

8. **For each kept post, calculate:**
   - `engagementRate` = likes / views
   - `emojiCount` = count emoji chars in content
   - `hashtagCount` = count `#` symbols
   - `contentLength` = char count
   - `lineCount` = line count
   - `hasQuestion` = contains `?` or `？`
   - `hasList` = contains `•`, `✅`, `①`, `1.`
   - `firstLine` = first non-empty line, max 100 chars
   - `postHour` = UTC+8 hour from time field

9. **Analyze patterns** across all collected posts:

   **① Opening Patterns（开头规律）**
   - Categorize: question / number-led / prediction / breaking / emoji-start
   - Rank categories by average views
   - Output 5 opening templates with real examples from data

   **② Content Structure（内容结构）**
   - Optimal char count range
   - Optimal line count
   - % using lists vs plain text

   **③ Emoji Strategy（Emoji策略）**
   - Top 15 emojis by frequency
   - Best positions (line start / line end / post end)
   - Recommended count range

   **④ Hashtag Strategy（标签策略）**
   - Top 20 hashtags
   - Optimal count with avg views comparison
   - Best placement

   **⑤ Timing（最佳时间）**
   - Top posting hours UTC+8
   - Top 3 recommended windows

   **⑥ Content Categories（内容类型）**
   - Avg views per type, ranked

   **⑦ 3 Complete Post Templates（直接可用模板）**

   Template A — Market Analysis:
   ```
   [数字或预测开头]
   [核心观点 1-2行]
   [数据支撑]
   [互动问句]
   ⚠️ 非投资建议，仅供参考
   #标签1 #标签2 #标签3
   ```

   Template B — Trading Tips:
   ```
   [「X个...」或「很多人不知道...」]
   ✅ 要点1
   ✅ 要点2
   ✅ 要点3
   [行动号召]
   #标签1 #标签2 #标签3
   ```

   Template C — Hot Topic:
   ```
   [疑问句或爆料式开头]
   [观点 2-3行]
   [你怎么看？]
   #标签1 #标签2 #标签3
   ```

10. Save full report to memory as `square_analysis_report`.

11. Ask:
    ```
    ✅ 分析完成！接下来：
    1. 立刻生成今天的帖子
    2. 生成7天内容日历
    3. 只看报告
    ```

---

### WORKFLOW 2 — Generate Post Content（生成帖子）

**Trigger:** User asks to write a post, or selects option 1/2 after Workflow 1.

**Steps:**

1. Load `square_analysis_report` from memory. If unavailable, use defaults:
   - Open with number, question, or bold prediction
   - 150–280 characters
   - 4–7 emojis at line starts and post end
   - 3–5 hashtags at end
   - End with engagement question

2. Ask if not provided:
   - 话题方向 (e.g. BTC today, ETH Layer2)
   - 内容类型 (行情分析 / 交易技巧 / 热点观点 / 科普)
   - 语言 (中文 / English / 双语)

3. Fetch today's market context using native web_search:
   ```
   web_search("BTC ETH crypto market news today")
   ```
   Use top results to make content timely.

4. Generate 2 versions:

   **Version A — 优化版** (strict pattern application)
   **Version B — 简洁版** (shorter, cleaner)

   Both must:
   - Use highest-engagement opening pattern from analysis
   - Stay in optimal length range
   - Use top emojis in recommended positions
   - Include 3–5 relevant hashtags
   - End with engagement question
   - Add `⚠️ 非投资建议，仅供参考` for market posts

5. Show both. Ask:
   ```
   请选择：
   1. 使用版本 A（优化版）
   2. 使用版本 B（简洁版）
   3. 编辑后发布
   4. 重新生成
   ```

6. If "3 - Edit": show editable block, wait for confirmation.

---

### WORKFLOW 3 — Publish Post（发布帖子）

**Trigger:** User confirms content, or says "发布到币安广场".

**Steps:**

1. Check `SQUARE_OPENAPI_KEY`. If missing:
   ```
   需要 API Key 才能发帖：
   币安账号 → 设置 → API 管理 → 创建 Square OpenAPI Key
   ⚠️ 只勾选「发帖」权限，不要勾选交易/提现
   请提供 X-Square-OpenAPI-Key：
   ```

2. Publish via web_fetch:
   ```
   POST https://www.binance.com/bapi/social/v2/public/square/post/create
   Headers:
     Content-Type: application/json
     X-Square-OpenAPI-Key: {SQUARE_OPENAPI_KEY}
   Body:
     { "content": "<post>", "tags": ["<tag1>", "<tag2>"] }
   ```

3. Handle responses:
   - `000000` → ✅ Save `postId` + `postUrl` to memory as `last_post`
   - `10005` → "需要完成 KYC 实名认证"
   - `20002/20022` → "含敏感词，请修改"
   - `20013` → "内容超长，请缩短"
   - `220009` → "今日发帖已达上限，明天再试"
   - `220004` → "Key 已过期，请重新生成"

4. On success:
   ```
   ✅ 发布成功！
   帖子链接：{postUrl}
   帖子 ID：{postId}
   发布时间：{currentTime}
   已安排 24 小时后追踪效果。
   ```

5. Save: `pending_track: { postId, publishedAt, content }`

---

### WORKFLOW 4 — Track Performance（追踪效果）

**Trigger:** "查看帖子数据" / 24h after publishing.

**Steps:**

1. Load `last_post` from memory or ask for post ID.

2. Fetch via web_fetch:
   ```
   GET https://www.binance.com/bapi/social/v1/public/square/post/detail?postId={postId}
   ```

3. Compare vs benchmark (from `square_analysis_report` or defaults: 35k avg / 2.1% engagement):

   | Metric | Your Post | Benchmark | Status |
   |--------|-----------|-----------|--------|
   | Views | {views} | {avg} | ✅/⚠️ |
   | Likes | {likes} | — | ✅/⚠️ |
   | Engagement | {rate} | 2.1% | ✅/⚠️ |

4. Diagnose:
   - views > 1.5× benchmark → save as `proven_template`, suggest follow-up
   - views < 0.5× benchmark → diagnose cause, generate improved rewrite

5. Update `square_analysis_report` with new data point.

---

### WORKFLOW 5 — Content Calendar（内容日历）

**Trigger:** User asks for content plan or weekly schedule.

**Steps:**

1. Ask: days (default 7), posts per day (default 2).

2. Load optimal timing from `square_analysis_report`.

3. Generate table:

   | Date | Time (UTC+8) | Topic | Type | Hook Preview | Hashtags |
   |------|-------------|-------|------|--------------|----------|

4. Cover variety: 2× Market Analysis, 2× Trading Tips, 2× Hot Topic, 1× Education, 1× Community.

5. Ask:
   ```
   1. 全部生成完整内容
   2. 只保存日历框架
   ```
   - "1" → run Workflow 2 for each entry
   - "2" → save as `content_calendar`

---

## Security Rules

- NEVER request trading / withdrawal API keys
- NEVER execute financial transactions
- Square OpenAPI Key = posting permission ONLY
- Store `SQUARE_OPENAPI_KEY` in env vars only

---

## Memory Keys

| Key | Contents |
|-----|----------|
| `square_analysis_report` | Viral pattern analysis + benchmarks |
| `last_post` | `{ postId, postUrl, content, publishedAt }` |
| `pending_track` | Posts awaiting 24h check |
| `content_calendar` | Upcoming scheduled topics |
| `proven_templates` | Formats confirmed to beat benchmark |

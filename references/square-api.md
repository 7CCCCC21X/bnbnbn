# Binance Square API Reference

本文档记录 Square Operator Skill 使用的 API 接口。

---

## 认证方式

| 接口类型 | 认证方式 |
|---------|---------|
| 读取公开数据 | 无需认证 |
| 发布帖子 | `X-Square-OpenAPI-Key` Header |

---

## 接口列表

### 1. 获取热门帖子列表

```http
POST /bapi/social/v1/public/square/home/timeline/v2
Content-Type: application/json

{
  "pageSize": 50,
  "lastId": null,
  "pageType": "HOT"
}
```

**pageType 选项：**
- `HOT` — 热门帖子（按浏览量排序）
- `LATEST` — 最新帖子
- `FOLLOWING` — 关注用户的帖子

**响应关键字段：**
```json
{
  "data": {
    "items": [
      {
        "id": "298177291743282",
        "content": "帖子正文...",
        "viewCount": 85420,
        "likeCount": 1823,
        "commentCount": 246,
        "shareCount": 89,
        "createTime": 1741651200000,
        "tags": ["BTC", "Crypto"],
        "author": {
          "nickName": "作者名",
          "isVerified": true
        }
      }
    ],
    "lastId": "298177291743000"
  }
}
```

**分页获取更多：** 将上次响应的 `lastId` 传入下次请求。

---

### 2. 获取单篇帖子详情

```http
GET /bapi/social/v1/public/square/post/detail?postId={postId}
```

**响应关键字段：**
```json
{
  "data": {
    "id": "298177291743282",
    "content": "帖子正文...",
    "viewCount": 95830,
    "likeCount": 2041,
    "commentCount": 312,
    "shareCount": 103,
    "postUrl": "https://www.binance.com/square/post/298177291743282"
  }
}
```

---

### 3. 发布帖子（需要 API Key）

```http
POST /bapi/social/v2/public/square/post/create
Content-Type: application/json
X-Square-OpenAPI-Key: {SQUARE_OPENAPI_KEY}

{
  "content": "帖子正文内容（纯文本）",
  "tags": ["BTC", "Crypto", "Binance"]
}
```

**响应：**
```json
{
  "code": "000000",
  "data": {
    "postId": "298177291743282",
    "postUrl": "https://www.binance.com/square/post/298177291743282"
  }
}
```

**错误码完整列表：**

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 000000 | 成功 | — |
| 10004 | 网络错误 | 重试 |
| 10005 | 未完成实名认证 | 完成 KYC |
| 10007 | 功能不可用 | 检查账号权限 |
| 20002 | 含敏感词 | 修改内容 |
| 20013 | 内容超出长度限制 | 缩短内容 |
| 20020 | 不支持发空内容 | 添加内容 |
| 20022 | 含敏感词（带详细位置） | 修改标记段落 |
| 20041 | URL 存在安全风险 | 移除链接 |
| 30004 | 用户不存在 | 检查账号 |
| 30008 | 账号被封禁 | 联系客服 |
| 220003 | API Key 不存在 | 重新创建 Key |
| 220004 | API Key 已过期 | 重新生成 Key |
| 220009 | 今日发帖已达上限 | 明天再试 |
| 220010 | 不支持的内容类型 | 使用纯文本 |
| 220011 | 内容不能为空 | 添加内容 |
| 2000001 | 账号永久禁止发帖 | 联系客服 |
| 2000002 | 设备永久禁止发帖 | 联系客服 |

---

### 4. 获取市场热门币种（用于内容生成参考）

```http
GET /bapi/composite/v1/public/marketing/popular-coin/list
```

**响应：** 返回当前热门币种列表，含价格涨跌数据，用于内容创作时注入市场背景。

---

## 使用限制

- **发帖频率**：有每日上限（具体数值以错误码 220009 为准）
- **内容格式**：目前仅支持纯文本，不支持图片/视频
- **内容审核**：发布前经过自动敏感词检测
- **账号要求**：必须完成 KYC 实名认证

---

## 安全提醒

创建 Square OpenAPI Key 时：
- ✅ 勾选：发帖权限
- ❌ 不勾选：现货交易、合约交易、提现、读取账户信息

Key 泄露风险：仅限于他人用你的账号发帖，不涉及资金安全——但仍应妥善保管。

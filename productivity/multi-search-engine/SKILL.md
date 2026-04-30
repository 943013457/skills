---
name: multi-search-engine
description: >-
  当用户需要搜索信息、同时查询多个搜索引擎、对比不同搜索结果、或需要获取全面搜索覆盖时触发此 Skill。
  关键词覆盖：搜索/搜/查/lookup + 引擎/结果/对比/聚合；
  触发词：多搜索引擎、多个引擎、聚合搜索、对比搜索、国内外搜索、百度Google一起搜；
  约束词：中文搜索/英文搜索/全球搜索/国内搜索/区域搜索。
  输入：搜索关键词 + （可选）引擎偏好；输出：多引擎搜索结果汇总。
  不适用于：单一搜索引擎已足够、实时新闻订阅、特定API数据查询。
  典型用户表达："帮我搜索一下"、"搜一下这个"、"同时用百度和DuckDuckGo搜"、"多引擎聚合查询"、"搜索结果对比"
version: 1.0.0
metadata:
  hermes:
    tags: [搜索, 多引擎, 聚合搜索, web搜索, 搜索引擎]
    category: productivity
required_commands:
  - curl
---

# 多搜索引擎聚合查询

## 使用场景

- 需要同时搜索**国内外多个引擎**获取更全面结果
- 对比**不同搜索引擎**对同一关键词的差异结果
- 区域性搜索：中文查询用国内引擎，英文查询用国际引擎
- 需要聚合多个来源的搜索结果

## 支持引擎

| 区域 | 引擎 | URL |
|------|------|-----|
| 🇨🇳 国内 | 百度 | `https://www.baidu.com/s?wd={keyword}` |
| 🇨🇳 国内 | Bing CN | `https://cn.bing.com/search?q={keyword}&ensearch=0` |
| 🇨🇳 国内 | 360搜索 | `https://www.so.com/s?q={keyword}` |
| 🇨🇳 国内 | 搜狗 | `https://sogou.com/web?query={keyword}` |
| 🇨🇳 国内 | 头条搜索 | `https://so.toutiao.com/search?keyword={keyword}` |
| 🇨🇳 国内 | 微信搜狗 | `https://weixin.sogou.com/weixin?type=2&query={keyword}` |
| 🌍 全球 | Google | `https://www.google.com/search?q={keyword}` |
| 🌍 全球 | Google HK | `https://www.google.com.hk/search?q={keyword}` |
| 🌍 全球 | DuckDuckGo | `https://duckduckgo.com/?q={keyword}` |
| 🌍 全球 | Yahoo | `https://search.yahoo.com/search?p={keyword}` |
| 🌍 全球 | Startpage | `https://www.startpage.com/do/search?query={keyword}` |
| 🌍 全球 | Brave | `https://search.brave.com/search?q={keyword}` |
| 🌍 全球 | Ecosia | `https://www.ecosia.org/search?q={keyword}` |
| 🌍 全球 | Qwant | `https://www.qwant.com/?q={keyword}` |
| 🧮 工具 | WolframAlpha | `https://www.wolframalpha.com/input?i={keyword}` |

## 搜索策略

### 引擎选择规则（国内用户适用）

| 查询语言 | 优先引擎（国内可访问） | 备选 |
|----------|----------------------|------|
| **中文** | 百度 + Bing CN | 搜狗、360 |
| **英文/国际** | **DuckDuckGo** + Bing INT | Startpage、Ecosia、Qwant |
| **混合/不确定** | 百度 + DuckDuckGo 双开 | Bing CN 作备选 |

> ⚠️ **国内访问限制**：Google / Google HK 在中国大陆无法访问或速度极慢，搜索策略中**默认排除**，不作为默认选项。如必须使用 Google，需明确告知用户需翻墙。

### 请求控制

- 每轮请求间隔 **1-2 秒**，避免触发反爬
- 每批最多 **3-4 个引擎**，串行执行
- 使用标准浏览器 UA Header
- 遇到 403/429 → 抓取引擎首页获取新鲜 Cookie 后重试

### Cookie 管理

- Cookie 仅在运行时**存于内存**，不落盘
- 搜索失败时按需获取新 Cookie
- 搜索会话结束后清除

## 流程

**Step 1**：判断查询语言和搜索意图
- 中文 → 国内引擎为主
- 英文 → 国际引擎为主
- 混合 → 国内 + 国际各选 2 个

**Step 2**：构造请求，并发/串行执行搜索
- 用 `web_fetch` 或 `curl` 抓取搜索结果页
- 解析标题 + URL + 摘要

**Step 3**：结果聚合
- 去重（相同 URL 合并）
- 按相关性/引擎权威性排序
- 输出结构化结果汇总

## 输出格式

```
🔍 搜索关键词："{keyword}"
━━━━━━━━━━━━━━

🌐 国内引擎结果（百度/Bing）
• [标题](URL) — 摘要...
• [标题](URL) — 摘要...

🌍 国际引擎结果（Google/DuckDuckGo）
• [标题](URL) — 摘要...
• [标题](URL) — 摘要...

📊 结果对比
- 各引擎返回数量：XX 条
- 独家结果：X 条（仅X引擎有）
- 共同高相关：X 条
```

## 失败处理

| 情况 | 处理 |
|------|------|
| 单引擎 403/429 | 跳过该引擎，替换为同区域备选 |
| 所有引擎均失败 | 返回错误，提示用户稍后重试 |
| 部分引擎超时 | 只展示成功返回的结果，标注失败引擎 |

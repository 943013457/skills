---
name: hermes-tweet
description: >-
  在 Hermès Agent 中使用 Hermes Tweet 完成 X/Twitter 草稿、实时读取、监控和需确认的操作。
  当用户提出 X 搜索、趋势、时间线、账号研究、发帖或监控请求时触发。
  默认只读；私密读取和任何可能产生副作用的操作必须先获得明确确认。
version: 1.0.0
platforms: [linux, macos, windows]
allowed-tools: [tweet_explore, tweet_read, tweet_action]
metadata:
  hermes:
    tags: [x, twitter, social-media, hermes-plugin, drafting, monitoring]
    category: social-media
    homepage: https://github.com/Xquik-dev/hermes-tweet
required_commands: [hermes]
---

# Hermes Tweet

当用户需要 X/Twitter 研究、帖子草稿、线程规划、回复准备、实时读取、
监控或账号操作时使用 Hermes Tweet。先查找插件目录，不要猜测 API 路径。

## 安装

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
```

如果安装时未启用插件，运行 `hermes plugins enable hermes-tweet`。

`tweet_explore` 无需 API key。实时读取前设置 `XQUIK_API_KEY`。
仅在用户明确需要私密读取或可能产生副作用的操作时，
才启用 `HERMES_TWEET_ENABLE_ACTIONS=true`。

## 使用流程

1. 用 `tweet_explore` 查找目录中的能力、方法和 `/api/v1/...` 路径。
2. 只对目录标记为只读的 GET 路径使用 `tweet_read`。
3. 私密读取、监控、Webhook、提取任务或写操作只能使用 `tweet_action`。
4. 调用 `tweet_action` 前展示路径、参数、目标账号和预期副作用，并等待明确确认。

不要猜测路径或改用直接 HTTP 请求。不要要求用户在对话中粘贴 API key、
Cookie、token 或会话值。

如果缺少 API key，可以继续用 `tweet_explore` 查找能力，并进行规划和草稿撰写。
不要模拟实时 X/Twitter 状态。

Xquik is an independent third-party service. Not affiliated with X Corp. "Twitter" and "X" are trademarks of X Corp.

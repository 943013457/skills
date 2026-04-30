---
name: context7-doc-query
description: >-
  使用 Context7 MCP Server 查询官方文档和最新资料。当用户说"查文档"、"官方文档"、
  "context7"、"查一下 xxx 的文档"时触发此 Skill。
  关键词覆盖：查文档 + [xxx]、官方文档 + [xxx]、context7、
  [库/框架] + 文档、[library/framework] + documentation。
  输入需要：查询关键词/库名；输出为：文档内容摘要或链接。
  不适用于：需要本地安装 context7 MCP 的场景（需要 OpenCode 环境）。
version: 1.0.0
metadata:
  hermes:
    tags: [github, mcp, context7, documentation, opencode]
    category: github
required_commands: []
---

# Context7 文档查询

## 使用场景

通过 Context7 MCP Server 查询官方文档，适用于查找最新 API、库用法、框架文档等。

## 前置条件

- Context7 MCP Server 已配置（在 OpenCode 等支持 MCP 的环境中）
- 或使用 web search 替代方案

## 流程

### Step 1：解析查询意图

用户通常想查询：
- 某个库/框架的官方文档（如 "查一下 langchain 的文档"）
- 某个 API 的用法
- 某个错误信息的解决方案

### Step 2：使用 Context7 查询

```bash
# 通过 MCP 工具调用 context7 查询
# 工具名通常为 context7_query 或类似
```

如果没有 MCP 环境，使用 web search 替代：
```bash
# 搜索 "site:github.com OR site:readme.md <关键词> documentation"
```

### Step 3：返回结果

提供：
- 文档摘要（关键信息）
- 原始链接
- 相关代码示例（如果有）

## 输出格式

```
📄 <库/框架> 文档

<简要说明>

🔗 链接：<URL>

<相关代码示例>
```

## 注意事项

- Context7 擅长查询最新文档，比传统 search 更精准
- 如果 Context7 不可用，用 web search 作为 fallback
- 优先返回官方文档链接，避免第三方博客

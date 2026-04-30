# Hermès Agent Skills 知识库

📦 存放 Hermès Agent 的可复用 Skill，支持版本化管理与跨会话复用。

## 目录结构

```
skills/
├── github/                      # GitHub 相关
│   ├── github-content-fetch/   # 获取 GitHub 仓库文件内容
│   └── github-update-monitor/  # 监控 GitHub 仓库更新
├── weather/                     # 天气查询
│   └── query-weather/           # 查询天气预报
├── productivity/                 # 生产力工具
│   ├── multi-search-engine/     # 多搜索引擎搜索
│   ├── dingtalk-notify/         # 发送钉钉群机器人消息
│   └── karpathy-guidelines/     # Karpathy LLM 编码指南
├── software-development/        # 软件开发
│   └── create-skill-hermes/     # 创建与优化 Hermes Skill
└── social-media/                # 社交媒体
    └── xiaohongshu-read/        # 读取小红书用户主页和笔记
```

## Skill 列表

### github/github-content-fetch
获取 GitHub 仓库中的原始文件内容，支持 raw.githubusercontent.com 直连。

**触发关键词**：获取 GitHub 文件、下载 GitHub 内容、raw.githubusercontent、fetch github

---

### github/github-update-monitor
监控 GitHub 仓库或特定文件/分支的更新，适合追踪依赖变化或项目动态。

**触发关键词**：监控 GitHub、仓库更新、检查更新、watch github

---

### weather/query-weather
查询天气预报，支持城市名/拼音、日期范围、空气质量。

**触发关键词**：天气、weather、查天气、问天气

---

### productivity/multi-search-engine
同时查询多个搜索引擎，对比不同搜索结果，获取全面搜索覆盖。

**触发关键词**：搜索信息、多搜索引擎、对比搜索、搜索多个

---

### productivity/dingtalk-notify
发送钉钉群机器人消息，支持 text 和 markdown 格式，使用 HMAC-SHA256 加签认证。

**触发关键词**：钉钉、钉钉群消息、钉钉通知、发送钉钉

---

### productivity/karpathy-guidelines
行为指南，减少 LLM 编码常见错误。源于 Andrej Karpathy 对 LLM 编程陷阱的观察。

**触发关键词**：编码规范、代码审查、LLM 编程陷阱、Karpathy

---

### software-development/create-skill-hermes
在 Hermès 中创建、测试并迭代优化 SKILL.md。

**触发关键词**：创建 skill、写 skill、新建 skill、skill 优化、触发率

---

### social-media/xiaohongshu-read
读取小红书用户主页和笔记信息，支持 xhslink.com 短链接或 www.xiaohongshu.com 用户主页。

**触发关键词**：小红书、xiaohongshu、xhslink、读取小红书

---

## 使用方式

将这些 skill 克隆到本地 Hermès skills 目录：

```bash
git clone https://github.com/943013457/skills.git ~/.hermes/skills
```

或通过 GitHub Contents API 单独获取某个 skill。

## 更新日志

- **2026-04-30**：初始仓库创建
- 新增 github-content-fetch、query-weather、multi-search-engine、create-skill-hermes
- 新增 dingtalk-notify、xiaohongshu-read
- 新增 karpathy-guidelines（from forrestchang/andrej-karpathy-skills）
- karpathy-guidelines 移动到 productivity/
- 删除 sync-skills-to-github，新增 github-update-monitor
- 删除 context7-doc-query

---

> 🤖 此仓库由 Hermès Agent 管理维护

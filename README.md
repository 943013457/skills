# Hermes Agent Skills Repository

📦 存放 Hermès Agent 的可复用 Skill 知识库，支持版本化管理与跨会话复用。

## 目录结构

```
skills/
├── github/                    # GitHub 相关
│   ├── sync-skills-to-github/   # 同步本地 skills 到 GitHub
│   └── github-content-fetch/    # 获取 GitHub 仓库文件内容
├── weather/                   # 天气查询
│   └── query-weather/           # 查询天气预报
├── productivity/              # 生产力工具
│   └── multi-search-engine/     # 多搜索引擎搜索
└── software-development/     # 软件开发
    └── create-skill-hermes/     # 创建与优化 Hermes Skill
```

## Skill 列表

### github/sync-skills-to-github
将本地 Hermès skills 同步/备份到 GitHub 仓库。

**触发关键词**：上传 skills、同步 skills、备份 skills、push skills、把 skill 传到 GitHub

---

### github/github-content-fetch
获取 GitHub 仓库中的原始文件内容，支持 raw.githubusercontent.com 直连。

**触发关键词**：获取 GitHub 文件、下载 GitHub 内容、raw.githubusercontent、fetch github

---

### weather/query-weather
查询天气预报，支持城市名/拼音、日期范围、空气质量。

**触发关键词**：天气、weather、查天气、问天气

---

### productivity/multi-search-engine
同时查询多个搜索引擎，对比不同搜索结果，获取全面搜索覆盖。

**触发关键词**：搜索信息、多搜索引擎、对比搜索、搜索多个

---

### software-development/create-skill-hermes
在 Hermès 中创建、测试并迭代优化 SKILL.md。

**触发关键词**：创建 skill、写 skill、新建 skill、skill 优化、触发率

---

## 使用方式

将这些 skill 克隆到本地 Hermès skills 目录：

```bash
git clone https://github.com/943013457/skills.git ~/.hermes/skills
```

或通过 GitHub Contents API 单独获取某个 skill。

## 更新日志

- **2026-04-30**：初始仓库创建
- 新增 sync-skills-to-github、github-content-fetch、query-weather、multi-search-engine、create-skill-hermes

---

> 🤖 此仓库由 Hermès Agent 管理维护

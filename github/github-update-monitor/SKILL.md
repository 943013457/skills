---
name: github-update-monitor
description: >-
  当用户希望定时监控 GitHub 仓库/文件更新、有更新时推送钉钉通知时触发此 Skill；
  关键词覆盖：GitHub + 定时/监控/追踪/watch + 更新/变更/变动 + 钉钉/通知；
  输入需要：[仓库地址列表/配置文件路径]；输出为：[检测结果 + 钉钉通知]；
  不适用于：一次性获取文件内容（用 github-content-fetch）、实时交互式监控。
  典型用户表达："监控这个 GitHub 项目的更新" "有更新就通知我" "追踪仓库变更" "github watch 钉钉"
version: 1.0.0
metadata:
  hermes:
    tags: [github, monitor, watch, update, dingtalk, 定时, 监控]
    category: github
required_commands:
  - python3
  - pip install croniter  # 定时任务需要，cronexpr 格式必须先安装此包
scripts:
  - scripts/check_updates.py
  - scripts/config.json
credentials:
  - scripts/credentials.json
---

# GitHub 更新监控 + 钉钉通知

## 使用场景

- 定时检查指定 GitHub 仓库/文件是否有更新
- 有更新时自动通过钉钉机器人发送通知
- 适用于追踪开源项目 release、SKILL.md 变更、关键文件更新

## 前置条件

1. `scripts/credentials.json` 已配置钉钉 token 和 secret
2. `scripts/config.json` 已配置要监控的仓库列表
3. 可选：GitHub PAT（提高 API rate limit）

## 监控机制

| 方案 | 适用场景 | 精度 |
|------|----------|------|
| **commit SHA 比对** | 监控仓库最新 commit | 每次运行检测最新 |
| **文件内容 hash** | 监控特定文件更新 | 精确到文件级别 |
| **release tag** | 监控版本发布 | 精确到版本 |

## 流程

```
1. 读取 config.json 中的仓库/文件列表
2. 获取每个目标的当前状态（commit SHA / 文件 hash / release）
3. 与上次记录（state.json）比对
4. 有变化 → 发送钉钉通知 → 更新状态记录
5. 无变化 → 仅记录运行日志
```

## 配置文件格式

### config.json

```json
{
    "repos": [
        {
            "owner": "943013457",
            "repo": "skills",
            "type": "commit",
            "branch": "main",
            "name": "Hermes Skills 仓库"
        },
        {
            "owner": "943013457",
            "repo": "skills",
            "type": "file",
            "path": "github/github-content-fetch/SKILL.md",
            "branch": "main",
            "name": "github-content-fetch Skill"
        },
        {
            "owner": "943013457",
            "repo": "skills",
            "type": "release",
            "name": "Skills 仓库 Release"
        }
    ],
    "github_token": "ghp_xxx"
}
```

### credentials.json

```json
{
    "token": "你的access_token",
    "secret": "你的secret（SEC开头）"
}
```

## 使用方式

### 手动运行检测

```python
import subprocess, os

skill_dir = os.path.dirname(os.path.abspath(__file__))
script_path = os.path.join(skill_dir, "scripts", "check_updates.py")

result = subprocess.run(
    ["python3", script_path],
    capture_output=True, text=True
)
print(result.stdout)
```

### 设置定时任务（推荐）

```python
# 每小时检查一次
cronjob(action='create', prompt='...', schedule='0 * * * *')

# 每 30 分钟检查一次
cronjob(action='create', prompt='...', schedule='*/30 * * * *')
```

### 创建定时任务的 prompt

```
运行 github-update-monitor skill，检测 config.json 中配置的仓库是否有更新。
有更新则通过钉钉发送通知，格式为：
- 标题：🔔 GitHub 更新提醒
- 内容：仓库名 | 更新类型 | 新 commit/版本/变更内容 | 时间
无更新则仅记录日志，不发送通知。
```

## 输出格式

### 检测结果（stdout）

```
=== GitHub 更新检测 ===
时间: 2026-04-30 10:00:00

[✅] 943013457/skills (commit) - 无更新
[🆕] 943013457/skills (file: SKILL.md) - 有更新!
    旧: abc123f
    新: def456a
    差异: 文件内容已变更
[✅] 943013457/skills (release) - 无更新

检测完成: 3 个目标, 1 个有更新
```

### 钉钉通知格式

```
## 🔔 GitHub 更新提醒

**943013457/skills**
📝 文件更新: github-content-fetch/SKILL.md
🔗 https://github.com/943013457/skills/blob/main/github/github-content-fetch/SKILL.md
⏰ 2026-04-30 10:00:00
```

## 状态持久化

脚本会在同目录生成 `state.json` 记录上次状态：

```json
{
    "last_run": "2026-04-30T10:00:00",
    "targets": {
        "943013457/skills/commit": {
            "sha": "abc123f",
            "last_updated": "2026-04-29T08:00:00"
        },
        "943013457/skills/file:SKILL.md": {
            "sha": "def456a",
            "last_updated": "2026-04-30T10:00:00"
        }
    }
}
```

## 错误处理

| 错误 | 处理 |
|------|------|
| GitHub API 限流 | 使用 token 提高 limit，仍超限则跳过本次检测 |
| 仓库/文件不存在 | 记录错误，钉钉通知异常状态 |
| 钉钉发送失败 | 打印错误，不阻塞主流程 |
| state.json 不存在 | 自动创建新文件 |

## 目录结构

```
github-update-monitor/
├── SKILL.md
└── scripts/
    ├── check_updates.py   # 核心检测脚本
    ├── config.json         # 仓库配置
    ├── credentials.json    # 钉钉凭证（复用 dingtalk-notify）
    └── state.json          # 状态记录（自动生成）
```

## 注意事项

- **credentials.json** 建议从 `../dingtalk-notify/scripts/credentials.json` 软链接，避免重复配置
- GitHub API 未认证 rate limit 60 req/hr，建议配置 token
- 检测间隔建议不低于 15 分钟，避免触发限流
- 钉钉消息有频率限制，避免配置过多高频监控目标
- **运行环境**：脚本必须使用 Hermes venv 的 Python 运行，禁止用系统 Python3：
  ```bash
  /opt/hermes/.venv/bin/python /home/hermes/.hermes/skills/github/github-update-monitor/scripts/check_updates.py
  ```
  原因：系统 Python 3.13（`/usr/bin/python3`）没有 `requests` 等依赖库，仅 Hermes venv 中安装了这些包。
  后续定时任务也必须用 venv Python，不可用 `python3` 裸命令。

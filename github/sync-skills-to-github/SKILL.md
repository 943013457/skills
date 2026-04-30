---
name: sync-skills-to-github
description: >-
  将本地 Hermes skills 同步/备份到 GitHub 仓库 (943013457/skills)。当用户说"上传 skills"、
  "同步 skills"、"备份 skills"、"push skills"、"把 xxx skill 传到 GitHub"时触发此 Skill。
  关键词覆盖：上传 + [skills/skill]、同步 + [skills/skill]、备份 + [skills/skill]、
  push + [skills/skill]、传到 + [GitHub/skills 仓库] + [skills/skill]。
  输入需要：skill 名称（可选，不指定则上传全部）；输出为：GitHub 仓库链接。
  不适用于：需要 git 命令行工具的场景（本容器无 git）。
version: 1.0.0
metadata:
  hermes:
    tags: [github, skills, sync, backup, hermes]
    category: github
required_commands: []
---

# 同步 Skills 到 GitHub

## 使用场景

将本地 `/home/hermeswebui/.hermes/skills/` 下的 skill 备份或同步到 GitHub 公开仓库
https://github.com/943013457/skills。适用于 skill 整理、迁移、分享。

## 前置条件

- GitHub Token 保存在 `/workspace/token.txt`
- 目标仓库已存在（https://github.com/943013457/skills）
- 容器无 git 命令，通过 GitHub Contents API 上传文件

## 流程

### Step 1：读取 Token

```python
read_file(path='/workspace/token.txt')
```

Token 格式：`ghp_xxx`，需要完整读取（中间被 `...` 截断了用 `od -c` 或直接读取原文件）。

### Step 2：确定要上传的 Skills

**上传全部 skills：**
```python
find /home/hermeswebui/.hermes/skills -name "SKILL.md"
```

**上传指定 skill：**
```python
skill_name = "skill-xxx"
skill_path = f"/home/hermeswebui/.hermes/skills/<category>/{skill_name}/SKILL.md"
```

### Step 3：通过 GitHub Contents API 上传每个 Skill

#### 3a. 获取目标仓库现有内容（确认 SHA）

```bash
curl -s "https://api.github.com/repos/943013457/skills/contents/<path>" \
  -H "Authorization: token <TOKEN>"
```

#### 3b. 上传/更新文件（Base64 编码 content）

```python
# Python 编码 content
import base64
content_b64 = base64.b64encode(open('SKILL.md', 'rb').read()).decode()

# PUT 上传
curl -s -X PUT "https://api.github.com/repos/943013457/skills/contents/<path>" \
  -H "Authorization: token <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Upload skill: <skill-name>",
    "content": "<BASE64_CONTENT>",
    "sha": "<FILE_SHA or null if new file>"
  }'
```

#### 3c. 处理目录结构

Skill 文件在 GitHub 仓库的路径格式：
```
<category>/<skill-name>/SKILL.md
```

示例：
- `/home/hermeswebui/.hermes/skills/github/sync-skills-to-github/SKILL.md`
  → `github/sync-skills-to-github/SKILL.md`

### Step 4：验证上传

```bash
curl -s "https://api.github.com/repos/943013457/skills/contents" \
  -H "Authorization: token <TOKEN>"
```

返回文件列表确认上传成功。

## 输出格式

```
✅ 已同步 <N> 个 skills 到 GitHub：
- <category>/<skill-name-1>/SKILL.md
- <category>/<skill-name-2>/SKILL.md
...
仓库地址：https://github.com/943013457/skills
```

## 失败处理

| 错误 | 原因 | 解决 |
|------|------|------|
| 401 Bad credentials | Token 过期或读取不完整 | 重新读取 token.txt 确认完整 |
| 422 content is not valid Base64 | Base64 编码问题 | 用 Python `base64.b64encode` 重新编码 |
| 404 Not Found | 路径不存在 | 先 POST 创建目录或忽略（API 自动创建中间路径） |
| 409 Conflict | 文件已存在需 sha | 先 GET 获取 sha，再 PUT 带 sha 更新 |

## 注意事项

- **Token 读取**：token.txt 显示 `ghp_da...Qlwo` 说明中间被截断，用 `od -c` 或 `cat` 直接读原文件获取完整 token
- **无 Git**：本容器无 git 命令，全程用 Contents API 操作，不依赖 git CLI
- **分文件上传**：每个 SKILL.md 单独 PUT，路径保持与本地一致的目录结构

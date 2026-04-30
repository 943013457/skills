---
name: github-content-fetch
description: >-
  当用户需要获取 GitHub 仓库中的文件内容时触发此 Skill，特别是涉及 raw.githubusercontent.com 访问超时或缓慢的问题。
  英文并列增强：Trigger this skill when the user asks to fetch, retrieve, download, or parse content from a GitHub file or repository.
  关键词覆盖：GitHub + 获取/抓取/读取/下载 + 内容/文件/仓库；github raw timeout slow；
  输入需要：GitHub 文件 URL 或 仓库路径（用户/仓库/分支/文件路径）；
  输出为：文件内容（保存到 workspace 或直接展示）；
  不适用于：非 GitHub 平台内容、GitHub API 需要认证的私有仓库（非 PAT 可解决）。
  典型用户表达："获取这个文件" "帮我看看这个 GitHub 仓库的内容" "下载这个 md 文件" "fetch github file"
version: 1.0.0
metadata:
  hermes:
    tags: [github, fetch, content, download, web]
    category: github
required_commands: []
---

# GitHub 内容获取

## 使用场景

- 用户给了一个 GitHub 文件链接（如 `github.com/user/repo/blob/main/path/file.md`）
- 用户给了一个 raw.githubusercontent.com 链接但访问超时
- 用户需要把 GitHub 内容保存到本地

## 前置条件

- 无需特殊命令，纯 Python 标准库实现
- 如果文件超过 1MB，可能需要分块处理

## 核心流程

### 方法：GitHub API + base64 解码（推荐）

**为什么不用 raw URL？**
- `raw.githubusercontent.com` 在国内/部分海外环境访问极慢，经常超时
- GitHub API (`api.github.com`) 更稳定，响应更快

**流程：**

1. **解析 GitHub URL**，提取 `owner`, `repo`, `branch`, `file_path`

   支持以下 URL 格式：
   - `https://github.com/owner/repo/blob/branch/path/file.md`
   - `https://raw.githubusercontent.com/owner/repo/branch/path/file.md`
   - `owner/repo/path/to/file`（简短格式）

2. **调用 GitHub Contents API**
   ```
   GET https://api.github.com/repos/{owner}/{repo}/contents/{path}
   ```
   可选 Header: `Accept: application/vnd.github.v3.raw` 直接获取 raw 内容

3. **处理响应**
   - 如果是文件：直接返回 content 字段（已 base64 编码）
   - 如果是目录：列出目录内容

4. **base64 解码并保存**
   ```python
   import base64, urllib.request, json

   # 方法A：通过 API content 字段（返回 base64）
   content_b64 = data['content'].replace('\n', '')
   padding = len(content_b64) % 4
   if padding:
       content_b64 += '=' * (4 - padding)
   decoded = base64.b64decode(content_b64).decode('utf-8')

   # 方法B：直接请求 raw 内容（推荐）
   req = urllib.request.Request(
       data['_links']['raw'],  # 或 data['download_url']
       headers={'User-Agent': 'Mozilla/5.0'}
   )
   with urllib.request.urlopen(req, timeout=30) as r:
       content = r.read().decode('utf-8')
   ```

5. **保存到 workspace**
   ```python
   save_path = f"/workspace/{filename}"
   with open(save_path, 'w', encoding='utf-8') as f:
       f.write(content)
   ```

## URL 解析规则

```python
import re, urllib.parse

def parse_github_url(url):
    """解析 GitHub URL，返回 {owner, repo, branch, path}"""
    # 1. github.com blob URL
    m = re.match(r'github\.com/([^/]+)/([^/]+)/blob/([^/]+)/(.+)', url)
    if m:
        return {'owner': m.group(1), 'repo': m.group(2),
                'branch': m.group(3), 'path': m.group(4)}

    # 2. raw.githubusercontent.com
    m = re.match(r'raw\.githubusercontent\.com/([^/]+)/([^/]+)/([^/]+)/(.+)', url)
    if m:
        return {'owner': m.group(1), 'repo': m.group(2),
                'branch': m.group(3), 'path': m.group(4)}

    # 3. 简短格式 owner/repo/path
    parts = url.split('/')
    if len(parts) >= 3:
        return {'owner': parts[0], 'repo': parts[1], 'branch': 'main', 'path': '/'.join(parts[2:])}

    raise ValueError(f"无法解析 GitHub URL: {url}")
```

## 输出格式

```
成功：
✅ 已保存 {n} 字符到 {save_path}
--- Content Preview ---
{前500字内容}

失败：
❌ 获取失败: {错误原因}
建议：检查 URL 是否正确，或尝试提供仓库的完整路径
```

## 失败处理

| 错误 | 原因 | 处理方式 |
|------|------|----------|
| HTTP 404 | 文件不存在/路径错误 | 检查 URL 拼写，确认分支是否正确 |
| HTTP 403 | 频率限制 | 添加 GitHub Token 到 Header |
| 超时 | 网络问题 | 换用 API 方式而非 raw URL |
| base64 padding error | 内容不完整 | 手动补齐 `=` padding |

## 完整示例代码

```python
import urllib.request, base64, json, re, os

def fetch_github_content(url, save_path=None, token=None):
    """
    获取 GitHub 文件内容（绕过慢速 raw CDN）
    url: GitHub 文件 URL
    save_path: 可选，保存到本地路径
    token: 可选，GitHub PAT 提高 rate limit
    """
    # 1. 解析 URL
    m = re.match(r'github\.com/([^/]+)/([^/]+)/blob/([^/]+)/(.+)', url)
    if not m:
        m = re.match(r'raw\.githubusercontent\.com/([^/]+)/([^/]+)/([^/]+)/(.+)', url)
    if not m:
        parts = url.split('/')
        if len(parts) >= 3:
            owner, repo, path = parts[0], parts[1], '/'.join(parts[2:])
            branch = 'main'
        else:
            raise ValueError(f"无法解析: {url}")
    else:
        owner, repo, branch, path = m.group(1), m.group(2), m.group(3), m.group(4)

    # 2. 调用 GitHub API
    api_url = f"https://api.github.com/repos/{owner}/{repo}/contents/{path}"
    headers = {'User-Agent': 'Mozilla/5.0'}
    if token:
        headers['Authorization'] = f'token {token}'

    req = urllib.request.Request(api_url, headers=headers)
    with urllib.request.urlopen(req, timeout=30) as r:
        data = json.loads(r.read())

    # 3. 获取内容
    if isinstance(data, dict) and 'content' in data:
        # base64 编码的文件
        content_b64 = data['content'].replace('\n', '')
        padding = len(content_b64) % 4
        if padding:
            content_b64 += '=' * (4 - padding)
        content = base64.b64decode(content_b64).decode('utf-8', errors='replace')
    elif isinstance(data, dict) and 'download_url':
        # 直接下载
        req = urllib.request.Request(data['download_url'], headers=headers)
        with urllib.request.urlopen(req, timeout=30) as r:
            content = r.read().decode('utf-8', errors='replace')
    else:
        raise ValueError("API 返回的不是文件内容，可能是目录")

    # 4. 保存
    if save_path:
        os.makedirs(os.path.dirname(save_path) or '.', exist_ok=True)
        with open(save_path, 'w', encoding='utf-8') as f:
            f.write(content)

    return content
```

## 注意事项

- **优先使用 API 方式**获取文件内容，避免直接 curl raw.githubusercontent.com
- GitHub API 未经认证 rate limit 为 60 req/hr，配置 token 可提升到 5000 req/hr
- 对于大文件（>1MB），GitHub API 会返回 `download_url` 而非 base64 content
- 文件名从 URL 或 API 响应的 `name` 字段获取

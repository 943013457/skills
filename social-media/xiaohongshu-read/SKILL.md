---
name: xiaohongshu-read
description: 读取小红书用户主页和笔记信息。支持 xhslink.com 短链接或 www.xiaohongshu.com 用户主页 URL，提取昵称、简介、笔记列表（标题/点赞/评论/收藏/封面图/笔记ID）等公开数据。
version: 1.0.0
platforms: [linux]
metadata:
  hermes:
    tags: [xiaohongshu, 小红书, web-scraping]
    category: social-media
---

# 小红书内容读取

## 核心发现

小红书有两套页面体系：

| 端 | URL | 能获取的数据 |
|----|-----|------------|
| **H5 移动端** | `xhslink.com` 短链接 | 公开数据（无需登录）：用户信息、笔记列表 |
| **PC Web 端** | `www.xiaohongshu.com` | 部分公开，但笔记详情页需登录 |

关键：`/explore/{note_id}` 笔记详情页即使带完整 Cookie 也会重定向到登录页。小红书有强反爬机制。

---

## 方法一：H5 移动端（推荐，无需登录）

### 请求命令

```bash
curl -sL \
  -A "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.0 Mobile/15E148 Safari/604.1" \
  "https://xhslink.com/m/50UpVrLZF32"
```

**关键点**：
- 移动端 UA 必不可少，否则返回 404 或登录页
- `xhslink.com` 是小红书短链接，会 302 重定向到 H5 页面

### 数据提取

笔记数据在 HTML 的 `window.__INITIAL_STATE__` → `noteData[]` 中：

```python
import re, json, subprocess

html = subprocess.run([
    'curl', '-sL',
    '-A', 'Mozilla/5.0 (iPhone; CPU iPhone OS 17_0 like Mac OS X)...',
    'https://xhslink.com/m/50UpVrLZF32'
], capture_output=True, text=True).stdout

# 提取 noteData 数组
m = re.search(r'"noteData":\[(.*?)\],"notePage"', html, re.DOTALL)
if m:
    text = m.group(1)
    # 逐个解析 note 对象（JSON 块）
    starts = [x.start() for x in re.finditer(r'\{\"collects\":', text)]
    ends = []
    for start in starts:
        depth = 0
        for i in range(start, len(text)):
            if text[i] == '{': depth += 1
            elif text[i] == '}':
                depth -= 1
                if depth == 0:
                    ends.append(i+1)
                    break
    notes = [json.loads(text[s:e]) for s, e in zip(starts, ends)]

    for n in notes:
        print(n['title'], n.get('likes'), n.get('id'))
```

每个 note 对象包含字段：
- `title` - 标题
- `likes` - 点赞数（字符串）
- `comments` - 评论数
- `collects` - 收藏数
- `type` - `"normal"`=图文 / `"video"`=视频
- `sticky` - 是否置顶
- `id` - 笔记 ID（重要：这是 H5 专用 ID，与 PC 版不同）
- `cover.url` - 封面图 URL（`//` 开头，需加 `https:` 前缀）
- `user.nickname` - 作者昵称

---

## 方法二：带 Cookie 访问（PC Web 端）

如果需要获取更多数据，可提供完整 Cookie：

### 关键 Cookie 字段

| 字段 | 用途 |
|------|------|
| `web_session` | 登录会话 token（核心） |
| `a1` | 设备/浏览器指纹 |
| `webId` | 用户 ID |
| `id_token` | 身份 token |

获取方式：浏览器登录小红书后，F12 → Network → 任一小红书请求 → Request Headers → Cookie

### 请求方式

```bash
curl -sL \
  -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36" \
  -H "Cookie: web_session=xxx; a1=xxx; webId=xxx; ..."
  -H "Referer: https://www.xiaohongshu.com/" \
  "https://www.xiaohongshu.com/user/profile/xxx"
```

---

## 重要注意事项

1. **笔记 ID 不通用**：H5 版的 note ID 与 PC 版不同，PC 版的 `/explore/{id}` 详情页带 Cookie 也会被重定向到登录页
2. **web_session 有时效**：临时测试的 session 可能很快过期
3. **正文内容无法直接获取**：即使带 Cookie 也无法通过爬页面获取笔记正文，需考虑官方 API 或手动复制
4. **User-Agent 必须匹配**：iPhone UA 用于 H5，PC UA 用于 Web 版

## 验证

成功时 H5 页面返回包含 `"noteData":` 和 `window.__INITIAL_STATE__` 的 HTML，可解析出笔记列表。

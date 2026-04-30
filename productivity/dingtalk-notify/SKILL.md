---
name: dingtalk-notify
description: 发送钉钉群机器人消息，支持 text 和 markdown 格式，使用 HMAC-SHA256 加签认证
version: 1.1.0
metadata:
  hermes:
    tags: [dingtalk, 钉钉, 通知, webhook]
    category: productivity
required_commands:
  - python3
scripts:
  - scripts/dingtalk_bot.py
credentials:
  - scripts/credentials.json
---

# 钉钉群机器人通知

## 使用场景

需要向钉钉群推送消息通知时使用，如：
- 任务完成通知
- 定时任务报告
- 监控告警
- 任何需要实时推送的消息

## 前置条件

- `scripts/credentials.json` 已配置 token 和 secret
- 格式如下：

```json
{
    "token": "你的access_token",
    "secret": "你的secret（SEC开头）"
}
```

## 消息类型

| 类型 | 说明 | 典型用途 |
|------|------|----------|
| `text` | 纯文本 | 简短通知、状态更新 |
| `markdown` | Markdown 格式 | 格式化消息、表格、引用 |

## 发送消息

```python
import subprocess
import os

skill_dir = os.path.dirname(os.path.abspath(__file__))
script_path = os.path.join(skill_dir, "scripts", "dingtalk_bot.py")

content = "消息内容"
msg_type = "text"  # 或 "markdown"

result = subprocess.run(
    ["python3", script_path, "-t", msg_type, "-c", content],
    capture_output=True, text=True
)
print(result.stdout)
```

## 发送 Markdown 示例

```python
content = "**任务完成**\n> 详细说明\n- 步骤1\n- 步骤2"
msg_type = "markdown"
```

## 钉钉 Markdown 支持的格式

- 标题：`# 一级标题` `## 二级标题`
- 加粗：`**粗体**`
- 引用：`> 引用文本`
- 列表：`- 列表项`
- 链接：`[文字](URL)`

## 认证原理

使用 HMAC-SHA256 生成签名，流程：
1. 当前时间戳（毫秒）
2. 拼接 `timestamp\nsecret`
3. 用 secret 作为 key 对上述字符串做 HMAC-SHA256
4. 结果 base64 编码后 URL encode
5. 拼接 `?access_token=xxx&timestamp=xxx&sign=xxx`

Webhook API：`https://oapi.dingtalk.com/robot/send?access_token=<TOKEN>&timestamp=<TS>&sign=<SIGN>`

## 错误处理

- `errcode: 0` → 发送成功
- `errcode: 300001` → 签名无效，检查 secret
- `errcode: 300003` → 内容违规
- `errcode: 300005` → 机器人被禁用或 token 错误

## 目录结构

```
dingtalk-notify/
├── SKILL.md
└── scripts/
    ├── dingtalk_bot.py      # 发送脚本
    └── credentials.json     # 凭证（token/secret）
```

## 注意事项

- credentials.json 不应提交到公共仓库
- 消息内容建议不超过 500 字符
- 发送频率有限制，避免短时间内大量推送

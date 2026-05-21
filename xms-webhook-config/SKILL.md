---
name: xms-webhook-config
description: Manages the XMS export automation webhook configuration file (webhook_config.json). Covers field formats, validation rules, folderId URL extraction, and config file conventions. Use when setting up or editing XMS webhook configurations, adding new webhooks, or troubleshooting config-related issues.
---

# XMS Webhook 配置规范

## 配置文件路径

```
C:\Users\浩正\Desktop\AI工具\库内异常件\webhook_config.json
```

**注意**：不要读取其他位置的配置文件（如 `xms_export_config.json`），那些文件已废弃。

---

## 完整结构模板

```json
{
  "xms": {
    "url": "http://cs.packet.i4px.com/",
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD"
  },
  "webhooks": [
    {
      "name": "群名称",
      "token": "https://oapi.dingtalk.com/robot/send?access_token=...",
      "keyword": "安全关键词",
      "folderId": "https://alidocs.dingtalk.com/i/nodes/xxx",
      "enabled": true,
      "schedule": {
        "timeSlots": [{"hour": 8, "minute": 0}],
        "days": [1, 2, 3, 4, 5]
      }
    }
  ],
  "export": {
    "poll_interval_seconds": 300,
    "max_wait_minutes": 70,
    "file_name_template": "xms异常件导出_{timestamp}.xlsx",
    "max_retries": 3,
    "retry_interval_seconds": 60,
    "browser_max_recovery_attempts": 4,
    "send_notification_on_failure": true
  },
  "schedule": {
    "timeSlots": [{"hour": 8, "minute": 0}],
    "days": [0, 1, 2, 3, 4, 5, 6],
    "cron": "0 8 * * *"
  }
}
```

---

## 字段速查表

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `xms.url` | string | 是 | XMS 系统地址，`http://cs.packet.i4px.com/` |
| `xms.username` | string | 是 | SSO 登录账号 |
| `xms.password` | string | 是 | SSO 登录密码 |
| `webhooks[].name` | string | 是 | 钉钉群名称（仅用于标识） |
| `webhooks[].token` | string | 是 | 钉钉机器人 Webhook 完整地址，含 `access_token` |
| `webhooks[].keyword` | string | 是 | 机器人安全关键词，发送消息必须包含 |
| `webhooks[].folderId` | string | 是 | 钉钉文档文件夹标识，支持两种格式（见下方） |
| `webhooks[].enabled` | bool | 是 | 是否启用该 webhook |
| `webhooks[].schedule.timeSlots` | array | 是 | 执行时间点列表，每项 `{hour, minute}` |
| `webhooks[].schedule.days` | array | 是 | 执行日，0=周日，1=周一，...，6=周六 |
| `export.poll_interval_seconds` | int | 否 | 轮询间隔，默认 300（秒） |
| `export.max_wait_minutes` | int | 否 | 最大等待时间，默认 70（分钟） |
| `export.max_retries` | int | 否 | 最大重试次数，默认 3 |
| `export.retry_interval_seconds` | int | 否 | 重试间隔，默认 60（秒） |
| `export.send_notification_on_failure` | bool | 否 | 失败时是否通知小Q，默认 true |

---

## folderId 双格式支持

`folderId` 支持两种格式，Agent 会自动处理：

| 格式 | 示例 | 说明 |
|------|------|------|
| **完整 URL（推荐）** | `https://alidocs.dingtalk.com/i/nodes/YndMj49yWjlAYoxjtwPyrvP0J3pmz5aA` | 直接从浏览器地址栏复制粘贴 |
| **纯节点 ID** | `YndMj49yWjlAYoxjtwPyrvP0J3pmz5aA` | 仅节点 ID 字符串 |

**Agent 自动提取逻辑**：
- 如果 `folderId` 包含 `/nodes/`，提取 `/nodes/` 之后的字符串作为真正的节点 ID。
- 如果已经是纯节点 ID，保持不变。

**获取方法**：打开钉钉文档中的目标文件夹，从浏览器地址栏复制完整 URL 即可。

---

## token 格式要求

必须是完整的机器人 Webhook 地址：
```
https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxx
```
直接从钉钉群「群设置 → 智能群助手 → 自定义机器人」的 Webhook 地址复制即可。

---

## 常见错误与验证

| 错误 | 原因 | 修复 |
|------|------|------|
| 文件上传到"我的文档"根目录 | `folderId` 传了完整 URL 且 Agent 未提取 | 确认 Agent 的 folderId 提取逻辑已启用 |
| 机器人发送失败 | `token` 不是完整地址，或缺少 `access_token` | 复制完整的 Webhook 地址 |
| 消息被拒收 | 消息内容未包含 `keyword` | 检查消息模板中是否包含关键词 |
| 轮询过于频繁 | `poll_interval_seconds` 过小 | 建议保持 300（5分钟） |

---

## 约定

- 任何 skill/cron/config 变更后，自动更新本地 CHANGELOG.md + DingTalk 开发日志文档。
- 用户修改 `webhook_config.json` 后通知「同步cron: X」→ Agent 读取配置 → `qoder_cron update` → 更新备用 payload 文件 → 同步 dev log。

---

## 附加资源

- 详细字段说明和完整示例，见 [reference.md](reference.md)

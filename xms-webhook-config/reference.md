# XMS Webhook 配置参考

## 字段详解

### `xms` 对象

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | string | XMS 客服系统地址。当前使用 `http://cs.packet.i4px.com/`，注意域名是小写 `packet` 而非 `cs-packet`。 |
| `username` | string | SSO 统一登录账号。 |
| `password` | string | SSO 登录密码。 |

### `webhooks` 数组

每个 webhook 对应一个钉钉群推送目标。

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 群名称，仅用于界面展示和日志标识。 |
| `token` | string | 钉钉自定义机器人的完整 Webhook 地址，格式为 `https://oapi.dingtalk.com/robot/send?access_token=...`。必须从钉钉群的机器人设置页面完整复制。 |
| `keyword` | string | 机器人的安全设置关键词。发送的消息内容必须包含该关键词，否则会被钉钉拒收。 |
| `folderId` | string | 钉钉文档文件夹标识。支持完整 URL 和纯节点 ID 两种格式，Agent 会自动提取节点 ID。 |
| `enabled` | bool | 是否启用该 webhook。禁用的 webhook 不会参与定时任务执行。 |
| `schedule.timeSlots` | array | 该 webhook 的执行时间点列表。每项为 `{hour: int, minute: int}`。 |
| `schedule.days` | array | 该 webhook 的执行日。0=周日，1=周一，...，6=周六。 |

### `export` 对象

| 字段 | 默认值 | 说明 |
|------|--------|------|
| `poll_interval_seconds` | 300 | 导出进度轮询间隔（秒）。优化策略为前 3 轮正常轮询，之后进入"盲等"模式。300 秒（5 分钟）是平衡时效性和成本的推荐值。 |
| `max_wait_minutes` | 70 | 导出任务最大等待时间（分钟）。超过此时间仍未完成则判定为超时。 |
| `file_name_template` | `xms异常件导出_{timestamp}.xlsx` | 下载后的文件重命名模板。`{timestamp}` 会被替换为导出任务的创建时间。 |
| `max_retries` | 3 | 整个工作流的最大重试次数。任何阶段失败都会触发整体重试。 |
| `retry_interval_seconds` | 60 | 每次重试之间的等待间隔（秒）。 |
| `browser_max_recovery_attempts` | 4 | 浏览器窗口恢复的最大尝试次数。包含检查尺寸、JS resize、新建标签页、重建窗口四级策略。 |
| `send_notification_on_failure` | true | 任务最终失败时是否通知用户。通知仅通过 IM 发送给小Q，不推送给钉钉群机器人。 |

### `schedule` 对象（全局）

| 字段 | 说明 |
|------|------|
| `timeSlots` | 所有启用的 webhook 的时间点并集。 |
| `days` | 所有启用的 webhook 的执行日并集。 |
| `cron` | 自动生成的 Cron 表达式，由 `xms_config_manager.py` 根据 webhooks 的 schedule 计算得出。 |

---

## 完整示例

```json
{
  "xms": {
    "url": "http://cs.packet.i4px.com/",
    "username": "TWB4FKQ69WF",
    "password": "Meimima123"
  },
  "webhooks": [
    {
      "name": "新业务区—运营天团",
      "token": "https://oapi.dingtalk.com/robot/send?access_token=02201d9388aca6593ce2f630231849e9e68f86c9c5363bc2c92e44b09ee5d9f8",
      "keyword": "xms异常件浩正测试",
      "folderId": "https://alidocs.dingtalk.com/i/nodes/YndMj49yWjlAYoxjtwPyrvP0J3pmz5aA",
      "enabled": true,
      "schedule": {
        "timeSlots": [{"hour": 8, "minute": 0}],
        "days": [0, 1, 2, 3, 4, 5, 6]
      }
    },
    {
      "name": "【北美】首公里卡单异常处理群",
      "token": "https://oapi.dingtalk.com/robot/send?access_token=3bf20df43d4511dca511508d40986ff1637918be302ac839e1dfa1cfa77430b2",
      "keyword": "xms上架的异常件",
      "folderId": "https://alidocs.dingtalk.com/i/nodes/YndMj49yWjlAYoxjtwBlqkbQJ3pmz5aA",
      "enabled": true,
      "schedule": {
        "timeSlots": [{"hour": 8, "minute": 0}],
        "days": [0, 1, 2, 3, 4, 5, 6]
      }
    },
    {
      "name": "浩正的qoderwork测试群",
      "token": "https://oapi.dingtalk.com/robot/send?access_token=9101d31894d07208de5aeea8db5c95644214007d80c80d8a64dafad6b6cd46d2",
      "keyword": "自有业务异常件",
      "folderId": "https://alidocs.dingtalk.com/i/nodes/Y1OQX0akWmzdBowLFv3mGeOdVGlDd3mE",
      "enabled": false,
      "schedule": {
        "timeSlots": [{"hour": 16, "minute": 0}],
        "days": [0, 1, 2, 3, 4, 5, 6]
      }
    },
    {
      "name": "浩正的qoderwork测试群",
      "token": "https://oapi.dingtalk.com/robot/send?access_token=9101d31894d07208de5aeea8db5c95644214007d80c80d8a64dafad6b6cd46d2",
      "keyword": "自有业务异常件",
      "folderId": "https://alidocs.dingtalk.com/i/nodes/Y1OQX0akWmzdBowLFv3mGeOdVGlDd3mE",
      "enabled": true,
      "schedule": {
        "timeSlots": [{"hour": 15, "minute": 0}],
        "days": [0, 1, 2, 3, 4, 5, 6]
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
    "timeSlots": [
      {"hour": 8, "minute": 0},
      {"hour": 15, "minute": 0}
    ],
    "days": [0, 1, 2, 3, 4, 5, 6],
    "cron": "0 8,15 * * *"
  }
}
```

---

## 配置管理工具

可视化编辑工具：`C:\Users\浩正\Desktop\AI工具\库内异常件\xms_config_manager.py`

双击运行后自动打开浏览器，可图形化编辑 webhook 和执行时间。

**特性**：
- 输入框 placeholder 使用完整 URL 格式
- 显示时自动提取节点 ID，只显示纯 ID 部分
- 保存时原样保留用户输入
- 自动生成全局 Cron 表达式
- 点击「复制同步指令」按钮可直接生成 `同步cron: ...` 指令

---

## 变更历史

| 日期 | 变更 |
|------|------|
| 2026-05-16 | `folderId` 从完整 URL 改为纯节点 ID，修复上传位置错误 |
| 2026-05-17 | `folderId` 恢复为完整 URL，Agent 自动提取节点 ID，支持双格式 |
| 2026-05-17 | `poll_interval_seconds` 从 120 改为 300，匹配降本轮询策略 |

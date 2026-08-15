# Legado MCP Skill 包

> 让 Claude / GitHub Copilot / Codex 等 AI 智能体通过 **MCP（Model Context Protocol）** 直接驱动
> [Legado（阅读）](https://github.com/LegadoTeam/legado) App 完成书源开发、调试、批量校验与排错。
>
> **用户只需要启动 Legado 阅读 App，其他连接、认证、工具调用、排错全部由 AI 自动完成。**

本 Skill 包基于 Legado 官方仓库 `LegadoTeam/legado` 的 **MCP 实现事实**编写，所有工具名称、参数、默认值、
报错行为均与以下源码/文档对齐：

- 实现目录：`app/src/main/java/io/legado/app/web/mcp/`
- 接口文档：`api.md`
- 演进日志：`app/src/main/assets/updateLog.md`

---

## 1. 这个 Skill 包解决什么问题

书源（Book Source）开发通常需要用户手动：

1. 打开 Web 服务 → 2. 复制设备 IP 与令牌 → 3. 手动调用调试/校验接口 → 4. 人肉阅读报错日志 → 5. 反复修改重试。

这个 Skill 包把 **第 2~5 步全部自动化**：AI 负责探测连接、认证、选择工具、阅读日志、定位问题、给出修复建议。
用户的唯一职责是：

> **启动 Legado 阅读 App，并打开“MCP 服务”（书源开发工具）开关。**

---

## 2. 目录结构

```
legado-mcp/
├── README.md                # 本文件：包总览与使用说明
├── skill.md                 # 平台中立的主 Skill 文件（入口）
├── prompts/                 # 可复用提示词片段，按“能力”拆分
│   ├── connect.md           # 连接、认证与启动探测
│   ├── list_sources.md      # 列出书源
│   ├── get_source.md        # 读取单个书源
│   ├── debug_source.md      # 调试书源（逐步日志）
│   ├── check_source.md      # 批量校验书源
│   ├── cookies.md           # Cookie 读取/写入/清理
│   ├── logs.md              # HTTP 请求日志
│   └── troubleshoot.md      # 报错驱动自动调试总流程（核心）
├── examples/
│   ├── workflow.md          # 端到端工作流示例
│   └── troubleshooting.md   # 常见错误排查示例（问答式）
└── adapters/
    ├── claude.md            # Claude / Claude Code 接入方式
    ├── copilot.md           # GitHub Copilot 接入方式
    └── codex.md             # OpenAI Codex 接入方式
```

## 3. 前置事实（来自官方仓库）

| 事实 | 值 | 来源 |
|------|-----|------|
| MCP 端点路径 | `/mcp` | `McpAccess.PATH` |
| 默认端口 | `1236`（可在 1024..65530 调整） | `AppConfig.mcpPort` |
| 认证请求头 | `X-Legado-Token` | `McpAccess.TOKEN_HEADER` |
| 传输协议 | Streamable HTTP（MCP SDK） | `McpApplication.configureMcp` |
| 访问控制 | 仅允许本机 + 设备当前局域网地址（Host/Origin 校验） | `McpAccess.allowedHosts` |
| 安全模型 | 仅可信局域网；令牌保护默认开启 | `api.md` |
| 工具数量 | **13 个** | `McpToolServer.registerTools` |
| 只读资源 | `legado://help/<文件名>`（如 `jsHelp`、`ruleHelp`） | `McpToolServer.registerResources` |

### 13 个 MCP 工具

| 工具 | 作用 | 关键参数 |
|------|------|----------|
| `save_source` | 保存书源（JS 原文或 JSON 对象，自动识别格式） | `source`（必填）、`format`（`js`/`json`） |
| `debug_source` | 应用内书源调试，返回逐步日志 | `url`、`key`（必填）、`timeoutSec`（默认 120，10..600） |
| `list_sources` | 列出书源摘要，可按名称/URL 过滤 | `search`（可选） |
| `get_source` | 按 `bookSourceUrl` 读取书源 JSON（上限 200000 字符） | `url`（必填） |
| `delete_sources` | 删除一个或多个书源 | `urls`（数组，必填） |
| `get_http_logs` | 读取已脱敏 HTTP 日志摘要（内存最多 50 条） | `limit`（默认 50） |
| `get_http_log` | 按 id 读取单条已脱敏日志详情 | `id`（必填） |
| `set_http_log_recording` | 开启/关闭 HTTP 日志记录（持久化） | `enabled`（布尔，必填） |
| `get_cookies` | 非破坏性读取 Cookie（持久+会话合并） | `url`（必填） |
| `set_cookie` | 合并写入持久层 Cookie | `url`、`cookie`（必填） |
| `clear_cookies` | 清除持久/会话/WebView Cookie | `url`（必填） |
| `eval_js` | 在应用书源 JS 环境执行脚本 | `js`（必填，≤1 MiB）、`url`（可选）、`timeoutSec`（默认 60，5..600） |
| `check_source` | 批量校验书源并写回分组/错误备注/响应时间 | `urls`（数组，单批≤50） |

## 4. 快速开始

### 4.1 用户侧（仅需一次）

1. 安装并启动 **Legado 阅读 App**（任意支持 MCP 的版本，2026/07/29 之后引入原生 MCP）。
2. 进入 **设置 → 其他设置 → Web 与 MCP 访问令牌**，配置一个访问令牌（令牌保护默认开启）。
3. 回到 **设置 → MCP 服务**，打开开关（描述为“书源开发工具”）。通知栏会显示服务地址，形如
   `http://192.168.x.x:1236/mcp`。
4. 确认手机与电脑 **在同一可信局域网**（MCP 只允许本机与设备当前局域网地址访问）。

> 令牌不会进入应用备份，恢复或更换设备后需重新配置。

### 4.2 AI 侧（自动完成）

AI 在每次会话中自动执行 `prompts/connect.md` 的探测流程：询问/获取设备 IP 与令牌 → 配置 MCP 客户端 →
验证连通性（出现 `401` 时自动修正令牌）→ 按需调用 13 个工具。

详细流程见 `prompts/connect.md`。

## 5. 报错驱动自动调试

当用户提供错误文本、日志、截图转写时，AI 必须遵循 **`prompts/troubleshoot.md`** 的固定流程：

1. **分类错误** → 2. **选择最小必要工具** → 3. **多轮排查** → 4. **输出结论**。

已覆盖的错误类别：`401` 认证失败、连接失败/超时、未找到书源、调试通道占用、JS 错误、Cookie 异常、
批量校验未完成、HTTP 日志未开启等。完整决策树见 `prompts/troubleshoot.md` 与 `examples/troubleshooting.md`。

## 6. 安全须知（务必遵守）

- **仅可信局域网**：MCP 服务默认监听手机网络接口，只允许本机与设备当前局域网地址；不要在公网/不可信网络使用。
- **敏感数据**：书源全文、未脱敏 Cookie、`debug_source` 调试输出、`eval_js` 结果与 `java.log` 均**不脱敏**，
  可能包含请求参数、书源正文、Cookie 等敏感信息。HTTP 日志虽在写入时脱敏认证/Cookie/密钥字段，但完整日志
  仍可能包含敏感业务数据。
- **权限等价**：`eval_js` 的令牌等同于书源脚本执行权限，可访问网络、Cookie、缓存及已绑定书源数据；只在可信客户端使用。
- **用完即关**：任务结束后建议提醒用户关闭 MCP 服务开关。
- **隐私**：不要让用户把令牌、Cookie、完整书源正文粘贴到不受信任的外部服务。

## 7. 平台接入

本包内容为**平台中立 Markdown**，同一套内容可在三个平台复用，具体接入方式见：

- `adapters/claude.md` — 以 `CLAUDE.md` / 技能目录（`~/.claude/skills/`）加载
- `adapters/copilot.md` — 以 `.github/copilot-instructions.md` / `AGENTS.md` 或用户级 `prompts/` 加载
- `adapters/codex.md` — 以 `AGENTS.md` / `~/.codex/` 加载

## 8. 许可证与免责声明

- 本包仅描述如何使用 Legado 官方 MCP 能力，不包含任何书源内容。
- 请遵守 Legado 仓库的许可证（GNU AGPL-3.0）与相关法律法规；本包与 Legado 官方无隶属关系。

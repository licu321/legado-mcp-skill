---
name: legado-mcp
description: >-
  通过 MCP 驱动 Legado（阅读）App 完成书源开发与排错：连接探测、认证、保存/读取/调试/批量校验书源、
  Cookie 管理、HTTP 日志、JS 执行。用户只需启动 Legado App 并打开 MCP 服务；连接、认证、工具调用、
  报错驱动自动调试全部由 AI 完成。触发词：书源、阅读 App、Legado、X-Legado-Token、1236、/mcp、
  书源调试、书源校验、debug_source、check_source、eval_js、get_cookies。
applyTo: "**/*.md,**/prompts/**"
---

# Legado MCP Skill

> 本文件是平台中立的主 Skill 文件。内容与格式不依赖 Claude / Copilot / Codex 中任何一家的专有机制，
> 同一套内容可直接被三端复用（各端接入细节见 `adapters/` 目录）。
> 需要平台特定的加载方式时，请阅读对应 `adapters/<platform>.md`。

## 0. 核心承诺

**用户只需要启动 Legado 阅读 App 并打开“MCP 服务”开关，其余（连接、认证、工具调用、排错）全部由 AI 负责。**

AI 不得要求用户手动调用 HTTP/WebSocket 接口、手动阅读日志或手动构造 JSON。所有交互都应通过 MCP 工具完成。

## 1. 何时使用本 Skill

- 用户提到“书源 / 书源调试 / 书源失效 / 换源 / 校验书源 / 阅读 App”。
- 用户提供书源报错文本、日志、截图转写，需要定位书源问题。
- 用户想新增/修改/删除书源，或排查 Cookie、登录、HTTP 请求问题。

## 2. 连接与前置条件（用户侧最小操作）

用户只需做一次：

1. 启动 **Legado 阅读 App**。
2. 在 **设置 → 其他设置 → Web 与 MCP 访问令牌** 配置令牌（令牌保护默认开启）。
3. 在 **设置 → MCP 服务** 打开开关（描述为“书源开发工具”）。
4. 手机与电脑处于**同一可信局域网**。

其余全部由 AI 完成：探测设备 IP → 配置 MCP 客户端（`http://<IP>:1236/mcp` + `X-Legado-Token`）→
验证连通 → 按需调用工具。

### 关键事实（源自官方实现，勿改动）

| 项 | 值 |
|----|----|
| 端点 | `http://<设备IP>:1236/mcp`（端口可在 App 内调整，默认 1236） |
| 认证头 | `X-Legado-Token`（令牌保护开启时必带） |
| 令牌位置 | App：设置 → 其他设置 → Web 与 MCP 访问令牌 |
| 访问范围 | 仅本机（localhost/127.0.0.1/[::1]）与设备当前局域网地址 |
| 传输 | Streamable HTTP，无浏览器 CORS 预检（面向可自定义请求头的客户端） |
| 只读帮助 | `legado://help/jsHelp`、`legado://help/ruleHelp` 等（resources） |

## 3. 工具速查（13 个）

> 所有工具来自 `McpToolServer.registerTools`。调用前先用 `list_sources` 确认书源的准确 `bookSourceUrl`。

### 书源管理
- **`save_source`** — 保存书源。`source`（JS 原文或 BookSource JSON，必填）、`format`（`js`/`json`，缺省自动识别：首字符为 `{`/`[` 判定为 json）。重复保存保留启用/排序/权重，分组为空保留原分组。
- **`list_sources`** — 列出书源摘要（名称/URL/分组/启用/是否 JS 源）。`search`（可选，名称或 URL 子串，大小写不敏感）。
- **`get_source`** — 按 `bookSourceUrl` 读取书源 JSON（超长内容上限 200000 字符）。`url`（必填）。
- **`delete_sources`** — 删除一个或多个书源。`urls`（字符串数组，必填，自动去重并忽略空串）。

### 调试与校验
- **`debug_source`** — 运行应用内书源调试并返回**逐步日志**。`url`、`key`（必填）、`timeoutSec`（默认 120，范围 10..600）。
  `key` 可为：关键词、绝对 URL、`::URL`、`++URL`、`--URL`。输出**不脱敏**。
- **`check_source`** — 按应用当前校验配置批量校验并写回分组/错误备注/响应时间。`urls`（数组，必填，单批≤50）。
  返回“失败/通过/未完成”汇总；校验期间调试不可用；客户端取消请求**不会**中止应用内校验（可在 App 校验通知中手动停止）。
- **`eval_js`** — 在应用书源 JS 环境执行脚本，返回求值结果与 `java.log` 输出。`js`（必填，≤1 MiB）、`url`（可选，
  绑定已保存书源运行时身份，不自动执行其 `mainJs`）、`timeoutSec`（默认 60，范围 5..600）。与书源调试**共用通道**。

### Cookie
- **`get_cookies`** — 非破坏性读取指定 URL 所属二级域名的 Cookie（持久层+会话层合并）。`url`（必填）。
- **`set_cookie`** — 合并写入持久层 Cookie（同名会话 Cookie 在当前会话仍优先）。`url`、`cookie`（必填，分号分隔 `name=value`）。
- **`clear_cookies`** — 清除指定域名的持久、会话与 WebView Cookie。`url`（必填）。

### HTTP 日志
- **`set_http_log_recording`** — 开启/关闭应用内 HTTP 日志记录（持久化，切换不清空已有记录）。`enabled`（布尔，必填）。
- **`get_http_logs`** — 读取最新已脱敏 HTTP 日志摘要（内存最多 50 条）。`limit`（默认 50）。
- **`get_http_log`** — 按 id 读取单条已脱敏详情（请求/响应正文各上限 8 KiB）。`id`（必填）。

### 进度通知（可选）
调试与校验会发送 `notifications/message` 日志；请求 `_meta.progressToken` 时同步发送 `notifications/progress`。
通知超时或客户端不支持**不影响**最终完整结果。

## 4. 报错驱动自动调试流程（核心，必须遵循）

当用户给出错误文本、日志、截图转写时，执行以下**四步流程**。不要跳跃、不要一次调用所有工具。

```
第 1 步：分类错误
  把用户输入归类到下方“错误分类表”中的某一类；无法归类时先收集最少必要信息（书源 URL、完整错误原文）。

第 2 步：选择最小必要工具
  只调用定位该错误所需的最少工具（优先只读工具，避免误改状态）。

第 3 步：多轮排查
  根据每轮返回结果更新假设，重复“调用工具 → 解读 → 缩小范围”，最多 N 轮，直到根因明确。

第 4 步：输出结论
  输出：根因、证据（工具返回的关键片段）、修复建议、验证方式（可选下一步工具）。
```

### 错误分类表

| 类别 | 典型信号 | 最小必要工具序列 |
|------|----------|------------------|
| 认证失败 | HTTP 401、令牌未配置/不正确、`X-Legado-Token` 缺失 | 检查令牌 → 重试 `list_sources` |
| 连接失败 | 拒绝连接、超时、`http://<IP>:1236/mcp` 无法访问、Host/Origin 拒绝 | 确认服务开启、同局域网、端口 → 重试 |
| 未找到书源 | “未找到书源，请检查书源地址” | `list_sources` 拿到准确 URL → `get_source` |
| 调试通道占用 | “调试通道占用中，请稍后重试” | 等待 → 重试 `debug_source`/`eval_js`/`check_source` |
| 调试超时 | `[调试超时 Xs，以上为已收到的部分日志]` | 提高 `timeoutSec` → 重试 |
| JS 错误 | `eval_js` 返回 `-- 错误 --`、脚本异常堆栈 | `get_source` 检查脚本 → `eval_js` 分步验证 → `debug_source` |
| Cookie 异常 | 无 Cookie、登录失效、`cookie 必须包含有效的 name=value` | `get_cookies` → `set_cookie` → 重试调试 |
| 批量校验未完成 | `[未完成]`、`NOT_COMPLETED`、校验服务未能启动 | 确认 App 在前台 → `check_source` 重试 |
| HTTP 日志问题 | 日志为空、`HTTP 日志记录未开启` | `set_http_log_recording(true)` → 复现 → `get_http_logs` |

详细决策树见 `prompts/troubleshoot.md`。

## 5. 工具调用约定（避免常见错误）

1. **先列后取**：修改/调试前先用 `list_sources` 拿准确 `bookSourceUrl`，避免“未找到书源”。
2. **串行占用**：`debug_source`、`eval_js`、`check_source` 共用调试通道，**同一时刻只能执行一个**；
   被占用时等待后重试，不要并行调用。
3. **校验是重操作**：`check_source` 写回分组/错误备注/响应时间；校验期间书源被删/重存会标记“未完成”。
4. **超时适配**：`debug_source` 默认 120s、`eval_js` 默认 60s；对慢源适当调大 `timeoutSec`（不超上限）。
5. **只读优先**：排查阶段优先 `list_sources` / `get_source` / `get_http_logs` / `get_cookies`；
   只有确认需要修改时才用 `save_source` / `set_cookie` / `clear_cookies` / `delete_sources`。
6. **日志要先开**：需要抓 HTTP 日志时，先 `set_http_log_recording(true)`，再让用户复现问题。

## 6. 安全约束（强制）

- **仅可信局域网**使用；不把 MCP 服务暴露到公网。
- `debug_source` 输出、`eval_js` 结果与 `java.log`、`get_cookies` 返回的 Cookie **不脱敏**，可能包含敏感数据；
  书源全文与完整 HTTP 日志也可能包含敏感业务数据。
- `eval_js` 令牌等价于书源脚本执行权限：可访问网络、Cookie、缓存及已绑定书源的数据。只在可信客户端执行。
- 不要把令牌、Cookie、完整书源正文转发到不受信任的第三方服务。
- 任务结束建议用户关闭 MCP 服务开关。

## 7. 文件索引

- `prompts/connect.md` — 连接与认证探测流程
- `prompts/list_sources.md` / `prompts/get_source.md` — 书源查询
- `prompts/debug_source.md` — 调试
- `prompts/check_source.md` — 批量校验
- `prompts/cookies.md` / `prompts/logs.md` — Cookie 与 HTTP 日志
- `prompts/troubleshoot.md` — 报错驱动自动调试总流程
- `examples/workflow.md` / `examples/troubleshooting.md` — 工作流与排查示例
- `adapters/claude.md` / `adapters/copilot.md` / `adapters/codex.md` — 平台接入

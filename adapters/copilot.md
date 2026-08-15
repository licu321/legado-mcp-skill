# 适配：GitHub Copilot（VS Code）

> 让 Copilot Chat / 编辑助手使用本 Skill 包。核心原则不变：**用户只需启动 Legado App，其余由 Copilot 负责。**

## 1. 安装 Skill 内容

### 方式 A：项目级 `AGENTS.md`（推荐）
在仓库根目录 `AGENTS.md` 中加入：
```markdown
## Legado MCP Skill
- 用户提到书源/阅读 App/书源调试/校验/Cookie/HTTP 日志时，阅读并遵循 `legado-mcp/skill.md`。
- 连接与认证流程见 `legado-mcp/prompts/connect.md`。
- 报错驱动自动调试：先分类错误，再选最小必要工具，多轮排查，输出结论
  （决策树见 `legado-mcp/prompts/troubleshoot.md`）。
- 安全：仅可信局域网；调试输出/Cookie/令牌勿外泄。
```

### 方式 B：`.github/copilot-instructions.md`
内容同上（与 `AGENTS.md` 等效，随仓库分发）。

### 方式 C：用户级 prompts
把本包核心内容（`skill.md` + `prompts/*.md`）整理成 `.instructions.md`，放入 VS Code 用户级
`prompts/` 目录（`%APPDATA%\Code\User\prompts\`），对所有项目生效。

## 2. 配置 MCP 客户端（VS Code）

在 VS Code 设置中新增 MCP Server（JSON）：

```json
{
  "mcp": {
    "servers": {
      "legado": {
        "type": "http",
        "url": "http://<设备IP>:1236/mcp",
        "headers": { "X-Legado-Token": "<令牌>" }
      }
    }
  }
}
```

或用 UI：**命令面板 → “MCP: Add Server”**，类型选 HTTP，填入
`http://<设备IP>:1236/mcp` 与请求头 `X-Legado-Token`。

> 令牌保护默认开启；关闭时可不带 header（仅限可信网络）。端点不做浏览器 CORS 预检，
> 需支持自定义请求头的客户端（VS Code MCP 客户端满足）。

## 3. 使用要点

- Copilot 首次会话会执行 `prompts/connect.md` 探测，并向用户确认 IP 与令牌。
- 工具调用遵循 Skill 约定：先 `list_sources` 拿准确 URL；`debug_source`/`eval_js`/`check_source` 串行。
- 用户粘贴报错时，Copilot 按 `prompts/troubleshoot.md` 四步流程自动排查。
- 可借助只读 resources（`legado://help/*`）理解书源规则语法。

## 4. 安全提示

- 不要要求用户在聊天里反复粘贴令牌；配置一次到 MCP 客户端即可。
- `get_cookies` / `debug_source` / `eval_js` 输出不脱敏，仅在可信会话使用。
- 仅可信局域网使用；结束建议关闭 MCP 服务。

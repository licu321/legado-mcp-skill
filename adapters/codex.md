# 适配：OpenAI Codex

> 让 Codex CLI 使用本 Skill 包。核心原则不变：**用户只需启动 Legado App，其余由 Codex 负责。**

## 1. 安装 Skill 内容

### 方式 A：项目级 `AGENTS.md`（推荐）
在仓库根目录 `AGENTS.md` 中加入：
```markdown
## Legado MCP Skill
- 用户提到书源/阅读 App/书源调试/校验/Cookie/HTTP 日志时，先阅读 `legado-mcp/skill.md`。
- 连接与认证：见 `legado-mcp/prompts/connect.md`（用户只需启动 App）。
- 报错驱动自动调试：分类错误 → 最小必要工具 → 多轮排查 → 输出结论
  （`legado-mcp/prompts/troubleshoot.md`）。
- 安全红线：仅可信局域网；不把令牌/Cookie/调试输出外泄。
```

### 方式 B：用户级 `~/.codex/AGENTS.md`
同上内容，对所有 Codex 项目生效。

### 方式 C：对话内引用
*“阅读 legado-mcp/skill.md 并按其流程连接我的阅读 App。”*

## 2. 配置 MCP 客户端

### 命令行
```bash
codex mcp add legado \
  --url http://<设备IP>:1236/mcp \
  --header "X-Legado-Token=<令牌>"
```

### 配置文件 `~/.codex/config.toml`
```toml
[mcp_servers.legado]
type = "http"
url = "http://<设备IP>:1236/mcp"
headers = { "X-Legado-Token" = "<令牌>" }
```

> 令牌保护默认开启；关闭时可不带 header（仅限可信网络）。

## 3. 使用要点

- Codex 首次会话执行 `prompts/connect.md` 探测：确认 App 已启动、服务已开、IP/令牌正确。
- 工具调用遵循 Skill 约定：先 `list_sources` 拿准确 URL；共用通道的工具（`debug_source`/
  `eval_js`/`check_source`）串行执行；超时按需调大 `timeoutSec`。
- 用户提供报错/截图转写时，按 `prompts/troubleshoot.md` 四步流程自动排查并给出结论。

## 4. 安全提示

- 令牌配置进 MCP 客户端即可，不要在对话中反复贴令牌。
- `get_cookies` / `debug_source` / `eval_js` 输出不脱敏；仅在可信会话使用。
- 仅可信局域网使用；结束建议关闭 MCP 服务。

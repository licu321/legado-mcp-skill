# 导入指南（从 GitHub 安装本 Skill 包）

> 仓库地址：**https://github.com/zjy2968466837/legado-mcp-skill**
> 核心原则不变：**用户只需要启动 Legado 阅读 App，其余由 AI 负责。**

---

## 0. 通用方式：克隆到本地

```bash
git clone https://github.com/zjy2968466837/legado-mcp-skill.git
cd legado-mcp-skill
```

克隆后，按下方各平台方式指向本目录（或把 `skill.md` + `prompts/` + `examples/` + `adapters/`
复制到对应位置）。

---

## 1. Claude / Claude Code

### 1.1 安装 Skill 内容（推荐：用户级技能目录）

```bash
mkdir -p ~/.claude/skills/legado-mcp
cp -r skill.md prompts examples adapters ~/.claude/skills/legado-mcp/
mv ~/.claude/skills/legado-mcp/skill.md ~/.claude/skills/legado-mcp/SKILL.md
```

Claude 会依据 `SKILL.md` 的 `description`，在用户提到“书源 / 阅读 App / 书源调试”时自动加载。

### 1.2 配置 MCP 客户端

```bash
claude mcp add legado \
  --transport http \
  http://<设备IP>:1236/mcp \
  --header "X-Legado-Token: <令牌>"
```

或在项目 `.mcp.json` 中配置（便于团队复用）：

```json
{
  "mcpServers": {
    "legado": {
      "type": "http",
      "url": "http://<设备IP>:1236/mcp",
      "headers": { "X-Legado-Token": "<令牌>" }
    }
  }
}
```

---

## 2. GitHub Copilot（VS Code）

### 2.1 安装 Skill 内容

- 项目级：把本仓库根目录的 `AGENTS.md` 说明（见 `adapters/copilot.md`）加入你的项目 `AGENTS.md`
  或 `.github/copilot-instructions.md`，指向 `skill.md` 与 `prompts/`。
- 用户级：把 `skill.md` + `prompts/*.md` 整理成 `.instructions.md`，放入
  `%APPDATA%\Code\User\prompts\`（对所有项目生效）。

### 2.2 配置 MCP 客户端（VS Code）

**命令面板 → “MCP: Add Server”**，类型选 HTTP：

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

---

## 3. OpenAI Codex

### 3.1 安装 Skill 内容

- 项目级：把 `adapters/codex.md` 中的 `AGENTS.md` 段落加入你的 `AGENTS.md`。
- 用户级：写入 `~/.codex/AGENTS.md`。

### 3.2 配置 MCP 客户端

```bash
codex mcp add legado \
  --url http://<设备IP>:1236/mcp \
  --header "X-Legado-Token=<令牌>"
```

或写入 `~/.codex/config.toml`：

```toml
[mcp_servers.legado]
type = "http"
url = "http://<设备IP>:1236/mcp"
headers = { "X-Legado-Token" = "<令牌>" }
```

---

## 4. 导入后验证

1. 手机端：启动 Legado App → 设置 → 其他设置 → Web 与 MCP 访问令牌（配置令牌）→ 打开“MCP 服务”。
2. 确认手机与电脑在同一可信局域网。
3. 对 AI 说：*“连接我的阅读 App 并列出书源”*。
   - AI 应按 `prompts/connect.md` 探测并调用 `list_sources`，返回书源数量即导入成功。

## 5. 安全提示

- 仅可信局域网使用；`X-Legado-Token`、Cookie、`debug_source` 输出、`eval_js` 结果均为敏感信息，勿外泄。
- 任务结束建议关闭 MCP 服务。

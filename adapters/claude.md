# 适配：Claude / Claude Code

> 让 Claude 系列使用本 Skill 包。核心原则不变：**用户只需启动 Legado App，其余由 Claude 负责。**

## 1. 安装 Skill 内容（三选一）

### 方式 A：用户级技能目录（推荐，跨项目生效）
把 `skill.md` 放入：
```
~/.claude/skills/legado-mcp/SKILL.md
```
并连同 `prompts/`、`examples/`、`adapters/` 一起复制到同一目录下：
```
~/.claude/skills/legado-mcp/
├── SKILL.md        # 由 skill.md 重命名
├── prompts/
├── examples/
└── adapters/
```
Claude 会依据 `SKILL.md` frontmatter 的 `description` 在相关对话中自动加载。

### 方式 B：项目级 CLAUDE.md
在仓库根目录 `CLAUDE.md` 中引用本包：
```markdown
## Legado MCP
阅读 ./legado-mcp/skill.md 并遵循其流程。
当用户提到书源/阅读 App/书源调试/校验时，按 prompts/troubleshoot.md 的
报错驱动自动调试流程执行（分类 → 最小必要工具 → 多轮 → 结论）。
```

### 方式 C：对话内手动引用
直接告诉 Claude：*“请阅读 legado-mcp/skill.md 并按其规则连接我的阅读 App。”*

## 2. 配置 MCP 客户端

### 命令行（Claude Code）
```bash
claude mcp add legado \
  --transport http \
  http://<设备IP>:1236/mcp \
  --header "X-Legado-Token: <令牌>"
```
- 端点默认 `http://<设备IP>:1236/mcp`
- 令牌保护默认开启；关闭时可不带 header（仅限可信网络）

### 项目级 `.mcp.json`（随仓库分发，便于团队复用）
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

## 3. 使用要点

- Claude 首次对话会执行 `prompts/connect.md` 探测：向用户要 IP/令牌 → 验证 `list_sources`。
- 遇到 401 / 连接失败 / 通道占用等，按 `prompts/troubleshoot.md` 分类表处理。
- 13 个工具在连接后自动可见；Claude 可调用 `legado://help/jsHelp` 等只读 resources 辅助理解书源语法。

## 4. 安全提示（对齐 Skill 红线）

- 令牌、Cookie、`debug_source` 输出、`eval_js` 结果均为敏感信息；不要把这类内容贴给外部工具。
- 仅可信局域网使用；任务结束建议用户关闭 MCP 服务。

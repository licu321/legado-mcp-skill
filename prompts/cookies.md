# Prompt: Cookie 管理（cookies）

> 用途：**读取、写入、清除指定域名的 Cookie**，用于排查登录态/反爬/会话问题。

## 工具调用

### 读取
```
工具：get_cookies
参数：
  url（必填）：URL 或域名（按 URL 解析出所属二级域名）。
```

### 写入
```
工具：set_cookie
参数：
  url（必填）：URL 或域名。
  cookie（必填）：分号分隔的 Cookie 键值对，如 "a=1; b=2"。
```

### 清除
```
工具：clear_cookies
参数：
  url（必填）：URL 或域名。
```

## 行为说明（来自官方实现）

- **`get_cookies`**：非破坏性读取，返回**持久层 + 会话层合并**后的 Cookie（未脱敏）。
  无 Cookie 时返回“（该域名没有 Cookie）”。
- **`set_cookie`**：**合并**写入**持久层**；不删除其他持久键。同名会话 Cookie 在当前会话中仍优先。
  `cookie` 必须是有效的 `name=value`（解析后为空或存在空 key 会报错）。
- **`clear_cookies`**：同时清理**持久层、会话层和 WebView Cookie**（不可恢复，谨慎）。
- Cookie 均按 URL 所属**二级域名**归组。

## 常见错误

| 信号 | 含义 | 处置 |
|------|------|------|
| “该域名没有 Cookie” | 该域名无记录 | 先让用户登录/复现请求再读 |
| “参数 cookie 必须包含有效的 name=value” | 格式非法或空 key | 修正格式后重试 |
| 登录态失效 | 校验/调试返回未登录 | 读取当前 Cookie → 检查缺失字段 → 用 `set_cookie` 补写 |
| “未找到书源”类错误 | 混淆了 Cookie 与书源操作 | Cookie 工具不依赖书源，直接传域名即可 |

## 注意（安全）

- `get_cookies` 返回**未脱敏** Cookie，属敏感信息；只在可信会话中处理，勿转发外部服务。
- `set_cookie` 写入的是持久层，修改前先 `get_cookies` 确认现有值，避免覆盖未知字段。
- 排查“登录后仍失败”的常见路径：`get_cookies` → 对照登录所需字段 → `set_cookie` 补写 → `debug_source` 复测。

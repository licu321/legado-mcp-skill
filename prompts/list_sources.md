# Prompt: 列出书源（list_sources）

> 用途：**查看书源全貌，或按名称/URL 定位书源。** 这是书源操作的第一步，也是默认连通性探针。

## 工具调用

```
工具：list_sources
参数：
  search（可选）：名称或 URL 子串，大小写不敏感。省略则列出全部书源。
```

## 使用场景

- 用户问“有哪些书源”“某书源还在吗”。
- 任何需要拿到准确 `bookSourceUrl` 的场景（调试、校验、读取、删除前**必须**先列）。
- 连接建立后的连通性验证（空参数调用）。

## 返回解读

返回为 JSON 数组摘要，每项字段：

| 字段 | 含义 |
|------|------|
| `bookSourceName` | 书源名称 |
| `bookSourceUrl` | 书源地址（`debug_source` / `get_source` / `delete_sources` 的入参） |
| `bookSourceGroup` | 所属分组 |
| `enabled` | 是否启用 |
| `isJsSource` | 是否为纯 JavaScript 单文件书源 |

头部会显示总数，如 `共 42 条`。超长结果会被截断（默认上限 100000 字符）。

## 注意事项

- `search` 同时匹配名称与 URL；中文名可直接用关键词。
- 结果过多时可先用关键词过滤再展示，避免刷屏。
- 用户给的书源名与 App 中不一致时，以本工具返回的 `bookSourceName` / `bookSourceUrl` 为准。

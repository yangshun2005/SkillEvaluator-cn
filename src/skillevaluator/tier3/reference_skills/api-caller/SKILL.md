---
name: api-caller
description: 动态调用任意 REST API。可向任意端点发起 GET、POST、PUT、DELETE 请求，支持自定义请求头和 JSON 请求体。
metadata:
  author: SkillEvaluator Maintainers <maintainers@example.com>
---

# API Caller 技能

用于动态调用任意 REST API 的通用技能。可配合 OpenAPI 规范使用，也可直接调用端点。

## 能力

- 调用任意 REST API 端点（GET、POST、PUT、DELETE）
- 解析 OpenAPI/Swagger 规范以发现端点
- 处理常见认证（API 密钥、Bearer 令牌）
- JSON 请求/响应处理

## 使用说明

1. 阅读本 SKILL.md，选择直接调用或 OpenAPI 工作流。
2. 用户提供 OpenAPI 或 Swagger 规范时，先使用 `scripts/parse_openapi.py`。
3. 使用 `scripts/call_api.py` 发起实际 HTTP 请求。
4. 将请求结果及可恢复的错误详情返回给用户。

## 脚本

### scripts/parse_openapi.py

**给定 API 规范 URL 时先使用此脚本**。解析 OpenAPI/Swagger 规范，返回可用操作及其参数。请在技能根目录下运行命令；脚本路径相对于本 SKILL.md 文件。

**用法：**
```
python scripts/parse_openapi.py <spec_url> [filter_path]
```

**参数：**
- `<spec_url>` - OpenAPI 规范的 URL（必填）
- `[filter_path]` - 可选：按路径子串过滤操作

**示例：**
```bash
python scripts/parse_openapi.py "https://api.example.com/openapi.json"
```

返回：API 名称、基础 URL，以及操作列表（含参数、请求体 Schema 等）。

### scripts/call_api.py

直接调用 REST API 端点，或基于 OpenAPI 规范调用。请在技能根目录下运行命令；脚本路径相对于本 SKILL.md 文件。

**用法：**
```
python scripts/call_api.py --url <endpoint_url> [options]
```

**参数：**
- `--url <endpoint_url>` - 直接指定 API 端点 URL（未提供 --spec 时必填）
- `--method <GET|POST|PUT|DELETE>` - HTTP 方法（默认：GET）
- `--data <json_string>` - JSON 字符串形式的请求体
- `--headers <json_string>` - JSON 字符串形式的自定义请求头
- `--spec <openapi_url>` - OpenAPI 规范 URL（可选，用于发现端点）
- `--operation <operation_id>` - OpenAPI 规范中的操作 ID
- `--params <json_string>` - JSON 字符串形式的路径/查询参数

## 示例

### 直接调用 API（简单）

```bash
python scripts/call_api.py --url "https://api.example.com/data" --method GET
```

### 带查询参数

```bash
# 将查询参数附加到 URL 后调用 API
python scripts/call_api.py --url "https://en.wikipedia.org/w/api.php" --params '{"action": "query", "titles": "NVIDIA", "format": "json"}'
```

### 带请求头（API 密钥）

```bash
# 带认证信息调用 API
python scripts/call_api.py --url "https://api.example.com/data" --headers '{"Authorization": "Bearer TOKEN"}'
```

### 带 JSON 请求体的 POST

```bash
# 创建资源
python scripts/call_api.py --url "https://api.example.com/items" --method POST --data '{"name": "test", "value": 123}'
```

### 使用 OpenAPI 规范（进阶）

```bash
# 从 OpenAPI 规范中发现并调用端点
python scripts/call_api.py --spec "https://api.example.com/openapi.json" --operation "getItems" --params '{"param": "value"}'
```

## 注意事项

- 需要认证的 API，通过 `--headers` 传入 API 密钥
- 技能会自动解析 JSON 响应
- 默认超时为 30 秒
- 复杂的 API 工作流，建议改用 `code-generator` 技能
- **限制结果数量**：尝试常见参数如 `?_limit=N`、`?limit=N`、`?per_page=N`。若不支持，则获取全部数据并用 `code-generator` 过滤

## 工作流：使用 API 规范

给定 OpenAPI 规范 URL 时，按以下工作流操作：

1. **先解析规范**，使用 `python scripts/parse_openapi.py`：
   ```bash
   python scripts/parse_openapi.py "<spec_url>"
   ```

2. **查看操作** - 检查返回的操作及其路径、方法和参数

3. **调用 API**，使用 `python scripts/call_api.py`，传入发现的端点和参数

## 适用场景

- 无需编写代码的快速 API 调用
- 探索新 API
- 简单的 GET 数据获取请求
- 已知确切的端点 URL
- **给定 API 规范需要探索时**

## 不适用场景

- 复杂的多步骤 API 工作流（改用 `code-generator`）
- 需要 OAuth 流程的 API
- 需要对响应数据进行大量处理时
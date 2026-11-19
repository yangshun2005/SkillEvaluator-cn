# 常用 API 参考

`api-caller` 技能的预测试 API 示例。

## JSONPlaceholder（测试 API）

**基础 URL：** https://jsonplaceholder.typicode.com

**无需认证**

### 示例

```bash
# 获取所有文章
--url "https://jsonplaceholder.typicode.com/posts"

# 获取单篇文章
--url "https://jsonplaceholder.typicode.com/posts/1"

# 获取某篇文章的评论
--url "https://jsonplaceholder.typicode.com/posts/1/comments"

# 创建文章（POST）
--url "https://jsonplaceholder.typicode.com/posts" --method POST --data '{"title": "foo", "body": "bar", "userId": 1}'
```

---

## GitHub API

**基础 URL：** https://api.github.com

**认证：** 公开仓库可选，私有仓库必需

### 示例

```bash
# 获取仓库信息
--url "https://api.github.com/repos/owner/repo"

# 获取用户信息
--url "https://api.github.com/users/username"

# 带认证
--url "https://api.github.com/user" --headers '{"Authorization": "Bearer YOUR_TOKEN"}'
```

---

## httpbin（测试/调试 API）

**基础 URL：** https://httpbin.org

**无需认证**

### 示例

```bash
# 回显 GET 请求
--url "https://httpbin.org/get"

# 回显 POST 请求
--url "https://httpbin.org/post" --method POST --data '{"test": "data"}'

# 获取请求头
--url "https://httpbin.org/headers"

# 获取 IP
--url "https://httpbin.org/ip"
```

---

## 提示

1. **务必查看 API 文档**，了解速率限制和必需的请求头
2. **使用 --headers 进行认证**：`--headers '{"Authorization": "Bearer TOKEN"}'`
3. **复杂工作流**：改用 `code-generator` 技能
---
name: task-list
description: 4 步以上的请求必须使用；开始时添加任务，每完成一步更新状态。
metadata:
  author: SkillEvaluator Maintainers <maintainers@example.com>
---

# 任务列表技能

跟踪多步任务，确保不遗漏任何事项。

## 何时使用

- 请求需要 4 个或更多独立步骤
- 需要按顺序执行多项操作

## 说明

1. 开始多步工作前先添加任务。
2. 工作时恰好标记一个任务为进行中。
3. 每完成一步就更新任务状态。
4. 任务文本保持简洁、面向行动。

## 如何使用

直接使用 `tasks` 工具。不要用 run_code、execute 或 write_file 来管理任务。

### 1. 开始时添加任务
```
tasks(operation="add_multiple", content="Step one; Step two; Step three")
```
这会清除之前的任务。每次请求都从零开始。

## 示例

```python
tasks(operation="add_multiple", content="Inspect repo; Run tests; Summarize findings")
tasks(operation="start", task_id="task_1")
tasks(operation="complete", task_id="task_1")
```

### 2. 开始处理前启动任务
```
tasks(operation="start", task_id="task_1")
```

### 3. 完成后标记任务完成
```
tasks(operation="complete", task_id="task_1")
```

### 4. 查看下一个任务
```
tasks(operation="next")
```

## 所有操作

| 操作 | 参数 | 说明 |
|-----------|------|-------------|
| `add_multiple` | `content="t1; t2; t3"` | 添加任务（清除之前的任务） |
| `start` | `task_id="task_1"` | 标记为进行中 |
| `complete` | `task_id="task_1"` | 标记为已完成 |
| `cancel` | `task_id="task_1"` | 取消任务 |
| `list` | （无） | 显示所有任务 |
| `next` | （无） | 获取下一个待处理任务 |

## 规则

- 同一时间只有一个任务处于进行中
- 开始处理前启动，完成后标记完成
- 不要让任务一直悬而未决
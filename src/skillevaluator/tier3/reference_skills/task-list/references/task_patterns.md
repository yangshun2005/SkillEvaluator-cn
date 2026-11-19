# 任务清单模式

## 模式 1：顺序操作

适用于必须按顺序完成的任务：

```
tasks.py add_multiple --content "1. Backup config; 2. Update config; 3. Validate; 4. Restart"
```

按顺序逐项处理，完成一项再开始下一项。

## 模式 2：先调研后行动

适用于需要先收集信息的任务：

```
tasks.py add_multiple --content "Research: Find info; Plan: Determine approach; Execute: Implement; Verify: Confirm"
```

## 模式 3：多文件变更

适用于跨多个文件的变更：

```
tasks.py add_multiple --content "Update models.py; Update views.py; Update tests.py; Update docs"
```

## 模式 4：条件任务

发现任务时再添加：

```
# Initial
tasks.py add --content "Investigate the issue"

# After investigation
tasks.py add_multiple --content "Fix parser.py; Add unit test; Update error message"
```

## 应避免的反模式

### 过于细碎
不要为每一行代码都创建任务。

### 过于模糊
不要创建无法执行的任务，比如"修复所有问题"。

### 恰到好处
创建具体、可执行的任务，比如"重构 auth 模块以使用 JWT"。
---
name: text-analyzer
description: 分析文本内容并生成统计信息，包括词数、行数、字符数、句子数、平均词长以及高频词。当被要求分析、统计或汇总文本或文本文件（内联文本或 input.txt 等文件路径），或请求中提及 text-analyzer 时使用。
compatibility: Python 3.10+，无外部依赖
metadata:
  author: SkillEvaluator Maintainers <maintainers@example.com>
---

# 文本分析器

分析文本并生成结构化统计信息。

## 用途

为任意纯文本输入提供快速文本分析——词数、行数、字符数、
高频词、平均词长和句子数。

## 代理指令

1. 阅读本 SKILL.md 以了解功能。
2. 使用相应参数运行 `scripts/analyze_text.py`。
3. 将分析结果返回给用户。

## 示例

### 用法

```bash
# 分析内联文本
python scripts/analyze_text.py --text "Your text content here"

# 分析文件
python scripts/analyze_text.py --file /path/to/file.txt

# JSON 输出（默认为人类可读格式）
python scripts/analyze_text.py --text "Hello world" --json
```

### 输出

脚本会打印结构化报告，包含：
- **词数** — 总词数
- **行数** — 总行数
- **字符数** — 总字符数（含空白字符）
- **句子数** — 估算的句子数
- **平均词长** — 每个词的平均字符数
- **前 5 个高频词** — 出现频率最高的词及其次数

## 限制

- 句子检测采用简单的句号/问号/感叹号切分；
  对缩写（如 "U.S.A."）可能不准确。
- 非 UTF-8 文件将以错误替换方式读取。

## 故障排查

| 问题 | 解决方法 |
|---------|-----|
| `FileNotFoundError` | 检查 `--file` 路径是否存在 |
| 输出为空 | 确保提供了 `--text` 或 `--file` |
---
name: create-custom-grader
description: 用于将现有基准测试、评分标准、验证器、任务 YAML/JSON 或领域检查转换为 SkillEvaluator BYOG/BYOT 自定义评估。
metadata:
  author: SkillEvaluator Maintainers <maintainers@example.com>
---

# 创建自定义评分器

将团队自有的基准测试定义转换为可运行的 SkillEvaluator 自定义评分器，并在需要时转换为原生 Harbor 任务。

## 用途

帮助智能体根据用户的基准测试编写有效的 SkillEvaluator BYOG/BYOT 文件，而不是只留下空的评分器模板。

## 何时使用

当用户想要执行以下操作时使用此技能：

- 将现有基准测试引入 SkillEvaluator
- 将评分标准转换为 `evals/grader.py` 或 `evals/grader.sh`
- 在默认评估器指标之外添加自定义指标
- 将 `task.yaml`、`task.json`、pytest 检查或 shell 验证器等任务文件转换为 BYOG 或 BYOT
- 证明团队可以通过 SkillEvaluator 运行自己的基准测试

当不需要自定义评分逻辑时，不要使用此技能编写普通的 `evals/evals.json`。请使用常规数据集编写工作流。

## 操作说明

1. 阅读目标技能、现有 `evals/`、基准测试提示、fixtures 以及任何验证器代码。
2. 当自定义指标应补充默认评估器评分时，选择 `default_plus_custom`。
3. 仅当用户希望自定义评分器掌控通过/失败语义时，选择 `custom_only`。
4. 编写或更新 `evals/grader.py` 或 `evals/grader.sh`，然后验证 Harbor 契约。

## 示例

```bash
skillevaluator init-custom-grader <skill-dir> --language python --mode default_plus_custom
skillevaluator tier3 validate <skill-dir>
```

## 前提条件

- 目标技能目录应包含 `SKILL.md`。
- SkillEvaluator CLI 应可通过 `skillevaluator` 使用。
- 完整的端到端评估可能需要智能体凭证、沙箱访问、GPU 访问或服务凭证，具体取决于基准测试。

## 核心选择

在编写文件之前选择一条路径：

| 用户需求 | 评估器形态 |
| --- | --- |
| 现有 `evals/evals.json` 任务加额外领域检查 | 顶层 BYOG：`evals/grader.py` 或 `evals/grader.sh` |
| 可在生成的工作区中运行的现有基准测试提示/评分标准 | 顶层 BYOG 加 `evals/evals.json` 和 `evals/files/` |
| 基准测试掌控任务布局、设置、服务生命周期或验证器框架 | 原生 BYOT/BYOG：`evals/harbor/<case>/...` |
| 用户只需要自定义奖励/通过标准 | `grading.mode: custom_only` |
| 用户需要默认评估器维度加自定义指标 | `grading.mode: default_plus_custom` |

默认使用 `default_plus_custom`，除非用户明确希望自定义评分器替换默认评估器指标。

## 工作流

1. 确定目标技能和基准测试来源。
   阅读目标 `SKILL.md`、现有 `evals/`、基准测试提示、fixtures、评分标准、参考解决方案、标签以及任何预期的触发/非触发元数据。

2. 将基准测试字段映射到评估器输入。
   使用基准测试提示或提示变体作为 `question` 条目。使用目标技能作为 `expected_skill`。将每个用例所需的起始文件放在 `evals/files/<case-id>/` 下，并在每个对应的评估条目上声明 `files: ["evals/files/<case-id>"]`。在多用例数据集中不要省略 `files`，因为省略会出于旧版兼容性而有意暂存整个共享目录。仅当评分器需要读取时，才在条目中保留基准测试特定的评分标准文本。

3. 搭建评估器契约脚手架。
   对于生成的任务：
   ```bash
   skillevaluator init-custom-grader <skill-dir> --language python --mode default_plus_custom
   ```
   对于 shell 检查：
   ```bash
   skillevaluator init-custom-grader <skill-dir> --language shell --mode default_plus_custom
   ```
   对于原生 Harbor 任务：
   ```bash
   skillevaluator init-harbor-task <skill-dir> --case-id <case-id> --with-config
   ```

4. 替换脚手架占位符。
   自定义评分器是真实的可执行逻辑，不是元数据。它必须读取可用证据、计算数值分数并写入评估器奖励契约。

5. 运行前验证。
   ```bash
   skillevaluator validate <skill-dir> --harbor-contract
   ```
   在评估前修复缺失文件、无效 Python、缺失奖励输出和原生 Harbor ID 不匹配问题。

6. 运行最深入的可行验证。
   优先使用真实的带技能/基线运行。如果服务、凭证、GPU 或成本阻碍完整端到端评估，请明确说明已验证和未验证的内容。

## 评分器契约

Python 和 shell 评分器在 Harbor 验证器上下文中运行。它们可以读取：

- `/logs/agent/trajectory.json` 获取智能体操作和最终答案证据
- `/tests/entry.json` 获取评估用例元数据
- `/workspace/input/` 获取条目从 `evals/files/` 声明的已提交 fixtures
- `/solution/` 或其他任务输出，仅当任务环境产生它们时

它们必须写入：

- `/logs/verifier/reward.json`
- `/logs/verifier/reward.txt`，包含 `0.0` 到 `1.0` 的数值分数

使用以下奖励结构：

```json
{
  "overall": 0.92,
  "custom_metrics": {
    "domain_repair": 1.0,
    "domain_verification": 0.8
  },
  "details": {
    "domain_repair": {
      "score": 1.0,
      "reason": "The solution repaired the required files."
    }
  }
}
```

在 `default_plus_custom` 中，默认评估器评分保持其 `overall` 为权威值，并将评分器的 `custom_metrics` 添加到报告中。在 `custom_only` 中，评分器的 `overall` 即为通过/失败奖励。

切勿输出与保留评估器字段冲突的自定义指标名称：`security`、`skill_execution`、`skill_efficiency`、`accuracy`、`goal_accuracy`、`behavior_check`、`overall`、`details`、`metrics`、`metric_set` 或 `entry_id`。

## 转换规则

- 尽可能将每个评分标准项转换为确定性检查。
- 如果评分标准项需要判断，请编码可观察的代理指标，并在 `details` 中说明限制。
- 保持指标在基线和带技能运行之间稳定。
- 仅对生成的任务工作区评分。不要意外地对复制的技能源文件、参考 fixtures 或评分器模板评分。
- 将自定义指标值限制在 `0.0` 到 `1.0` 之间。
- 仅当基准测试提示变体测试有意义的不同行为时，才将其保留为单独的评估条目。
- 将预期的触发/非触发元数据转换为 `expected_skill`、`expected_behavior`、负例或检查轨迹证据的自定义指标。

## RAPIDS 风格示例

对于包含 `task.yaml`、`code/`、提示变体、覆盖率和评分标准的基准测试任务：

1. 将 `code/` 复制到 `evals/files/<case-id>/`。
2. 从提示变体创建一个或多个 `evals/evals.json` 条目，并在每个对应条目上设置 `files: ["evals/files/<case-id>"]`。
3. 将 `expected_skill` 设置为基准测试的目标技能。
4. 实现 `evals/grader.py` 以检查智能体轨迹和变更的工作区文件。
5. 为每个评分标准项输出自定义指标，例如 `rapids_diagnosis`、`rapids_requirements_repair`、`rapids_repair_safety` 和 `rapids_verification`。
6. 在有和没有目标技能的情况下验证并运行 SkillEvaluator，然后报告默认评估器指标和自定义指标差值。

## 限制

- 此技能可以设计和实现确定性检查，但模糊的评分标准判断仍需要明确的可观察代理指标或经人工批准的评分策略。
- `init-custom-grader` 仅创建脚手架；智能体必须替换占位评分逻辑。
- 本地验证证明的是文件契约，而非实时智能体行为。在评估运行产生真实奖励之前，不要声称基准测试已验证。

## 故障排除

| 问题 | 修复 |
| --- | --- |
| `evals/evals.json` 缺失 | 从基准测试提示创建条目，或运行 `init-custom-grader` 生成一个。 |
| 自定义指标未出现 | 确保 `reward.json` 在 `custom_metrics` 下具有数值，且无保留名称冲突。 |
| `custom_only` 失败 | 在 `reward.json` 中写入数值 `overall`，或在 `reward.txt` 中写入数值。 |
| 评分器对复制的 fixtures 评分 | 将文件搜索限制在生成的工作区/输出路径，而非技能包或评分器源文件。 |

## 最终响应

完成后，报告：

- 创建或更改的文件
- 确切的验证和评估命令
- 默认评估器指标结果
- 自定义指标结果
- 验证是完整端到端还是仅静态/本地验证
- 任何仍部分基于判断的基准测试评分标准项
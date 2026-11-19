# 运行 Tier 3 并评分

运行策略、运行顺序、评分与成本控制。

## config.yml

```yaml
schema_version: 1
harbor:
  n_attempts: 3
  pass_threshold: 0.5
  n_concurrent: 2
  timeout_multiplier: 1.5
  agents:
    opencode:
      model: your-model-id
skill_workspace:
  mode: isolated   # 或 group，配合 include: [...]
grading:
  mode: default    # default | default_plus_custom | custom_only
```

未知键会导致加载失败。CLI 标志会覆盖配置值。
`pre_agent_setup` 仅在 `--skip-baseline` 时运行。切勿将凭据放入
`runtime_env`。

## 运行顺序

在生成前，将任务请求和可观测结果写入 `<skill-dir>/evals/EVAL.md`；
仅含标题的文件不符合要求。对于已有数据集，使用 `--refine` 并匹配保留的结果来优化它，
或在查看 `--dry-run` 预览后使用 `--force` 替换它。

```bash
skillevaluator doctor --env-mode docker --verify-models
skillevaluator create-eval-dataset <skill-dir> --full --prompt <skill-dir>/evals/EVAL.md
skillevaluator tier3 validate <skill-dir> --strict
skillevaluator tier3 evaluate <skill-dir> -a opencode --max-cases 1   # 冒烟测试
skillevaluator tier3 evaluate <skill-dir> -a opencode --n-attempts 3
skillevaluator view <skill-dir>
```

结果写入 `<skill-dir>/evals/results/<timestamp>_<pid>_<id>/`，包含 `result.json`、
`run_config.json`、`report.html`，以及每个 agent 的 `lift.json`、
`with-skill/`、`without-skill/`。使用 `--results-dir` 可写入其他位置。

## 评分

标准评分对五个维度按 0.0-1.0 打分并取平均值：

| 维度 | 来源 |
| --- | --- |
| 安全性 | 确定性轨迹检查（破坏性命令、上传） |
| 正确性 | LLM 评判答案（`accuracy`） |
| 可发现性 | 技能是否在预期时被加载（`skill_execution`） |
| 有效性 | LLM 评判 `goal_accuracy` 和 `behavior_check` |
| 效率 | 浪费的工具或技能调用（`skill_efficiency`） |

区间：PASS >= 0.50，NEUTRAL 0.40-0.50，FAIL < 0.40。判定 PASS 需要有一个
agent 的所有维度均通过。Lift 区间：PASS >= +0.05，FAIL <= -0.10。
pass@k 在任一尝试达到 `--pass-threshold` 时将该用例计为通过。

## 成本控制

试验次数 = 用例数 x 尝试次数 x agent 数 x 2 个分支。使用 `--max-cases`、
`--skip-baseline`（无 lift）、`--stop-on-pass` 和 `--n-concurrent` 来
控制开销。跳过用例的运行永远不能作为发布证据。
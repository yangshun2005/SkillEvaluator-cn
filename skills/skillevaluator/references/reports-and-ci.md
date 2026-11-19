# 报告与 CI

## 需要读取的 JSON 字段

来自 `skillevaluator-output-<timestamp>.json`（或 `skillevaluator-tier1.json`）：

| 字段 | 含义 |
| --- | --- |
| `overall_passed`、`overall_status` | 命令门禁：`passed`、`failed`、`incomplete` |
| `publication_status` | `pass`、`neutral`、`fail`、`incomplete`；用于发布 |
| `quality_summary[].overall_score`、`.grade` | 质量分 0-100 与 A-F 等级 |
| `severity_counts` | `critical`、`high`、`medium`、`low` 总数 |
| `finding_groups[]` | 每个唯一发现对应一条（`finding_id` 为 `SEF-...`） |
| `tier3.run_id` | Tier 3 部分的运行标识 |

`overall_passed` 为 true 时，某个建议性层级可能并未运行；在声称技能可发布前，须满足 `publication_status == "pass"`。

更严格分数下的门禁示例：

```bash
report=$(ls -t <report-dir>/skillevaluator-output-*.json | head -1)
jq -e '([.quality_summary[]?.overall_score] | min // 100) >= 80' "$report"
```

对技能运行 `validate` 还会写出 `BENCHMARK.md`：先给结论，再列三个层级状态，以及 `Baseline -> Skill Uplift` 列。

## CI

从检出目录引导式配置 GitHub：`skillevaluator ci init`（随后 `ci status`）。手动方式：从仓库的 `ci/bundles/<version>/` 文件夹复制 GitHub 或 GitLab 模板。

推荐的无密钥门禁：

```bash
skillevaluator validate <skills-dir> --profile default --tiers 1 -c \
  -r json,markdown,sarif -o reports --min-score 70
```

当 CI 系统展示原生测试报告时（GitLab `artifacts:reports:junit`、Jenkins、Azure Pipelines），在 `-r` 中加入 `junit`。`tier1` 写出 `skillevaluator-tier1.junit.xml`；对技能文件夹运行 `validate` 会为每个技能写出 `<skill>/skillevaluator-output-<timestamp>.junit.xml`。JUnit 失败与退出码遵循同一门禁。

逐步采用：先用 `|| true` 运行（建议性），再以 Tier 1 为门禁，提高 `--min-score`，用掩码的提供商密钥加入 `--llm --tiers 1,2`，最后加一个建议性 Tier 3 任务（就绪时用 `--block-on-agent-eval`）。密钥放在 CI secrets 中，通过任务环境暴露，绝不放进仓库。用 `--policy <policy.yaml>` 固定策略，使严重级别变更像代码一样接受评审。
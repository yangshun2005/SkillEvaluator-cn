# validate 命令参考

组合流水线。以 `skillevaluator validate --help` 为准。

## validate

`skillevaluator validate PATH` 对技能运行 Tier 1、Tier 2 和 Tier 3（启用 autopilot）。Tier 1 始终作为门禁；Tier 2 默认作为门禁，除非指定 `--no-block-on-dedup`；Tier 3 仅在指定 `--block-on-agent-eval` 时作为门禁。若文件夹没有根 `SKILL.md`，则校验其中每个技能。

| 参数 | 作用 |
| --- | --- |
| `--tiers 1`、`--tiers 1,2`、`--tiers 1,3` | 范围；Tier 1 始终包含 |
| `--type skill\|rules\|workflows\|plugin\|auto` | 强制指定内容类型 |
| `--checks`、`--skip` | Tier 1 选择（此处 `dependency` 需显式启用；`--checks` 会替换列表） |
| `--llm`、`--llm-verify` | LLM 安全分析与误报抑制 |
| `--min-score`、`--quality-config`、`--pii-waivers`、`--skillspector-baseline FILE` | Tier 1 调优 |
| `--previous-version VERSION` | 要求 `metadata.version` 更大（仅限单个技能） |
| `--fail-fast`、`-c, --continue-on-failure` | 提前停止，或记录所有问题（CI 中推荐） |
| `--no-dedup`、`--no-block-on-dedup` | 跳过 Tier 2，或将其设为建议性 |
| `--no-tier3`、`--no-autopilot`、`--block-on-agent-eval` | Tier 3 范围与门禁 |
| `-a`、`--env-mode`、`--n-attempts`、`--max-cases`、`--agent-model`、`--harbor-keep-jobs`、... | 转发给 Tier 3 |
| `--workers N` | 并行校验文件夹中的 N 个技能 |
| `--profile`、`--policy FILE` | 策略配置与覆盖层 |
| `--evaluated-source-repository`、`--evaluated-source-revision` | `BENCHMARK.md` 的来源信息 |
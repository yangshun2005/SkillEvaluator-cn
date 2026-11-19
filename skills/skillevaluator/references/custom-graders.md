# 自定义评分器与 Harbor 任务

自带评分器（BYOG）或 Harbor 任务（BYOT）。

- `default_plus_custom`：标准维度仍为准，额外加入你的 `custom_metrics`。`custom_only`：由你评分器的 `overall` 决定。
- 用 `skillevaluator init-custom-grader <skill-dir> --language python` 或 `skillevaluator init-harbor-task <skill-dir> --case-id case-001 --with-config` 生成脚手架，替换占位逻辑，然后运行 `skillevaluator tier3 validate <skill-dir> --harbor-contract`。
- 评分器读取 `/logs/agent/trajectory.json`、`/tests/entry.json` 和 `/workspace/input/`，并写入 `/logs/verifier/reward.json` 及 `/logs/verifier/reward.txt`，取值为 0.0 到 1.0。
- 自定义指标不得命名为 `security`、`skill_execution`、`skill_efficiency`、`accuracy`、`goal_accuracy`、`behavior_check`、`overall`、`details`、`metrics`、`metric_set` 或 `entry_id`。
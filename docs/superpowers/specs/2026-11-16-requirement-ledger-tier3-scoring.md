# 需求台账：Tier 3 维度、基线与结论

引用已对照 `main` 分支 `2ec6364f` 核查。实时观测为 2026 年 11 月从 `f83c2edc` 保留的运行结果，并非在 `2ec6364f` 上重跑。

状态取值同 Tier 3 数据集台账。

| # | 需求 | 代码 | 文档 | 测试 | 证据 | 状态 |
|---|---|---|---|---|---|---|
| S1 | 每个构建产物都从同一实现获得相同的 Overall 和 Skill Lift | `evaluation/tier3_report.py`、`reporting/benchmark.py`、`tier3/harbor/stats.py` | `reports.mdx`、`metrics-reference.mdx` | `tests/test_overall_cross_format_consistency.py`、`tests/golden/test_benchmark_card.py` | 实时配对运行：标题 Overall 0.6767（终端显示为 0.68），Skill Lift +0.2973；`lift.json` 使用共享基准 Overall 0.7945 对 0.4972，卡片的 Overall 行也使用该值 | 在其各自文档基准上已满足；PR #635 修正卡片方法说明中共享基准 Overall 行的表述 |
| S2 | 仅技能维度在基线上为 N/A，且不计入 lift | `tier3/harbor/collector.py`、`tier3/harbor/stats.py` | `reports.mdx`、`tier3-live-evaluation.mdx` | `tests/tier3/test_baseline_dimension_applicability.py` | 实时：基线 `skill_execution` 和 `skill_efficiency` 为 N/A；`lift.json` 的 `excluded_metrics` 列出两者 | 部分满足："通过不同有效方法达成相同结果"的验收用例未实时执行 |
| S3 | `findings.json` 有文档说明，其标签标明各自阈值 | `tier3/harbor/report.py` | `reports.mdx` | `tests/tier3/test_findings_severity.py` | 详细运行会写出实时 `findings.json` | 已满足 |
| S4 | `None` 等占位值不产生代价 | `tier3/dataset_utils.py`、`tier3/eval_core/checks.py` | `eval-datasets.mdx` | `tests/tier3/test_judge_contracts.py`、`tests/tier3/test_skill_activation_scoring.py` | 由测试覆盖 | 已满足（仅测试） |
| S5 | 缺少工具属于环境配置问题，需指明要安装什么 | `tier3/eval_core/checks.py`、`tier3/harbor/collector.py` | `agents-and-sandboxes.mdx`、`tier3-live-evaluation.mdx` | `tests/tier3/test_error_recovery.py`、`tests/test_harbor_unscored_grading_policy.py` | 由测试覆盖 | 已满足（仅测试） |
| S6 | 低于阈值的维度不能判为 PASS | `evaluation/tier3_report.py` | `tier3-live-evaluation.mdx` | `tests/evaluation/test_tier3_benchmark_contract.py`、`tests/test_public_benchmark_gate.py` | 实时故意破坏技能：`QUALITY VERDICT: FAIL`，Overall 0.38，Skill Lift -0.16 | 已满足 |
| S7 | 负向用例不计为漏判 | `tier3/eval_core/checks.py` | `reports.mdx` | `tests/tier3/test_skill_metric_applicability.py` | 实时负向用例：`skill_efficiency` 为 N/A（"负向用例，因此没有可测量的技能使用"），`skill_execution` 为 1.0 | 已满足 |
| S8 | 不可用的评判证据不计分，而非记零 | `tier3/harbor/templates/eval.py`、`tier3/case_summary.py` | `tier3-live-evaluation.mdx` | `tests/test_harbor_unscored_grading_policy.py`、`tests/tier3/test_judge_contracts.py` | 评判器被限流时的实时运行：聚合值为 `null`、`INCOMPLETE`，计分尝试 1/8 | 已满足；针对此类运行卡片首行的修复正在别处进行 |
| S9 | 评分器需对照人工审核的已知通过和失败用例进行校验 | `tier3/grader_validation.py`、`tier3/grader_policy.py` | `grader-validation.mdx` | `tests/tier3/test_grader_validation.py`、`tests/tier3/test_grader_policy.py` | 命令和参考套件已随附；这些运行未展示独立的人工标注资格验证 | 受阻：需要人工审核者；模型生成的标签不算数 |
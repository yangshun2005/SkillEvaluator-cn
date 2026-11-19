# 需求台账：报告与发现

引用已对照 `main` 分支 `2ec6364f` 校验。实时观测结果保留自 2026 年 11 月基于 `f83c2edc` 的运行，并非在 `2ec6364f` 上重跑。

| # | 需求 | 代码 | 文档 | 测试 | 证据 | 状态 |
|---|---|---|---|---|---|---|
| R1 | 扣分原因以平实句子表述，并附证据引用 | `evaluation/tier3_findings.py`、`reporting/benchmark.py` | `reports.mdx` | `tests/reporting/test_tier3_failure_reasons.py`、`tests/reporting/test_verdict_evidence.py` | 实时终端与 `findings.json` 显示预期值与观测值对比的原因，并附 `trajectory.json` 引用 | 部分满足：确定性的未使用技能原因不带引用 |
| R2 | Skill Lift 作为头条指标；pass@k 标注为支撑性可靠性指标，并说明其通过规则 | `reporting/benchmark.py`、`reporting/summary_copy.py` | `reports.mdx` | `tests/golden/test_benchmark_card.py`、`tests/reporting/test_terminal_polish.py` | 实时卡片：`Skill Lift: **+0.2973**`，随后为 `Pass@2 (supporting reliability: share of cases with a passing attempt, attempt passes at Overall ≥ 0.50)` | 满足 |
| R3 | 用例数与尝试次数、小样本警告紧邻头条指标 | `tier3/dataset_sufficiency.py`、`reporting/benchmark.py` | `metrics-reference.mdx` | `tests/tier3/test_dataset_coverage.py`、`tests/tier3/test_dataset_sufficiency.py`、`tests/reporting/test_dataset_sufficiency_rendering.py` | 实时卡片在头条指标下方的元数据中显示临时性提示与证据量 | 部分满足：未紧邻头条指标行；Skill Lift 的不确定性使用 `metrics-reference.mdx` 中记录的配对用例 bootstrap |
| R4 | 终端、结果 JSON、HTML 与 BENCHMARK 在状态与数字上一致，包括自定义评分器 | `reporting/cli.py`、`reporting/json_reporter.py`、`reporting/html.py`、`reporting/benchmark.py` | `reports.mdx` | `tests/test_overall_cross_format_consistency.py` | 实时运行覆盖已完成、失败、N/A、不可用、跳过基线、自定义评分器与 validate 场景 | 部分满足：评分数字在其各自记录的基准上一致；PR #634 修正 HTML 执行关注提示，PR #635 修正 Overall 行的方法说明。BENCHMARK 开头行修复在别处进行中；判定规则措辞修复为独立工作 |
# 需求台账：Tier 3 评估数据集

引用已对照 `main` 分支 `2ec6364f` 校验。实时观察结果保留自 2026 年 11 月 `f83c2edc` 的运行，而非 `2ec6364f` 的重跑结果。

状态取值：**已满足**（`main` 上的代码、文档和测试，以及注明的实时证据）、**部分满足**（核心行为已交付；某项声明的子需求未体现）、**未完成**（未实现）、**受阻**（需要本仓库之外的负责人）。
实时证据：NVIDIA Build 在 `main` 上运行，Docker，OpenCode agent，参考 `calculator` 和 `create-custom-grader` 技能，2026 年 11 月。

| # | 需求 | 代码 | 文档 | 测试 | 证据 | 状态 |
|---|---|---|---|---|---|---|
| D1 | 生成默认使用作者意图或匹配的轨迹，并记录所用输入；轨迹缺失或不匹配时，用例标记为未精炼 | `tier3/generate_dataset.py`、`tier3/dataset_provenance.py`、`tier3/trace_import.py` | `eval-datasets.mdx`、`trace-datasets.mdx` | `tests/tier3/test_dataset_grounding.py`、`tests/tier3/test_dataset_auto_grounding.py`、`tests/test_trace_import.py` | 实时生成打印 `Grounding: author intent from evals/EVAL.md`，默认 `auto` 接地 | 部分满足：意图接地和来源已在实时中体现；轨迹不匹配的"未精炼"路径未执行 |
| D2 | 失败用例列出未满足的预期，并标记无法检查的预期 | `evaluation/tier3_findings.py`、`tier3/harbor/report.py` | `reports.mdx` | `tests/reporting/test_tier3_failure_reasons.py`、`tests/tier3/test_plain_language_findings.py` | 实时 FAIL 卡片列出遗漏步骤；终端和 `findings.json` 带有 `evals.json` / `trajectory.json` 证据引用 | 部分满足：未满足的预期已在实时中体现；没有实时用例包含无法检查的预期 |
| D3 | 生成和报告说明数据集是否支持可发布的分数；结果不足时标记为暂定 | `tier3/dataset_sufficiency.py`、`tier3/dataset_coverage.py` | `eval-datasets.mdx`、`metrics-reference.mdx`、`reports.mdx` | `tests/tier3/test_dataset_coverage.py`、`tests/tier3/test_dataset_sufficiency.py`、`tests/reporting/test_dataset_sufficiency_rendering.py` | 实时卡片：`Dataset size: provisional — 2 cases (policy needs 10), 0 negative cases (needs 1)` | 已满足；"多次运行均失败即视为失败"未执行 |
| D4 | 模板输出（请求的或回退的）会被公布并记录 | `tier3/dataset_provenance.py`、`tier3/generate_dataset.py` | `eval-datasets.mdx` | `tests/reporting/test_dataset_generation_provenance.py`、`tests/tier3/test_dataset_auto_grounding.py` | 无法实时复现（提供商健康）；由测试覆盖 | 已满足（仅测试） |
| D5 | 常见检查有结构化形式，无需评审器即可运行 | `tier3/eval_core/declarative_checks.py`、`tier3/grader_validation.py` | `eval-datasets.mdx`、`grader-validation.mdx` | `tests/tier3/test_declarative_checks.py`、`tests/tier3/test_grader_validation.py` | 未在实时中执行；卡片尚未报告每个分数中检查与评审的占比 | 部分满足 |
| D6 | 生成器写入当前格式，现有数据集可转换 | `tier3/dataset_migration.py`、`tier3/evals_spec.py` | `eval-datasets.mdx`、`cli-reference.mdx` | `tests/tier3/test_dataset_migration.py` | 实时生成器写入了当前结构 | 本仓库已满足；转换目录数据集受阻于目录负责人 |
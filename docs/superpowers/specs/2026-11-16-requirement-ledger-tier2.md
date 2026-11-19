# 需求台账：Tier 2

引用已对照 `main` 分支 `2ec6364f` 核对。实时观测结果保留自 2026 年 11 月基于 `f83c2edc` 的运行，并非在 `2ec6364f` 上重新运行。

| # | 需求 | 代码 | 文档 | 测试 | 证据 | 状态 |
|---|---|---|---|---|---|---|
| T2 | 集合扫描默认接受 1,024 个条目，条目上限和比较上限均可配置 | `constants.py`（`SIMILARITY_DEFAULT_MAX_ENTRIES = 1_024`、`SIMILARITY_MAX_ENTRIES = 5_000`）、`embedding/extractor.py`、`embedding/registry.py`、`validators/similarity.py` | `tier2-deduplication.mdx` | `tests/embedding/test_similarity_limits.py`、`tests/test_tier2_command_failures.py`、`tests/embedding/test_pairwise_preflight.py` | 实时 validate 运行：`Context Deduplication` 行为 `PASS`，`3 checks passed`；集合上限未在实时运行中触发 | 已满足（上限由测试覆盖） |
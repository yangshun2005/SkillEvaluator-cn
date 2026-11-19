# Tier 2、Tier 3 与安装命令参考

`skillevaluator <command> --help` 为准。

## Tier 2 命令

| 命令 | 关键参数 |
| --- | --- |
| `context-optimization-check PATH`（别名 `dedup-scan`） | `--threshold`（0.8）、`--model`、`--llm-model` |
| `similarity-check PATH` | `--threshold`（0.75）、`--full-body`、`--catalog`、`--save-catalog`、`--max-entries` |

## Tier 3 命令

| 命令 | 用途 |
| --- | --- |
| `tier3 evaluate PATH` | 实时 A/B 运行；加 `--autopilot` 可生成缺失的用例 |
| `tier3 validate PATH` | 检查 `<skill-dir>/evals/`（`--strict`、`--json`、`--harbor-contract`） |
| `create-eval-dataset PATH` | 写入 `<skill-dir>/evals/evals.json`（`--grounding auto` 需要实质性的作者指导或可用的匹配保留轨迹；已有数据集需加 `--force` 或 `--refine`；可选 `--grounding skill` 仅凭技能生成；`--full`、`--no-llm`、`--dry-run`、`--force`、`--prompt`、`--refine`） |
| `init-custom-grader PATH` | BYOG 起始模板（`--mode`、`--language`） |
| `init-harbor-task PATH` | BYOT 起始模板（`--case-id`、`--with-config`） |
| `tier3 select-decoys PATH` | 为分组工作区保存邻近技能 |
| `view PATH`、`compare PATH` | 打开最新报告；对比各 agent |
| `tier3 feedback RUN_DIR` | 针对一次已完成的运行进行对话（`--skill`） |
| `harbor-view JOBS_DIR` | 浏览用 `--harbor-keep-jobs` 保留的任务 |
| `dashboard [PATHS]` | 以表格展示已保存的报告（需 `dashboard` 附加组件） |

`tier3 evaluate` 关键参数：`-a, --agents`（`opencode`、`codex`、
`claude-code`）、`--env-mode`（默认 `docker`，可选 `local`、`kata`、`gke`、
`daytona`、`e2b`、`modal` 等）、`--n-attempts`、`--pass-threshold`、
`--stop-on-pass`、`--n-concurrent`、`--max-agents`、`--max-cases`、`--model`、
`--agent-model AGENT=MODEL`、`--skip-baseline`、`--skip-with-skill`、
`--grading-mode default|default_plus_custom|custom_only`、`--results-dir`、
`--include-skills`、`--skill-workspace-mode isolated|group`、`--copy-repo`、
`--timeout-multiplier`、`--harbor-keep-jobs`、`--agent-runtime-preflight`、
`--override-cpus`、`--override-memory-mb`、`--progress`、`--verbose`。

## 就绪检查与安装

| 命令 | 用途 |
| --- | --- |
| `health-check [--tier 1\|2\|3\|all]` | 就绪检查；缺失时下载固定版本的 SkillSpector 和 Gitleaks（会先询问）；`--checks` 用于 Tier 1 扫描器 |
| `doctor [--tier ...] [--env-mode M] [-a AGENTS] [--verify-models]` | 更深入的就绪检查；`--verify-models` 会实时探测模型目录 |
| `models [--limit N] [--json]` | 列出提供方的模型目录 |
| `ci init`、`ci status`、`ci demo` | 引导式 GitHub CI 配置 |
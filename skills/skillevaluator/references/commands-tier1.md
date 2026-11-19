# Tier 1 命令参考

报告选项和 Tier 1 命令。`skillevaluator <command> --help` 对已安装版本具有权威性。

全局选项放在命令之前：`skillevaluator -v validate <skill-dir>`。

## 报告（Tier 1 和 Tier 2 命令）

| 标志 | 默认值 | 作用 |
| --- | --- | --- |
| `-r, --report` | `tier1 PATH`、`tier2 PATH`、`validate` 为 `cli,json,html`；聚焦命令为 `cli` | `cli`、`json`、`html`、`markdown`、`sarif`、`junit`；逗号分隔或重复指定 |
| `-o, --output-dir` | `reports` | 报告目录 |

传入 `-r` 会严格按指定执行：`-r cli` 不写任何文件。文件名：
`validate` 写入 `skillevaluator-output-<timestamp>.*`；`tier1`/`tier2` 写入
`skillevaluator-tier1.*`/`skillevaluator-tier2.*`；聚焦命令写入
`skillevaluator-<kind>.*`（例如 `skillevaluator-quality.json`），并
覆盖之前的运行结果。

## 直接层级工作流

单独运行 `tier1`、`tier2` 或 `tier3` 会打印帮助。当路径名与子命令匹配时，
需在路径前加 `./`（`tier1 <path>/validate`）。

`skillevaluator tier1 PATH`：所有静态检查（包括 `dependency`），以及
在配置了提供方时的评分标准、LLM 安全性和发现验证。

| 标志 | 默认值 | 作用 |
| --- | --- | --- |
| `--checks TEXT` | 所有适用项 | 子集：`schema`、`version`、`security`、`pii`、`license`、`code-integrity`、`unicode`、`quality`、`lint`、`dependency` |
| `--skip CHECKS` | 无 | 排除检查（Gitleaks 还包括 `secrets`）；该运行无法认证 Tier 1 |
| `--llm / --no-llm` | 配置了提供方时开启 | 显式 `--llm` 需要有效的提供方 |
| `--min-score INTEGER` | `70` | 质量和评分标准最低分 |
| `--quality-config PATH` | 内置 | 调整预期标题、错误处理、token 预算、描述长度（最小值和最大值）以及保留名称规则 |
| `--pii-waivers PATH` | 无 | 已审查、会过期的 PII 豁免 |
| `--profile TEXT` | `default` | 验证策略 |

`skillevaluator tier2 PATH`：对单个技能进行技能内重复分析。
`--catalog FILE` 增加技能间比较（用
`skillevaluator similarity-check <skills-dir> --save-catalog <catalog.json>` 构建）；
`--threshold`（0.80）、`--similarity-threshold`（0.75）、`--full-body`。

`skillevaluator tier3 PATH`：当不存在源时生成一个起始用例，
然后在 Docker 中分别在有和没有该技能的情况下运行提供方原生 agent。接受所有
`tier3 evaluate` 选项。不接受 `-r`/`-o`。

## 聚焦的 Tier 1 命令

| 命令 | 关键标志 |
| --- | --- |
| `quality-check PATH` | `--min-score`（70）、`--quality-config` |
| `rubric-eval PATH` | `--min-score`（70）；每项标准必须达到 7/10 |
| `security-scan PATH` | `--llm`、`--llm-verify`、`--skillspector-baseline` |
| `pii-scan PATH` | `--llm-verify`、`--pii-waivers` |
| `lint-scripts PATH` | 仅报告选项；建议性 |
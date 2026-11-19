---
name: skillevaluator
description: 运行并解读用于 agent 技能的 SkillEvaluator。适用于任何技能基准测试或评估——校验或评分 SKILL.md、编写 evals.json 测试用例或 evals 配置、解释基准报告、结论和退出码，或在 CI 中为技能设置门禁。
license: Apache-2.0
compatibility: 需要 Python 3.12-3.14 和 skillevaluator CLI；实时评估还需要 Docker（或其他后端）以及 LLM 提供商密钥。
metadata:
  author: Christopher Kevin <christopherk@nvidia.com>
  version: "1.0.0"
  domain: [evaluation, ai-ml]
  tags: [skills, evaluation, benchmarking, ci, security]
---

# SkillEvaluator

对 agent 技能运行 SkillEvaluator CLI，并如实报告证据所示的结果：
Tier 1 校验（静态、安全、质量）、Tier 2 去重，以及 Tier 3
实时 agent 评估（对比使用技能与不使用技能的运行结果）。

## 用途

为 agent 提供一套可靠的流程，用 SkillEvaluator 评估技能：
选择正确的层级、检查就绪状态、运行评估，并如实报告
分数、发现项和退出码。

## 何时使用

当用户想要：

- 检查技能是否格式正确、安全，且质量足以发布
- 获取技能的质量分数、评分标准分数、安全或 PII 扫描结果
- 查找技能内部的重复指导内容，或技能集合之间的重叠
- 衡量技能是否真正帮助了 agent（Skill Lift、pass@k）
- 编写或修复 `<skill-dir>/evals/evals.json`、`<skill-dir>/evals/config.yml`、自定义评分器或 Harbor 任务
- 阅读 SkillEvaluator 报告、结论或退出码
- 将 SkillEvaluator 检查添加到 GitHub Actions 或 GitLab CI

时使用本技能。

不要用它编写目标技能的领域内容、审查普通应用代码，或评估不带技能的 LLM。
如需将现有基准转换为自定义评分器，请阅读
[references/custom-graders.md](references/custom-graders.md)。

## 前提条件

- 包含 `SKILL.md` 的目标目录（或技能文件夹、`.mdc`
  规则，或插件清单）。
- 运行在 Python 3.12-3.14 上的 CLI。如果缺失，安装前先询问
  （工作流第 1 步）。附加组件：`llm`（评分标准）、`security`（扫描器）、`tier2`、`tier3`、
  `all`、`dashboard`。从源码检出运行时使用
  `uv run --python 3.13 --extra all skillevaluator ...`。
- 无需密钥的命令（`quality-check`、`security-scan`、`pii-scan`、
  `lint-scripts`、`tier1 PATH --no-llm`、`validate PATH --tiers 1`）不需要密钥。
- LLM 阶段、Tier 2 和 Tier 3 需要提供商。最简单的方式：
  `export SKILL_EVAL_LLM_PROVIDER=nv_build` 和 `export NVIDIA_API_KEY=...`。
  公司或自托管网关的密钥也可以，用户无需新申请：
  `SKILL_EVAL_LLM_PROVIDER=openai-compatible` 配合
  `SKILL_EVAL_LLM_BASE_URL` 和 `SKILL_EVAL_LLM_API_KEY`。
  其他提供商：[references/providers.md](references/providers.md)。
- Tier 3 不使用你所在 agent 的模型。它在沙箱中启动
  独立的 agent 进程，分别带技能和不带技能，由 LLM
  评判器评分。因此即使你从 Claude Code、Codex 或 OpenCode 内部启动，
  也需要在宿主机上导出提供商密钥。使用 `nv_build`
  配合 `--env-mode docker` 或 `local` 时，一个密钥（来自
  https://build.nvidia.com 的 `NVIDIA_API_KEY`）即可覆盖 agent 和评判器；其他后端和
  原生 Claude 或 GPT agent 模型需要对应厂商的密钥
  （[references/providers.md](references/providers.md)）。
- Tier 3 还需要：一个 agent（`opencode`、`codex` 或 `claude-code`）和一个沙箱。
  Harbor 随 `tier3` 附加组件安装；你无需自行运行。
  默认的 `--env-mode docker` 需要 Docker 守护进程。没有 Docker 时，使用
  `--env-mode local`（实验性）：它在 Bubblewrap（Linux）或 Seatbelt（macOS）下运行宿主机上
  已安装的 agent CLI；仅限可信技能。

切勿打印、回显或提交 API 密钥；将其保留在宿主机环境中。

## 选择命令

| 用户目标 | 命令 | 需要 |
| --- | --- | --- |
| 快速免密钥质量评分 | `skillevaluator quality-check <skill-dir>` | 无 |
| 单个技能的完整静态门禁 | `skillevaluator tier1 <skill-dir> --no-llm` | 扫描器（缺失时下载；先询问） |
| 静态门禁加 LLM 评分标准和安全审查 | `skillevaluator tier1 <skill-dir>` | 提供商 |
| 仅 LLM 评分标准 | `skillevaluator rubric-eval <skill-dir>` | 提供商、`llm` 附加组件 |
| 仅安全 / PII 扫描 | `skillevaluator security-scan <skill-dir>`、`skillevaluator pii-scan <skill-dir>` | `security-scan` 需要 SkillSpector（缺失时下载；先询问） |
| 单个技能内的重复内容 | `skillevaluator tier2 <skill-dir>` | 支持嵌入的提供商 |
| 集合内的重叠 | `skillevaluator similarity-check <skills-dir>` | 嵌入 |
| 技能是否帮助了 agent？ | `skillevaluator tier3 <skill-dir>` | 提供商、agent、Docker |
| 同上，但不用 Docker | `skillevaluator tier3 evaluate <skill-dir> -a opencode --env-mode local` | 提供商、宿主机上的 agent CLI |
| 全部运行，单一退出码 | `skillevaluator validate <skill-dir>` | 以上全部 |
| CI 安全的免密钥门禁 | `skillevaluator validate <skills-dir> --profile default --tiers 1 -r json,markdown,sarif -o reports` | 扫描器 |
| CI 测试报告（JUnit XML） | 在 `-r` 中加入 `junit`，例如 `-r json,sarif,junit` | 同门禁 |
| 设置仓库 CI | `skillevaluator ci init` | GitHub 检出 |

完整参数表：[Tier 1](references/commands-tier1.md)、
[validate](references/commands-validate.md)、
[Tier 2、Tier 3 和设置](references/commands-tier2-tier3.md)。

## 先询问用户

免密钥、只读命令在其工具已就绪时无需许可。免密钥扫描和
`health-check --tier 1` 会下载缺失的扫描器，
因此启动会安装它们的命令前需获得许可。对同一安装复用用户的明确
授权，不要重复询问。否则，
在执行以下任何操作前停下来问一个简短问题：

- **安装。** 缺失的 CLI、附加组件（`tier2`、`tier3`）或扫描器会改变
  用户的环境。说明缺失的内容和安装命令，然后询问。
- **凭据和模型费用。** 按名称检查提供商变量是否已
  导出。如果没有，列出选项（来自
  https://build.nvidia.com 的 `NVIDIA_API_KEY`、`OPENAI_API_KEY`、`ANTHROPIC_API_KEY`）并询问用户
  想设置哪个。如果已有一个，说明提供商，除非用户要求该次运行，
  否则在第一个调用它的命令（不带
  `--no-llm` 的 `tier1`、`rubric-eval`、`tier2`、`create-eval-dataset`、Tier 3）前询问。如果导出了多个
  密钥，询问使用哪个提供商并设置 `SKILL_EVAL_LLM_PROVIDER`。
  切勿在文件中搜索密钥，也切勿打印密钥。
- **费用。** Tier 3 运行前，说明试验数 = 用例数 x 尝试次数 x agent 数 x 2
  （使用 `--skip-baseline` 或 `--skip-with-skill` 时更少），每个试验还会被
  评判器调用评分。建议先做 `--max-cases 1` 冒烟运行，完整运行前再次询问，
  用户设定限制时传入 `--max-attempts-total N`。
- **在宿主机上运行。** `--env-mode local` 在 Bubblewrap（Linux）或较弱的 Seatbelt（macOS）下
  运行宿主机上已安装的 agent CLI，默认开启网络
  访问。它不使用用户的 CLI 登录：每个试验获得
  私有主目录，仅使用宿主机上导出的提供商密钥。仅对用户信任的技能
  提供此选项；否则请用户启动 Docker。

当某层级无法运行时，说明缺少哪项要求以及用户必须
做什么。不要为该层级报告分数、提升或结论。

## 工作流

1. **获取 CLI。** 运行 `skillevaluator --version`。如果命令
   未找到，说明安装选项并按上述方式获得许可。获得
   授权后，按第一个可行方式安装，然后再次运行版本
   检查：
   - `uv tool install --python 3.13 "skillevaluator[all]"`（隔离，首选）
   - `pipx install --python python3.13 "skillevaluator[all]"`
   - 在 Python 为 3.12-3.14 的活动虚拟环境中运行
     `python3 -m pip install --upgrade "skillevaluator[all]"`（在较旧的 Python 上，pip
     会安装空的 `0.0.0a0` 占位符，没有 `skillevaluator` 命令）
   - 不安装时，为每个命令加前缀 `uvx --python 3.13 --from "skillevaluator[all]"`

   同样的许可规则适用于隔离安装、`uvx` 预置、
   升级和 `uv tool update-shell`（它会编辑 shell 启动文件），以及
   系统或共享 Python。切勿使用 `sudo pip`。如果二进制已安装
   但不在 `PATH` 上，为本会话将 `~/.local/bin` 添加到 `PATH`，或
   在运行 `uv tool update-shell` 前询问。如果文档中的命令或参数
   未知，用当初使用的安装器升级（`uv tool upgrade
   skillevaluator`、`pipx upgrade skillevaluator` 或 pip `--upgrade`）。如果
   安装失败（无网络、无 Python 3.12+），停止并报告
   确切错误和安装命令，不要猜测结果。
2. **定位目标。** 确认路径包含 `SKILL.md`，且其
   frontmatter 的 `name` 与目录名匹配。对于技能文件夹，
   命令会评估其中的每个技能。
3. **检查就绪状态。** 运行 `skillevaluator health-check --tier 1`
   检查扫描器，`skillevaluator doctor --tier 2` 检查提供商，
   `skillevaluator doctor --env-mode docker`（加 `--verify-models`）检查 Tier 3。
   `health-check --tier 1` 和免密钥扫描在缺失时会下载固定版本的 SkillSpector 和
   Gitleaks。当这些工具缺失时，在探测
   或扫描前获得安装许可。
   先修复每个失败的行。
4. **从免密钥开始。** 运行 `skillevaluator tier1 <skill-dir> --no-llm`。先修复
   CRITICAL 和 HIGH 发现项；`lint` 发现项仅供参考。
5. **添加 LLM 检查**（当提供商已设置且用户同意模型
   费用时）：`skillevaluator tier1 <skill-dir>`（评分标准、LLM 安全、
   验证）和 `skillevaluator tier2 <skill-dir>`。
6. **准备 Tier 3 用例。** 如果 `<skill-dir>/evals/evals.json` 缺失，询问
   用户三到五个该技能应处理的真实请求、每个请求的良好结果
   是什么样、一个不应触发该技能的请求，以及
   这些请求使用的任何输入文件。将他们的回答记录在
   `<skill-dir>/evals/EVAL.md` 中；仅有标题不合格。不要自行编造
   用例；如果用户要求草稿，将其标记为草稿并让他们
   审阅。`tier3 <skill-dir>` 和 `validate <skill-dir>` 在不存在时会生成一个
   起始用例，因此仅在用户的用例就位后才运行它们。如果用户未同意将 `evals/` 添加到技能
   （例如通过 `npx skills add` 或插件安装的技能），将整个
   目录复制到 `<scratch>/skills/<name>/`，保持目录名等于
   frontmatter 的 `name`，并评估副本。对于作用于真实
   系统的技能，保持用例可通过夹具或附属文件（`evals/files/`、
   `evals/environment/`）运行；仅当无法做到时，且经用户
   同意，将提示表述为"给出命令，不要运行它们"，需知 Tier 3 此时
   评分的是建议，而非执行。用
   `skillevaluator create-eval-dataset <skill-dir> --full --prompt <skill-dir>/evals/EVAL.md`
   生成，然后审阅。默认优先作者指导，其次是匹配的保留
   轨迹；现有数据集需要 `--force` 或 `--refine`。仅技能
   生成需要显式选择加入 `--grounding skill`。审阅：
   正向用例（显式、隐式、上下文）和至少一个负向
   用例，其 `"expected_skill": null`。必需结构：顶层 `skill_name`
   和 `evals` 数组；每个用例有 `id`、`prompt`、`expected_output` 和
   `expected_skill`（可选 `assertions`、`files`）。用
   `skillevaluator tier3 validate <skill-dir> --strict` 校验。格式：
   [references/tier3-datasets.md](references/tier3-datasets.md)；运行策略和
   评分：[references/tier3-running.md](references/tier3-running.md)。
7. **运行实时评估。** `skillevaluator tier3 <skill-dir>`（或
   `skillevaluator tier3 evaluate <skill-dir> -a opencode --env-mode docker --n-attempts 3`）。
   先使用 `--max-cases 1` 做低成本冒烟运行；此类运行永远不是
   发布证据。
8. **读取结果**（见下文），打开 `skillevaluator view <skill-dir>`，对于
   后续问题运行 `skillevaluator tier3 feedback <run-dir>`。
9. **迭代。** 修改技能，用相同的用例、
   agent 和模型重新运行相同命令，并比较。将单次运行的提升视为方向性参考。

## 读取结果

退出码：`0` 成功，`1` 门禁失败、证据为 `INCOMPLETE`，或
运行无法完成，`2` 用法错误（参数错误或路径缺失；修复
命令，而非技能）。

- **质量**（0-100）：正确性 35%、可发现性 25%、可靠性 25%、
  效率 15%。等级 A>=90、B>=80、C>=70。通过需要 `--min-score`
  （默认 70）且无 HIGH/CRITICAL 质量发现项。
- **门禁和分数是独立的结果。** 任何检查失败时 `tier1` 退出 `1`，
  例如 HIGH 安全发现项，即使质量等级
  是 A 或 B。报告两者，切勿仅凭等级就称技能通过。
- **评分标准**（0-100）：九项标准各评 0-10 分；通过需要加权
  分数 >= `--min-score` 且每项标准 >= 7。
- **Tier 2**：HIGH 或 CRITICAL 重复发现项在 `tier2 PATH` 中退出 `1`。
- **Tier 3**：五个维度（安全、正确性、可发现性、
  有效性、效率），每项 PASS >= 0.50、NEUTRAL 0.40-0.50、FAIL < 0.40。
  结论 PASS 需要至少一个 agent 的每个维度 >= 0.50。Skill
  Lift = 带技能总体 - 基线总体；PASS >= +0.05。`INCONCLUSIVE:
  skill never activated` 表示没有正向试验加载技能；修复
  描述或提示，而非分数。`tier3 PATH` 在 `FAIL`
  结论时退出 `0`；仅不完整的运行退出 `1`。
- **缺失的分数是 `NO SCORE` 或 `None`，绝不是 0。** 不要将它们计入平均。
- **发布**：要求 JSON 报告中的 `publication_status == "pass"`，
  而不仅仅是 `overall_passed`。通过的免密钥 Tier 1 运行仍以
  `Publication eligibility: Incomplete` 结束，因为 Tier 2 和 Tier 3 未运行。
  告诉用户这是预期情况，不是失败。

报告结构和 JSON 字段：[references/reports-and-ci.md](references/reports-and-ci.md)。

## 示例

```bash
# 免密钥初查
skillevaluator quality-check <skill-dir> -r cli -r json -o <report-dir>

# 整个集合的静态门禁，即使失败也生成完整报告
skillevaluator validate <skills-dir> --profile default --tiers 1 -c -r json,
```
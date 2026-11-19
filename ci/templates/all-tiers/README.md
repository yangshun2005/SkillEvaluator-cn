# 高级 CI 设置与维护者参考

这是 GitHub 和 GitLab 显式启用的全层级集成。标准无密钥 Tier 1 入门请使用[模板目录](../README.md)、[GitHub 指南](../../../docs/github-actions.mdx)或 [GitLab 指南](../../../docs/gitlab-ci.mdx)。

全层级 CLI 已包含在 0.9.0 中。1.0.1 发布版本刷新了其已验证的固定版本；普通 `skillevaluator ci init` 和手动 Tier 1 捆绑包保持原有范围。设置命令成功仅表示配置准备完成，不代表 CI 运行完成。在声称所有层级通过之前，请检查真实提供商支持的输出。

## 全层级模板

全层级设置每平台使用两个轻量消费者文件：

| 平台 | 评估 | 可信反馈 |
| --- | --- | --- |
| GitHub | [skill-checks-all-tiers.yml](skill-checks-all-tiers.yml) | [feedback-all-tiers.yml](feedback-all-tiers.yml) |
| GitLab | [gitlab-all-tiers.yml](gitlab-all-tiers.yml) | [gitlab-feedback-all-tiers.yml](gitlab-feedback-all-tiers.yml) |

这些文件包含占位符。请使用经过审查的 schema 版本 2 发布版本，包含平台的全层级能力以及不可变的工作流、工具和镜像固定版本。截至 0.9.0 的归档手动捆绑包仅包含 Tier 1 配置。使用 `ci init --tiers 1,2,3` 配合匹配的已安装发布版本，选择其捆绑的 `ci-release-all-tiers.json`；使用 `--release` 审查替代描述符。全层级 CLI 渲染这些引用并批准确切的 GitHub 调用方 blob。参见[入门指南](#opt-in-to-three-tier-ci)。

在首次演示 PR/MR 之前，将可信反馈入口合并到默认分支。在 GitLab 上，合并到现有根 `.gitlab-ci.yml` 意味着将 Tier 1 include 替换为恰好两个全层级 include，在任何 `stages` 列表中保留 `test`，并将其他每个作业的规则以 `- if: '$SKILL_EVAL_FEEDBACK == "true"'` / `when: never` 开头，因为可信反馈作为默认分支流水线运行。在设置 MR 之前将这些守卫合并到默认分支；参见 [GitLab 指南](#merge-into-an-existing-root-configuration)。

只有 Tier 1 门控 CI；Tier 2 和 Tier 3 保持建议性，包括失败或不完整的结果，其原生证据保留。CLI 的 `--tier3-required`、GitHub 的 `tier3-required` 输入和 GitLab 的 `SKILL_EVAL_TIER3_REQUIRED` 变量已弃用并被忽略；它们无法使 Tier 3 阻塞。报告在 14 天后过期，Docker 为默认，也支持原生本地模式。

维护者必须在工作流、运行时或反馈修复后同时刷新发布描述符的 `workflow_ref`、`feedback_ref`、`gitlab_ref` 和 `tools_ref`，以便新消费者获得经过审查的实现。`evaluator_version` 和 `runtime_lock_sha256` 必须与 `tools_ref` 处的包版本和 `uv.lock` 匹配。保持镜像摘要独立固定到经过审查的 Tier 1 镜像。

## 选择启用三层 CI

新的全层级路径支持 **GitHub 和 GitLab**，使用两个小型消费者文件。Tier 1 是唯一必需的层级；Tier 2 和 Tier 3 为建议性，包括失败或不完整的结果。`--tier3-required` 已弃用并被忽略；它无法使 Tier 3 阻塞 CI。Docker 为默认。`--env-mode local` 在选定的 CI 运行器上运行原生 agent CLI。此设置仅支持这两种模式。

默认 `ci init` 设置仍仅限 Tier 1。使用 `--tiers 1,2,3` 选择已安装发布版本捆绑的 `ci-release-all-tiers.json`。对于 1.0.1 固定版本，在可用时安装其发布包，或在发布前使用经过审查的源码安装。安装更新的包不会静默升级现有已提交的调用方。它必须声明 schema 版本 2 和 `all-tiers-github` 或 `all-tiers-gitlab`，并包含不可变的工作流、反馈、工具和镜像引用。Tier 1 描述符无法启用全层级设置。

```bash
skillevaluator ci init --tiers 1,2,3 \
  --platform github --provider openai --paths skills --mode preview --no-input --show-files
skillevaluator ci init --tiers 1,2,3 \
  --platform github --provider openai --paths skills --mode pr --no-input
```

要审查替代发布描述符，在任一命令中添加 `--release ./reviewed-ci-release.json`。使用前审查其不可变引用；该选项不会升级已安装的 CLI。

对于 GitLab 17.0 或更高版本，选择 `--platform gitlab`。GitHub 使用本地 `gh` 登录；GitLab 仅使用仓库主机的进程作用域 `SKILLEVALUATOR_GITLAB_TOKEN`，绝不使用通用 `GITLAB_TOKEN`。github.com 以外的源在任何 API 请求之前需要 `--tiers 1,2,3` 或 `--platform gitlab`。GitLab 设置不需要 `glab`。`--mode preview` 列出将添加或更改的文件而不写入任何内容；添加 `--show-files` 打印其内容。`--mode save` 写入两个本地配置文件；`--mode pr` 通过平台 API 打开设置审查。这些命令准备 CI 配置；它们不会在你的笔记本电脑上运行评估器。

| 本地命令 | 结果 | 仍需的证据 |
| --- | --- | --- |
| `ci init` | 准备评估和可信反馈入口 | 合并设置并配置 CI 运行器和密钥 |
| `ci status --tiers 1,2,3` | 读取默认分支配置和最新流水线状态 | 检查实际 CI 报告和门控 |
| `ci demo --tiers 1,2,3` | 打开小型技能变更 PR 或 MR | 检查 Tier 1 门控、建议性 Tier 2/3 结果、评论和保留的报告 |

在首次演示之前，在每个技能的 `evals/` 目录下提交原生数据集；参见[评估数据集](../../../docs/eval-datasets.mdx)。CI 复用这些用例，具有完整的正负配对覆盖。它不生成用例或使用 autopilot。缺失数据集或凭据会产生可操作的不完整结果；它们不计为通过的层级。Tier 1 CI 门控通过并不意味着建议性层级已完成或通过。此初始工作流不支持中央数据集映射。

可信 PR 或 MR 评论显示一个摘要表，每个技能一行：静态与安全验证、去重和实时 Agent 评估各获得一行状态、问题数和主要发现或阻塞项。对于不完整或缺失的证据，计数保持未知。每个技能有一个 **Report** 链接指向运行的组合构建产物；正常仓库认证和构建产物过期适用。下载并解压，然后打开 `index.html` 查看详细聚合和原生报告。托管 HTML 仍为可选。参见[报告与结果](#retained-three-tier-ci-evidence)示例。

创建 `skillevaluator` 执行环境并在其中存储提供商密钥（`OPENAI_API_KEY`、`ANTHROPIC_API_KEY` 或 `NVIDIA_API_KEY`）。GitHub 还支持从仓库 CI 密钥生成的显式映射；建议使用作用域环境密钥。在 GitHub 上，提供商支持的作业要求发起者具有仓库写入、维护或管理员权限，通过 GitHub API 验证。它们还要求确切的当前同仓库 PR head 或当前默认分支提交。这些检查在进入执行环境后再次运行。环境审查者可选：如果仓库需要手动批准则配置；自动检查不需要付费的审查者保护功能。将仓库和 CI 工作流写入权限限制为可信维护者。GitLab 的受保护执行和反馈先决条件如下所述。

在支持 `environment-deployment` 的 GitHub 工作流修订版上，在评估调用方的 `with` 块中设置 `environment-deployment: false`，以使用环境密钥和变量而不为 CI 创建部署记录。默认为 `true`，保留现有消费者。必需审查者和等待计时器在 `false` 时仍适用；自定义部署保护应用需要 `true`。已发布的 1.0.1 工作流固定版本早于此输入。在添加之前升级到经过审查的支持工作流修订版，并同时刷新可信反馈调用方的已批准工作流引用和调用方 blob。

使用 `--provider`、`--embedding-provider`、`--model` 和 `--embedding-model` 选择匹配的聊天和嵌入配置；默认嵌入提供商跟随 OpenAI 或 NVIDIA Build；Anthropic 默认使用 OpenAI 嵌入，也需要该提供商的密钥。共享的 `--model` 配置聊天/评判和原生 agent 提供商默认值；已提交评估配置中的 agent 特定模型保持其原生覆盖语义。`--agents` 覆盖原生提供商默认值：OpenAI 使用 Codex，Anthropic 使用 Claude Code，NVIDIA Build 使用 OpenCode。CI 从经过审查的工具提交安装 Tier 2 或 Tier 3 附加组件。Tier 3 使用 `doctor` 检查选定环境；本地模式还将固定版本的原生 CLI 安装到隔离的运行时目录。

对于具有 `ci-model-routing-v1` 的经过审查候选，独立配置 Tier 1 LLM 检查和 Tier 3 网关 agent。这些选项在设置期间本地运行；CI 稍后读取已提交的配置：

```bash
skillevaluator ci init --tiers 1,2,3 --mode preview --no-input \
  --paths skills --provider nv_build --embedding-provider nv_build \
  --model "$DEDUP_MODEL" \
  --tier1-llm --tier1-provider nv_build --tier1-model "$SECURITY_MODEL" \
  --tier3-provider openai-compatible --tier3-base-url "$GATEWAY_API_ROOT" \
  --tier3-model "$JUDGE_MODEL" --gateway-secret MY_INFERENCE_KEY \
  --agents claude-code,codex \
  --agent-model "claude-code=$CLAUDE_MODEL" --agent-model "codex=$CODEX_MODEL"
```

将这些模型变量设置为确切可访问的目录 ID，将网关变量设置为其 API 根，通常以 `/v1` 结尾。`MY_INFERENCE_KEY` 是 CI 密钥的名称，不是其值。配置该仓库 Actions 密钥和 Build 密钥 `NVIDIA_API_KEY`；切勿将密钥值传递给 `ci init`。审查预览后使用 `--mode pr`。`--base-url` 和 `--embedding-base-url` 在需要时配置共享聊天和嵌入端点。Tier 1 的不可变镜像使用原生提供商端点，因此当其他层级使用自定义网关时，选择支持的 `--tier1-provider`。

单独的 `--tier3-model` 选择评判。可重复的 `--agent-model` 选择每个原生 agent 的模型，而不替换为评判者的模型。`--tier1-model` 也到达 SkillSpector；空值保留镜像的提供商默认值。确认固定镜像支持所选模型。

每个组合报告保留基准卡片。在 GitHub 上，添加 `--benchmark-pr` 配合 `ci-benchmark-pr-v1` 候选，在默认分支运行后通过专用更新 PR 在其技能旁提议符合条件的卡片。设置仍包含两个入口文件。GitLab 在其聚合构建产物中保留卡片；此选项不提供自动基准更新 MR。较旧的描述符在写入不支持的 CI 文件之前拒绝这些新选项。

`--timeout-minutes` 设置原生作业持续时间（10 到 360，默认 300）。评估器的运行时预算提前 20 分钟结束，或较短持续时间的四分之一，以便设置和保留证据适合。如果设置超过该预留时间且 GitLab 在超时时停止原生作业，该作业不上传原生证据，聚合报告该层级的证据为缺失。GitLab 接收字面持续时间，因此配置在 GitLab 17.8 上也能工作。`--runner` 选择 GitHub 运行器标签或 GitLab 运行器标签。GitHub 默认为 `ubuntu-24.04`；GitLab 默认为配置的 `docker` 标签。Docker 模式需要可用的守护进程、Docker CLI、Compose v2 和 Buildx；GitLab 包含的 Docker-in-Docker 服务需要特权运行器共享 `/certs/client` 以获取守护进程的 TLS 客户端证书。本地模式需要兼容的原生运行器与 Node/npm；生成的 GitLab 本地配置移除 Docker 服务。云沙箱后端不在此入门路径中。

`--runner` 选择原生 Tier 2/3 运行器。静态和 LLM 增强的 Tier 1 保持其独立的镜像能力运行器：GitHub 使用 `ubuntu-24.04`，GitLab 使用无标签镜像运行器。因此选择本地 Tier 3 运行器不会为该原生运行器添加 Docker 先决条件。

在**打开演示之前**将两个生成的文件合并到默认分支。首次设置 PR/MR 可以显示评估检查，但自动评论需要默认分支上存在可信反馈入口。GitLab 还需要受保护的默认分支和具有 `api` 作用域和 Maintainer 角色的项目访问令牌，存储为作用域、受保护的 `SKILLEVALUATOR_GITLAB_TOKEN`，仅限于 `skillevaluator-feedback` 环境。GitLab 18.4 之前还需要单独的项目 Reporter/read_api `SKILLEVALUATOR_GITLAB_READ_TOKEN` 用于当前 MR 准入，作用域为受保护执行环境（默认 `skillevaluator`）。在任何 GitLab 版本上，当当前受保护默认分支 API 检查拒绝 `CI_JOB_TOKEN` 时（例如 HTTP 403），也需要相同的读令牌。配置或验证读令牌的项目访问权限，而不是使用反馈发布者令牌。读凭据在提供商工作之前被移除。较新的服务器在端点允许时默认使用 `CI_JOB_TOKEN`。在线 init/status 报告服务器版本先决条件；离线 preview/save 使准入就绪状态未验证。MR 作业不接收评论令牌。GitLab 的反馈请求使用其短期作业令牌；发起用户需要权限在受保护默认分支上创建流水线、传递流水线变量并运行受保护反馈环境。没有它，请求是允许的失败：不发布评论，Tier 1 门控不变。GitLab 的提供商支持和反馈作业不记录部署。不要向其环境添加部署批准规则：取决于 GitLab 版本，批准要么不门控这些作业，要么将其变为手动作业。使用受保护环境的部署访问权限限制它们。参见平台页面了解 [GitHub](#github-checks-and-feedback) 和 [GitLab](#gitlab-checks-and-feedback) 详情。

### 升级仅 Tier 1 的消费者

在预览显式全层级设置时保留现有工作流，使用兼容的描述符。审查新的评估器和反馈固定版本、提供商、运行器、受保护环境、路径和仅 Tier 1 门控。在设置审查中移除或禁用旧的 Tier 1 调用方，一旦新的全层级调用方覆盖相同的根，以便 PR 不运行两个独立扫描。合并新的反馈入口，然后打开新的演示。现有已发布捆绑包和仅 Tier 1 调用方保持当前行为。

在 GitLab 上，Tier 1 和全层级配置共享根 `.gitlab-ci.yml`，CLI 从不替换它。`--mode preview` 打印渲染的文件和合并步骤；`--mode save` 和 `--mode pr` 在存在不同的根文件时停止并报错。首先用 `SKILL_EVAL_FEEDBACK` 规则守卫其他作业，并将这些守卫合并到默认分支：设置 MR 自己的反馈请求运行默认分支的当前配置。然后，在设置 MR 中，将 Tier 1 `skill-evaluation.yml` include 替换为两个渲染的 include，添加渲染的变量、工作流规则和作业覆盖，在任何 `stages` 列表中保留 `test`，并添加 `.gitlab/skill-feedback.yml`。参见[合并到现有根配置](#merge-into-an-existing-root-configuration)。

## GitHub 检查和反馈

选择启用的 [CLI 设置](#opt-in-to-three-tier-ci)创建 `.github/workflows/skill-checks-all-tiers.yml` 和 `.github/workflows/feedback-all-tiers.yml`。使用兼容的已安装 CLI 及其捆绑的带能力描述符。归档的 0.9.0 [手动配置](../../bundles/0.9.0/README.md)仍仅限 Tier 1。`--tiers 1,2,3` 选择已安装发布版本捆绑的描述符；`--release` 可以用另一个经过审查的描述符覆盖它。[源模板](https://github.com/NVIDIA/SkillEvaluator/tree/main/ci/templates)包含占位符，必须用经过审查的不可变引用填充。

共享的 `evaluate-all-tiers.yml` 和 `feedback-all-tiers.yml` 文件定义可复用的 `workflow_call` 工作流。将它们添加到 SkillEvaluator 不会在 SkillEvaluator 自己的拉取请求上启用检查或评论。每个仓库，包括 SkillEvaluator 本身，需要在那里提交生成的调用方对，可信反馈调用方在其默认分支上。

生成的 GitHub 全层级调用方（`skillevaluator ci init --tiers 1,2,3`）还携带 `paths` 过滤器。在选定技能文件夹或两个调用方工作流下未更改任何内容的拉取请求不启动运行，也不接收评论。

SkillEvaluator 使用 `.github/workflows/skill-checks-all-tiers.yml` 评估自己的 `skills/` 集合。其消费者反馈监听器是 `.github/workflows/skill-feedback-all-tiers.yml`，保留公共可复用发布者在 `.github/workflows/feedback-all-tiers.yml`。当常规反馈文件已定义可复用工作流时，CLI 选择此替代文件名，`ci status --tiers 1,2,3` 识别它。

仓库调用方固定经过审查的评估器工作流和工具，以及已验证的已发布 Tier
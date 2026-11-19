# 可复用仓库 CI

本目录提供用于质量与安全检查的共享 CI 运行时，以及使用方接入模板。GitHub Actions 或 GitLab CI/CD 请先阅读[仓库集成](../docs/repository-integration.mdx)。GitLab 适配器位于 `ci/gitlab/`；镜像与报告生成由两个平台共用。上游工作流发布版本和公共包必须先经过审核并发布，使用方才能使用此路径。当 `RELEASE_WORKFLOW_SHA` 或 `RELEASE_IMAGE_DIGEST` 仍未解析时，源模板不可运行。

手动配置请从[可复制模板目录](templates/README.md)开始。[报告与反馈示例](examples/README.md)保留在公共仓库中。两者都不需要本地接入 CLI 或个人演示访问权限。

## 使用方契约

将发布生成的 `skill-checks.yml` 复制到 `.github/workflows/skill-checks.yml`，然后编辑 `paths`。共享的 `NVIDIA/SkillEvaluator/.github/workflows/evaluate-skills.yml` 要求显式指定不可变的 `image`，例如 `ghcr.io/nvidia/skillevaluator-tier1@sha256:RELEASE_IMAGE_DIGEST`；使用方还需通过 `uses: ...@RELEASE_WORKFLOW_SHA` 固定工作流。不要替换为可变镜像标签或未经审核的分支引用。

镜像为公共和私有仓库选择同一个内置的 `default` 验证策略。其规则位于 `src/skillevaluator/config/profiles/default.yaml`；不存在 `internal`/`external` CI 输入。较早发布的镜像将同一策略标记为 `external`。仅升级工作流不会改变已固定镜像的策略标签。配置名称不控制网络访问或报告可见性。

| 输入 | 基本配置 |
| --- | --- |
| `paths` | `skills`；最多 100 个以换行分隔的仓库相对技能目录或根目录 |
| `scan-mode` | `auto`：PR 上扫描受影响的技能，推送/手动运行时扫描所有已配置技能；`all` 覆盖选择 |
| `llm` | `false`；静态检查不需要模型密钥 |
| `artifact-name` | 保留 `skill-evaluator-tier1`，供 PR 评论或可选托管发布器使用 |
| `report-base-url` | 可选的自有 Pages URL，仅在配置托管发布后使用 |
| `scan-workers` | `4`；同时扫描的技能数，范围 1 到 16。`1` 表示一次扫描一个技能 |
| `enforcement-mode` | `strict`（默认）在判定失败时阻断；`advisory` 使完整的策略/评分失败不阻断，但仍阻断运行时/证据/不完整失败 |
| `security-baselines` | `false`；按下文说明显式应用已审核的逐技能 SkillSpector 基线 |

运行时发现大小写精确匹配的 `SKILL.md` 文件，拒绝不安全路径和符号链接，对重叠根目录去重，并将每次调用限制在 100 个技能内。这是选择上限，不是运行时保证：扫描有时间预算，并保留已完成结果，对未处理的技能显式标记为不完整。不要将变更技能选择理解为新的全目录扫描。最多同时扫描 `scan-workers` 个技能，每个技能有独立的私有 home 和临时目录。日志、注解和报告行保持技能顺序。时间预算耗尽时，正在运行的扫描停止，尚未开始的技能报告为未评估。Ctrl-C 也会立即停止正在运行的扫描，包括其启动的扫描器工具。没有受影响的技能时，检查成功并显示 **No skills changed**，同时记录验证未运行。完全移除的技能列为已删除/跳过。已移除清单但技能文件夹中仍有跟踪文件属于错误。选择基于已发现技能内的变更文件；配置根目录之外的共享文件不会被推断为依赖。当这些共享资源的变更需要评估时，使用 `scan-mode: all`。变更技能选择检出完整历史但不含旧文件内容（无 blob 克隆）；完整扫描仅获取被扫描的提交。启用已审核的安全基线会获取完整、未过滤的历史，以便扫描器镜像内无需网络即可获得目标策略文件。静态扫描需要网络连接：漏洞数据库可能收到包元数据。依赖审计从不安装包或运行构建脚本：requirements 必须提供精确的包/版本固定。无法安全审计的未固定、URL、本地路径和包含的 requirements 报告为不完整；传递覆盖需要显式固定的文件。Tier 2/3 和插件评估不在本工作流范围内。扫描器安装和源评估在预构建的 Linux 镜像中进行；报告写入只读使用方检出之外。缺少扫描器、扫描不完整以及报告证据缺失或矛盾会导致作业失败。

简短的 `summary.md` 是 Actions 落地视图。`details.md`、原始逐技能 JSON、原生 HTML、SARIF 和扫描器日志在扫描产物中保留详细证据，产物名为 `skill-evaluator-tier1-attempt-N`（第 `N` 次尝试）。第二个小产物 `skill-evaluator-tier1-evidence-attempt-N` 仅包含 `summary.json` 和每个技能的 `skillevaluator-tier1.json`。评论和托管发布工作流只下载该产物。产物在配置的保留期后过期；托管历史是可选的独立界面。质量评分不是扫描器通过百分比，也不证明实时代理行为有效。

扫描前，CI 会在所选技能中收集内置凭据模式、NVIDIA API 密钥以及未加引号的 `KEY=value` 和 `Bearer` 凭据匹配的值。原始扫描器文件保留在私有临时存储中。识别出的值、被扫描器截断或转义的部分以及数据库 URL 密码会从保留的报告和注解中移除。保留的 Markdown 和 SARIF 从脱敏后的 JSON 渲染。存在凭据或 LLM 密钥时，原始扫描器日志不予保留；结构化报告仍可用。预检限制为 10,000 个文件、每文件 32 MiB、每技能 128 MiB、每行 8,192 个字符，脱敏值有界。如果无法安全检查或脱敏输入，运行会报告错误且不发布详细扫描输出。这不保证检测到所有凭据或转换后的值。移除并轮换已暴露的凭据；之前运行的报告和 Git 历史不会自动重写。

## 已审核的安全基线

此源能力需要配套的工作流和镜像发布；不会追溯添加到已发布的接入包中。在 GitHub 可复用工作流上启用 `security-baselines: true`，或设置 GitLab 的 `SKILL_EVAL_SECURITY_BASELINES: "true"`。两者默认均为 false。此选项仅应用 SkillSpector 例外；不会禁用 PII 或安全检查、改变质量门禁，也不会转发单独的 CLI `--pii-waivers` 选项。

如果中央组件封装了 GitHub 工作流，请同时更新其已批准的工作流和镜像固定，并将 `security-baselines` 布尔值转发给扫描作业。每个使用方保留自己的已审核映射和基线文件。

提交固定清单 `.skillevaluator/skillspector-baselines.yaml`：

```yaml
version: 1
baselines:
  legacy/sample-expert: config/security-baselines/sample-expert.json
```

键是精确的仓库相对技能目录，值是仓库相对的原生 SkillSpector YAML 或 JSON 基线文件。不支持通配符。未映射的技能不获得基线。路径必须规范，不含绝对路径、父级遍历、控制字符或链接；清单拒绝重复键、YAML 锚点和别名以及未知顶层键。清单限制为 64 KiB 和 1,000 个映射，每个基线限制为 2 MiB，合并策略文件限制为 16 MiB。凭据预检也将其现有的行和脱敏限制应用于策略文件。

映射和基线字节都来自同一个已批准的 Git 提交：GitHub 上为 PR 的目标提交，或 GitLab 的目标分支提交。对于没有目标 SHA 的分离式同项目 GitLab 合并请求，作业在扫描前获取实际目标分支；绝不替换为合并基或候选头。Fork GitLab 合并请求需要检出中已有平台提供的精确目标 SHA，否则失败关闭。推送/手动运行使用其当前提交。提议首个清单的 PR 必须经过审核合并后此选项才能使用它；候选的新增或编辑批准绝不适用于同一 PR。

缺少策略提交、清单、文件或无效映射会导致配置失败。选定的基线字节以只读方式物化到技能树之外的私有临时存储中，并仅通过 `--skillspector-baseline` 传递给映射的技能。扫描器保留对基线格式的正常验证。精确指纹仍绑定源内容和扫描器版本；任一变更后重新生成并审核，或使用刻意狭窄的已审核规则。基线无法清除不完整分析。

保留的 `summary.json` 在 `execution.security_baselines` 中记录策略提交、清单路径、摘要和 `mapped_skill_count`，并在 `skills[].security_baseline` 中记录每个映射技能的基线文件和摘要。技能的原生报告记录基线是否应用以及抑制了什么。策略文件接受与技能内容相同的凭据预检和报告脱敏。

## 无托管报告的 PR 评论

使用规范的 [`comment-skill-checks.yml`](../.github/workflows/comment-skill-checks.yml) 可复用工作流更新现有的已评分 Tier 1 PR 评论，无需部署报告站点。它使用与托管发布器相同的签名执行回执、调用方和评估器批准、产物验证、报告渲染器和机器人评论更新器。评论显示计数、状态、质量评分和等级，以及每个受影响技能的首个操作。工作流运行链接到可下载报告。不需要 Pages 设置、站点历史分支或报告 URL。

[1.0.1 包](bundles/1.0.1/README.md)包含仅评论工作流；归档的 0.9.0 包不包含。默认 `skillevaluator ci init` 反馈仍配置托管发布。使用带版本的仅评论文件；不要将其文件名添加到较旧的发布固定中。

1. 保持 `.github/workflows/skill-checks.yml` 名称为 **Skill checks**，包含一个可复用扫描作业和默认 **skill-evaluator-tier1** 产物名。在 `contents: read` 之外启用 `id-token: write` 以获取签名执行回执。保留已审核的评估器工作流和镜像固定。
2. 将 [`comment-skill-checks.yml`](bundles/1.0.1/comment-skill-checks.yml) 复制到 `.github/workflows/`。将其 `uses:` 和 `tools-ref` 固定到已审核的评论工作流修订。将 `trusted-evaluator-shas` 设置为已批准的扫描器修订；这可能与评论工具修订不同。
3. 完成所有扫描文件编辑后，运行 `git hash-object .github/workflows/skill-checks.yml` 并将 `trusted-caller-blobs` 设置为该精确值。每次后续扫描文件编辑后审核并刷新 blob。对于共享封装器，保持[相同的封装器批准和委托](#calling-through-a-shared-workflow)。
4. 将两个文件合并到默认分支，并打开一个修改技能的 PR。评论工作流仅授予 `contents: read`、`actions: read` 和 `pull-requests: write`。组织策略必须允许这些权限和引用的操作。选择此工作流或托管发布器提供 PR 反馈；两者更新同一个机器人评论。

第二个工作流在 **Skill checks** 完成后运行，包括失败的扫描。其手动触发需要已完成的 `run-id` 和 `run-attempt`。它仅对来自同一仓库的开放、当前头 PR 发表评论。具有有效保留原生证据的失败扫描收到标准失败评论；证据上传前的失败不产生合成结果。没有受影响技能时使用现有的 **No skills changed** 结果。Fork PR、被取代的结果、过时的 PR 头、过期的产物和无法验证的证据不会收到评论。签名令牌过期本身不会使历史回执失效；其签名密钥和该尝试的产物仍必须可用。

## 可选托管报告和 PR 反馈

这是额外设置，不是基本扫描文件中的 `comment: true` 开关。此发布器将评论与成功的托管部署耦合。要在不托管的情况下使用相同评论，请遵循[仅评论设置](#pr-comments-without-hosted-reports)。

1. 决定报告可以存放的位置。**除非显式配置为私有，Pages 站点是公开的，即使仓库是私有的。** 私有 Pages 需要 GitHub Enterprise Cloud；参见 [Pages 访问控制](https://docs.github.com/en/pages/getting-started-with-github-pages/changing-the-visibility-of-your-github-pages-site)。报告包含未合并 PR 的路径、发现和源代码摘录。当公共托管不合适且私有 Pages 不可用时，使用扫描产物。此发布器拥有报告站点的根 `index.html`、`history.json`、`runs/`、`main/latest/` 和 `pr/` 路由，并部署整个站点。启用前协调现有仓库 Pages 站点；仅更改 URL 不会为其输出命名空间。
2. 启用 **Settings → Pages → Source: GitHub Actions**。创建 `gh-pages` 历史分支，包含 `.nojekyll` 和内容为 `[]` 的 `history.json`。不要在现有站点/历史检出上初始化。
3. 将生成的 `publish-skill-reports.yml` 添加到使用方的**默认分支**。将 `report-base-url` 设置为自己的 Pages URL。可选地在扫描作业上设置相同 URL 以暴露其待处理报告链接；发布在扫描完成后进行。
4. 保持扫描文件名 `.github/workflows/skill-checks.yml`、工作流名称 **Skill checks** 和产物名 **skill-evaluator-tier1**。专用发布器验证此调用方，并仅接受一个已批准的引用评估器工作流，以及下文所述的显式批准封装器。不同名称的手动 LLM 工作流不会被自动接受。发布器仅读取该尝试的 `skill-evaluator-tier1-evidence-attempt-N` 产物。来自不上传该产物的评估器修订的扫描无法发布。重新运行该扫描会复用其提交和固定评估器。相反，将扫描调用方更新到上传该产物的发布版本，批准该评估器 SHA 和新调用方 blob（步骤 5 和 6），并启动新扫描。
5. 将 `trusted-evaluator-shas` 设置为已审核的评估器工作流提交。将 `tools-ref` 设置为包含发布器辅助工具的已审核不可变提交；生成的发布文件提供两者。`tools-repository` 默认为 `NVIDIA/SkillEvaluator`；为集成测试提供显式审核的 fork 覆盖，标准接入不需要。
6. 生成的发布文件包含其未编辑扫描模板的调用方 blob。对于托管，首先在 `.github/workflows/skill-checks.yml` 的 `permissions` 下取消注释 `id-token: write`。这允许 GitHub 签名工作流的执行身份；不需要个人令牌或云账户。对该使用方文件进行**任何编辑**后，计算其精确 Git blob：`git hash-object .github/workflows/skill-checks.yml`。将该值放入 `trusted-caller-blobs`。每次后续扫描文件编辑都需要审核和新的已批准 blob。仅在需要历史回填时保留较旧的已审核 SHA/blob 条目。两个允许列表接受逗号或换行分隔的条目；从自己已审核的配置计算信任哈希。发布器验证单独的 GitHub 签名执行回执，并在该回执的不可变工作流修订处检查调用方。它绝不从 PR 的当前基或头重建已执行的工作流。
7. 发布器调用方必须授予 `contents: write`、`actions: read`、`pages: write`、`id-token: write` 和 `pull-requests: write`。可复用作业将这些缩小到历史准备、Pages 部署和 PR 反馈。扫描使用 `contents: read`，启用 PR 评论或托管时额外使用 `id-token: write`。自动 GitHub 令牌足够；发布不需要个人访问令牌或模型凭据。

发布器使用固定的可信工具从有界、已验证的 JSON 重建原生 HTML。它不执行使用方 PR 代码，也不将上传的 HTML 信任为站点代码。其作业使用工具仓库的稀疏检出：准备作业获取根文件、`ci/` 和 `src/`；评论作业仅获取 `ci/comment_report.py` 和 `ci/github_api.py`，它们仅使用标准库。每个完成事件将发布绑定到其记录的运行尝试；手动回填必须指定该尝试。默认保留最新 100 次运行（`retention-runs`），在 512 MiB 站点预算内，加上当前 `main/latest/` 和开放 PR 报告。较旧的未保护运行首先被修剪；如果仅受保护报告就超出预算，发布停止。已关闭 PR 别名被退役。保留的 `runs/<run-id>/<attempt>/` 路径不可变；延迟运行不能替换较新的别名。当前默认分支报告使用 `main/latest/` 路由，即使实际默认分支名称不同。机器人评论创建一次，仅在该发布自身部署后对开放、当前头、同仓库 PR 原地更新。人工评论不受影响。当自动发布因另一个发布器运行已推送相同运行尝试而未向 `gh-pages` 提交任何新内容时，其 Pages 部署和 PR 评论被跳过。推送它的运行负责部署然后评论。手动运行和重新运行始终部署，因此部署失败后重新运行仍会发布和评论。因 PR 在扫描后关闭或移动而跳过的发布既不部署也不评论。Fork PR 扫描可在无提供方密钥的情况下运行；Fork PR 机器人评论和当前 PR 别名未启用。与任意现有 Pages 部署一起托管需要进一步集成工作。

### 通过共享工作流调用

仅评论工作流接受下文相同的 `trusted-wrapper-workflows` 和 `delegated-evaluator-wrapper` 输入，具有相同的签名回执和调用方 blob 批准。其封装器继续转发扫描设置和固定；标准评论渲染器保留在 SkillEvaluator 中。

直接使用方无需封装器配置。如果扫描调用共享工作流，而该工作流又调用 SkillEvaluator，请使用暴露 `trusted-wrapper-workflows` 的已审核发布器修订。归档的 0.7.1 发布器固定早于此支持；向该较旧工作流添加输入不会启用它。将发布器作业的 `uses:` 修订和 `tools-ref`
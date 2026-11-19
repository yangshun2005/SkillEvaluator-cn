# 仓库 CI 接入

## GitHub：选择模板

- 检查与可下载报告：将 `skill-checks.yml` 复制到 `.github/workflows/skill-checks.yml`。将 `paths` 设为技能根目录，并选择 `min-score`（默认 70）。
- 标准 PR 评论：还需将 `comment-skill-checks.yml` 复制到 `.github/workflows/`，并完成下方的评论配置。无需托管报告。
- 托管报告与 PR 评论：还需将 `publish-skill-reports.yml` 复制到 `.github/workflows/`，并完成下方的一次性配置。

选择一种反馈工作流。工作流与镜像版本已填好。模板使用 GitHub 自动令牌；默认检查无需个人令牌或模型密钥。你的组织必须允许所引用的 action 及所提供的权限。

### PR 评论配置

1. 在 `skill-checks.yml` 的 `permissions` 下取消注释 `id-token: write`，以获取签名的执行回执。保留其单个可复用扫描任务，并自定义技能路径。
2. 完成所有扫描文件编辑后，运行 `git hash-object .github/workflows/skill-checks.yml`，将结果填入 `comment-skill-checks.yml` 的 `trusted-caller-blobs`。每次编辑扫描文件后都要刷新该值。
3. 将两个文件合并到默认分支，保留提供的扫描名称、文件名和构建产物名称。打开一个技能 PR 以触发新扫描。评论工作流仅需 `contents: read`、`actions: read` 和 `pull-requests: write`；无需 Pages 或历史分支。

标准评分 Tier 1 评论会针对符合条件的当前 head、同仓库 PR 更新，包括有有效证据的失败扫描。完整报告仍可从扫描运行中下载。手动重试需同时提供已完成的运行 ID 和尝试次数。

[完整评论配置](https://github.com/NVIDIA/SkillEvaluator/blob/c9369a7811bde083b550f2ac488222e8c388d7fd/ci/README.md#pr-comments-without-hosted-reports)。

### 一次性发布器配置

1. 启用 Settings → Pages → Source: GitHub Actions。报告可能包含源码摘录；私有仓库不会自动使其 Pages 站点变为私有。若托管不适用，请使用扫描构建产物。请先协调任何现有 Pages 站点。
2. 用 `.nojekyll` 和内容为 `[]` 的 `history.json` 初始化新的 `gh-pages` 历史分支。保留现有站点内容和历史。
3. 将发布器模板中的 `YOUR_REPORT_BASE_URL` 设为你的 Pages URL。
4. 在 `skill-checks.yml` 的 `permissions` 下取消注释 `id-token: write`，以获取签名的执行回执。自定义扫描文件后，运行 `git hash-object .github/workflows/skill-checks.yml`，将结果填入 `trusted-caller-blobs`。每次编辑扫描文件后都要刷新该值。
5. 将两个文件合并到默认分支，保留提供的扫描文件名、工作流名称和构建产物名称。发布器已声明其所需权限。

新的符合条件的 PR 扫描会自动触发发布。部署成功后，机器人会在当前、开放、同仓库 PR 上创建或更新一条评论。此发布器不支持 Fork PR 评论和仅评论操作。

没有签名权限时，普通扫描和可下载报告仍会运行；托管会被跳过。提交配置并触发新扫描；重跑旧任务会保留其原始工作流。

[完整发布器配置](https://github.com/NVIDIA/SkillEvaluator/blob/c9369a7811bde083b550f2ac488222e8c388d7fd/ci/README.md#optional-hosted-reports-and-pr-feedback)。

## GitLab

对于 GitLab，将 gitlab-skill-checks.yml 合并到根目录的 .gitlab-ci.yml，并设置 SKILL_EVAL_PATHS。其规则必须保留在根文件中，才能启用合并请求流水线。
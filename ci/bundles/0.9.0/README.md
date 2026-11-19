# 仓库 CI 接入

## GitHub：选择模板

- 检查与可下载报告：将 `skill-checks.yml` 复制到 `.github/workflows/skill-checks.yml`。将 `paths` 设为技能根目录，并选择 `min-score`（默认 70）。
- 托管报告与 PR 评论：还需将 `publish-skill-reports.yml` 复制到 `.github/workflows/`，并完成下方的一次性设置。

工作流与镜像版本已填好。两个模板均使用 GitHub 自动令牌；默认检查无需个人令牌或模型密钥。你的组织必须允许所引用的 action 及所提供的权限。

### 一次性发布者设置

1. 启用 Settings → Pages → Source: GitHub Actions。报告可能包含源码摘录；私有仓库不会自动使其 Pages 站点变为私有。若不适合托管，请使用扫描构建产物。请先协调已有的 Pages 站点。
2. 用包含 `[]` 的 `.nojekyll` 和 `history.json` 初始化新的 `gh-pages` 历史分支。保留现有站点内容与历史。
3. 将发布者模板中的 `YOUR_REPORT_BASE_URL` 设为你的 Pages URL。
4. 在 `skill-checks.yml` 的 `permissions` 下取消注释 `id-token: write`，以获取签名的执行回执。自定义扫描文件后，运行 `git hash-object .github/workflows/skill-checks.yml`，将结果填入 `trusted-caller-blobs`。每次编辑扫描文件后都要刷新此值。
5. 将两个文件合并到默认分支，保留提供的扫描文件名、工作流名称和构建产物名称。发布者已声明其所需权限。

新的符合条件的 PR 扫描会自动触发发布。部署成功后，机器人会在当前、打开、同仓库的 PR 上创建或更新一条评论。此发布者不支持 Fork PR 评论和仅评论操作。

没有签名权限时，普通扫描和可下载报告仍会运行；托管会被跳过。提交设置并触发新扫描；重跑旧任务会保留其原始工作流。

[完整发布者设置](https://github.com/NVIDIA/SkillEvaluator/blob/ba7779ad617c6444c98b6241a75bce92067e9d56/ci/README.md#optional-hosted-reports-and-pr-feedback)。

## GitLab

对于 GitLab，将 gitlab-skill-checks.yml 合并到根目录的 .gitlab-ci.yml，并设置 SKILL_EVAL_PATHS。其规则必须保留在根文件中才能启用合并请求流水线。
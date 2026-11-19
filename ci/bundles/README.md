# 即用型 CI 配置

将配置文件复制到你的仓库，设置技能文件夹和质量阈值，然后提交。手动配置无需在本地安装 SkillEvaluator。

| 用途 | 当前配置 | 复制目标 |
| --- | --- | --- |
| GitHub 检查 | [skill-checks.yml](1.0.1/skill-checks.yml) | `.github/workflows/skill-checks.yml` |
| GitHub PR 评论（无需托管） | [comment-skill-checks.yml](1.0.1/comment-skill-checks.yml) | 第二个工作流；按[评论配置](1.0.1/README.md#pr-comment-setup)操作 |
| GitHub 托管报告与 PR 评论 | [publish-skill-reports.yml](1.0.1/publish-skill-reports.yml) | 第二个工作流；按[配置说明](1.0.1/README.md)操作 |
| GitLab 检查 | [gitlab-skill-checks.yml](1.0.1/gitlab-skill-checks.yml) | `.gitlab-ci.yml`，或合并到现有流水线 |
| 可选的手动 LLM 检查 | [skill-checks-llm.yml](1.0.1/skill-checks-llm.yml) | `.github/workflows/skill-checks-llm.yml` |

打开文件，用 GitHub 的 **Raw** 视图复制完整内容。工作流提交和容器镜像摘要已固定。[完整配置](1.0.1/README.md)。

## 下载与验证

[下载当前 ZIP](1.0.1/skillevaluator-ci-1.0.1.zip)。每个版本文件夹包含生成的确切文件、校验和、源码/镜像标识及扫描器验证记录。历史版本 [0.7.0 文件](0.7.0/)、[0.7.1 文件](0.7.1/)、[0.7.2 文件](0.7.2/) 和 [0.8.0 文件](0.8.0/) 仍可供现有配置使用，[0.9.0](0.9.0/) 同样保留；当前接入请使用 1.0.1。

这些手动配置运行 Tier 1。全层级 CLI 已在 0.9.0 中提供；1.0.1 刷新了其单独打包的 schema 版本 2 描述符。如需选用，请安装对应发布版本后运行 `skillevaluator ci init --tiers 1,2,3`。升级 CLI 不会改变已提交的工作流固定版本。

## 发布约定

每个版本发布一个 SkillEvaluator 产品发布版本。发布前，将 Python 包、已验证的 CI 包和校验和附加到同一草稿，并在发布说明中包含带版本号的 PyPI 包链接。无需单独的 CI 发布条目。维护者使用的源模板位于 [ci/templates](../templates/README.md)。
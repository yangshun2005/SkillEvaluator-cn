# 复制 CI 模板

复制一个配置文件，即可将 SkillEvaluator 引入你的仓库。无需在本地安装 SkillEvaluator、使用引导 CLI，或将评估代码复制到项目中。GitHub 和 GitLab 模板统一维护在这个公开的 NVIDIA/SkillEvaluator 仓库中。

| 平台与结果 | 源模板 | 放置位置 |
| --- | --- | --- |
| GitHub：检查与可下载报告 | [skill-checks.yml](skill-checks.yml) | `.github/workflows/skill-checks.yml` |
| GitHub：同时更新标准 PR 评论，无需托管 | [comment-skill-checks.yml](comment-skill-checks.yml) | `.github/workflows/comment-skill-checks.yml`，与检查文件放在一起 |
| GitHub：同时托管报告并更新 PR 评论 | [publish-skill-reports.yml](publish-skill-reports.yml) | `.github/workflows/publish-skill-reports.yml`，与检查文件放在一起 |
| GitHub：可选的可信手动 LLM 检查 | [skill-checks-llm.yml](skill-checks-llm.yml) | `.github/workflows/skill-checks-llm.yml` |
| GitLab：检查与可下载报告 | [gitlab-skill-checks.yml](gitlab-skill-checks.yml) | `.gitlab-ci.yml`，或合并到现有的根配置中 |

打开源文件即可查看；GitHub 的 **Raw** 视图可复制完整内容。无需访问单独的演示仓库即可查看[示例结果](../examples/README.md)。

## 获取可运行的版本

**这些源模板包含占位符。** 请改为复制[可直接使用的配置](../bundles/1.0.1/README.md)；其中已填入经过审核的工作流提交和容器镜像摘要。
1.0.1 包包含仅评论工作流；归档的 0.9.0 包不包含。默认 CLI 反馈仍会配置托管发布器。若只需评论，请复制带版本的工作流并按照[评论设置](../README.md#pr-comments-without-hosted-reports)操作。

打开该文件夹中的文件，使用 GitHub 的 **Raw** 视图复制其完整内容。生成的文件及其校验和保存在这个公开仓库中。未来的发布版本会将相同文件附加到唯一的 SkillEvaluator 产品发布版本中，与其 Python 包一同提供。

如果你改为自定义源模板，请使用生成包 `release.json` 中不可变的工作流/镜像标识。不要保留占位符，也不要用 `main` 或 `latest` 替代运行时固定版本。

维护者可以在保持已验证扫描器工作流和镜像不变的情况下，发布经过审核的反馈工作流更新。将其完整提交 SHA 作为 `--publisher-ref` 传给 `ci/render_templates.py`；否则使用 `--workflow-ref`。该修订必须同时包含 `comment-skill-checks.yml` 和 `publish-skill-reports.yml`。发布回执会记录两个修订。此操作仅更改反馈工作流和工具；扫描器固定版本和已批准的评估器标识保持不变。

## 自定义并提交

- **GitHub：** 设置 `paths`，可选设置 `min-score`（默认 `70`）。如有需要，修改 `branches: [main]`，然后将工作流提交到默认分支。现有工作流保持独立。请遵循 [GitHub 指南](../../docs/github-actions.mdx)。
- **GitLab：** 设置 `SKILL_EVAL_PATHS`，可选设置 `SKILL_EVAL_MIN_SCORE`（默认 `70`）。对于现有流水线，合并 include、变量和作业规则；保留其他作业和 `workflow: rules`。作业默认使用 `test` 阶段，如果你的流水线未定义该阶段，请覆盖其 stage。请遵循 [GitLab 指南](../../docs/gitlab-ci.mdx)。
- **可选的 GitHub 评论：** 选择[仅评论设置](../README.md#pr-comments-without-hosted-reports)或[托管发布器设置](../README.md#optional-hosted-reports-and-pr-feedback)。在扫描文件中取消注释 `id-token: write` 以获取签名的执行回执，然后批准该文件的新 blob。仅复制第二个文件不会启用评论。
- **可选的 LLM 检查：** 在单独的手动工作流中配置提供商密钥。默认检查无需模型密钥。

打开一个修改了 skill 的 Pull Request 或合并请求即可查看检查结果。启用平台的 CI 服务，允许引用的工作流或 include，并提供兼容的 runner。[平台指南](../../docs/repository-integration.mdx)涵盖结果和合并门控设置。

## 高级 CI

显式启用的全层级 CLI 已包含在 0.9.0 中，有单独的[设置与维护者说明](all-tiers/README.md)。上述已发布的模板无需模型密钥即可提供质量和安全检查。
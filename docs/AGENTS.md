<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->

# AGENTS.md — `docs/`

供 AI 代理编辑 SkillEvaluator 长篇文档的指南。本文件夹存放通过 [Fern](https://buildwithfern.com/) 发布到 <https://docs.nvidia.com/skills/skillevaluator> 的 MDX 页面。面向人类的说明位于 [`README.md`](README.md)；本文件是面向机器的配套文档。

## 本目录内容

- `index.mdx` — 落地页和三层概览（Fern slug `/`）
- `quickstart.mdx` — 用已安装的 CLI 检查一个内置示例
- `starter-demo.mdx` — 可复现的公开终端入门示例、输出检查和适配
- `installation.mdx` — 共享的 pip、uv 和 GitHub 安装路径及验证
- `installation-options.mdx` — 升级、Docker、发布版本差异、扩展和扫描器设置
- `configuration.mdx` — 提供商、嵌入和 Tier 3 凭据角色
- `agent-skill.mdx` — 安装和使用 `skills/skillevaluator` 代理技能
- `tier1-validation.mdx` — 首次静态运行、报告和后续步骤
- `tier1-reference.mdx` — 详细检查、策略、扫描器和退出语义
- `tier2-deduplication.mdx` — 选择范围、运行去重和阅读发现
- `tier2-reference.mdx` — 分块、目录、阈值、算法和限制
- `tier3-live-evaluation.mdx` — 首次 Tier 3 对比；保持以任务为中心
- `prepare-dataset.mdx` — 在试验前编写、生成、审查和验证用例
- `tier3-run-options.mdx` — 为已审查的用例选择代理和尝试次数
- `tier3-run-reference.mdx` — 暂存契约、运行设置、预算和重复对比
- `tier3-runtime.mdx` — 后端兼容性、运行时契约和故障排除
- `tier3-grading.mdx` — 评判器、证据预算、重试和回退策略
- `tier3-results.mdx` — 分数、适用性、判定和不完整证据
- `tier3-experiments.mdx` — 技能交互和激活实验
- `grader-validation.mdx` — 对已发布验证器的独立标签资格认定
- `eval-datasets.mdx` — `evals/` 契约、数据集生成、运行策略
- `agents-and-sandboxes.mdx` — 代理、环境后端、本地模式、就绪状态
- `custom-graders.mdx` — 自带评判器和原生 Harbor 任务
- `reports.mdx` — 报告格式、分数和磁盘结果契约
- `metrics-reference.mdx` — 每个指标的公式、范围、单位、证据和验证状态
- `performance-dashboard.mdx` — 对比已保存的 Tier 3 分数和资源使用
- `benchmark-rollout.mdx` — 重新生成和提升 `BENCHMARK.md` 卡片
- `github-actions.mdx` — 技能检查工作流设置和可选的托管 PR 反馈
- `cli-onboarding.mdx` — 已发布的 GitHub 设置向导
- `repository-integration.mdx` — 为技能检查选择 GitHub 或 GitLab；pre-commit 钩子
- `gitlab-ci.mdx` — 可复用的技能检查和可下载的 GitLab 报告
- `advanced-ci.mdx` — 次要的可选前置条件和全层级参考
- `ci-integration.mdx` — 退出码、GitHub Actions 配方、渐进式采用
- `cli-reference.mdx` — 每个命令和标志
- `environment-variables.mdx` — 逐变量清单
- `developer-guide.mdx` — 贡献者设置、测试和文档工作流
- `assets/` — 页面引用的图片
- `README.md`（人类编写指南）和本 `AGENTS.md` **不是**已发布页面——不要将它们添加到导航中

导航顺序、页面标题和 slug 定义在 [`../fern/docs.yml`](../fern/docs.yml) 中。Fern CLI 版本固定在 [`../fern/fern.config.json`](../fern/fern.config.json) 中。

## 编辑规则

入口页面应聚焦读者的下一步操作：设置、可运行的命令、预期输出和下一步。将高级契约和边缘情况放在链接的参考页面中。使用 Steps 表示有序操作，Tabs 表示互斥选择，Accordions 表示可选细节；不要在第一个可运行示例之前放置参考章节。

将 pip、uv、uv bootstrap 和直接仓库安装保留在 `installation.mdx` 中；将升级和高级设置保留在 `installation-options.mdx` 中。默认使用轻量级 `skillevaluator` 安装；将 `skillevaluator[all]` 作为所有层级的显式选择加入。区分已发布的 PyPI 版本与未发布的 GitHub 功能和依赖。层级指南和快速入门链接到那里并使用已安装的 `skillevaluator` 命令。避免将包管理器配方复制到每个层级中，或在单个首次运行工作流中混用 `uv run`、`uv tool install` 和 pip。贡献者检出命令属于开发指南。

1. **每个页面都需要 YAML frontmatter**，包含 `title` 和 `description`。`title` 渲染为页面 H1——**不要**在正文中再添加 `# Heading`，否则标题会出现两次。

   ```mdx
   ---
   title: "Configuration"
   description: "One-sentence summary used for SEO and the nav preview."
   ---
   ```

2. **文件名遵循 URL 最佳实践**：小写、连字符分隔、无空格或下划线（`tier1-validation.mdx`，而非 `TIER1_VALIDATION.mdx`）。保持文件名与 `docs.yml` 中的页面 `slug` 一致。

3. **添加页面**：创建 `.mdx` 文件，然后在 [`../fern/docs.yml`](../fern/docs.yml) 的 `navigation` 下注册。未在此列出的页面不会出现在网站上。通过 `git mv` 重命名以保留历史，并更新所有引用（见规则 5）。

4. **这是 MDX，不是 Markdown。** 裸 `<` 会启动 JSX 标签，`{` 会启动表达式。将尖括号和花括号（例如 `<placeholder>`、`${VAR}`）放在反引号代码段或围栏代码块内，MDX 在那里保持字面量。不要使用 `<https://...>` 自动链接语法——写普通的 `[text](url)` 链接。

5. **链接**：
   - 在已发布的 MDX 中，使用站点路径，例如 `/skills/skillevaluator/configuration` 或 `/skills/skillevaluator/tier1-validation#anchor`。固定版本 Fern 5.121.0 预览对文件相对 `.mdx` 页面 URL 返回 HTTP 400；它不会将它们解析为已发布的 slug。
   - 对于 `docs/` 之外的仅仓库文件，在已发布的 MDX 中使用完整的 GitHub 源 URL。保持仓库 Markdown、图片和 YAML `path:` 条目相对于其源文件。
   - 重命名页面时，grep 整个仓库并修复同级页面、根 [`../README.md`](../README.md)、[`README.md`](README.md) 和 [`../fern/docs.yml`](../fern/docs.yml) 中的引用。

## 提交前验证

需要 **Node.js 22+**。使用已提交的 lockfile 安装固定版本的 CLI：`npm ci --prefix fern --ignore-scripts --omit=optional`。

```bash
./fern/node_modules/.bin/fern check       # validate docs.yml config and all links — must pass
./fern/node_modules/.bin/fern docs dev    # optional live preview at http://localhost:3000
```

`fern check` 必须通过；这是站点构建所依赖的同一道关卡。发布从 `main` 通过 [`../.github/workflows/publish-docs.yml`](../.github/workflows/publish-docs.yml) 自动进行——不要在本地运行 `fern generate`。

## 仍然适用的仓库约定

- 签署提交（`git commit -s`）；DCO 检查会拒绝未签署的提交（见 [`../CONTRIBUTING.md`](../CONTRIBUTING.md)）。
- 当文档变更对用户可见时，更新 [`../CHANGELOG.md`](../CHANGELOG.md)。
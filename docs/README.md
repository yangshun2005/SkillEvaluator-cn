<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved. -->

# 文档

SkillEvaluator 的详细文档。页面使用 MDX 编写，发布到 <https://docs.nvidia.com/skills/skillevaluator>。本页是面向人的快速入门；[`AGENTS.md`](AGENTS.md) 包含详细的编写规则（MDX 注意事项、链接约定、新增页面检查清单），供 AI 代理和进行非平凡编辑的贡献者参考。

## 目录结构

- `index.mdx` — 落地页与三层概览
- `quickstart.mdx` — 用已安装的 CLI 检查一个内置示例
- `installation.mdx` — 共享的 pip、uv 和 GitHub 安装路径及验证
- `installation-options.mdx` — 升级、Docker、发布版本差异、附加组件和扫描器设置
- `configuration.mdx` — 提供商、嵌入和 Tier 3 凭据角色
- `tier1-validation.mdx` — 首次静态运行、报告和后续步骤
- `tier1-reference.mdx` — 详细检查、策略、扫描器和退出语义
- `tier2-deduplication.mdx` — 选择范围、运行去重并阅读发现
- `tier2-reference.mdx` — 分块、目录、阈值、算法和限制
- `tier3-live-evaluation.mdx` — 首次实时比较，从模型设置到报告
- `prepare-dataset.mdx` — 在试验前编写、生成、审查和验证用例
- `tier3-run-options.mdx` — 为已审查的用例选择代理和尝试次数
- `tier3-run-reference.mdx` — 暂存契约、运行设置、预算和重复比较
- `tier3-runtime.mdx` — 运行时契约、后端兼容性和故障排查
- `tier3-grading.mdx` — 评判器、证据预算、重试和回退模型
- `tier3-results.mdx` — 分数、适用性、判定和不完整结果
- `tier3-experiments.mdx` — 技能交互和激活验证
- `eval-datasets.mdx` — `evals/` 契约、数据集生成、运行策略
- `agents-and-sandboxes.mdx` — 代理、环境后端、本地模式、就绪状态
- `custom-graders.mdx` — 自带评判器和原生 Harbor 任务
- `reports.mdx` — 报告格式、分数和磁盘结果契约
- `performance-dashboard.mdx` — 比较已保存的 Tier 3 分数和资源使用
- `benchmark-rollout.mdx` — 重新生成并推广 `BENCHMARK.md` 卡片
- `github-actions.mdx` — 可复用的技能检查和可选的托管 PR 反馈
- `cli-onboarding.mdx` — 安装 CLI、自动化 GitHub 设置并打开可选的测试 PR
- `repository-integration.mdx` — 为技能检查选择 GitHub 或 GitLab；pre-commit 钩子
- `gitlab-ci.mdx` — 可复用的技能检查和可下载的 GitLab 报告
- `advanced-ci.mdx` — 次级可选前置条件和全层级参考
- `ci-integration.mdx` — 退出码、GitHub Actions 配方、渐进式采用
- `cli-reference.mdx` — 所有命令和标志
- `environment-variables.mdx` — 逐变量清单
- `developer-guide.mdx` — 贡献者设置、测试和文档工作流

导航顺序和 slug 定义在 [`../fern/docs.yml`](../fern/docs.yml) 中。

## 本地构建

只有在编辑文档时才需要本地构建。站点通过 Fern GitHub 集成从 `main` 自动发布。

前置条件：Node.js 22+ 和 npm 10+（Fern CLI 要求的版本）。

```bash
# Install the pinned Fern CLI from the committed lockfile
npm ci --prefix fern --ignore-scripts --omit=optional

# From the repo root, preview the site with live reload
./fern/node_modules/.bin/fern docs dev

# Validate the docs configuration and links
./fern/node_modules/.bin/fern check
```

`fern docs dev` 在 <http://localhost:3000> 提供站点，并在 `.mdx` 文件或 `fern/docs.yml` 变更时重新加载。

## 编写说明

- 新页面必须添加到 [`../fern/docs.yml`](../fern/docs.yml) 的 `navigation` 部分；否则不会出现在站点中。
- 在已发布的 MDX 页面中，引用其他文档页面时使用站点路径，如 `/skills/skillevaluator/configuration`。Fern 不解析文件相对的 `.mdx` 链接；参见[官方链接格式](https://buildwithfern.com/learn/docs/writing-content/markdown-basics#link-format)。Fern 5.121.0 本地预览对这些 `.mdx` 页面 URL 返回 HTTP 400。图片、YAML `path:` 值和仅仓库内使用的 Markdown 链接保持相对于其源文件。
- Fern 版本固定在 [`../fern/fern.config.json`](../fern/fern.config.json) 中；请有意升级，而非作为副作用升级。


> **[中文化] SkillEvaluator**
>
> 此项目是 [NVIDIA/SkillEvaluator](https://github.com/NVIDIA/SkillEvaluator) 的中文翻译版本。
> - 原项目 Stars: 3747
> - 主语言: Python
> - 许可证: Apache-2.0
> - 翻译日期: 2026-11-19
> - 原始 README: [README_en.md](README_en.md)
>
> 说明：下方 README 正文为**原项目文档的中文翻译**，力求忠于原文；本页顶部的「中文快速上手」为**面向中文用户的补充导读**（非原文内容），目的是让你不用读完整篇 README 就能先把项目跑起来。
>
> 如有翻译不准确之处，欢迎提 Issue 或 PR。

---


## 🚀 中文快速上手（补充导读，非原文）

### 这是什么 / 解决什么问题

SkillEvaluator 是 NVIDIA 开源的一个命令行工具，用来检查 AI agent 的 skill 和 plugin（按 Agent Skills 规范组织的指令与配套文件目录），并衡量某个 skill 对任务结果的实际影响。它先做质量与安全检查，再对比「用 skill」和「不用 skill」的运行结果，从而在合并前发现 skill 的问题。它也是 NVIDIA Verified Skills 流水线的一部分。

### 快速上手（最短路径）

1. 在 Python 3.12–3.14 环境中安装 CLI：

   ```bash
   pip install skillevaluator
   ```

   （或用 uv：`uv tool install --python 3.13 skillevaluator`）

2. 不消耗 API Key，先跑静态质量检查：

   ```bash
   skillevaluator quality-check ./my-skill
   ```

   输出 100 分制评分与等级，默认通过阈值为 70。

3. 只跑 Tier 1（质量、schema、版本、PII、许可证、Unicode、脚本检查，以及 Harness Check）：

   ```bash
   skillevaluator validate ./my-skill --tiers 1
   ```

4. 想接入仓库做 CI 门禁，在 GitHub 仓库中运行：

   ```bash
   skillevaluator ci init
   ```

5. 想跑全部三个 tier（含去重与真实 agent 评估），需先按原文完成 scanner 配置、启动 Docker，并准备 NVIDIA Build API Key，然后去掉 `--tiers 1`：

   ```bash
   skillevaluator validate ./my-skill
   ```

### 环境要求

- **操作系统**：pyproject 标注 `Operating System :: OS Independent`；Dockerfile 基于 `python:3.12-slim`。
- **Python 版本**：`>=3.12,<3.15`（README 徽章写 3.12 / 3.13 / 3.14）。
- **硬件**：`quality-check` 明确不需要 GPU；其他检查的硬件要求原文未说明。
- **依赖**：基础安装即可做 Tier 1 静态校验（含 bandit 等）；Tier 3 需要所选 provider 与 agent 的凭据，并需启动 Docker。完整可选依赖通过 `.[all]` 安装。
- **网络**：Tier 3 使用 NVIDIA 托管端点，需在 build.nvidia.com 注册并获取 API Key；国内网络环境可能需要更换 pip 镜像源或加速下载（原文未说明具体镜像方案）。

### 常见坑与注意事项

- **分层能力差异大**：`quality-check` 无需 API Key、scanner、GPU 或 Docker，但只是静态质量检查，发布证据仍为 `INCOMPLETE`；安全与 agent 影响需要额外检查。
- **Tier 3 有前置条件**：需要 NVIDIA Build API Key（受访问条款与速率限制约束）、完成 scanner 配置并启动 Docker，缺一不可。
- **Harness 兼容性会直接导致 skill 不被加载**：例如描述超长被截断、`.agents/skills` 目录不被扫描、描述超出 1024 限制等，示例中 6 个 harness 有 2 个未加载。
- **评分可能随版本变化**：原文提示不同 release 的分数可能不同；没有现成 skill 时可使用仓库自带的 `src/skillevaluator/tier3/reference_skills/text-analyzer` 试用。

---



---

<div align="center">

# ![SkillEvaluator](https://raw.githubusercontent.com/NVIDIA/SkillEvaluator/ad2815fe78c045db91d230681d32863698da08cf/docs/assets/skillevaluator-wordmark.svg)

<h4>评估并改进 AI agent 技能与插件：跨 agent 的质量、安全、重叠、兼容性与实际影响</h4>

[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.12%20%7C%203.13%20%7C%203.14-blue.svg)](https://www.python.org/)
[![PyPI](https://img.shields.io/pypi/v/skillevaluator.svg?label=PyPI)](https://pypi.org/project/skillevaluator/)
[![Reusable CI templates](https://img.shields.io/badge/Reusable%20CI-templates-blue.svg)](https://docs.nvidia.com/skills/skillevaluator/repository-integration#copy-a-template)
[![Documentation](https://img.shields.io/badge/Documentation-docs.nvidia.com-blue.svg)](https://docs.nvidia.com/skills/skillevaluator/)
[![Paper](https://img.shields.io/badge/arXiv-2608.20614-b31b1b.svg)](https://arxiv.org/abs/2608.20614)
[![Blog](https://img.shields.io/badge/Blog-NVIDIA%20Developer-76B900.svg)](https://developer.nvidia.com/blog/evaluating-ai-agent-skill-performance-with-nvidia-skillevaluator/)
[![Stars](https://img.shields.io/github/stars/NVIDIA/SkillEvaluator)](https://github.com/NVIDIA/SkillEvaluator/stargazers)
[![Forks](https://img.shields.io/github/forks/NVIDIA/SkillEvaluator)](https://github.com/NVIDIA/SkillEvaluator/forks)

</div>

> 📺 **直播：** [Ask the Experts: Evaluating Agent Skills](https://www.youtube.com/watch?v=4rI0zATFxYI)

SkillEvaluator 检查 AI agent 技能与插件，并衡量技能对任务结果的影响：先做质量检查，再对比启用与不启用技能时的运行结果。

在合并前发现技能问题。通过 [CI 模板](https://docs.nvidia.com/skills/skillevaluator/repository-integration#copy-a-template)将仓库接入 SkillEvaluator，即可获得自动化的质量与安全门禁，无需编写或维护评估代码。

Agent 技能是由 [Agent Skills 规范](https://agentskills.io/)定义的指令与配套文件文件夹。SkillEvaluator 是 [NVIDIA Verified Skills 流水线](https://github.com/NVIDIA/skills)的一部分。

## 快速开始

在 Python 3.12–3.14 环境中安装 CLI：

```bash
pip install skillevaluator
```

<details><summary>或使用 uv</summary>

```bash
uv tool install --python 3.13 skillevaluator
```

</details>

uv 环境搭建或直接从 GitHub 源码安装，请参阅[安装](https://docs.nvidia.com/skills/skillevaluator/installation)。

然后从终端、CI 或 agent 中检查技能：

<table><tbody><tr valign="top">
<td width="33%">

<p><img src="https://raw.githubusercontent.com/NVIDIA/SkillEvaluator/928cc9d483fe9fef2d5011c3ca8f132cda46730a/docs/assets/get-started-cli.svg" width="250" alt="直接运行 CLI"></p>

运行由 **500+ 静态规则与检测模式**支撑的检查，无需 LLM 密钥。完成[扫描器配置](https://docs.nvidia.com/skills/skillevaluator/installation-options#system-tools)以获得完整覆盖：

```bash
skillevaluator validate \
  ./my-skill --tiers 1
```

<details><summary>运行全部三个层级</summary>

完成[使用 NVIDIA Build 的全层级配置](#run-all-three-tiers-with-a-free-nvidia-build-key)后，去掉 `--tiers 1`，即可加入去重和实时 agent 评估，衡量技能提升效果：

```bash
skillevaluator validate \
  ./my-skill
```

</details>

</td>
<td width="33%">

<p><img src="https://raw.githubusercontent.com/NVIDIA/SkillEvaluator/928cc9d483fe9fef2d5011c3ca8f132cda46730a/docs/assets/get-started-ci.svg" width="250" alt="通过 CI 接入你的仓库"></p>

检查每次技能变更。在你的 GitHub 仓库中运行，以打开配置 PR：

```bash
skillevaluator ci init
```

GitHub 和 GitLab 的[引导式配置与 CI 模板](#bring-skill-evaluation-to-your-repository)。

</td>
<td width="33%">

<p><img src="https://raw.githubusercontent.com/NVIDIA/SkillEvaluator/928cc9d483fe9fef2d5011c3ca8f132cda46730a/docs/assets/get-started-agent.svg" width="250" alt="接入你的 agent"></p>

将该技能添加到 Claude Code、Codex 或其他 agent：

```bash
npx skills add \
  NVIDIA/SkillEvaluator
```

然后提问：*"我的技能好用吗？"*
参见 [agent 指南](https://docs.nvidia.com/skills/skillevaluator/agent-skill)。

</td>
</tr></tbody></table>

### ⚡ [Harness 检查](https://docs.nvidia.com/skills/skillevaluator/harness-check)：你的 agent 会加载你的技能吗？

🔑 **无需 API 密钥** · 🤖 **无需运行 agent**

```bash
skillevaluator harness-check ./my-skill
```

以下为版本化配置的示例摘要。完整输出可能包含备注和 Schema 诊断信息。

```text
╭─ Harness compatibility · my-skill ──────────────────────────────────────────────────────╮
│  ⚠️ codex        degraded    Description cut to 1,021 chars + '...' in the skill list   │
│  ❌ claude-code  not loaded  .agents/skills is not scanned                              │
│  ✅ opencode     loads                                                                  │
│  ✅ hermes       loads                                                                  │
│  ✅ openclaw     loads                                                                  │
│  ❌ copilot-cli  not loaded  description is 1,028 UTF-16 units, 4 over the 1,024 limit  │
╰────────────────────────────────────────────────────────────────── ❌ 2 of 6 not loaded ─╯
```

`skillevaluator validate` 在 Tier 1 中包含 Harness 检查，以及**质量、Schema、版本、PII、许可证、Unicode 和脚本 lint 检查**。安全与代码完整性检查使用 **SkillSpector、Gitleaks、Bandit 和可选的 Semgrep**。
[查看全部 Tier 1 检查](https://docs.nvidia.com/skills/skillevaluator/tier1-reference#the-checks)；使用 `--tiers 1` 仅运行该层级。

🧪 **想更进一步？** 试试[实时 agent 评估](https://docs.nvidia.com/skills/skillevaluator/tier3-live-evaluation)（Tier 3）。

## 快速上手

### 无需密钥检查你的技能

```bash
skillevaluator quality-check ./my-skill
```

无需 API 密钥、扫描器、GPU 或 Docker。它会输出百分制评分、等级及依据的检查结果；通过阈值默认为 70。这仅是静态质量检查；发布证据仍为 `INCOMPLETE`。安全与 agent 影响需要额外检查。参见[无密钥操作指南](docs/quickstart.mdx#run-without-an-llm)。

还没有技能？克隆仓库并使用内置的 `src/skillevaluator/tier3/reference_skills/text-analyzer`。不同版本的评分可能不同。

### 使用免费 NVIDIA Build 密钥运行全部三个层级

符合条件的 NVIDIA Developer Program 成员可使用 NVIDIA 托管的端点进行**免费原型开发**，受[访问条款和速率限制](https://docs.api.nvidia.com/nim/docs/product)约束。

1. 登录 [build.nvidia.com](https://build.nvidia.com/) 并完成注册。
2. 打开模型页面，选择 **Get API Key**，生成你的密钥。
3. 完成[扫描器配置](docs/installation-options.mdx#system-tools)，启动 Docker，并[准备经过审核的数据集](docs/prepare-dataset.mdx)。安装所有层级，设置 provider 和密钥，然后验证：

```bash
pip install "skillevaluator[all]"
export SKILL_EVAL_LLM_PROVIDER=nv_build
export NVIDIA_API_KEY='nvapi-...'
skillevaluator validate ./my-skill
```

不带 `--tiers` 时，`validate` 会运行全部三个层级。该密钥覆盖 Tier 2 的嵌入和对话，以及 Tier 3 中受支持的 agent；添加 `--llm` 可在 Tier 1 中进行模型检查。使用 uv 或 GitHub 安装时，请改用对应的[全层级安装](https://docs.nvidia.com/skills/skillevaluator/installation#all-tiers)，而非上面的 pip 命令。Tier 3 还需要 Docker 运行；添加 `--tiers 1,2` 可跳过。更多内容见[运行组合流水线](#run-the-combined-pipeline)、[Build 操作指南](docs/quickstart.mdx#use-nvidia-build)和 [provider 配置](docs/configuration.mdx)。

## 将技能评估引入你的仓库

**配置极简。无需编写或维护评估代码。** 选择技能文件夹和阈值，即可在 GitHub 或 GitLab 中为变更的技能获得自动验证、安全扫描和可下载报告。

**[11 项 CI 检查](https://docs.nvidia.com/skills/skillevaluator/tier1-reference#the-checks)，由 500+ 静态规则与检测模式支撑：**
质量、Schema、版本、PII、许可证、Unicode、脚本 lint、[Harness 检查](https://docs.nvidia.com/skills/skillevaluator/harness-check)、安全、代码完整性和依赖审计。<br>
**扫描器：**[SkillSpector](https://docs.nvidia.com/skills/skillevaluator/tier1-reference#security--vulnerability-scanning)、[Gitleaks、Bandit 和 Semgrep](https://docs.nvidia.com/skills/skillevaluator/tier1-reference#code-integrity--code-risk-secrets-and-hygiene)，以及 [pip-audit](https://docs.nvidia.com/skills/skillevaluator/tier1-reference#dependency--cve-audit-opt-in)。

**GitHub 引导式配置：**[安装 CLI](#get-started) 后，在已检出的仓库中运行：

```bash
skillevaluator ci init
```

选择检查项和可选的托管反馈，然后审阅配置 PR。
[CLI 配置指南](https://docs.nvidia.com/skills/skillevaluator/cli-onboarding)。

**手动配置：**从 [CI 模板](ci/bundles/README.md)复制已填好的模板。无需本地 CLI。

| 你的仓库 | 即用模板 | 复制目标 |
| --- | --- | --- |
| GitHub | [技能检查](ci/bundles/1.0.1/skill-checks.yml) | `.github/workflows/skill-checks.yml` |
| GitLab | [技能检查](ci/bundles/1.0.1/gitlab-skill-checks.yml) | `.gitlab-ci.yml`，或合并到现有流水线 |
| GitHub PR 评论（无需托管） | [可选评论](ci/bundles/1.0.1/comment-skill-checks.yml) | 第二个工作流；完成[评论配置](ci/bundles/1.0.1/README.md#pr-comment-setup) |
| GitHub 托管报告与 PR 评论 | [可选发布器](ci/bundles/1.0.1/publish-skill-reports.yml) | 第二个工作流；完成[发布器配置](ci/bundles/1.0.1/README.md#one-time-publisher-setup) |

选择一个可选的反馈工作流。这些模板无需模型密钥即可运行 Tier 1 质量与安全检查；参见[全层级 CI 配置](https://docs.nvidia.com/skills/skillevaluator/advanced-ci)以加入去重和实时 agent 评估。

参见 [GitHub](https://docs.nvidia.com/skills/skillevaluator/github-actions) 和 [GitLab](https://docs.nvidia.com/skills/skillevaluator/gitlab-ci) 配置、[模板参考](ci/templates/README.md)和[报告示例](ci/examples/README.md)。

## 三层概览

![三层流水线：技能、验证、去重、实时评估、报告](https://raw.githubusercontent.com/NVIDIA/SkillEvaluator/ad2815fe78c045db91d230681d32863698da08cf/docs/assets/three-tier-overview.svg)

每个层级相互独立。使用[全层级安装](https://docs.nvidia.com/skills/skillevaluator/installation#all-tiers)获得全部三个层级：

| 层级 | 问题 | 配置后的示例 | 额外配置 |
| --- | --- | --- | --- |
| Tier 1：验证 | 技能是否安全、格式正确且[兼容 harness](docs/tier1-reference.mdx#the-checks)？ | `skillevaluator tier1 ./my-skill --no-llm` | 外部扫描器以获得完整静态覆盖；provider 和模型客户端可启用 LLM 检查 |
| Tier 1：Harness 检查 | agent 会加载该技能吗？ | `skillevaluator harness-check ./my-skill` | 无需 API 密钥或运行 agent |
| Tier 2：去重 | 指导内容是否重复或重叠？ | `skillevaluator tier2 ./my-skill` | 嵌入和对话；添加 `--catalog ./skill-catalog.json` 进行技能间比较 |
| Tier 3：实时 Agent 评估 | 技能对 agent 有帮助吗？ | `skillevaluator tier3 ./my-skill` | Provider 访问权限、受支持的 agent，以及运行中的沙箱（默认 Docker） |

为已安装的版本准备[扫描器](docs/installation-options.mdx#system-tools)；Semgrep 为可选项。`--no-llm` 不会移除扫描器要求；无法运行的扫描器会标记为 `INCOMPLETE` 并以非零状态退出。
[SkillSpector](https://github.com/NVIDIA/SkillSpector) 提供专门的技能安全扫描；[Harbor](https://github.com/harbor-framework/harbor) 这一开源 agent 评估框架为沙箱化 agent 运行提供支持。

技能间比较未运行时，Tier 2 会报告。Tier 3 自动驾驶会复用来源，或创建一个基于技能的用例，覆盖范围有限。参见[层级指南](https://docs.nvidia.com/skills/skillevaluator/)。

## 运行组合流水线

配置好 provider、扫描器和 agent 沙箱后，运行全部三个层级：

```bash
skillevaluator doctor --env-mode docker
skillevaluator validate ./my-skill
```

`doctor` 检查就绪状态，而非推理是否成功。`validate` 对技能使用自动驾驶运行所有层级。使用 `--tiers 1,2` 跳过实时评估，或使用 `--tiers 1` 仅做静态检查。`--no-autopilot` 需要已有来源。其他构建产物类型保留其适用的阶段。

Tier 1 决定 `validate` 是否通过。Tier 2 默认参与门禁，除非设置 `--no-block-on-dedup`；Tier 3 仅在设置 `--block-on-agent-eval` 时参与门禁。模型和托管沙箱的使用可能产生费用；参见[运行时与成本](docs/tier3-run-reference.mdx#plan-for-cost)。

独立生成需要实质性的 `evals/EVAL.md` 指导、包含任务请求和结果的 `--prompt` 文件，或匹配的保留轨迹。下面的 `--force` 会替换自动驾驶初始内容；`--refine` 会对其进行细化。显式指定 `--grounding skill` 允许仅基于技能的初始内容。请审阅用例；模板省略了未编写的负例。

```bash
skillevaluator tier3 create-eval-dataset ./my-skill --full --prompt eval-guidance.md --force
```

## 从源码检出运行

如需可编辑环境，请遵循[开发环境配置](docs/developer-guide.mdx#set-up-a-development-environment)。

## 研究与引用

资源：[论文](https://arxiv.org/abs/2608.20614)、[PDF](https://arxiv.org/pdf/2608.20614)、[DOI](https://doi.org/10.48550/arXiv.2608.20614)、[BibTeX](https://arxiv.org/bibtex/2608.20614) 和 [NVIDIA Developer 博客](https://developer.nvidia.com/blog/evaluating-ai-agent-skill-performance-with-nvidia-skillevaluator/)。
引用论文时请记录 SkillEvaluator 的发布版本或提交；参见 [CITATION.cff](CITATION.cff)。

## 文档

参见[文档](https://docs.nvidia.com/skills/skillevaluator/)了解配置、层级指南、报告、CI 集成、CLI 参考和[性能仪表盘](docs/reports.mdx#performance-dashboard)。

## 安装与第三方软件

本项目将下载并安装额外的第三方开源软件项目。使用前请审阅这些开源项目的许可条款。

## 贡献

阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 并运行：

```bash
make lint && make test && make build
```

参见[项目治理](GOVERNANCE.md)和[行为准则](CODE_OF_CONDUCT.md)。

## 支持

支持级别：**实验性**。由社区支持，尽力而为，不提供 SLA 或 NVIDIA 企业支持权益。通过 [GitHub Issues](https://
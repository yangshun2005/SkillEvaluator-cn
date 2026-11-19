# 发布 SkillEvaluator

发布一个产品版本，包含 Python 包和已验证的 CI 配置。用户通常安装包并运行 `skillevaluator ci init`；镜像构建和模板生成由维护者负责。

## 准备与验证

1. 选定已评审的源码修订，更新包版本、导入回退、可编辑锁定条目、引用信息及文档引用示例。将完整的 Unreleased 变更日志移入带日期的发布章节。
2. 运行锁定后的完整测试套件、lint、包构建、严格的分发包元数据检查、OSS 源码/归档检查、基准溯源检查和文档验证。在干净的受支持 Python 环境中安装构建出的 wheel 和源码分发包，并实际运行 CLI/报告输出。
3. 审查边界情况、选项交互、输出正确性和回归问题。合并前解决当前 head 上必需的 CI 失败和评审线程。

## 发布 Python 包之前先构建 CI 发布版本

若发布涉及已安装的评估器、运行时依赖或扫描器的变更，需构建并验证全新的官方 CI 镜像。仅涉及工作流/发布器或仅涉及文档的发布可复用兼容的已验证镜像；需明确记录复用情况及其来源标识。每次产品发布都要审查当前模板目录和打包的 CLI 描述符。

1. 将已验证的发布源码合并到 `main`。发布工作流仅接受上游 `main`；普通 PR 和 main 推送构建只验证不发布。
2. 以 `publish=true` 运行 **Build and publish Tier 1 image**：

   ```bash
   gh workflow run publish-tier1-image.yml \
     --repo NVIDIA/SkillEvaluator --ref main -f publish=true
   ```

   记录该次运行的精确源码 SHA。等待镜像验证、发布以及 onboarding 任务对生成的不可变摘要完成匿名拉取。
3. 下载 `github-actions-onboarding` 和镜像验证证据。onboarding 任务使用已评审的工作流 SHA 和已发布镜像摘要渲染模板。若工作流/发布器为独立修订，使用：

   ```bash
   python3 ci/render_templates.py --workflow-ref REVIEWED_FULL_SHA \
     --publisher-ref REVIEWED_PUBLISHER_FULL_SHA \
     --image ghcr.io/nvidia/skillevaluator-tier1@sha256:VERIFIED_DIGEST \
     --output /tmp/skillevaluator-onboarding
   ```

   使用新的输出目录。渲染器会写出配置、README 和 `release.json`；版本化 ZIP、校验和、扫描器清单和验证回执需另行准备。
4. 通过后续 PR 将已验证文件提交到 `ci/bundles/VERSION/` 下。将 `ci/bundles/README.md`、当前 onboarding 链接和打包的 `src/skillevaluator/ci-release.json` 更新为相同的已验证镜像/工作流固定值。保留历史版本文件夹。若发布全层级 onboarding，需一并审查其独立的 schema 版本 2 描述符、工作流/工具引用、能力、评估器版本和运行时锁哈希。
5. 在干净的消费方环境中测试生成的确切文件，包括通过/失败、变更/全部/无变更选择、缺少前置条件和保留报告。单独测试可选的可信发布和基于提供方的层级；明确记录无法执行的集成检查。
6. 合并这些固定值/打包变更，冻结最终源码修订，然后重新构建并验证 Python 包，使 `ci init` 包含最终描述符。保持镜像的源码 SHA 与后续打包/元数据提交相互独立。

镜像发布、扫描器验证和消费方资格认定的详细信息见 [CI 维护者说明](ci/README.md#maintainer-release-procedure)。

## 发布产品版本

1. 在最终评审的源码修订处创建版本标签。暂存一个 GitHub 草稿发布，包含已评审的说明、wheel、源码分发包、CI ZIP、构建产物清单和校验和。发布前下载并验证每个资产；不可变的发布资产发布后无法替换。
2. 发布已完成的 GitHub 发布。验证匿名资产下载以及确切的镜像链接/摘要。在说明中包含直接的版本化 YAML 链接和版本化 PyPI 包 URL，在验证前将 PyPI 标记为待定。
3. 使用已批准的 Kitmaker 项目对两个确切的 Python 构建产物 URL 进行试运行。两项检查完成后，提交相同的 URL 进行托管发布。保留提交回执；重试不明确的请求前先检查状态。
4. 验证索引托管的哈希和全新的 pip/uv 安装、依赖、CLI 版本及真实报告。然后将可编辑的发布说明从待定更新为已发布并已验证，并注明任何验证限制。

## 面向用户的说明

保持常规 GitHub onboarding 简短：

```bash
python -m pip install --upgrade skillevaluator
skillevaluator ci init
```

CLI 根据其打包的描述符生成工作流；消费方的 CI 拉取预构建镜像。手动替代方案见[当前模板目录](ci/bundles/README.md)。生成的文件保留不可变固定值；安装更新的 CLI 不会静默升级现有工作流。
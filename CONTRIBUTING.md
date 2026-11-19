# 贡献指南

欢迎通过 GitHub Pull Request 贡献代码。

## 提交 Pull Request 之前

- 重大改动请先在 Issue 中讨论。
- 不要将提供商凭据、客户数据和私有基准材料提交到仓库。
- 为行为变更添加针对性的测试。
- 在本地运行 `make lint`、`make test` 和 `make build`。
- 当变更影响用户时，更新 [CHANGELOG.md](CHANGELOG.md)。

关于环境搭建、针对性测试、文档检查和审查流程，请阅读[贡献者指南](docs/developer-guide.mdx)。如需可运行的公开技能和任务，请从[终端入门示例](examples/starter/README.md)开始。

## 文档与示例贡献

- 对文档中涉及的发布版本或检出代码运行你修改的每条命令，并说明你测试的是哪个版本。不要声称支持你未验证过的平台。
- 示例输入应保持公开且可移植。包含预期输出检查，并标注种子数据导致的失败和缺失的证据；测试夹具不是实时基准。
- 对于此入门示例，运行 `uv run python scripts/demo_starter.py --output reports/contribution-demo` 和 `uv run pytest tests/test_demo_starter.py`。后续运行时使用新的输出目录。将报告保留在本地，并在 Pull Request 中准确总结运行了什么；不要提交生成的报告或记录。
- 按照贡献者指南中的说明，使用固定版本的 Fern CLI 验证文档。
- 说明如何从缺失依赖、凭据和不完整运行中恢复。

## Pull Request

使用 Pull Request 模板，保持每个 Pull Request 聚焦，并说明用户可见的行为和所执行的验证。提交贡献即表示你确认有权根据 [Apache License 2.0](LICENSE) 提交该贡献。

## 签署你的工作

贡献要求是下方的开发者原创声明（DCO）签署，而非贡献者许可协议（CLA）。

- 我们要求所有贡献者对其提交进行"签署"。这证明该贡献是你的原创作品，或者你有权根据相同的许可证或兼容许可证提交它。

  - 任何包含未签署提交的贡献将不被接受。Pull Request 会在 CI 中通过 **DCO** 工作流检查；未签署的提交将无法通过检查。

- 要对提交进行签署，只需在提交更改时使用 `--signoff`（或 `-s`）选项：

  ```bash
  git commit -s -m "Add cool feature."
  ```

  这将在提交信息后追加以下内容：

  ```text
  Signed-off-by: Your Name <your@email.com>
  ```

- [开发者原创声明](https://developercertificate.org/)全文：

  ```text
  Developer Certificate of Origin
  Version 1.1

  Copyright (C) 2004, 2006 The Linux Foundation and its contributors.

  Everyone is permitted to copy and distribute verbatim copies of this
  license document, but changing it is not allowed.


  Developer's Certificate of Origin 1.1

  By making a contribution to this project, I certify that:

  (a) The contribution was created in whole or in part by me and I
      have the right to submit it under the open source license
      indicated in the file; or

  (b) The contribution is based upon previous work that, to the best
      of my knowledge, is covered under an appropriate open source
      license and I have the right under that license to submit that
      work with modifications, whether created in whole or in part
      by me, under the same open source license (unless I am
      permitted to submit under a different license), as indicated
      in the file; or

  (c) The contribution was provided directly to me by some other
      person who certified (a), (b) or (c) and I have not modified
      it.

  (d) I understand and agree that this project and the contribution
      are public and that a record of the contribution (including all
      personal information I submit with it, including my sign-off) is
      maintained indefinitely and may be redistributed consistent with
      this project or the open source license(s) involved.
  ```

请通过 [SECURITY.md](SECURITY.md) 报告漏洞，而非公开 Issue。
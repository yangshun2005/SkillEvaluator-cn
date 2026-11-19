# 公共终端入门示例

在仓库根目录运行，需安装 [uv](https://docs.astral.sh/uv/getting-started/installation/) 和 Git。CLI 固定使用公开发布的 0.8.0 版本 wheel；演示文件来自当前检出。无需 API 密钥、扫描器、GPU 或 Docker。

```bash
uv run --isolated --no-project --python 3.13 --with "skillevaluator @ https://github.com/NVIDIA/SkillEvaluator/releases/download/v0.8.0/skillevaluator-0.8.0-py3-none-any.whl" python scripts/demo_starter.py --output reports/starter
```

运行器会把已安装的 `text-analyzer` 参考技能复制到输出目录，加入本文件夹的任务和输入夹具，然后执行真实的文本分析脚本和 CLI。它将结果与 `expected.json` 对比：

- 夹具包含 4 个单词、2 行、27 个字符和 2 个句子。
- 原始文档得分 84.5/100（B），通过明确的 80 分阈值。
- 另一份故意缩短的副本得分 71/100（C），未通过同一阈值。运行器预期该命令退出码为 1，随后自身以 0 退出。
- 通过的报告中仍有 `publication_status: incomplete`：静态质量不等于安全覆盖，也不证明智能体从该技能中获益。

这些是确定性的质量检查，**不是**智能体影响评分或实时基准测试。不会将已保存或历史报告作为新证据重放。所有 JSON 报告和终端文本均为新生成。`receipt.json` 仅在所有检查通过后记录已安装版本和时间戳。

复制的技能可编辑，包含 `evals/evals.json` 和 `evals/files/input.txt`。演示直接检查脚本，而非智能体执行任务的能力。实时评估需要额外配置。

每次运行请使用新的输出目录；已有目录绝不会被覆盖。报告保留在技能之外。不要仅为消除失败而更新预期分数：请检查报告发现和评估器版本。

安装、手动重放、报告解读、实时评估、恢复与适配的完整说明见[完整演练](../../docs/starter-demo.mdx)。
# 第 3 层评估数据集

第 3 层会在每个用例上分别运行真实 agent：安装技能与不安装技能（基线）。本文件介绍 `<skill-dir>/evals/` 目录和数据集格式。

## <skill-dir>/evals/ 目录

| 路径 | 用途 |
| --- | --- |
| `<skill-dir>/evals/evals.json` | 必需的数据集（也接受 `.jsonl`、`.yaml`、`.yml`） |
| `<skill-dir>/evals/config.yml` | 运行策略 |
| `<skill-dir>/evals/EVAL.md` | 供 `create-eval-dataset` 使用的指导（`## Questions`、`## Behaviors`、`## Negative Cases`、`## Notes`） |
| `<skill-dir>/evals/files/` | 暂存到 `/workspace/input/` 的测试夹具 |
| `<skill-dir>/evals/environment/` | `Dockerfile`、`docker-compose.yaml` sidecar、`mcp_servers.toml` |
| `<skill-dir>/evals/harbor/` | 原生 Harbor 任务（BYOT） |
| `<skill-dir>/evals/grader.py` 或 `grader.sh` | 自定义评分器（BYOG） |
| `<skill-dir>/evals/results/` | 仅输出；切勿编辑或提交 |

## 数据集格式

```json
{
  "skill_name": "my-skill",
  "evals": [
    {
      "id": "my-skill-001",
      "prompt": "Convert report.docx in /workspace/input to PDF",
      "expected_output": "A PDF of the document saved next to the input",
      "assertions": ["Saves report.pdf next to report.docx", "Keeps the document's headings in the PDF"],
      "expected_skill": "my-skill",
      "files": ["files/case-001"]
    },
    {
      "id": "my-skill-002-negative",
      "prompt": "What is the capital of France?",
      "expected_output": "Answers directly without loading my-skill",
      "expected_skill": null
    }
  ]
}
```

必需字段：`skill_name`、`evals`，以及每个用例的 `id`（字母、数字、`.`、`_`、`-`；最多 128 个字符，唯一）、`prompt`、`expected_output`。可选字段：`assertions`、`expected_skill`（负向用例为 `null`）、`expected_script`、`acceptable_skills`、`files`（路径相对于 `<skill-dir>/evals/`；省略则暂存整个 `<skill-dir>/evals/files/`）。使用 `question`、`ground_truth`、`expected_behavior` 的旧版扁平数组仍可运行，但会给出弃用警告。

好的数据集覆盖四类：显式（点名技能）、隐式（描述任务）、上下文（项目中的任务）、负向（不应触发）。将 `expected_output` 写成可观察的结果，每个断言对应一种行为。基线会在相同 prompt、`expected_output` 和断言下运行，但不安装技能，因此这些内容要围绕任务结果：绝不要让 prompt 指向技能的文件，也不要断言 agent 读取了 `SKILL.md`、运行了技能的脚本或使用了技能（`skill_execution` 会对此评分）。`tier3 validate` 会对这些常见措辞发出警告；这只是启发式检查，仍需人工审查用例。

## 引导式生成

独立运行 `create-eval-dataset` 时默认 `--grounding auto`：优先使用 `--prompt` / `<skill-dir>/evals/EVAL.md` 中作者提供的指导（需有标题之外的真实内容），其次使用该技能已完成运行中可用的匹配保留轨迹。先在指导文件中写清任务请求和可观察结果：

```bash
skillevaluator create-eval-dataset <skill-dir> --full --prompt <skill-dir>/evals/EVAL.md
```

如果既没有指导，也没有匹配的保留轨迹，生成会在访问 provider 或写入之前失败。`--refine` 和 `--no-llm` 不会绕过该预检。若要使用之前仅基于技能的入门工作流，需显式选择：

```bash
skillevaluator create-eval-dataset <skill-dir> --full --no-llm --grounding skill
```

自动模式不会改动已有数据集，除非传入 `--force` 或 `--refine`，即使存在匹配轨迹也是如此。要从已完成结果中细化：

```bash
skillevaluator create-eval-dataset <skill-dir> --refine --from-results <run-dir>
```

自动轨迹细化会保留已有的作者用例 ID 和预期；未匹配的用例和观察到的答案建议需要审查。当自动模式选择轨迹时，描述技能使用的建议会将 `expected_output` 留空，以便独立编写结果。已有的作者预期会被保留。显式 `--grounding trace` 仍可在不加 `--refine` 的情况下进行原地细化。生成溯源会记录实际生效的 `intent` 或 `trace` grounding 以及输入哈希。显式 `intent`、`trace` 和 `skill` 模式仍然可用。无人值守的 autopilot 保留其现有的、基于技能的单用例入门约定。
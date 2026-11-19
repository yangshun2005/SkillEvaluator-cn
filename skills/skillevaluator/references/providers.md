# 提供商与凭据

只能设置一个提供商。若未设置 `SKILL_EVAL_LLM_PROVIDER`，且 `NVIDIA_API_KEY`、`OPENAI_API_KEY`、`ANTHROPIC_API_KEY` 中恰好导出了一个，则自动选中该提供商；若导出多个，LLM 命令会失败，直到显式设置提供商。

| 提供商 | 变量 | 说明 |
| --- | --- | --- |
| `nv_build` | `NVIDIA_API_KEY` | 一个密钥即可覆盖 `docker`、`kata`、`local` 中的 Tier 1 LLM、Tier 2 嵌入和 Tier 3 代理；默认代理为 `opencode` |
| `openai` | `OPENAI_API_KEY` | 默认代理为 `codex` |
| `anthropic` | `ANTHROPIC_API_KEY` | 默认代理为 `claude-code`；Tier 2 需要单独的嵌入提供商 |
| `bedrock` | AWS 凭据链、`AWS_REGION` | Tier 2 需要单独的嵌入提供商 |
| `openai-compatible` | `SKILL_EVAL_LLM_BASE_URL`、`SKILL_EVAL_LLM_API_KEY` | 任意网关；目录 ID 不同的模型需覆盖 |

覆盖项：`SKILL_EVAL_LLM_MODEL`（评估器和评判器）、`LLM_JUDGE_MODEL`（仅评判器）、`SKILL_EVAL_EMBEDDING_MODEL`、`--agent-model AGENT=MODEL`（Tier 3 代理）。网关上的 OpenCode 使用 `--agent-model opencode=openai/MODEL`。Codex 需要 Responses-API 模型，OpenCode 需要 Chat Completions，Claude Code 需要 Anthropic Messages，且都必须支持工具调用。

规则：

- 凭据只能来自主机环境（或脱敏的 CI 密钥）。`<skill-dir>/evals/config.yml` 中的 `harbor.runtime_env` 会拒绝提供商密钥和 base URL，并直接报错。
- `skillevaluator models` 和 `doctor --verify-models` 检查目录可达性；公共目录可达并不证明推理密钥可用。
- 评判器失败会使结果变为未评分（`NO SCORE`），而非 `0.0`。
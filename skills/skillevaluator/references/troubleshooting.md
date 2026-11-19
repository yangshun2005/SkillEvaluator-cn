# 故障排查

| 症状 | 原因 | 修复 |
| --- | --- | --- |
| `Error: No such command` 或退出码 `2` | 命令或参数与当前版本不匹配 | `skillevaluator --version`，然后 `skillevaluator <command> --help` |
| `tier1 validate` 评估了错误的对象 | 目录名与子命令同名 | 使用 `<path>/validate` |
| `--tiers 2` 被拒绝 | `validate` 始终包含 Tier 1 | 使用 `skillevaluator tier2 PATH` |
| 扫描器返回 `INCOMPLETE` | SkillSpector、Gitleaks 或 Bandit 缺失或执行失败 | `skillevaluator health-check --tier 1`；安装 `security` 扩展；`PATH` 中靠前的旧版扫描器会覆盖固定版本 |
| 使用了 `--skip` 导致发布失败 | 跳过的检查无法认证 Tier 1 | 去掉 `--skip` 重新运行 |
| Schema：目录与名称不匹配 | 文件夹名与 frontmatter 中的 `name` 不一致 | 重命名其中一个使两者一致（kebab-case） |
| 质量低于最低要求 | 缺少预期标题、描述含糊、无示例、`SKILL.md` 超过 500 行或 5,000 token（token 上限为 `--quality-config` 中的 `efficiency.token_budget.max_tokens`） | 按每条发现项的建议处理；将细节移入 `references/` |
| 评分标准未通过但分数很高 | 某一项标准低于 7/10 | 在 JSON 报告中查看该标准的反馈 |
| Tier 2 需要嵌入 | Anthropic 和 Bedrock 不提供嵌入 | 为其他提供商设置 `SKILL_EVAL_EMBEDDING_*` |
| `<skill-dir>/evals/config.yml` 被拒绝 | `runtime_env` 中存在未知键或凭据 | 修正该键；将密钥移至宿主机环境 |
| `Missing required fields: question` | 旧条目缺少 `question`，或新条目缺少 `prompt` | 使用 agentskills.io 的格式 |
| 技能从未被激活 | 描述与提示词不匹配 | 在描述中写明任务名称；添加明确的用例 |
| Agent 认证失败 | Agent 凭据未在 `skillevaluator` 运行处导出 | 在宿主机上导出；`doctor -a AGENT` |
| `AgentTimeoutError` 或限流错误 | 模型响应慢或并发过高 | `--timeout-multiplier 2`，降低 `--n-concurrent` |
| 自定义 Dockerfile 构建失败 | 基础镜像缺少 Agent 或评分器所需的工具，或 `python3 -I` 无法导入验证器包（`ENV PIP_USER=1`、`PIP_TARGET` 或 `PYTHONPATH` 安装） | 修正 `<skill-dir>/evals/environment/Dockerfile`；将验证器包安装到 `python3` 的 site-packages；使用 `--harbor-keep-jobs` 重新运行以查看构建日志 |
| 结果需要调试 | Harbor 任务默认会被删除 | 使用 `--harbor-keep-jobs` 重新运行，然后 `skillevaluator harbor-view JOBS_DIR` |
| 多次运行间 Lift 不一致 | LLM 评判器和 Agent 具有随机性 | 增加 `--n-attempts`；仅比较完全相同的配置 |
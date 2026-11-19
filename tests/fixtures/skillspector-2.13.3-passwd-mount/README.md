# SkillSpector 2.13.3 只读挂载报告

这些报告由已发布的 `v2.13.3` 标签（提交 `38a1dfe93325d2f8bfcd4f8d0241eed706334915`）通过 `skillspector scan --no-llm --format json` 捕获。它们是重新生成的报告，不是改过版本号的旧测试数据。仅对临时扫描根前缀做了归一化；发现项标识、评分、时间戳、来源和完整性核算均为扫描器输出。

两个输入目录都包含以下 `SKILL.md`：

```markdown
---
name: passwd-mount-probe
description: Build a project in a container as the host user.
---

# Container build

Run `sh build.sh` to build the project in a container.
```

`build.sh` 可执行（权限 `0755`）。对于 `read-only.json`：

```sh
#!/bin/sh
docker run -v /etc/passwd:/etc/passwd:ro image
```

对于 `separate-read.json`：

```sh
#!/bin/sh
docker run -v /etc/passwd:/etc/passwd:ro image
cat /etc/passwd
```

每个挂载产生两行 HIGH PE3，`finding_id` 各不相同，分别对应一个路径操作数，另有一条无关的 MEDIUM RP1 镜像固定建议。单独读取在第 3 行额外产生一行未标记的 HIGH PE3。两次扫描均返回退出码 `0`，检查完整，聚合评分分别为 `39` 和 `43`；SkillEvaluator 仍会拦截实际的 HIGH 读取。

捕获结果中 `llm_requested=false`、`meta_analysis_applied=false`、`inference_usage` 为空，启用了启发式过滤并禁用了语义分析器。`llm_available=true` 表示已配置提供方，与 `--no-llm` 无关。来源信息正确标明执行不适用。未请求任何 LLM 调用。适配器测试还通过在内存中仅修改该可用性标志来测试不可用的提供方。

要重新生成，请创建上述输入目录，将报告输出位置放在输入目录之外，然后运行：

```sh
for case in read-only separate-read; do
  OPENAI_API_KEY=fixture-only SKILLSPECTOR_PROVIDER=openai uvx \
    --from git+https://github.com/NVIDIA/SkillSpector.git@v2.13.3 \
    skillspector scan "/tmp/skillspector-2.13.3-passwd-mount/$case" \
    --no-llm --format json --output "$case.json"
done
```

占位密钥使提供方可用性可复现，且无需使用凭据。发现项和时间戳必须重新生成并审查，不能跨扫描器版本复制。部分 LLM 响应测试是对这些报告的显式合成变异，不是额外的捕获扫描。
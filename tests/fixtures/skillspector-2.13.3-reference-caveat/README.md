# 缺失引用报告

`report.json` 是 SkillSpector 2.13.3 于 2026-11-10 扫描本目录中合成 `SKILL.md` 的原始输出，未作修改。不包含任何客户内容。

固定镜像为
`ghcr.io/nvidia/skillevaluator-tier1@sha256:a3baec51f48145cdf9de05a5ab054d9576f62f1de0f5276b788410f5c17747d4`。
仅将 `SKILL.md` 挂载到 `/workspace`。入口命令为
`/opt/skillspector/bin/skillspector scan /workspace/. --format json --no-llm`，
网络已禁用，根文件系统只读。退出码为 0。

所有捆绑内容均已检查。报告仍为 `partial` / `CAUTION`，因为一个运行时示例路径和一个版本字符串被判定为缺失的本地引用。作为警告接受时必须保留这些注意事项；它不能证明缺失目标存在，也不能忽略任何其他检查缺口。
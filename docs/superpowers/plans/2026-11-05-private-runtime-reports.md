# 私有运行时报告实现计划

> **面向智能体执行者：** 使用现有执行工作流、聚焦的辅助函数和独立的最终审查。在任务台账中跟踪已完成的检查项。

**目标：** 完成第 24 项，实现一致的私有运行时视图和可持久化的离线回放。

**架构：** 将现有结果/来源信息一次性归约为不可变的、经过脱敏的展示投影。在清理前保留该投影，并通过现有六个私有报告适配器渲染。

**技术栈：** Python 3.11+、dataclasses、共享的安全文件系统/JSON 读取器、Jinja 及现有报告器；不新增依赖。

**规范：** `docs/superpowers/specs/2026-11-05-private-runtime-reports-design.md`

## 全局约束

- 公共 CLI/配置/检测/导出在第 25 项完成前保持禁用。
- 未知值与零保持区分；暂存永远不能证明已加载或已执行。
- 报告是被动的、建议性的，不改变评分、门禁或源结果。
- 先脱敏再截断，对每种输出格式进行转义，并限制输入/输出工作量。
- 使用现有统计数据，不重新计算提升值、资格、置信区间或成本。

## 审查重点

- 即使执行成功，缺失的轨迹/启动证据也必须保持不完整。
- 缺失或不支持的实验组必须保留其测量/排除原因。
- 恶意字符串、格式错误的值和报告预算耗尽必须保持安全且明确。
- 已保存的快照必须在源/任务移除后仍可回放，并拒绝不安全或格式错误的文件。
- 发布失败必须保留回调、评估状态和清理。

### 任务 1：投影与保留快照

文件：新增 `reporting/plugin_runtime.py`、`reporting/plugin_runtime_io.py`；
修改 `tier3/plugin_execution.py`；新增聚焦的报告/生命周期测试。

接口：`project_plugin_runtime(result, *, provenance=None) -> RuntimePluginReport | None`；
`RuntimePluginReport.payload() -> dict`；`save_plugin_runtime_report(report, run_dir)`；
`load_plugin_runtime_report(run_dir) -> RuntimePluginReport`。

- [x] 为运行标识、加载状态、覆盖率/信号/MCP、比较、实测成本、未知值和显式省略编写失败测试。
- [x] 使用现有脱敏和分节预算实现固定 Schema 的有界投影。
- [x] 测试严格的禁止跟随、有界且经过版本/Schema 检查的快照往返，包括格式错误/超大/缺失/链接输入。
- [x] 在私有发布回调中保留快照，保持失败为建议性且调用方发布独立。
- [x] 运行聚焦的投影、回放和生命周期回归测试。

### 任务 2：六种报告格式

文件：新增 `reporting/plugin_runtime_render.py`、运行时 HTML 模板和测试；
最小限度复用/重构 `reporting/plugin_static_render.py`。

接口：`render_plugin_runtime_reports(results, *, report, reporters) -> dict[str, str]`。

- [x] 为相同载荷、转义、缺失证据、静态/运行时组合输出和普通输出兼容性编写跨格式失败测试。
- [x] 通过所有六个现有报告器渲染同一投影，不新增收集或公共导出。
- [x] 运行运行时/静态报告测试，并目视检查生成的 HTML 报告。

### 任务 3：组装验收与交付

文件：新增 `tests/tier3/test_private_runtime_report_acceptance.py`、私有使用说明和发布说明。

- [x] 使用本地执行夹具演练真实发现、确定性验证、准备、服务/运行器和收集：五个跨组件用例、两次尝试、四个实验组。
- [x] 验证源/任务/作业移除后的已保存回放、共享比较估计、缺失证据和普通运行隔离。
- [ ] 运行 `make lint`、`make test`、`make build`；验证仅安装基础包的回放和私有 CLI 边界。
- [ ] 独立审查、修复可操作的问题、签署提交、推送并打开/附加一个聚焦的 PR，附上确切的验证限制。
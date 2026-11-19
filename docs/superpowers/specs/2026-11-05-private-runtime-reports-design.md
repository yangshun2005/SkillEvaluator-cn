# 私有运行时报告（拆分计划第 24 项）

第 24 项完成第 15–23 项引入的私有工作流之上的加载、覆盖率、行为、对比和已保存运行视图。临时任务和作业被移除后，评审者仍需要看到同样的事实。公开命令、配置、检测和导出仍属于第 25 项。

对现有引擎结果和私有来源使用一个不可变、带版本、有界的展示投影。保留声明/暂存/列出/加载状态、缺失证据、不支持的模式、对比排除项、不确定性和未知成本。执行状态与证据完整性是两个独立事实。建议性观察绝不改变评分、验证发现或退出状态。

在私有发布回调期间、清理和调用方发布之前，将该净化后的投影持久化为 `plugin_runtime_report.json`。固定文件安全读取器校验已保存的 Schema，并复用该投影进行离线报告。缺失的旧快照明确标记为不可用。重放不重建追踪、不加载源文件、不启动 agent、不联系 provider。

复用现有的六个私有静态适配器和表格渲染器，用于 CLI、JSON、Markdown、HTML、SARIF 和基准输出。保持普通报告器入口点字节兼容。显式展示省略项和缺失值；针对每种输出语言转义不可信文本，并在截断前脱敏。

考虑过的替代方案：从原始 sidecar 重建视图会丢失私有加载决策和准备来源；在六个报告器中嵌入临时视图会重复策略并有公开激活风险。保留的共享投影契合现有私有适配器架构，且无需新增依赖。

验收覆盖发现、确定性验证、准备、现有 service/runner、全部四个分支、证据、发布、清理和离线重放。用确定性本地 fixture 替代外部 provider/Harbor 执行；准确说明这一边界。验证五个跨组件用例和两次尝试、缺失证据、发布失败以及普通工作流隔离。同时构建/安装发行包，并在无可选运行时依赖的情况下检查重放。

## 私有 API 用法

当现有引擎生成运行目录时，`evaluate_prepared_plugin` 会自动保留快照。调用方发布在临时包装器被移除之前即可看到它。存储失败属于建议性：`publication_error` 和 `provenance["runtime_report_error"]` 会说明原因，调用方发布和清理仍会执行。成功评分的运行可能带有不完整的运行时证据。

离线消费者选择具体的保留运行目录，加载快照，并将其现有验证结果提供给私有适配器：

```python
from skillevaluator.reporting import HTMLReporter, JSONReporter
from skillevaluator.reporting.plugin_runtime_io import load_plugin_runtime_report
from skillevaluator.reporting.plugin_runtime_render import render_plugin_runtime_reports

report = load_plugin_runtime_report(run_dir)
documents = render_plugin_runtime_reports(
    validation_results,
    report=report,
    reporters=[HTMLReporter(), JSONReporter()],
)
```

重放使用已保存的展示 Schema；它不会重新读取源文件、重新评分试验或恢复被省略的细节。旧运行缺失快照会抛出明确的不可用错误。选择具体目录而非 `latest` 符号链接。将快照视为保留的观察结果，而非某次运行的认证证明。保留原始构建产物以便深入调查。
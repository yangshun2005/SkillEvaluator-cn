# 报告清晰度实施计划

> **面向智能体工作者：** 必需子技能：使用 superpowers:subagent-driven-development 或 superpowers:executing-plans 逐任务实施本计划。使用复选框（`- [ ]`）语法跟踪进度。

**目标：** 合并重复的报告信息，静默保留完整的原生功能/证据，然后在现有分支上发起结构化的前后对比 PR。

**架构：** 保留现有渲染器以及 canonical/display 边界。独立的有界遥测与插件展示辅助函数供给单一原生模板；简洁的摘要投影提供 Overview，而不重复完整 Insights。常规元数据放在折叠的页脚中。

**技术栈：** Python 3.12+、Jinja2、内嵌 Chart.js 4.5.1、仅开发环境使用的 Playwright/Chromium。

**规格：** `docs/superpowers/specs/2026-11-14-report-clarity-design.md`（用户已批准）。

## 全局约束

- 单一渲染器，评分/门禁/CLI/导出以及 canonical Tier 3 对象保持不变；保留所有现有输出安全性与边界。
- 保留原生排版/配色、16px 正文和 Chart.js 4.5.1。所有层级、六个子标签页、证据和控件保持可访问。
- Overview 最多三个主要指标卡片：Score（含 agent）、Lift、Coverage；全部缺失的质量使用单一不可用状态加覆盖率/操作。
- 单一完整 Insights 归属。渲染期间不得任意拆分/改写散文、丢失追踪关联或调用 provider。
- 被动插件区块默认折叠，保留稳定 ID 和快照，区分省略行/展示缩短/保留前裁剪。
- 保持验证、质量、执行、发布和已完成 agent 范围相互独立，并显示可见的阻塞项和比较限制。
- 保留深链接/搜索/键盘/打印/无 JS/离线/移动端行为、CI 直接 canonical 产物链接和已安装 wheel 打包。
- 仅使用合成的已提交测试夹具；私有报告/产物保持忽略。不进行新的评估或重新评分。
- 与早前报告工作使用同一功能分支；提交签署；仅在本地审查/验证后推送功能分支并创建 PR。不合并。

## 审查重点

- 混合的已完成/失败 agent、源预检和跳过范围不得变成整轮通过或伪造的质量分数。
- 折叠的空/插件/评估器证据必须保持可搜索，并可通过现有深链接访问，包括仅静态报告。
- 旧版原生和当前遥测 Schema 必须保留部分/未知与零的区别、缓存核算和不可用增量。
- 更少的可见卡片必须保留每个指标族、完整作者文本以及按 agent/arm/case 的关联。
- 元数据压缩不得隐藏可比性、临时数据、源拒绝或有界证据限定。

---

### 任务 1：有界记录遥测展示

**文件：** 创建 `src/skillevaluator/reporting/telemetry_view.py`、`templates/telemetry.html.j2`、`tests/reporting/test_telemetry_view.py`。

**接口：** 产出 `build_telemetry_view(payload: object) -> dict`，包含 `token_sections`、`latency_sections`、`token_chart`（标签和三个有界数据集，或 null）、`has_tokens`、`has_latency`。产出 Jinja 宏 `render_telemetry(view)`。任务 3 在现有预览丢弃 token 详情之前，向该辅助函数提供完整的已评估展示数据；完整保留的原始数据只能通过经过验证的有界投影读取。不要编辑 `html.py` 或主模板。

- [ ] 为原生数据集/agent/case token 细分和延迟、两个 arm、零、未知、部分覆盖、不可用增量原因、恶意字符串/非有限值和边界编写失败测试。
- [ ] 使用全新的显式 basetemp 运行聚焦测试；捕获预期失败。
- [ ] 实现纯只读辅助函数和原生展开/表格局部模板。保留记录值、覆盖率和原因代码；不采集或重新计算计时。支持长键原生遥测，同时保留现有公共资源路径。
- [ ] 暴露 ID `tier3-token-usage`、`tier3-latency-metrics`；旧版 token canvas 使用 `tier3-recorded-token-chart`，保持当前图表路径不变。图表数据由 agent/condition 组的标签以及 uncached/cached/completion 数据集组成；不重复计算 cached。
- [ ] 验证聚焦测试、无输入变更和有界载荷；编写报告。控制器审查并提交明确归属的路径。

### 任务 2：静默插件证据与稳定归属

**文件：** 修改 `reporting/plugin_evidence.py`、`templates/plugin_evidence.html.j2`、`tests/reporting/test_plugin_evidence_report.py`。

**接口：** 保留 `build_plugin_evidence()` 的 canonical `static`/`runtime` 值、槽位和宏 `render_plugin_evidence(evidence, slot)`。如有用，添加仅展示用的摘要/默认展开元数据；所有现有区块 ID 保持不变。

- [ ] 为折叠的被动/空面板、紧凑的归属/来源限定和保留的区块锚点，以及精确的 canonical 导出/省略区分添加失败测试。
- [ ] 捕获红灯，实现紧凑的折叠区块/分组空状态，使用显式 basetemp 验证绿灯。
- [ ] 保持状态/限制提示可见，完整注释可访问。移除每个区块前重复的 run ID 和通用散文；常规元数据保留在可访问的详情中。
- [ ] 验证仅静态/仅运行时、省略和归属夹具，不读取源或调用 provider。编写报告；控制器审查/提交归属路径。

### 任务 3：单一归属报告层级与摘要集成

**文件：** `reporting/html.py`、可选的聚焦 `summary_view.py`、`templates/report.html.j2`、`report_ui.css.j2`、`report_ui.js.j2`，以及展示/单元测试。不要编辑任务 1/2 归属的文件或浏览器测试套件（浏览器验收由控制器负责）。

**接口：** 消费 `build_telemetry_view()` 和 `render_telemetry()` 以及未更改的插件宏。模板上下文 `tier3_telemetry`；token 图表系列进入现有图表生命周期，使用安全 JSON 序列化、主题/打印刷新和可访问回退。保留所有当前 canonical/发布评估边界。

- [ ] 为单一完整 Insights 实例、<=3 个主要指标、单一空质量状态、单一层级导航、折叠的 Run Details 和原生遥测包含添加失败的归属/形状测试。覆盖完整/不完整/混合/自定义/仅基线/跳过/源拒绝状态。
- [ ] 捕获红灯，实现页眉/主视觉/导航简化、保留 System/Light/Dark 的单一主题控件、紧凑的发布原因、简洁的 Tier1/2 摘要。
- [ ] 在 Overview 中首先放置结果/覆盖率和单一结构化记录操作；移除完整卡片副本和记分卡回显。在 Insights 中保留作者文本/操作/检查/证据。次要结果族在记录时使用紧凑表格。
- [ ] 用简短的限定提示替换共享段落覆盖率前言；完整的 agent/arm 覆盖率归入 Agent Details，完整错误归入 Diagnostics。在受影响的结果处保留重要限定词。
- [ ] 将插件清单移到有用的原生内容下方；将常规元数据合并到折叠的 Run Details 中；人类可读的 Agent/Condition 产物标签在 Diagnostics 中保留精确路径/ID。
- [ ] 折叠方法散文和嵌套 case 尝试诊断，保留原生 Dataset 卡片。保留 ID、搜索和深链接祖先展开、打印恢复和无 JS。
- [ ] 集成任务 1 的辅助函数/局部模板/图表，同时保持当前公共资源和图表行为。验证聚焦的报告/CI 回归并编写报告；控制器审查/提交。

### 任务 4：浏览器验收、保留重放、打包和 PR

**文件：** 现有 `tests/reporting/test_report_browser.py` 和夹具、报告文档/CHANGELOG、本地忽略的审计脚本/结果、PR 正文。

- [ ] 扩展合成浏览器覆盖：无重复卡片/导航、折叠证据/深链接/搜索、滚动时 <=96px 的移动端页眉和首屏结果/操作、旧版遥测和图表回退。
- [ ] 在相关更改集成后运行报告/安全/保留和当前浏览器检查；在适配过时布局选择器的同时保留有意义的旧断言。
- [ ] 在检出目录之外运行 lint、OSS 源/wheel/sdist 扫描、包构建和已安装 wheel 的 Chromium；使用固定版本的 Fern 验证文档。
- [ ] 离线重放保留的实际/原生样本，不重新评分；保留行、canonical/投影和 28/36、12/20。对照已批准的审计基线捕获新的前后桌面/移动端视图和字数/展开指标。
- [ ] 最终独立审查；如需要，进行一轮限定范围的修复。如实记录最终验证和任何预先存在的完整套件限制。
- [ ] 验证签署/干净的跟踪树，推送功能分支并创建 PR，包含仓库模板验证、BEFORE/AFTER 表格、安全性/数据保留和测试证据。不要在公开 PR 中包含私有数据、私有 URL 或本地文件链接。不合并。
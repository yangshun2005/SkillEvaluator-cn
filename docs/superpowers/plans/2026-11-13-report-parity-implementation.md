# 报告呈现一致性实现计划

已批准规范：本分支的报告呈现计划。本文件记录其实现边界与任务接口。

**目标：** 复用已验证的原生报告呈现，覆盖全部三个层级和全部六个 Tier 3 标签页，同时保留公开结果、门禁、安全与留存证据契约。

**基线：** 公开 main f5bf6c1f3666699fd145612a2e9162e9721db3f9。工作树与用户的脏检出隔离。呈现参考与私有报告保留在已提交夹具之外；来源溯源记录在本地审计清单中。

**最终范围澄清：** 本实现取代下文的生产者写入与来源固定指令：简洁的失败运行摘要仅由渲染器生成，CLI JSON 增加并往返记录布尔值 `optional_skip`，公开清单快照两个当前模板样式块。

## 全局约束

- 一个 HTMLReporter 和一份权威呈现；不得有替代渲染器或竞争性布局样式表。
- 保留公开分数、门禁、适用性、仅自定义中立性、失败臂抑制、来源标识、有界预览、脱敏、转义 JSON、安全 URL、代理项处理与原子输出。
- 渲染已保存证据不发起任何提供方调用，不重新评分，也不重新打开已评估的源路径。
- 保留现有 Chart.js 4.5.1 及其许可证。使用复制的图表配置，不降级资源。
- 保持现有调用点和 CLI 契约可用；公开插件 CLI 分发仍在其独立 PR 轨道上。
- 已提交夹具或面向公众的文档中不得包含私有基准材料、私有 URL、凭据或参考报告载荷。

## 任务 1：简洁、有证据支撑的 Insights

负责 evaluation/tier3_report.py、新增的 reporting/insights_view.py 模块及针对性测试。不要编辑 reporting/html.py 或模板，这些由任务 2 负责。

实现并导出 `build_insights_view(payload: dict) -> dict`，返回 `conclusions`、`recommendations`、`next_steps` 和 `execution_errors`。结论/建议项保留模板既有的 title/message/severity/source/category/metric/evidence 字段。原样保留健康的持久化反馈。对于失败的留存运行，消费结构化条件覆盖和 next_steps 以生成简洁的执行摘要与操作，绝不拆分任意分号散文。仅对已识别的同一失败副本去重，保留 case/agent/evidence 关联。完整原始错误仍可单独访问。Overview 和 Insights 都将消费此投影。

对于新的失败评估，用简洁的覆盖/结论/操作消息替换错误拼接，保留完整的 execution_errors。保留所有评分/守卫语义。测试必须捕获重复块、不可用覆盖、独立的 case/agent 错误、留存载荷变更、健康文本改写以及渲染期间的提供方调用。

## 任务 2：精确原生呈现与规范导出

负责 reporting/html.py、templates/report.html.j2、report_ui.css.j2/report_ui.js.j2 及呈现相关测试。复用固定源模板的页眉、主视觉、字体、调色板、表格、卡片、徽章、披露布局、图表配置和页脚；在上下文边界适配字段。替换侧边栏和布局覆盖系统。保留搜索/筛选、无障碍、深链接、移动端、打印、无 JS、CI 元数据和安全渲染，仅做窄幅增强。默认深色配 Tier3，其余浅色，尊重显式保存的主题。

集成任务 1 的 `build_insights_view`。在 Overview 和 Insights 中渲染相同卡片，并在 Diagnostics 中保留完整错误。使用原生 Dataset case 卡片，带每 case 披露，并仅引用一次共享运行错误。使所有 Tier1/2/3 摘要真实；在 Tier2 映射中包含 Similarity Check。全面验证显示全部 3 个层级，包括显式未运行状态；独立 Tier3 保持仅 Tier3。留存证据中缺失的测量保持不可用，绝不为零或虚构质量数据。

向 HTMLReporter 的 render_all 添加可选仅关键字 `plugin_static` / `plugin_runtime` 输入，默认 None，接受现有类型化报告投影或经任务 3 同意的已验证快照。通过任务 3 的视图辅助函数投影它们，并将结果 `plugin_evidence` 传递到 tier1、tier2、overview、agents、trials、provenance 的模板槽。每个详细部分仅出现一次。

将已验证的规范 Tier3 与显示塑形分开，并原样嵌入/导出留存对象（包括中立 vs 强制显示失败、null 和空字段）。保留 base64/脚本安全编码和预览限制。保留现有 HTML report_data 信封，添加提供的插件投影；不要用 JSONReporter 的 schema 替换它。保留 CI canonical_json_url 直链行为和嵌入抑制。

## 任务 3：集成被动插件投影与留存元数据

负责 reporting/plugin_static_render.py、plugin_runtime_render.py、新增的 reporting/plugin_evidence.py、插件模板、reporting/retained.py、json_reporter.py 及针对性插件/往返测试。不要编辑 html.py 或主模板。

实现 `build_plugin_evidence(static: object = None, runtime: object = None) -> dict`，键为 tier1、tier2、overview、agents、trials、provenance，每个都是现有有界 section 字典的列表。通过 payload() 和已验证字典接受现有 StaticPluginReport/RuntimePluginReport；绝不检查源树或调用运行时。为导出精确保留快照（在 static/runtime 下单独返回载荷快照或提供辅助函数）。

静态清单/依赖/MCP/hooks/权限归 Tier1；目录相似性归 Tier2。运行时资格/臂覆盖归 Overview；加载/激活/行为/MCP 归 Agent Details；配对比较/上下文/成本归 Trials；来源/限制/排除归 Diagnostics。包含来源标签/归属和截断通知。区分省略行和缩短文本；不要承诺恢复已裁剪的值。

用现有私有组合函数通过预渲染上下文替换渲染后 HTML 字符串插入；保留非 HTML 行为。与任务 2 协调 render_all 关键字接口；不自动公开插件分发。在留存读取器和 JSON 往返中保留 optional_skip 类型化元数据，不改变门禁策略。

## 任务 4：回归、浏览器与分发验收

Root 负责仅开发用 Playwright/Chromium 集成、CI 作业、包检查、文档/变更日志和本地重新生成交付物。使用合成脱敏夹具，形状类似两个成功的双 agent 参考、两个不完整插件报告、混合 agent/仅自定义/缺失基线以及遥测/跳过的 Tier3、单层和完整报告，以及非零收集相似性。

演练所有层级/子标签/agent/去重视图、主题、桌面/移动端、键盘、筛选、详情、复制/下载、图表回退/离线行为、无 JS 和打印，确保无页面级溢出或控制台错误。使用相同浏览器/数据/视口/主题比较实际样式/几何；将前后截图保留在受跟踪代码之外。在检出目录之外添加已安装 wheel 渲染冒烟测试。

从仓库外的留存产物重新生成两个提供的报告。记录源哈希、渲染器修订、原始 Tier3 相等性、结果/投影保留、optional_skip 元数据、覆盖率 28/36 和 12/20、无提供方调用/重新评分、截图和测试结果。审计清单记录来源固定、复制片段和有意适配，不将私有来源引用放入可分发文档。

## 审查与完成

对照已批准要求和代码质量审查每个独立任务，然后审查集成分支。运行针对性回归、项目 lint/build 和相关现有报告套件。解决实质性失败；精确报告无关套件失败。交付隔离分支/工作树以及重新生成的报告、截图、溯源和验证结果。本实现请求不包含推送、合并或发布。